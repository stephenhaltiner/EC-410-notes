# data.table

## Why use data.table?

1. Concise
2. Insanely fast
3. Memory efficient
4. Feature rich (and stable)
5. Dependency-free

## data.table basics

### The data.table object

The tidyverse provides its own enhanced version of a data.frame in the form of tibbles. The same is true for data.table. In fact, data.table functions only work on objects that have first been converted to data.tables.

* The specialized internal structure of data.table objects is a key reason the package is so fast.

Multiple ways to create a data.table:

* `data.table(x = 1:10)` creates a new data.table from scratch
* `as.data.table(df)` turns an existing data frame (df) into a data.table
* setDT(df) turns an existing data frame into a data.table *by reference*, meaning we don't have to reassign it

Note: CSVs imported via `fread()` are automatically converted to data.table.

### What does "modify by reference" mean?

There are two ways of changing or assigning objects in R:

1. **Copy-on-modify:** Creates a copy of your data. Implies extra computational overhead.
2. **Modify-in-place:** Avoids creating copies and simply changes the data where it sits in memory.

When we say data.table "modifies by reference", that means it modifies objects in place. Less memory and faster computation!

### data.table syntax

#### DT[i,j,by]

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

### j

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



## Keys

## Merging datasets

## Reshaping data

## data.table + tidyverse workflows

## Summary
