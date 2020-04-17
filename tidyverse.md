# Tidyverse

[What is tidy data?](#what-is-tidy-data?)  
[Tidyverse basics](#tidyverse-basics)  

* [Tidyverse vs. base R](#tidyverse-vs.-base-r)
* [Tidyverse packages](#tidyverse-packages)
* [An aside on pipes](#an-aside-on-pipes)

[Data wrangling with dplyr](#data-wrangling-with-dplyr)

* 

[Data tidying with tidyr](#data-tidying-with-tidyr)

* 

[Summary](#summary)

## What is tidy data?

* Each variable forms a column.
* Each observation forms a row.
* Each type of observational unit forms a table.

Basically, tidy data is more likely to be [long/"narrow" format](https://en.wikipedia.org/wiki/Wide_and_narrow_data) than wide format.

## Tidyverse basics

### Tidyverse vs. base R

Tidyverse is great because:

* The documentation and community support are outstanding.
* Having a consistent philosophy and syntax makes it easy to learn.
* It provides a convenient front-end to some key big-data tools.
* For data cleaning, wrangling, and plotting, it is a no-brainer.
  + (The **data.table** package is also pretty great, though.)

But, a combination of tidyverse and base R is often the best solution to a problem.

There is often a direct correspondence between a tidyverse command and its base R equivalent. These generally follow a `tidyverse::snake_case` vs `base::period.case` rule.

### Tidyverse packages

The tidyverse actually comes with a lot more packages than those that are loaded automatically, e.g. **lubridate**.

The two workhorse packages for cleaning and wrangling data are **dplyr** and **tidyr**.

### An aside on pipes

In R, the pipe operator is `%>%` and is automatically loaded with the tidyverse.

Why use pipes?  
These next two lines of code do exactly the same thing.  
`mpg %>% filter(manufacturer=="audi") %>% group_by(model) %>% summarise(hwy_mean = mean(hwy))`  
`summarise(group_by(filter(mpg, manufacturer=="audi"), model), hwy_mean = mean(hwy))`

The first one is much easier to understand.


## Data wrangling with dplyr

## Data tidying with tidyr

## Summary
