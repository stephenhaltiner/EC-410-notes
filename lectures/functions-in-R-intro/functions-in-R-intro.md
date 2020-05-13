Functions in R: Intro
================

Create a function (and assign it to an object):

    my_func <- 
    function(ARGUMENTS) {
      OPERATIONS
      return(VALUE)
    }

Short functions can use a single line:

    my_short_func <- function(ARGUMENTS) OPERATION

**[An example](#an-example)**

  - [Specifying return values](#specifying-return-values)
  - [Specifying default argument
    values](#specifying-default-argument-values)
      - [Environments and lexical
        scoping](#environments-and-lexical-scoping)

**[Control flow](#control-flow)**

  - [if and ifelse](#if-and-ifelse)
      - [ifelse gotchas and
        alternatives](#ifelse-gotchas-and-alternatives)
  - [case when (nested ifelse)](#case-when-\(nested-ifelse\))

**[Iteration](#iteration)**

  - [for loops](#for-loops)
  - [Functional programming](#functional-programming)
      - [lapply and co.](#lapply-and-co.)
      - [purrr](#purrr)
  - [Create and iterate over named
    functions](#create-and-iterate-over-named-functions)
  - [Iterate over multiple inputs](#iterate-over-multiple-inputs)
      - [`mapply()` and `pmap()`](#mapply\(\)-and-pmap\(\))
      - [Using a data frame of input
        combinations](#using-a-data-frame-of-input-combinations)

**[Further resources](#further-resources)**

## An example

``` r
square <-       ## Function name
  function(x) { ## Argument(s) the function takes as input
    x^2         ## Operation(s) the function performs
  }
square(3)
```

    ## [1] 9

On one line:

``` r
square <- function(x){x^2}
square(4)
```

    ## [1] 16

### Specifying return values

Above, we didn’t specify a return value for the function. By default, R
returns the final object you created within the function. You won’t
always want this, so make a habit of specifying the return object:

``` r
square <- 
  function(x){
    x_sq <- x^2   ## Create intermediary object (that will be returned)
    return(x_sq)  ## The value(s) / object(s) to return
  }
square(5)
```

    ## [1] 25

Specifying a return object is helpful when you want to return more than
one object. Let’s say we want to remind the use what variable they used
as an argument in our function:

``` r
square <- 
  function(x){
    x_sq <- x^2
    return(list(value=x, value_squared=x_sq))
  }
square(6)
```

    ## $value
    ## [1] 6
    ## 
    ## $value_squared
    ## [1] 36

We can make it return a tibble instead:

``` r
square <- 
  function(x){
    x_sq <- x^2
    df <- tibble(value=x, value_squared=x_sq) ## Bundle the input and output values into a convenient dataframe
    return(df)
  }
square(7)
```

    ## # A tibble: 1 x 2
    ##   value value_squared
    ##   <dbl>         <dbl>
    ## 1     7            49

### Specifying default argument values

``` r
square <- 
  function(x = 1){  ## Set default argument value to 1
    x_sq <- x^2
    df <- tibble(value=x, value_squared=x_sq)
    return(df)
  }
square() ## Leave argument blank to use default
```

    ## # A tibble: 1 x 2
    ##   value value_squared
    ##   <dbl>         <dbl>
    ## 1     1             1

#### Environments and lexical scoping

None of the intermediate objects that we created within the above
functions (`x_sq`, `df`, etc.) have entered the global environment.

R has [lexical
scoping](https://adv-r.hadley.nz/functions.html#lexical-scoping) rules,
which govern where it stores and evalueates the values of different
objects. Functions operate in a sort of sandboxed
[environment](https://adv-r.hadley.nz/environments.html). They don’t
return or use objects in the global environment unless they’re forced to
(e.g. with a `return()` command). Also, a function will only look to
outside environments (a level “up”) to find an object if it can’t be
found within.

## Control flow

The order (“flow”) of statements and operations out functions evaluate.

### if and ifelse

With an `if` condition, we can alert the user when the default value is
used:

``` r
square <- 
  function(x = NULL){  ## Set initial default to NULL
    if(is.null(x)) {
      x=1  ## Reassign default to 1
      message("No input value provided. Using defalut value of 1.")
      }
    x_sq <- x^2
    df <- tibble(value=x, value_squared=x_sq)
    return(df)
  }
square() ## Leave argument blank to use default
```

    ## No input value provided. Using defalut value of 1.

    ## # A tibble: 1 x 2
    ##   value value_squared
    ##   <dbl>         <dbl>
    ## 1     1             1

`ifelse()` statements are written as:

    ifelse(CONDITION, DO IF TRUE, DO IF FALSE)

This uses `ifelse()` to test if the function is working as intended:

``` r
eval_square <-
  function(x) {
    if (square(x)$value_squared == x*x) { ## Condition
      ## What to do if condition is TRUE
      message("Function working as intended")
    } else {
      ## What to do if condition is FALSE
      message("Function is not working as intended")
    }
  }
eval_square(64)
```

    ## Function working as intended

#### ifelse gotchas and alternatives

Base R `ifelse()` works great normally. However, it behaves oddly when
you return a date:

``` r
## If argument=TRUE, return today's date
## If argument=FALSE or something else, return yesterday's date
today <- function(...) ifelse(..., Sys.Date(), Sys.Date()-1)
today(TRUE)
```

    ## [1] 18395

It returns the date in number format, which is annoying.

``` r
## Convert it to date format
as.Date(today(TRUE), origin="1970-01-01")
```

    ## [1] "2020-05-13"

**dplyr** and **data.table** provide their own functions which guard
against this problem:

  - `dplyr::if_else()`
  - `data.table::fifelse()` (“fast if-else”)

### case when (nested ifelse)

Nested `ifelse()` statements can be hard to read and
    troubleshoot:

    ifelse(CONDITION1, DO IF TRUE, ifelse(CONDITION2, DO IF TRUE, ifelse(...)))

A better solution is to use `CASE WHEN`, which originally came from SQL.
**dplyr** and **data.table** have functions for this:

``` r
data.table::fcase(
  x <= 3, "small",  ## Evaluated first
  x <= 7, "medium", ## Evaluated second
  default = "large" ## Defalut value
)
```

``` r
x = 1:10
dplyr::case_when(
  x <= 3 ~ "small",  ## Evaulated first
  x <= 7 ~ "medium", ## Evaluated second
  TRUE ~ "large"     ## Default value. `x > 7 ~ "big"` also works
)
```

    ##  [1] "small"  "small"  "small"  "medium" "medium" "medium" "medium" "large" 
    ##  [9] "large"  "large"

It works with data frames/tables too:

``` r
data.table(x = 1:10)[, grp := fcase(x <= 3, "small",
                                    x <= 7, "medium",
                                    default = "large")][]
```

``` r
tibble(x = 1:10) %>%
  mutate(grp = case_when(x <= 3 ~ "small",
                         x <= 7 ~ "medium",
                         TRUE ~ "large"))
```

    ## # A tibble: 10 x 2
    ##        x grp   
    ##    <int> <chr> 
    ##  1     1 small 
    ##  2     2 small 
    ##  3     3 small 
    ##  4     4 medium
    ##  5     5 medium
    ##  6     6 medium
    ##  7     7 medium
    ##  8     8 large 
    ##  9     9 large 
    ## 10    10 large

## Iteration

You don’t always need to iterate in R because it’s a *vectorized*
language, meaning you can apply a function to every element of a vector
at once, rather than one at a time. For example, you can do this with
the `:` operator, or with `c()`.

``` r
square(1:5)
```

    ## # A tibble: 5 x 2
    ##   value value_squared
    ##   <int>         <dbl>
    ## 1     1             1
    ## 2     2             4
    ## 3     3             9
    ## 4     4            16
    ## 5     5            25

### for loops

``` r
for(i in 1:5) print(LETTERS[i])
```

    ## [1] "A"
    ## [1] "B"
    ## [1] "C"
    ## [1] "D"
    ## [1] "E"

To “grow” an object via a *for* loop, we first must create an empty (or
NULL) object:

``` r
kelvin <- 300:305
fahrenheit <- NULL
# fahrenheit <- vector("double", length(kelvin))  ## Makes it work faster, because specifying the type and length allows R to more efficiently allocate memory

for(k in 1:length(kelvin)) {
  fahrenheit[k] <- kelvin[k] * 9/5 - 459.67
}
fahrenheit
```

    ## [1] 80.33 82.13 83.93 85.73 87.53 89.33

Historically, basic *for* loops in R were slower than other methods.
This has since been resolved, but *for* loops can clutter the global
environment with intermediary objects. Also, they clash with the best
practices of functional programming.

### Functional programming

*for* loops tend to emphasize the *objects* we’re working with (e.g. a
vector of numbers) rather than the *operations* we want to apply to them
(e.g. get the mean or median). This is inefficient because it requires
us to write out the for-loop by hand for each operation.

*for* loops also pollute the global environment with counting variables
like `i`, `k`, etc. And to “grow” an object as we iterate over it, we
have to create an empty object first.

FP has two implementations in R:

  - The `*apply` family of functions in base R
  - The `map*()` family of functions from **purr**.

#### lapply and co.

`lapply()` stands for *list apply* because it returns a list.

``` r
lapply(1:3, function(i) LETTERS[i])
```

    ## [[1]]
    ## [1] "A"
    ## 
    ## [[2]]
    ## [1] "B"
    ## 
    ## [[3]]
    ## [1] "C"

Note this does not create any global variables.

`lapply()` can take vectors, data frames, lists, etc. as arguments, but
the output will always be a list, where each element of the list is a
result from one iteration of the loop.

To get a data frame instead of a list, pipe the output to
`dplyr::bind_rows()`.

``` r
lapply(1:3, function(i) {
  df <- tibble(num = i, let = LETTERS[i])
  return(df)
  }) %>%
  bind_rows()
```

    ## # A tibble: 3 x 2
    ##     num let  
    ##   <int> <chr>
    ## 1     1 A    
    ## 2     2 B    
    ## 3     3 C

`sapply()` (“simplify apply”) is a wrapper around `lapply()` that tries
to return a simplified output that matches the input type. (e.g. if you
feed it a vector, it will try to return a vector.)

``` r
sapply(1:10, function(i) LETTERS[i])
```

    ##  [1] "A" "B" "C" "D" "E" "F" "G" "H" "I" "J"

**Progress bars:** `pbapply::pblapply()` is a wrapper around `lapply()`
that adds progress bar and ETA functionality in the console.

#### purrr

Part of the tidyverse, **purr** offers enhanced implementations of the
base `*apply()` functions. The main function is `purrr:map()`. Syntax is
identical to `base::lapply()`:

``` r
map(1:2, function(i) { ## only need to swap `lapply` for `map`
  df <- tibble(num = i, let = LETTERS[i])
  return(df)
  })
```

``` 
## [[1]]
## # A tibble: 1 x 2
##     num let  
##   <int> <chr>
## 1     1 A    
## 
## [[2]]
## # A tibble: 1 x 2
##     num let  
##   <int> <chr>
## 1     2 B    
```

`purrr::map_df()` returns a data frame instead of a list:

``` r
map_df(1:3, function(i) { ## don't need bind_rows with `map_df`
  df <- tibble(num = i, let = LETTERS[i])
  return(df)
  })
```

    ## # A tibble: 3 x 2
    ##     num let  
    ##   <int> <chr>
    ## 1     1 A    
    ## 2     2 B    
    ## 3     3 C

### Create and iterate over named functions

We can split the function and the iteration into separate steps. This is
good because typically we create functions with the goal of reusing
them.

``` r
## Create a named function
num_to_alpha <- 
  function(i) {
  df <- tibble(num = i, let = LETTERS[i])
  return(df)
  }
```

Now iterate over the function using different input values:

``` r
# lapply(1:3, num_to_alpha) %>% bind_rows()  ## Also works
map_df(1:3, num_to_alpha)
```

    ## # A tibble: 3 x 2
    ##     num let  
    ##   <int> <chr>
    ## 1     1 A    
    ## 2     2 B    
    ## 3     3 C

### Iterate over multiple inputs

We can iterate over e.g. 1:5, but what if a function has multiple
inputs? Consider this function that takes two variables `x` and `y`,
combines them in a data frame, and uses them to create a third variable
`z`:

``` r
## Create a named function
multi_func <- 
  function(x, y) {
  df <- 
    tibble(x = x, y = y) %>%
    mutate(z = (x + y)/sqrt(x))
  return(df)
  }
multi_func(1, 6)
```

    ## # A tibble: 1 x 3
    ##       x     y     z
    ##   <dbl> <dbl> <dbl>
    ## 1     1     6     7

There are two ways to iterate over both `x` and `y`:

  - use `mapply()` or `pmap()`
  - use a data frame of input combinations

#### `mapply()` and `pmap()`

`purrr::pmap()`

``` r
## Note that the inputs must be combined in a list.
pmap_df(list(x=1:3, y=6:8), multi_func)
```

    ## # A tibble: 3 x 3
    ##       x     y     z
    ##   <int> <int> <dbl>
    ## 1     1     6  7   
    ## 2     2     7  6.36
    ## 3     3     8  6.35

`base::mapply()`

``` r
## Note that the inputs are now moved to the *end* of the call. 
## mapply is based on sapply, so we also have to tell it not to simplify if we want to keep the list structure.
mapply(
  multi_func,
  x=1:3, ## Our "x" vector input
  y=6:8, ## Our "y" vector input
  SIMPLIFY = F ## Tell it not to simplify, to keep the list structure
  ) %>%
  bind_rows()
```

    ## # A tibble: 3 x 3
    ##       x     y     z
    ##   <int> <int> <dbl>
    ## 1     1     6  7   
    ## 2     2     7  6.36
    ## 3     3     8  6.35

#### Using a data frame of input combinations

“Cheat” by feeding multi-input functions a *single* data frame that
specifies combinations of variables by row. This is good because:

  - You don’t have to worry about accidentally feeding separate inputs
    of different lengths. (`pmap()` would at least fail and give you a
    helpful error message, but `mapply()` would complete with misaligned
    colummns.)
  - You can run a function over all possible combinations of different
    inputs. Use `base::expand.grid()` to generate a data frame of all
    combinations, and feed that as input to the function.
  - It’s simpler and cleaner to organize your input into one object.

Example:

``` r
parent_func <-
  ## Main function: Takes a single data frame as an input
  function(input_df) {
    df <-
      ## Nested iteration function
      map_df(
      1:nrow(input_df), ## i.e. Iterate (map) over each row of the input data frame
      function(n) {
        ## Extract the `x` and `y` values from row "n" of the data frame
        x <- input_df$x[n]
        y <- input_df$y[n]
        ## Use the extracted values
        df <- multi_func(x, y)
        return(df)
      })
    return(df)
    }
```

There are three conceptual steps to the above code chunk:

1.  First, I create a new function called parent\_func(), which takes a
    single input: a data frame containing x and y columns (and
    potentially other columns too).
2.  This input data frame is then passed to a second (nested) function,
    which will iterate over the rows of the data frame.
3.  During each iteration, the x and y values for that row are passed to
    our original multi\_func() function. This will return a data frame
    containing the desired output.

Let’s test that it worked using two different input data frames.

``` r
## Case 1: Iterate over x=1:5 and y=6:10
input_df1 <- tibble(x=1:3, y=6:8)
parent_func(input_df1)
```

    ## # A tibble: 3 x 3
    ##       x     y     z
    ##   <int> <int> <dbl>
    ## 1     1     6  7   
    ## 2     2     7  6.36
    ## 3     3     8  6.35

``` r
## Case 2: Iterate over *all possible combinations* of x=1:5 and y=6:10
input_df2 <- expand.grid(x=1:3, y=6:8)
# input_df2 <- expand(input_df1, x, y) ## Also works
parent_func(input_df2)
```

    ## # A tibble: 9 x 3
    ##       x     y     z
    ##   <int> <int> <dbl>
    ## 1     1     6  7   
    ## 2     2     6  5.66
    ## 3     3     6  5.20
    ## 4     1     7  8   
    ## 5     2     7  6.36
    ## 6     3     7  5.77
    ## 7     1     8  9   
    ## 8     2     8  7.07
    ## 9     3     8  6.35

## Further resources

  - [R for Data Science](http://r4ds.had.co.nz/) — esp. chapters [19
    (“Functions”)](http://r4ds.had.co.nz/functions.html) and [21
    (“Iteration”)](http://r4ds.had.co.nz/iteration.html) — covers much
    of the same ground as we have here, with emphasis on the purrr
    package for iteration.
  - In-depth: Hadley’s [Advanced R](https://adv-r.hadley.nz/) (2nd ed.)
    Covers all the concepts, including more on his (and R’s) philosophy
    on functional programming (see [Section
    ||](https://adv-r.hadley.nz/fp.html)).
  - Concise overview of the \*apply() functions: [blog
    post](https://nsaunders.wordpress.com/2010/08/20/a-brief-introduction-to-apply-in-r/).
  - [purrr tutorial](https://jennybc.github.io/purrr-tutorial)
    mini-website. (Bonus: She goes into more depth about working with
    lists and list columns.)
