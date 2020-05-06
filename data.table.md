# data.table

data.table is a powerful data wrangling package. It has concise syntax and incredible speed. It is also lightweight.

**Basic syntax:** `DT[i, j, by]`

* `i` On which rows?
* `j` Do what?
* `by` Grouped by what?

data.table (re)introduces some new ideas like *modify by reference* (`:=`) and syntax (`.()`, `.SD`, `.SDcols`, etc).

* This helps maximize performance while maintaining concise, consistent syntax.

**Use keys** to order your data for a major speed boost.

**Contents:**

* [data.table basics](#data.table-basics)
  + [The data.table object](#the-data.table-object)
  + [Modify by reference](#modify-by-reference)
  + [Syntax](#syntax)
  
* [Working with rows: DT[i, ]](#working-with-rows-DT-i)
  + [Order by rows (arrange)](#order-by-rows-arrange)
  
* [Manipulating columns: DT[, j]](#manipulating-columns--dt---j-)
  + [Modifying columns](#modifying-columns)
    - [Aside: chaining data.table operations](#aside-chaining-data.table-operations)
    
  + [Subsetting on columns](#subsetting-on-columns)
  + [Aggregating](#aggregating)
  
* [Grouping: DT[,,by]](#grouping-dt-by)
  + [Efficient subsetting with .SD](#efficient-subsetting-with-.sd)
  + [keyby](#keyby)
  
* [Keys](#keys)
  + [Setting a key](#setting-a-key)
  
* [Merging datasets (joins)](#merging-datasets-joins)
* [Reshaping data](#reshaping-data)
  + [Wide to long](#wide-to-long)
  + [Long to wide](#long-to-wide)
  
* [data.table + tidyverse workflows](#data.table-tidyverse-workflows)

## data.table basics

### The data.table object

The tidyverse provides its own enhanced version of a data.frame in the form of tibbles. The same is true for data.table. In fact, data.table functions only work on objects that have first been converted to data.tables.

* The specialized internal structure of data.table objects is a key reason the package is so fast.

**Create a data.table:**

* `data.table(x = 1:10)` creates a new data.table from scratch
* `as.data.table(df)` turns an existing data frame (df) into a data.table
* setDT(df) turns an existing data frame into a data.table *by reference*, meaning we don't have to reassign it

Note: CSVs imported via `fread()` are automatically converted to data.table.

### Modify by reference

There are two ways of changing or assigning objects in R:

1. **Copy-on-modify:** Creates a copy of your data. Implies extra computational overhead.
2. **Modify-in-place:** Avoids creating copies and simply changes the data where it sits in memory.

When we say data.table "modifies by reference", that means it modifies objects in place. Less memory and faster computation!

### Syntax

**DT[i,j,by]**

* `i`: On which rows?
* `j`: What do do?
* `by`: Grouped by what?

dplyr equivalents:

* `filter()`, `slice()`, `arrange()`
* `select()`, `mutate()`
* `group_by()`

The tidyverse tends to break up operations step-by-step, but data.table aims to do everything in one operation.


## Working with rows: DT[i, ]

Subsetting by rows works the same as you'd expect from dplyr:

* `DT[x == "string", ]`: Subset to rows where variable x == "string"
* `DT[y > 5, ]`: Subset to rows where variable y > 5
* `DT[1:10, ]`: Subset to the first 10 rows

Multiple filters/conditions work too:

* `DT[x=="string" & y>5, ]`

When we're only subsetting on `i`, we don't need commas:

* `DT[1:10]` is equivalent to `DT[1:10, "]`

### Order by rows (arrange)

data.table provides an optimized `setorder()` function for reordering *by reference*:

* `setorder(starwars_dt, birth_year, na.last = TRUE)`

But you could still do it this way:

* `starwars_dt[order(birth_year)]` (temporarily) sort by youngest to oldest


## Manipulating columns: DT[, j]

Recall some dplyr verbs we used to manipulate our variables:

* `select()`, `mutate()`, `summarize()`, `count()`...

data.table recognizes all these are different versions of telling R: "Do something to this variable in my dataset." All these operations take place in the `j` slot.

* This concision requires some syntax tweaks, but it's not too bad.

### Modifying columns

Use the `:=` operator to add, delete, or change columns in a data.table. (Called the "walrus" operator.)

* `DT[, xsq := x^2]`: Create a new column (`xsq`) from an existing one (`x`)
* `DT[, x := as.character(x)]`: Change an existing column

**Important:** `:=` is *modifying by reference*, i.e. in place. So we don't have to (re)assign the object to save these changes.

Add `[]` to see changes printed to screen:

```
DT[, x_sq := x^2][]

##    x x_sq
## 1: 1    1
## 2: 2    4
```
Or type `DT` on a new line.

**Sub-assign by reference**

One implication of `:=` is data.table's [sub-assign by reference](https://rdatatable.gitlab.io/data.table/articles/datatable-reference-semantics.html#ref-i-j) functionality. You can specify which rows to target with `i` and sub-assign directly with `j`.

To modify multiple columns simultaneously, we have two options:

1. LHS := RHS form: `DT[, c("var1", "var2") := .(val1, val2)]`
2. Functional form: `DT[, `:=` (var1=val1, var2=val2)]

Grant prefers functional form, so that's what the examples will use.

* `DT[, `:=` (y = 3:4, y_name = c("three", "four"))]`

However, note that dynamically assigning columns in a single step (like with dplyr::mutate) doesn't work:

```
DT[, `:=` (z = 5.6, z_sq = z^2)][]

## Error: object 'z' not found
```

#### Aside: chaining data.table operations

The native data.table way is to append consecutive `[]` terms.

```
DT[, z := 5:6][, z_sq := z^2][]

##    x x_sq y y_name z z_sq
## 1: 1    1 3  three 5   25
## 2: 2    4 4   four 6   36
```
Or use the **magrittr** pipe (just prefix each step with `.`):
```
library(magrittr)
DT %>%
  .[, xyz := x+y+z] %>%
  .[, xyz_sq := xyz^2] %>%
  .[]

##    x x_sq y y_name z z_sq xyz xyz_sq
## 1: 1    1 3  three 5   25   9     81
## 2: 2    4 4   four 6   36  12    144
```

To remove a column from your dataset, set it to NULL.

* `DT[, y_name := NULL]`

### Subsetting on columns ()

We can use the `j` slot to subset our data on columns.

Subset by column position:
```
starwars_dt[, c(1:3, 10)] %>% head(2)
##              name height mass homeworld
## 1: Luke Skywalker    172   77  Tatooine
## 2:          C-3PO    167   75  Tatooine
```
Or by name:
```
# starwars_dt[, c("name", "height", "mass", "homeworld")] ## Also works
# starwars_dt[, list(name, height, mass, homeworld)] ## So does this
starwars_dt[, .(name, height, mass, homeworld)] %>% head(2)
##              name height mass homeworld
## 1: Luke Skywalker    172   77  Tatooine
## 2:          C-3PO    167   75  Tatooine
```

**`.()` is a data.table shortcut for `list()`.**

You can often use these three forms interchangeably in data.table:

* `.(var1, var2, ...)`
* `list(var1, var2, ...)`
* `c("var1", "var2", ...)`

Exclude columns through negation:

* `starwars_dt[, !c("name", "height")]`

Rename a column:

* `setnames(starwars_dt, old="name", new="alias")[]`

dplyr's `select()` and `rename()` are a little more user-friendly. If you want, you can use the same verbs on data.tables:

* `starwars_dt %>% select(homeworld, everything())`

### Aggregating

Aggregate within `j`:

* `starwars_dt[, mean(height, na.rm=T)]`

Note: we don't keep anything unless we assign the result to an object. To add the new aggregated column to your original dataset, use `:=`.

* `starwars_dt[, mean_height := mean(height, na.rm=T)]`

data.table also provides specialized [symbols](https://rdatatable.gitlab.io/data.table/reference/special-symbols.html) for common aggregation tasks in `j`.

E.g. count the number of observations using `.N`:

* `starwars_dt[, .N]`


## Grouping: DT[,,by]

data.table's `by` argument works like `dplyr::group_by`.

* `starwars_dt[, mean(height, na.rm=T), by = species]`: Collapse by single variable
* `starwars_dt[, .(species_height = mean(height, na.rm=T)), by = species]`: Same, but explicitly name the summary variable
* `starwars_dt[, mean(mass, na.rm=T), by = height>190]`: Conditionals work too.
* `starwars_dt[, species_n := .N, by = species][]`: Add an aggregated column to the data (here: number of observations by species group)

Summarize multiple variables at once (the tedious way):
```
starwars_dt[, 
            .(mean(height, na.rm=T),
              mean(mass, na.rm=T),
              mean(birth_year, na.rm=T)), 
            by = species]
```

### Efficient subsetting with .SD

The `.SD` symbol stands for "subsetting data."

```
starwars_dt[, 
            lapply(.SD, mean, na.rm=T),   ## Specify what to do on the data subset (.SD), iterate with lapply()
            by = species, 
            .SDcols = c("height", "mass", "birth_year")]  ## Specify which columns to subset
```
Note, the `.()` syntax doesn't work with `.SDcols`, but `:` does (e.g. `.SDcols = height:mass`).

Omitting `.SDcols` will apply the function to *all* variables in the dataset.

### keyby

The `keyby` argument works like `by` but it also orders the results and creates a **key**.


## Keys

Keys are for ordering the data, allowing for *extremely* fast subsetting.

Imagine we want to filter a dataset based on a particular value (e.g. find all human characters in the starwars dataset.)

* Normally, we'd have to search through the whole dataset to identify matching cases.
* But if we set an appropriate key, then the data are already ordered in such a way that the computer only has to search through a much smaller subset.

**Analogy:** A filing cabinet has a drawer for "ABC", another for "DEF", etc. To find Fred's file, we only have to search the second drawer, not the whole cabinet.

### Setting a key

You can set a key when you first create a data.table:

* `DT = data.table(x = 1:10, y = LETTERS[1:10], key = "x")`
* `DT = as.data.table(DF, key = "x")`
* `setDT(DF, key = "x")`

Or use `setkey()` on an existing data.table:

* `setkey(DT, x)` (Note the key doesn't have to be quoted here.)

Since keys just describe a particular ordering of the data, you can set a key on *multiple* columns:

* `DT = as.data.table(DF, key = c("x", "y"))`
* `setkey(DT, x, y)`

Use `key()` to see what keys are currently set for a data.table. A table can only have one key at a time.


## Merging datasets (joins)

data.table provides two ways to merge datasets:

* `DT1[DT2, on = "id"]`
* `merge(DT1, DT2, by = "id")`

The second way offers extra functionality (`?merge.data.table`).

Summary of the different join options: see [here](https://atrebas.github.io/post/2019-03-03-datatable-dplyr/#joinbind-data-sets).

**dplyr**
```
left_join(
  flights, 
  planes, 
  by = "tailnum"
  )
```

**data.table**
```
merge(
  flights_dt, 
  planes_dt, 
  all.x = TRUE, ## omit for inner join
  by = "tailnum")
```

When there is an ambiguous same-name column in each table, both methods create "column.x" and "column.y" variants. With dplyr, we avoid this by using `rename()`. With data.table, use `setnames()`:
```
merge(
  setnames(flights_dt, old = "year", new = "year_built"),
  planes_dt, 
  all.x = TRUE, 
  by = "tailnum")
```

**Use keys for super fast joins**
```
flights_dt_key = as.data.table(flights, key = "tailnum")
planes_dt_key = as.data.table(planes, key = "tailnum")
merge_dt_key = function() merge(flights_dt_key, planes_dt_key, by = "tailnum")
```

## Reshaping data

With tidyverse, we can reshape data using the `tidyr::pivot*` functions.

data.table has functions for reshaping data too:

* `dcast()`: convert wide data to long data
* `melt()`: convert long data to wide data

The **tidyfast** package implements data.table versions of the `tidyr::pivot*` functions:

* `tidyfast::dt_pivot_longer()`: wide to long
* `tidyfast::dt_pivot_wider()`: long to wide

### Wide to long
```
stocks
##          time          X         Y         Z
## 1: 2009-01-01  0.3522752  4.068978 7.3494008
## 2: 2009-01-02 -0.3102111 -1.763021 0.7670371
```
Convert to long: two options

* `melt(stocks, id.vars="time", variable.name="stock", value.name="price")`
* `stocks %>% dt_pivot_longer(X:Z, names_to="stock", values_to="price")`

### Long to wide
```
stocks_long
##          time stock      price
## 1: 2009-01-01     X  0.3522752
## 2: 2009-01-02     X -0.3102111
## 3: 2009-01-01     Y  4.0689778
## 4: 2009-01-02     Y -1.7630212
## 5: 2009-01-01     Z  7.3494008
## 6: 2009-01-02     Z  0.7670371
```

* `dcast(stocks_long, time ~ stock, value.var="price")`
* `stocks_long %>% dt_pivot_wider(names_from=stock, values_from=price)`


## data.table + tidyverse workflows

Use which package makes sense for a given situation. Remember you can use tidyverse verbs on data.tables:

* `starwars_dt %>% group_by(homeworld) %>% summarize(height = mean(height, na.rm=T))`

This incurs a performance penalty - luckily there is dtplyr.

**dtplyr** provides a data.table "back-end" for dplyr. Write your code as if using dplyr, and it gets translated as data.table code.

* dtplyr is 36x faster than dplyr, and 0.5x as fast as data.table.
* dtplyr automatically prints its data.table translation to screen.

