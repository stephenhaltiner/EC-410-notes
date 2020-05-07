# R Language Basics

**Introduction**

  * [Arithmetic](#arithmetic)
  * [Logical Operators](#logical-operators)
    + Floating-point numbers
  * [Assignment](#assignment)
    + Assignment with `<-`
    + Assignment with `=`
  * [Help](#help)
    + Vignettes

**Object-oriented programming in R**

  * [Everything is an object](#everything-is-an-object)
    + What are objects?
    + Object class, type, and structure
    + Global environment
    + Working with multiple objects
  * [Everything has a name](#everything-has-a-name)
    + Reserved words
    + Semi-reserved words
    + Namespace conflicts
  * [Indexing](#indexing)
    + Option 1: [ ]
    + Option 2: $
  * [Cleaning up](#cleaning-up)
    + Removing objects (and packages)
    + Removing plots

## Introduction

### Arithmetic

R recognizes all the standard arithmetic operators (`+`, `-`, `/`, `^`).  

We can also invoke modulo operators (integer division & remainder):  

* `100 %/% 60` (How many whole hours in 60 minutes?) returns `1`
* `100 %% 60` (How many minutes are left over?) returns `40`

### Logical Operators

* `1 > 2` returns `FALSE`
* `1 > 2 | 0.5` returns `TRUE` (`|` means "or")
* `1 > 2 & 0.5` returns `FALSE` (`&` means "and")
* `isTRUE (1 < 2)` returns `TRUE`

Negation (`!`) is handy for filtering data objects based on missing observations.

* `is.na(1:5)` returns `FALSE FALSE FALSE FALSE FALSE`
* `!is.na(1:5)` returns `TRUE TRUE TRUE TRUE TRUE`
* `Negate(is.na)(1:5)` also works.

Value matching (`%in%`): to see whether an object is contained within (i.e. matches one of) a list of items.

* `4 %in% 1:10` returns `TRUE`
* `4 %in% 5:10` returns `FALSE`

There's no "not-in" command, but we could create one:

* `` `%ni` <- Negate(`%in`) `` (The backticks help to specify functions.)

Evaluation: two equal signs

* `1 = 1` returns an error
* `1 == ` returns `TRUE`
* `1 != 2` returns `TRUE`

#### Floating-point numbers  

What happens when we evaluate `0.1 + 0.2 == 0.3`?
* returns `FALSE`

Why: Computers represent numbers as binary (base 2) floating-points. This is fast and memory efficient but can lead to unexpected behavior (similar to how base 10 can't precisely represent certain fractions, e.g. 1/3 = 0.3333...)

Solution: Use `all.equal()` for evaluating floats (i.e. fractions).

* `all.equal(0.1 + 0.2 + 0.3)` returns `TRUE`

### Assignment

In R, we can use either `=` or `<-` for assignment.  

#### Assignment with `<-`

`<-` is read as "gets." You can think of it as an arrow saying *assign in this direction*.  
`a <- 10 + 5`  
`a`  
returns `15`.

An arrow could point right, but this is rarely used.

#### Assignment with `=`

`b = 10 + 10`  
`b`  
returns `20`. (Note the assigned object *must* be on the left with `=`.)

Which to use?

* Most R users prefer `<-` for assignment, since `=` also has a specific role for evaluation *within* functions.
  + However, some say it doesn't matter. E.g. `=` is quicker to type and makes more sense if you're coming from another programming language.
  + Use whichever you prefer; just be consistent.
  
### Help

If you're struggling with a (named) function or object in R, type "help":

* e.g. `help(solve)`
* or simply use `?`, as in `?solve`.

For many packages you can also try the `vignette()` function, which provides an introduction to a package and its purpose with examples. This is often the best way to learn how to use a package.

* e.g. `vignette("dplyr")`

But, you must know the exact name of the package vignette(s).

* e.g. The `dplyr` package has several vignettes: "dplyr", "window-functions", "programming", etc.
* `vignette()` lists the vignettes of every *installed* package.
* `vignette(all = FALSE)` lists the vignettes of *loaded* packages.

#### Commenting:

* Comments in R are signified by `#`.
* Use `Ctrl+Shift+c` in RStudio to (un)comment sections of highlighted code.

## Object-oriented programming in R

### Everything is an object

#### What are objects?

There are many types (or classes) of objects. We regularly work with:

* vectors
* matrices
* data frames (there are different kinds of data frames; e.g. "tibbles")
* lists
* functions
* etc.

Each object class has its own set of rules governing how that object can be used in R.

* For example, you can perform many of the same operations on matrices and data frames, but some operations only work on a matrix, and vice versa.
* You can (often) convert an object from one type to another:
  + `df <- data.frame(x = 1:2, y = 3:4)` create a small data frame
  + `m <- as.matrix(df)` convert it to (i.e. create) a matrix called "m"
  
#### Object class, type, and structure

Use the `class`, `typeof`, and `str` commands to undertand more about a particular object.

* `class(df)` evaluates `df`'s class (returns `"data frame"`)
* `typeof(df)` evaluates its type (returns `"list"` [a data frame is a list of columns])
* `str(df)` shows its structure (returns ``` 'data.frame': 2 obs. of 2 variables' ```)

You can also inspect/print an object in the console (type `df` and hit Enter). The `View()` function is also helpful.

#### Global environment

If we run a regression on our simple data frame `df` like so:  
`lm (y ~ x)`  
we get an error: `object 'y' not found`. Why?

Because R can't find the variables we specified in our global environment. We have to tell R that the `x` and `y` we're looking for belong to the object `df`.

One solution is to specify the datasource:  
`lm(y ~ x, data = df)`

One of the main reasons R is better than Stata is that you can have multiple data frames loaded in memory at the same time in R.

#### Working with multiple objects

We can create a new data frame that exists happily alongside our existing objects in the global environment.  
`df2 <- data.frame(x = rnorm(10), y = runif(10))`

### Everything has a name

#### Reserved words

These are fundamental commands, operators, and relations in base R that you cannot (re)assign, even if you tried. For example:  
`if` `else` `while` `function` `for` `TRUE` `FALSE` `NULL` `Inf` `Nan` `NA`

#### Semi-reserved words

These are named functions of constants (e.g. `pi`) that you could re-assign, but already have important meanings from base R.

Arguably the most important one is `c()`, which is used for concatenation (creating vectors and binding different objects together).  
`my_vector <- c(1, 2, 5)`  
`my_vector`  
returns `1 2 5`

#### Namespace conflicts

Two loaded packages might have functions that share the same name.

* e.g. When you load `dplyr`, it "masks" some objects from `package:stats` and `package:base`.

When a namespace conflict arises, the most recently loaded package gains preference. So the `filter()` function refers to the `dplyr` variant.

If we want the `stats` variant, we have 2 options:

1. Temporarily use `stats::filter()` (sometimes it's good to be specific anyway)
2. Permanently assign `filter <- stats::filter` (permanent on a session-by-session basis)

The temporary `package::funtion()` option is generally preferred.

Another good rule of thumb is to load your most important packages last.

Pay attention to warnings when loading a new package. `?` is your friend if you're ever unsure (it will reveal which variant of an object is being used).

### Indexing

#### Option 1: [ ]

We've seen indexing in the form of R console output. For example:
```
1 + 2  

## [1] 3
```
The `[1]` denotes the first (and in this case, only) element of our output. In this case, it's a vector of length 1 equal to the value "3".

* Indexing in R begins at 1. In some other languages (e.g. Python and Javascript) it begins at 0.

We can use `[]` to index objects that we create:
```
a <- 1:10
a[4]      ## Get the 4th element of object "a"

## [1] 4

a[c(4, 6)]    ## Get the 4th and 6th elements

## [1] 4 6
```
It also works on larger arrays (vectors, matrices, data frames, lists). For example:
```
starwars[1:3, 1]  ## Show the cell corresponding to the 1st row & 1st column of the data frame.

## # A tibble: 3 x 1
##   name          
##   <chr>         
## 1 Luke Skywalker
## 2 C-3PO
## 3 R2-D2
```
**Lists** are a more complex type of array object in R.

* They can contain a random assortment of objects that don't share the same class, or have the same shape (e.g. rank) or common structure.
* e.g. A list can contain a scalar, a string, and a data frame. Or you can have a list of data frames, or a list of lists.

The relevance to indexing is that lists require two square brackets `[[]]` to index the parent list item and then the standard `[]` within that parent item. For example:
```
my_list <- list(a = "hello", b = c(1,2,3), c = data.frame(x = 1:5, y = 6:10))
my_list[[1]]    ## Return the 1st object of the parent list

## [1] "hello"

my_list[[2]][3]   ## Return the 3rd element of the 2nd object in the parent list

## [1] 3
```
#### Option 2: $

`$` is another indexing operator like `[ ]`.  
When you run `my_list` you get:
```
## $a
## [1] "hello"
## 
## $b
## [1] 1 2 3
## 
## $c
##   x  y
## 1 1  6
## 2 2  7
## 3 3  8
## 4 4  9
## 5 5 10
```
Notice how our (named) parent list objects are demarcated: `$a`, `$b`, and `$c`.

We can call these objects directly by name using the dollar sign:
```
my_list$a
## [1] "hello"

my_list$b[3]
## [1] 3

my_list$c$x
## [1] 1 2 3 4 5
```
The `$` form of indexing also works for other object types in R.

You could combine `[ ]` and `$` like so:  
`starwars$name[1]` returns `"Luke Skywalker"`

However, note the different output format compared to `starwars[1, 1]`. The output here is a string instead of a 1x1 tibble.

`$` provides another way to solve the "object not found" in global environment problem from before:

* `lm(y ~ x)` doesn't work
* `lm(df$y ~ df$x)` works!

### Cleaning up

#### Removing objects (and packages)

Use `rm()` to remove an object or objects from your working environment.

You could use `rm(list = ls())` to remove all objects in your working environment (except packages), but this is frowned upon. Better to just start a new R session.

Detaching packages is also complicated because there are many cross-dependencies. Better to just restart your R session.

#### Removing plots

`dev.off()` removes all plots that have been generated during your session. RStudio also has buttons for clearing your workspace environment and removing individual plots.
