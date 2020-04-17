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

### Key dplyr verbs

1. `filter`: Filter (i.e. subset) rows based on their values.
2. `arrange`: Arrange (i.e. reorder) rows based on their values.
3. `select`: Select (i.e. subset) columns by their names.
4. `mutate`: Create new columns.
5. `summarize`: Collapse multiple rows into a single summary value.


### dplyr::filter

We can chain multiple filter commands with pipes (`%>%`), or just separate them with commas within a single filter command:  
`starwars %>% filter(species == "Human", height >= 190)`

Regular expressions work too:  
`starwars %>% filter(grepl("Skywalker", name))`

* `grepl` is "logical" `grep` (it returns a search term match as true or false)

Identify missing data:  
`starwars %>% filter(is.na(height))`

* Use `!is.na` to exclude missing data


### dplyr::arrange

Order rows by value:  
`starwars %>% arrange(birth_year)`

Descending order:  
`starwars %>% arrange(desc(birth_year))`


### dplyr::select

Use commas to select multiple columns from a data frame. Use a colon to select consecutive columns. Deselect a column with `-`.  
`starwars %>% select(name:skin_color, species, -height`

Rename some (or all) variables in place:  
`starwars %>% select(alias=name, crib=homeworld, sex=gender)`

Select according to pattern (select all columns containing a certain phrase):  
`starwars %>% select(name, contains("color"))` returns hair_color, skin_color, eye_color

Bring some variables to the "front" of the data frame:  
`starwars %>% select(species, homeworld, everything())`

* *Note:* The new `relocate` function in dplyr 1.0.0 will bring more functionality to ordering of columns. See [here](https://www.tidyverse.org/blog/2020/03/dplyr-1-0-0-select-rename-relocate/).


### dplyr::mutate

Create new columns from scratch, or as transformations of existing columns:
```
starwars %>%
  select(name, birth_year) %>%
  mutate(dog_years = birth_year * 7) %>%
  mutate(comment = paste0(name, " is ", dog_years, " in dog years."))
```

*Note:* `mutate` is order aware. So you can chain multiple mutates in a single call:
```
starwars %>%
  select(name, birth year) %>%
  mutate(
    dog_years = birth_year * 7,
    comment = paste0(name, " is ", dog_years, " in dog years.")
    )
```

Boolean, logical, and conditional operators work with `mutate`:
```
starwars %>%
  select(name, height) %>%
  filter(name %in% c("Luke Skywalker", "Anakin Skywalker")) %>%
  mutate(tall1 = height > 180) %>%        ## values are FALSE and TRUE
  mutate(tall2 = ifelse(height > 180, "Tall", "Short"))   ## values are Short and Tall
```

* *Note:* Boolean values use less memory than strings.

Combine `mutate` with `across` to easily work on a subset of variables:  
```
starwars %>%
  select(name:eye_color) %>%
  mutate(across(is.character, toupper))   ## make all char-type variables uppercase
```

### dplyr::summarize

Very useful in combination with `group_by()`:
```
starwars %>%
  group_by(species, gender) %>%
  summarize(mean_height = mean(height, na.rm = T))
```

* Note that including `na.rm = T` is usually a good idea with summarize functions. Otherwise any missing value will propagate to the summarized value too.

The `across`-based workflow works with `summarize`:
```
starwars %>%
  group_by(species) %>%
  summarize(across(is.numeric, mean, na.rm=T))    ## Looks across columns, to keep numeric ones and return their respective mean
```

* Note: the `across`-based workflow is superseding old "scoped" variants of `summarize` as of dplyr v1.0.0.

### Other dplyr goodies

`group_by` and `ungroup`

* Useful with `summarize` and `mutate`.

`slice`: Subset rows by position rather than by values

* e.g. `starwars %>% slice(c(1, 5))`

`pull`: Extract a column from a data frame as a vector or scalar

* e.g. `starwars %>% filter(gender == "female") %>% pull(height)`

`count` and `distinct`: Number and isolate unique observations.

* e.g. `starwars %>% count(species)`, or `starwars %>% distinct(species)`

You could use a combination of `mutate`, `group_by`, and `n()`, e.g. `starwars %>% group_by(species) %>% mutate(num = n())`.

There are also a class of [window funcitons](https://cran.r-project.org/web/packages/dplyr/vignettes/window-functions.html) for getting leads and lags, ranking, creating cumulative aggregates, etc.

* See `vignette("window-functions")`.


### Joining operations

dplyr can do all kinds of join operations:

* `inner_join(df1, df2)`
* `left_join(df1, df2)`
* `right_join(df1, df2)`
* `full_join(df1, df2)`
* `semi_join(df1, df2)`
* `anti_join(df1, df2)`

Guide on join types [here](https://r4ds.had.co.nz/relational-data.html#understanding-joins).










## Data tidying with tidyr

## Summary
