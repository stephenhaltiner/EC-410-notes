# Tidyverse

[Key verbs](#key-verbs)  

**Tidyverse basics**

* [What is tidy data?](#what-is-tidy-data)
* [Tidyverse vs base R](#tidyverse-vs-base-r)
* [Tidyverse packages](#tidyverse-packages)
* [Pipes](#pipes)

**dplyr (for data wrangling)**

* [dplyr::filter](#dplyrfilter)
* [dplyr::arrange](#dplyrarrange)
* [dplyr::select](#dplyrselect)
* [dplyr::mutate](#dplyrmutate)
* [dplyr::summarize](#dplyrsummarize)
* [Other dplyr goodies](#other-dplyr-goodies)
* [Joining operations](#joining-operations)

**tidyr (for data cleaning)**

* [tidyr::pivot_longer](#tidyrpivot-longer)
* [tidyr::pivot_wider](#tidyrpivot-wider)
* [tidyr::separate](#tidyrseparate)
* [tidyr::unite](#tidyrunite)
* [tidyr::crossing](#tidyrcrossing)
* [tidyr::expand and tidyr::complete](#tidyrexpand-and-tidyrcomplete)


## Key verbs

**dplyr**

1. `filter`: Filter (i.e. subset) rows based on their values.
2. `arrange`: Arrange (i.e. reorder) rows based on their values.
3. `select`: Select (i.e. subset) columns by their names.
4. `mutate`: Create new columns.
5. `summarize`: Collapse multiple rows into a single summary value. Often used with `group_by`.

**tidyr**

* `pivot_longer`: Pivot wide data into long format (i.e. "melt").
  + Updated verstion of `tidyr::gather`.
* `pivot_wider`: Pivot long data into wide format (i.e. "cast").
  + Updated version of `tidyr::spread`.
* `separate`: Separate (i.e. split) one column into multiple columns.
* `unite`: Unite (i.e. combine) multiple columns into one.

**Also useful:**

* `%>%` (pipes)
* `group_by`
* `left_join` and other join functions

## Tidyverse basics

### What is tidy data?

* Each variable forms a column.
* Each observation forms a row.
* Each type of observational unit forms a table.

Basically, tidy data is more likely to be [long/"narrow" format](https://en.wikipedia.org/wiki/Wide_and_narrow_data) than wide format.

### Tidyverse vs base R

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

### Pipes

In R, the pipe operator is `%>%` and is automatically loaded with the tidyverse.

Why use pipes?  
These next two lines of code do exactly the same thing.  
`mpg %>% filter(manufacturer=="audi") %>% group_by(model) %>% summarise(hwy_mean = mean(hwy))`  
`summarise(group_by(filter(mpg, manufacturer=="audi"), model), hwy_mean = mean(hwy))`

The first one is much easier to understand.


## Data wrangling with dplyr

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

You can also pipe a data frame to a join operation:  
`df1 %>% left_join(df2)`

Guide on join types [here](https://r4ds.had.co.nz/relational-data.html#understanding-joins).

`left_join(flights, planes)`  
Note dplyr makes a guess about which columns to match.

Specify which columns to match with `by`:  
`left_join(flights, planes, by = "tailnum")`

You may want to rename ambiguous columns beforehand to avoid confusion:  
`planes %>% rename(year_built = year)`


## Data tidying with tidyr

### tidyr::pivot_longer
Example data frame:
```
stocks <- data.frame(
  time = as.Date('2009-01-01') + 0:1,
  X = rnorm(2, 0, 1),
  Y = rnorm(2, 0, 2),
  Z = rnorm(2, 0, 4)
  )

stocks
        time         X         Y         Z
1 2009-01-01  1.691159 -2.217228 12.697924
2 2009-01-02 -0.712833 -1.409637  2.661915
```

`tidy_stocks <- stocks %>% pivot_longer(-time, names_to="stock", values_to="price")`
```
tidy_stocks
  time       stock  price
  <date>     <chr>  <dbl>
1 2009-01-01 X      1.69 
2 2009-01-01 Y     -2.22 
3 2009-01-01 Z     12.7  
4 2009-01-02 X     -0.713
5 2009-01-02 Y     -1.41 
6 2009-01-02 Z      2.66 
```

### tidyr::pivot_wider
`tidy_stocks %>% pivot_wider(names_from=stock, values_from=price)`
```
  time            X     Y     Z
  <date>      <dbl> <dbl> <dbl>
1 2009-01-01  1.69  -2.22 12.7 
2 2009-01-02 -0.713 -1.41  2.66
```

`tidy_stocks %>% pivot_wider(names_from=time, values_from=price)`
```
  stock `2009-01-01` `2009-01-02`
  <chr>        <dbl>        <dbl>
1 X             1.69       -0.713
2 Y            -2.22       -1.41 
3 Z            12.7         2.66 
```

* Note that this second example has effectively transposed the data.

#### Remembering pivot syntax

The argument order is *"names"* and then *"values"*.

### tidyr::separate

Divide cells horizontally:
```
economists <- data.frame(name = c("Adam.Smith", "Paul.Samuelson", "Milton.Friedman"))

economists
##              name
## 1      Adam.Smith
## 2  Paul.Samuelson
## 3 Milton.Friedman
```

`economists %>% separate(name, c("first_name", "last_name"))`
```
##   first_name last_name
## 1       Adam     Smith
## 2       Paul Samuelson
## 3     Milton  Friedman
```
The command can guess the separation character, or you can specify it with `separate(..., sep=".")`.


Divide cells **vertically**:
`separate_rows` splits up cells that contain multiple fields or observations (which is common in survey data):
```
jobs <- data.frame(
  name = c("Jack", "Jill"),
  occupation = c("Homemaker", "Philosopher, Philanthropist, Troublemaker") 
  ) 
jobs

##   name                                occupation
## 1 Jack                                 Homemaker
## 2 Jill Philosopher, Philanthropist, Troublemaker
```
`jobs %>% separate_rows(occupation)`
```
##   name     occupation
## 1 Jack      Homemaker
## 2 Jill    Philosopher
## 3 Jill Philanthropist
## 4 Jill   Troublemaker
```

### tidyr::unite
```
gdp <- data.frame(
  yr = rep(2016, times = 4),
  mnth = rep(1, times = 4),
  dy = 1:4,
  gdp = rnorm(4, mean = 100, sd = 2)
  )
gdp

##     yr mnth dy       gdp
## 1 2016    1  1 100.97242
## 2 2016    1  2  99.80214
## 3 2016    1  3  99.76375
## 4 2016    1  4 100.68899
```
Combine "yr", "mnth", and "dy" into one "date" column:
`gdp_u <- gdp %>% unite(date, c("yr", "mnth", "dy"), sep = "-") %>% as_tibble()`
```
## # A tibble: 4 x 2
##   date       gdp
##   <chr>    <dbl>
## 1 2016-1-1 101. 
## 2 2016-1-2  99.8
## 3 2016-1-3  99.8
## 4 2016-1-4 101.
```
Note: `unite` automatically creates a character variable. We can fix this with `dplyr::mutate` and `lubridate::ymd`:
```
library(lubridate)
gdp_u %>% mutate(date = ymd(date))

## # A tibble: 4 x 2
##   date         gdp
##   <date>     <dbl>
## 1 2016-01-01 101. 
## 2 2016-01-02  99.8
## 3 2016-01-03  99.8
## 4 2016-01-04 101.
```

### tidyr::crossing

Gives you the full combination of a group of variables:  
`crossing(side=c("left", "right"), height=c("top", "bottom"))`
```
  side  height
  <chr> <chr> 
1 left  bottom
2 left  top   
3 right bottom
4 right top 
```

### tidyr::expand and tidyr::complete
`expand` and `complete` allow you to fill in (implicit) missing data or variable combinations in existing data frames.

