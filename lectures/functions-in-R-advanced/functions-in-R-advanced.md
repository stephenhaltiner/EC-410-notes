Functions in R: Advanced
================

[Debug mode](#debug-mode)

[Catching user errors](#catching-user-errors)

  - [Function-specific control flow](#function-specific-control-flow)

  - [base::tryCatch()](#base--trycatch\(\))
    
      - [Wrap tryCatch() around an entire
        function](#wrap-trycatch\(\)-around-an-entire-function)
      - [Use tryCatch() inside a
        function](#use-trycatch\(\)-inside-a-function)

  - [purrr::safely() and family](#purrr--safely\(\)-and-family)

[Caching (memoisation)](#caching-\(memoisation\))

  - [Caching across R sessions](#caching-across-r-sessions)
  - [Verbose output](#verbose-output)

[Further resources](#further-resources)

## Debug mode

[Download
cheatsheet](https://github.com/rstudio/cheatsheets/raw/master/rstudio-ide.pdf)

`debugonce()` lets you enter debug mode the next time you run the
function you specify. (It’s a version of `debug()` that prevents “debug
hell” caused by recursion.)

You may also be prompted by RStudio to debug when there is an error.

## Catching user errors

Take for example this bad input:

``` r
square("one")
```

    ## Error in x^2: non-numeric argument to binary operator

There are three main approaches to make your code user-proof:

### Function-specific control flow

We use `ifelse` to produce a warning or error message if the input
argument is invalid:

``` r
square_ifelse <- 
  function(x = 1) {
    if (is.numeric(x)) { ## Check that the input is valid for our function.
      x_sq <- x^2
      df <- tibble(value=x, value_squared=x_sq)
      return(df)
    } else { ## Return a warning message if not.
      message("You must provide a numeric input variable.")
    }
  }
square_ifelse("one")
```

    ## You must provide a numeric input variable.

We can get a similar result with less code, using `stop()`:

``` r
square_stop <- 
  function (x = 1) {
    if (!is.numeric(x)) stop("You must provide a numeric input variable.")
    x_sq <- x^2
    df <- tibble(value=x, value_squared=x_sq)
    return(df)
  }
square_stop("one")
```

    ## Error in square_stop("one"): You must provide a numeric input variable.

### base::tryCatch()

A more general option is to use `base::tryCatch()` for handling errors
and warnings. Two examples:

#### Wrap tryCatch() around an entire function

We invoke R’s built-in “error” class, which is passed to another
built-in function “message”. This tells R to produce our message when it
recognizes that (any) error has occured inside our function.

``` r
tryCatch(
  square("three"), 
  error = function(e) message("Something went wrong. Did you try to square a string instead of a number?")
  )
```

    ## Something went wrong. Did you try to square a string instead of a number?

This works, but throws away all our input because of a single error.
Take this for example:

``` r
tryCatch(
  square(c(1,2,"three")), 
  error = function(e) message("Something went wrong. Did you try to square a string instead of a number?")
  )
```

    ## Something went wrong. Did you try to square a string instead of a number?

Though most of our inputs were valid, we get a blanket-statement error
message. Ideally we would only get an error message for the third input
variable…

#### Use tryCatch() inside a function

``` r
square_trycatch <-
  function (x = 1) {
    x_sq <- tryCatch(x^2, error = function(e) NA_real_) ## tryCatch goes here now. Produce an NA value if we can't square the input.
    df <- tibble(value=x, value_squared=x_sq)
    return(df)
  }
square_trycatch(c(1,2,"three"))
```

    ## # A tibble: 3 x 2
    ##   value value_squared
    ##   <chr>         <dbl>
    ## 1 1                NA
    ## 2 2                NA
    ## 3 three            NA

It returned NA for “three” but why also for 1 and 2?

``` r
str(c(1,2,"three")) ## Return the structure of the data
```

    ##  chr [1:3] "1" "2" "three"

R coerces every element in this vector to character strings, so each
element got evaluated as NA.

Solution: use an input array that allows for different element types
(i.e. a *list*.)

``` r
map(list(1,2,"three"), square_trycatch)
```

    ## [[1]]
    ## # A tibble: 1 x 2
    ##   value value_squared
    ##   <dbl>         <dbl>
    ## 1     1             1
    ## 
    ## [[2]]
    ## # A tibble: 1 x 2
    ##   value value_squared
    ##   <dbl>         <dbl>
    ## 1     2             4
    ## 
    ## [[3]]
    ## # A tibble: 1 x 2
    ##   value value_squared
    ##   <chr>         <dbl>
    ## 1 three            NA

We might want to bind the results into a single data frame, like with
`purrr:map_df()`, but this produces errors because all the columns have
to contain only one data type.

Solution: Coerce to numeric the offending input, within the function
itself. This will introduce coercion warnings, but at least it won’t
fail:

``` r
square_trycatch2 <- 
  function (x = 1) {
    x_sq <- tryCatch(x^2, error = function(e) NA_real_)
    df <- tibble(value=as.numeric(x), value_squared=x_sq) ## Convert input to numeric
    return(df)
  }
map_df(list(1,2,"three"), square_trycatch2)
```

    ## Warning in eval_tidy(xs[[j]], mask): NAs introduced by coercion

    ## # A tibble: 3 x 2
    ##   value value_squared
    ##   <dbl>         <dbl>
    ## 1     1             1
    ## 2     2             4
    ## 3    NA            NA

### purrr::safely() and family

The tidyverse equivalent of `tryCatch()`. Related functions include
`purrr::possibly()`.

``` r
square_simple <-
  function (x = 1) {
    x_sq <- x^2
  }
square_safely <- safely(square_simple)
square_safely("three")
```

    ## $result
    ## NULL
    ## 
    ## $error
    ## <simpleError in x^2: non-numeric argument to binary operator>

You can specify default behavior in case of an error:

``` r
square_safely <- safely(square_simple, otherwise = NA_real_)
square_safely("three")
```

    ## $result
    ## [1] NA
    ## 
    ## $error
    ## <simpleError in x^2: non-numeric argument to binary operator>

## Caching (memoisation)

Caching is good beacuse problems happen: crashes, malfunctions, power
outages, memory limits, timeouts, invalid arguments buried in iterated
input, etc.

The problems are worse with parallel computations: the whole program
could run and then only reveal an error at the very end.

We use the [memoise](https://github.com/r-lib/memoise) package.
(Memoisation is a particular form of caching that saves the results of
expensive function calls, so we don’t have to repeat them later.)

To see the benefits of caching, let’s create a slow function and test
without, and then with, caching:

``` r
slow_square <- 
  function(x) {
    Sys.sleep(1)
    square(x)
  }
```

To enable caching, we feed the function to `memoise::memoise()`:

``` r
library(memoise)
mem_square <- memoise(slow_square)
```

``` r
## Run first time
system.time(
  m1 <- map_df(1:10, mem_square)
)
```

    ##    user  system elapsed 
    ##   0.048   0.000  10.074

``` r
## Run second time
system.time(
  m2 <- map_df(1:10, mem_square)
)
```

    ##    user  system elapsed 
    ##   0.003   0.000   0.003

You could verify that the results are equal:

``` r
all.equal(m1, m2)
```

    ## [1] TRUE

If we run the function with some additional inputs, only the new inputs
will add to the compute time:

``` r
system.time(
  m3 <- map_df(1:15, mem_square)
)
```

    ##    user  system elapsed 
    ##   0.011   0.000   5.028

### Caching across R sessions

Specify a dedicated cache directory for complex or time-consuming
analyses that you want to be able to access across R sessions.

To enable caching that persists across sessions, specify a cache
directory with the argument `cache = cache_filesystem(PATH)`. Recommend
you use use a `.rcache/` naming pattern to keep your paths organized.

``` r
## Cache directory path (which I've already created)
cache_dir <- here("R-funcs-adv/.rcache")

## (Re-)memoise our function with the persistent cache location
mem_square_persistent <-
  memoise(slow_square, cache = cache_filesystem(cache_dir))
```

### Verbose output

It’s often helpful to add verbose prompts to memoised functions. The
code below will:

1.  Check for and load previously cached results. Print results to
    screen.
2.  Run the memoised function on any inputs that haven’t yet been
    evaluated. (These results will be cached.) Print results to screen.

<!-- end list -->

``` r
mem_square_verbose <- 
  function(x) {
    ## 1. Load cached data if already generated
    if (has_cache(mem_square_persistent)(x)) {
      cat("Loading cached data for x =", x, "\n")
      my_data <- mem_square_persistent(x)
      return(my_data)
    }
    
    ## 2. Generate new data if cache not available
    cat("Generating data from scratch for x =", x, "...")
    my_data <- mem_square_persistent(x)
    cat("ok\n")
    
    return(my_data)
  }
```

``` r
system.time(
  m5 <- map_df(1:10, mem_square_verbose)
)
```

    ## Loading cached data for x = 1 
    ## Loading cached data for x = 2 
    ## Loading cached data for x = 3 
    ## Loading cached data for x = 4 
    ## Loading cached data for x = 5 
    ## Loading cached data for x = 6 
    ## Loading cached data for x = 7 
    ## Generating data from scratch for x = 8 ...ok
    ## Generating data from scratch for x = 9 ...ok
    ## Generating data from scratch for x = 10 ...ok

    ##    user  system elapsed 
    ##   0.022   0.003   6.031

## Further resources

  - [Debugging techniques in
    RStudio](https://rstudio.com/resources/rstudioconf-2018/debugging-techniques-in-rstudio/)
    (a recorded talk, from RStudio)
  - [Debugging with
    RStudio](https://support.rstudio.com/hc/en-us/articles/205612627-Debugging-with-RStudio)
    (also from RStudio)
  - [Object of type ‘closure’ is not
    subsettable](https://rstudio.com/resources/rstudioconf-2020/object-of-type-closure-is-not-subsettable),
    Grant’s favorite debugging talk
  - [Debugging](https://adv-r.hadley.nz/debugging.html) chapter of
    *Advanced R*
