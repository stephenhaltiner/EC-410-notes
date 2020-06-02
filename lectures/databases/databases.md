Databases
================

#### Learning and practicing SQL

**dplyr**’s `show_query()` function is a great way to get started. So is
the [SQL translation
vignette](https://dbplyr.tidyverse.org/articles/sql-translation.html).

The best way to learn is to write your own queries. The [BigQuery
UI](https://console.cloud.google.com/bigquery) is helpful for this.

#### Resources

  - ["Getting Global Fishing Watch Data from Google BigQuery using
    R](http://jsmayorga.com/post/getting-global-fishing-watch-from-google-bigquery-using-r)
  - Julia Evans’ [*Become A Select
    Star*](https://wizardzines.com/zines/sql/)
  - Official BigQuery
    [documentation](https://cloud.google.com/bigquery/docs/) overviews
    the functions and syntax for “standard SQL”

#### Contents

  - [Databases and the tidyverse](#databases-and-the-tidyverse)

  - [Getting started with SQLite](#getting-started-with-sqlite)
    
      - [Connecting to a database](#connecting-to-a-database)
      - [Generating queries](#generating-queries)
      - [Laziness as a virtue](#laziness-as-a-virtue)
      - [Collect the data into your local R
        environment](#collect-the-data-into-your-local-r-environment)

  - [Using SQL directly in R](#using-sql-directly-in-r)
    
      - [Translate with
        dplyr::show\_query()](#translate-with-dplyr--show-query--)
      - [Option 1: Use R Markdown `sql`
        chunks](#option-1--use-r-markdown--sql--chunks)
      - [Option 2: Use
        DBI::dbGetQuery()](#option-2--use-dbi--dbgetquery--)
      - [Recommendation: Use
        glue::glue\_sql()](#recommendation--use-glue--glue-sql--)
      - [Disconnect](#disconnect)

  - [Google BigQuery](#google-bigquery)
    
      - [Example 1: US birth data](#example-1--us-birth-data)
      - [Example 2: Global Fishing
        Watch](#example-2--global-fishing-watch)

Many “big data” problems could be described as “small data problems in
disguise”, meaning the data we care about is just a small subset or
aggregation of a larger dataset. For example, we might want to access US
Census data…but only for a few counties along a state border. Or, we
might want to analyze climate data collected from many weather
stations…but aggregated to the national/monthly level. In these cases,
the bottleneck is interacting with the original data, which is too large
to fit in memory. We can store and access this data with **relational
databases**.

Databases can exist locally or remotely (though remotely is more
common). They are stored *on-disk* somewhere, rather than in-memory. We
extract the information we want by submitting a *query* to the database.
In the query, we can request that the data be manipulated or subsetted
in a certain way before it is delivered to us.

**A table in a database is like a data frame in an R list.**

## Databases and the tidyverse

**dbplyr** allows for direct communication with databases from your
local R environment. It’s provides a database backend to **dplyr**. It
depends on the **DBI** package, which provides a common interface to
allow **dplyr** to work with many different databases using the same
type of code.

For **DBI** to work, you must install a specific backend package for the
type of database you want to connect to.
[Here](https://db.rstudio.com/dplyr/#getting-started) is a list of
commonly used backend packages. We will use:

  - **RSQLite**, which embeds a SQLite database.
  - **bigrquery**, which connects to Google BigQuery.

SQLite is a lightweight SQL database engine that can exist on our local
computers (no need to connect to a server). BigQuery is a convenient way
to get data if you already have a Google Cloud account.

## Getting started with SQLite

(For more, see RStudio’s [Databases using
dplyr](https://db.rstudio.com/dplyr) tutorial.)

### Connecting to a database

Start by opening an (empty) database connection via the
`DBI::dbConnect()` function, which we’ll call `con`. Note, we are
calling the **RSQLite** package in the backgroud for the SQLite backend
and telling R this is a local connection that exists in memory.

``` r
# library(DBI)
con <- dbConnect(RSQLite::SQLite(), path = ":memory:")
```

The first argument is the database backend, i.e. `RSQLite::SQLite()`
since that’s what we’re using in this case. SQLite only needs one other
argument: the `path` to the database. We use the special string
`":memory:"` which causes SQLite to create a temporary, in-memory
database.

Our makeshift database is empty, so let’s copy in the *flights* dataset
from the **nycflights13** package. We’ll use `dplyr::copy_to()` to do
that.

The indexes enable efficient database performance. These are set by the
database host platform or maintainer in normal applications.

``` r
# library(nycflights13)
# library(dplyr)

copy_to(
  dest = con,
  df = nycflights13::flights,
  name = "flights",
  temporary = FALSE,
  indexes = list(
    c("year", "month", "day"),
    "carrier",
    "tailnum",
    "dest"
    )
  )
```

Now that we’ve copied the data to the database, we can reference it via
`dplyr::tbl()`:

``` r
# library(dbplyr)

flights_db <- tbl(con, "flights")
flights_db
```

    ## # Source:   table<flights> [?? x 19]
    ## # Database: sqlite 3.30.1 []
    ##     year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
    ##    <int> <int> <int>    <int>          <int>     <dbl>    <int>          <int>
    ##  1  2013     1     1      517            515         2      830            819
    ##  2  2013     1     1      533            529         4      850            830
    ##  3  2013     1     1      542            540         2      923            850
    ##  4  2013     1     1      544            545        -1     1004           1022
    ##  5  2013     1     1      554            600        -6      812            837
    ##  6  2013     1     1      554            558        -4      740            728
    ##  7  2013     1     1      555            600        -5      913            854
    ##  8  2013     1     1      557            600        -3      709            723
    ##  9  2013     1     1      557            600        -3      838            846
    ## 10  2013     1     1      558            600        -2      753            745
    ## # … with more rows, and 11 more variables: arr_delay <dbl>, carrier <chr>,
    ## #   flight <int>, tailnum <chr>, origin <chr>, dest <chr>, air_time <dbl>,
    ## #   distance <dbl>, hour <dbl>, minute <dbl>, time_hour <dbl>

### Generating queries

**dplyr** auto-translates tidyverse style code into SQL. Some examples:

``` r
## Select some columns
flights_db %>% select(year:day, dep_delay, arr_delay)
```

    ## # Source:   lazy query [?? x 5]
    ## # Database: sqlite 3.30.1 []
    ##     year month   day dep_delay arr_delay
    ##    <int> <int> <int>     <dbl>     <dbl>
    ##  1  2013     1     1         2        11
    ##  2  2013     1     1         4        20
    ##  3  2013     1     1         2        33
    ##  4  2013     1     1        -1       -18
    ##  5  2013     1     1        -6       -25
    ##  6  2013     1     1        -4        12
    ##  7  2013     1     1        -5        19
    ##  8  2013     1     1        -3       -14
    ##  9  2013     1     1        -3        -8
    ## 10  2013     1     1        -2         8
    ## # … with more rows

``` r
## Filter according to a condition
flights_db %>% filter(dep_delay > 240)
```

    ## # Source:   lazy query [?? x 19]
    ## # Database: sqlite 3.30.1 []
    ##     year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
    ##    <int> <int> <int>    <int>          <int>     <dbl>    <int>          <int>
    ##  1  2013     1     1      848           1835       853     1001           1950
    ##  2  2013     1     1     1815           1325       290     2120           1542
    ##  3  2013     1     1     1842           1422       260     1958           1535
    ##  4  2013     1     1     2115           1700       255     2330           1920
    ##  5  2013     1     1     2205           1720       285       46           2040
    ##  6  2013     1     1     2343           1724       379      314           1938
    ##  7  2013     1     2     1332            904       268     1616           1128
    ##  8  2013     1     2     1412            838       334     1710           1147
    ##  9  2013     1     2     1607           1030       337     2003           1355
    ## 10  2013     1     2     2131           1512       379     2340           1741
    ## # … with more rows, and 11 more variables: arr_delay <dbl>, carrier <chr>,
    ## #   flight <int>, tailnum <chr>, origin <chr>, dest <chr>, air_time <dbl>,
    ## #   distance <dbl>, hour <dbl>, minute <dbl>, time_hour <dbl>

``` r
## Get the mean delay by destination
flights_db %>%
  group_by(dest) %>%
  summarise(delay = mean(dep_time)) %>%
  arrange(desc(delay))
```

    ## Warning: Missing values are always removed in SQL.
    ## Use `mean(x, na.rm = TRUE)` to silence this warning
    ## This warning is displayed only once per session.

    ## # Source:     lazy query [?? x 2]
    ## # Database:   sqlite 3.30.1 []
    ## # Ordered by: desc(delay)
    ##    dest  delay
    ##    <chr> <dbl>
    ##  1 CHO   2057.
    ##  2 TUL   2032.
    ##  3 LEX   2026 
    ##  4 ABQ   2006.
    ##  5 TYS   1997.
    ##  6 OKC   1996.
    ##  7 ILM   1974.
    ##  8 SMF   1946.
    ##  9 BHM   1944.
    ## 10 CRW   1882.
    ## # … with more rows

### Laziness as a virtue

**dplyr** tries to be as “lazy” as possible. Your R code is translated
into SQL and executed in the database, not in R. This is good because:

  - It never pulls data into R unless you explicitly ask.
  - It delays doing any work until the last possible moment: it collects
    everything you want to do and sends it to the database in one step.

For example, if we’re interested in the mean departure and arrival
delays for each plane (unique tail number):

``` r
tailnum_delay_db <- 
  flights_db %>%
  group_by(tailnum) %>%
  summarise(
    mean_dep_delay = mean(dep_delay),
    mean_arr_delay = mean(arr_delay),
    n = n()                            ## Count number of obs.
  ) %>%
  arrange(desc(mean_arr_delay)) %>%
  filter(n >= 100)
```

This sequence of operations never even touches the database. It’s not
until you ask for the data (e.g. by calling `tailnum_delay_db`) that
**dplyr** actually generates the SQL and requests the results from the
database. Even then, it limits the number of rows it pulls:

``` r
tailnum_delay_db
```

    ## # Source:     lazy query [?? x 4]
    ## # Database:   sqlite 3.30.1 []
    ## # Ordered by: desc(mean_arr_delay)
    ##    tailnum mean_dep_delay mean_arr_delay     n
    ##    <chr>            <dbl>          <dbl> <int>
    ##  1 N11119            32.6           30.3   148
    ##  2 N16919            32.4           29.9   251
    ##  3 N14998            29.4           27.9   230
    ##  4 N15910            29.3           27.6   280
    ##  5 N13123            29.6           26.0   121
    ##  6 N11192            27.5           25.9   154
    ##  7 N14950            26.2           25.3   219
    ##  8 N21130            27.0           25.0   126
    ##  9 N24128            24.8           24.9   129
    ## 10 N22971            26.5           24.7   230
    ## # … with more rows

### Collect the data into your local R environment

Typically you’ll iterate a few times before figuring out what data you
need from the database. Then, use `collect()` to pull it into a local
data frame:

``` r
tailnum_delay <- 
  tailnum_delay_db %>%
  collect()
tailnum_delay
```

    ## # A tibble: 1,218 x 4
    ##    tailnum mean_dep_delay mean_arr_delay     n
    ##    <chr>            <dbl>          <dbl> <int>
    ##  1 N11119            32.6           30.3   148
    ##  2 N16919            32.4           29.9   251
    ##  3 N14998            29.4           27.9   230
    ##  4 N15910            29.3           27.6   280
    ##  5 N13123            29.6           26.0   121
    ##  6 N11192            27.5           25.9   154
    ##  7 N14950            26.2           25.3   219
    ##  8 N21130            27.0           25.0   126
    ##  9 N24128            24.8           24.9   129
    ## 10 N22971            26.5           24.7   230
    ## # … with 1,208 more rows

Now we see it has 1,218 rows. It’s stored as a data frame, so we can use
it just like any other.

Question: What is the relationship between late departures and late
arrivals?

``` r
tailnum_delay %>%
  ggplot(aes(x = mean_dep_delay,
             y = mean_arr_delay,
             size = n)) +
  geom_point(alpha = 0.3) +
  geom_abline(intercept = 0, slope = 1, col = "orange") +
  coord_fixed()
```

    ## Warning: Removed 1 rows containing missing values (geom_point).

![](databases_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

Assuming we’re finished querying our SQLite database, we’d disconnect
from it by calling `DBI::dbDisconnect(con)`.

## Using SQL directly in R

### Translate with dplyr::show\_query()

Behind the scenes, **dplyr** is translating your R code to SQL. Use
`show_query()` to display the SQL code that was used to generate a
queried table:

``` r
tailnum_delay_db %>%
  show_query()
```

    ## <SQL>
    ## SELECT *
    ## FROM (SELECT *
    ## FROM (SELECT `tailnum`, AVG(`dep_delay`) AS `mean_dep_delay`, AVG(`arr_delay`) AS `mean_arr_delay`, COUNT() AS `n`
    ## FROM `flights`
    ## GROUP BY `tailnum`)
    ## ORDER BY `mean_arr_delay` DESC)
    ## WHERE (`n` >= 100.0)

There are some translation artifacts, because dplyr implements
safeguards to ensure the translated SQL will absolutely work. This
explains the repeated `SELECT` commands.

SQL uses a *lexical* order of operations that doesn’t always preserve
the *logical* order of operations, meaning the way we write a query is
not the same way we think through a query. (SQL has a strict “order of
execution”, so there is a hierarchy of commands.)
[Here](https://wizardzines.com/zines/sql/samples/from.png) is a graphic
explaining it, from the zine [“Become a Select
Star”](https://wizardzines.com/zines/sql/).

The main confusion point is, we always write `SELECT` first, even though
`SELECT` doesn’t come into play until later in the query’s logic.

The **DBI** package lets you write and submit SQL queries directly from
R.

``` r
## Show the equivalent SQL query for a group of dplyr commands
flights_db %>% filter(dep_delay > 240) %>% head(5) %>% show_query()
```

    ## <SQL>
    ## SELECT *
    ## FROM `flights`
    ## WHERE (`dep_delay` > 240.0)
    ## LIMIT 5

Note: the backticks around object names, and the parentheses around
`WHERE`, are just safeguards that are not always necessary.

### Option 1: Use R Markdown `sql` chunks

Just use the header ` ```{sql, connection = con} ` instead of
` ```{r} `. (Change `con` to your own database connection.)

Once knitted, this looks like:

``` sql
SELECT *
FROM flights
WHERE dep_delay > 240
LIMIT 5
```

<div class="knitsql-table">

| year | month | day | dep\_time | sched\_dep\_time | dep\_delay | arr\_time | sched\_arr\_time | arr\_delay | carrier | flight | tailnum | origin | dest | air\_time | distance | hour | minute | time\_hour |
| ---: | ----: | --: | --------: | ---------------: | ---------: | --------: | ---------------: | ---------: | :------ | -----: | :------ | :----- | :--- | --------: | -------: | ---: | -----: | ---------: |
| 2013 |     1 |   1 |       848 |             1835 |        853 |      1001 |             1950 |        851 | MQ      |   3944 | N942MQ  | JFK    | BWI  |        41 |      184 |   18 |     35 | 1357081200 |
| 2013 |     1 |   1 |      1815 |             1325 |        290 |      2120 |             1542 |        338 | EV      |   4417 | N17185  | EWR    | OMA  |       213 |     1134 |   13 |     25 | 1357063200 |
| 2013 |     1 |   1 |      1842 |             1422 |        260 |      1958 |             1535 |        263 | EV      |   4633 | N18120  | EWR    | BTV  |        46 |      266 |   14 |     22 | 1357066800 |
| 2013 |     1 |   1 |      2115 |             1700 |        255 |      2330 |             1920 |        250 | 9E      |   3347 | N924XJ  | JFK    | CVG  |       115 |      589 |   17 |      0 | 1357077600 |
| 2013 |     1 |   1 |      2205 |             1720 |        285 |        46 |             2040 |        246 | AA      |   1999 | N5DNAA  | EWR    | MIA  |       146 |     1085 |   17 |     20 | 1357077600 |

5 records

</div>

### Option 2: Use DBI::dbGetQuery()

To run SQL queries in regular R scripts, use `DBI::dbGetQuery()`:

``` r
## Run the query using SQL directly on the connection.
dbGetQuery(con, "SELECT * FROM flights WHERE dep_delay > 240 LIMIT 5")
```

    ##   year month day dep_time sched_dep_time dep_delay arr_time sched_arr_time
    ## 1 2013     1   1      848           1835       853     1001           1950
    ## 2 2013     1   1     1815           1325       290     2120           1542
    ## 3 2013     1   1     1842           1422       260     1958           1535
    ## 4 2013     1   1     2115           1700       255     2330           1920
    ## 5 2013     1   1     2205           1720       285       46           2040
    ##   arr_delay carrier flight tailnum origin dest air_time distance hour minute
    ## 1       851      MQ   3944  N942MQ    JFK  BWI       41      184   18     35
    ## 2       338      EV   4417  N17185    EWR  OMA      213     1134   13     25
    ## 3       263      EV   4633  N18120    EWR  BTV       46      266   14     22
    ## 4       250      9E   3347  N924XJ    JFK  CVG      115      589   17      0
    ## 5       246      AA   1999  N5DNAA    EWR  MIA      146     1085   17     20
    ##    time_hour
    ## 1 1357081200
    ## 2 1357063200
    ## 3 1357066800
    ## 4 1357077600
    ## 5 1357077600

### Recommendation: Use glue::glue\_sql()

Grant recommends the `glue_sql()` function from **glue**, part of the
tidyverse. It provides a more integrated approach that lets you use
local R variables in your SQL queries, and divide long queries into
sub-queries.

``` r
## Using R variables in a SQL query

# library(glue)

## Some local R variables
tbl <- "flights"
d_var <- "dep_delay"
d_thresh <- 240

## The "glued" SQL query string
sql_query <-
  glue_sql("
  SELECT *
  FROM {`tbl`}
  WHERE ({`d_var`} > {d_thresh})
  LIMIT 5
  ", 
  .con = con
  )

## Run the query
dbGetQuery(con, sql_query)
```

    ##   year month day dep_time sched_dep_time dep_delay arr_time sched_arr_time
    ## 1 2013     1   1      848           1835       853     1001           1950
    ## 2 2013     1   1     1815           1325       290     2120           1542
    ## 3 2013     1   1     1842           1422       260     1958           1535
    ## 4 2013     1   1     2115           1700       255     2330           1920
    ## 5 2013     1   1     2205           1720       285       46           2040
    ##   arr_delay carrier flight tailnum origin dest air_time distance hour minute
    ## 1       851      MQ   3944  N942MQ    JFK  BWI       41      184   18     35
    ## 2       338      EV   4417  N17185    EWR  OMA      213     1134   13     25
    ## 3       263      EV   4633  N18120    EWR  BTV       46      266   14     22
    ## 4       250      9E   3347  N924XJ    JFK  CVG      115      589   17      0
    ## 5       246      AA   1999  N5DNAA    EWR  MIA      146     1085   17     20
    ##    time_hour
    ## 1 1357081200
    ## 2 1357063200
    ## 3 1357066800
    ## 4 1357077600
    ## 5 1357077600

`glue_sql()` really pays off when working with large, nested queries.
[Documentation
here](https://glue.tidyverse.org/reference/glue_sql.html).

### Disconnect

``` r
## Terminate connection with the database
dbDisconnect(con)
```

## Google BigQuery

There are tons of public datasets available on BigQuery. The console has
a nice web UI too.

Here is a
[tutorial](https://towardsdatascience.com/bigquery-without-a-credit-card-discover-learn-and-share-199e08d4a064)
for using BigQuery to determine who is the most popular Alan (measured
by Wikipedia page views).

To use the **bigrquery** package, you must provide your GCP project
billing ID. We stored our credentials in the `.Renviron` file in our
home directory during the cloud computing lecture. In that case, you can
call it using `Sys.getenv()`.

``` r
library(bigrquery)
billing_id <- Sys.getenv("GCE_DEFAULT_PROJECT_ID")  ## Replace with your project ID if this doesn't work
```

### Example 1: US birth data

Establish a connection using `DBI::dbConnect()`:

``` r
# library(DBI)
# library(dplyr)

bq_con <- 
  dbConnect(
    bigrquery::bigquery(),  ## Specify the BigQuery backend
    project = "publicdata",
    dataset = "samples",
    billing = billing_id  ## Provide credentials
    )
```

This connection works for any tables in the database. Set up
auto-authentication, then list the tables:

``` r
bq_auth(path = Sys.getenv("GCE_AUTH_FILE")) ## Or you could authenticate manually instead
 
dbListTables(bq_con)
```

    ## [1] "github_nested"   "github_timeline" "gsod"            "natality"       
    ## [5] "shakespeare"     "trigrams"        "wikipedia"

Create a pointer to the “natality” table in our local environment:

``` r
natality <- tbl(bq_con, "natality")
```

Summarize as mean, and collect data into our session:

``` r
bw <- 
  natality %>%
  filter(!is.na(state)) %>% ## remove some outliers
  group_by(year) %>%
  summarise(weight_pounds = mean(weight_pounds, na.rm=T)) %>%
  collect()
```

``` r
bw %>%
  ggplot(aes(year, weight_pounds)) +
  geom_line()
```

![](databases_files/figure-gfm/unnamed-chunk-22-1.png)<!-- -->

Query again, but grouping by year, state, and gender:

``` r
bw_st <- 
  natality %>%
  filter(!is.na(state)) %>%
  group_by(year, state, is_male) %>%
  summarise(weight_pounds = mean(weight_pounds, na.rm=T)) %>%
  mutate(gender = ifelse(is_male, "Male", "Female")) %>%
  collect()
```

``` r
## Select arbitrary states to highlight
states <- c("CA","DC","OR","TX","VT")
## Rearranging the data will help with the legend ordering
bw_st <- bw_st %>% arrange(gender, year)

## Plot it
bw_st %>%
  ggplot(aes(year, weight_pounds, group=state)) + 
  geom_line(col="grey75", lwd = 0.25) + 
  geom_line(
    data = bw_st %>% filter(state %in% states), 
    aes(col=fct_reorder2(state, year, weight_pounds)),
    lwd=0.75
    ) +
  facet_wrap(~gender) + 
  scale_color_brewer(palette = "Set1", name=element_blank()) +
  labs(
    title = "Mean birth weight, by US state over time",
    subtitle = "Selected states highlighted",
    x = NULL, y = "Pounds",
    caption = "Data sourced from Google BigQuery"
    ) + 
  theme_ipsum(grid=F)
```

![](databases_files/figure-gfm/unnamed-chunk-24-1.png)<!-- -->

Remember to disconnect.

``` r
dbDisconnect(bq_con)
```

### Example 2: Global Fishing Watch

``` r
gfw_con <- 
  dbConnect(
    bigrquery::bigquery(),
    project = "global-fishing-watch",
    dataset = "global_footprint_of_fisheries",
    billing = billing_id
    )
```

``` r
dbListTables(gfw_con)
```

    ## [1] "fishing_effort"          "fishing_effort_byvessel"
    ## [3] "fishing_vessels"         "vessels"

``` r
effort <- tbl(gfw_con, "fishing_effort")
effort
```

    ## # Source:   table<fishing_effort> [?? x 8]
    ## # Database: BigQueryConnection
    ##    date   lat_bin lon_bin flag  geartype vessel_hours fishing_hours mmsi_present
    ##    <chr>    <int>   <int> <chr> <chr>           <dbl>         <dbl>        <int>
    ##  1 2012-…    -879    1324 AGO   purse_s…        5.76          0                1
    ##  2 2012-…   -5120   -6859 ARG   trawlers        1.57          1.57             1
    ##  3 2012-…   -5120   -6854 ARG   purse_s…        3.05          3.05             1
    ##  4 2012-…   -5119   -6858 ARG   purse_s…        2.40          2.40             1
    ##  5 2012-…   -5119   -6854 ARG   trawlers        1.52          1.52             1
    ##  6 2012-…   -5119   -6855 ARG   purse_s…        0.786         0.786            1
    ##  7 2012-…   -5119   -6853 ARG   trawlers        4.60          4.60             1
    ##  8 2012-…   -5118   -6852 ARG   trawlers        1.56          1.56             1
    ##  9 2012-…   -5118   -6850 ARG   trawlers        1.61          1.61             1
    ## 10 2012-…   -5117   -6849 ARG   trawlers        0.797         0.797            1
    ## # … with more rows

Which country fishes the most?

``` r
effort %>%
  group_by(flag) %>%
  summarise(total_fishing_hours = sum(fishing_hours, na.rm = T)) %>%
  arrange(desc(total_fishing_hours)) %>%
  collect()
```

    ## # A tibble: 126 x 2
    ##    flag  total_fishing_hours
    ##    <chr>               <dbl>
    ##  1 CHN             57711389.
    ##  2 ESP              8806223.
    ##  3 ITA              6790417.
    ##  4 FRA              6122613.
    ##  5 RUS              5660001.
    ##  6 KOR              5585248.
    ##  7 TWN              5337054.
    ##  8 GBR              4383738.
    ##  9 JPN              4347252.
    ## 10 NOR              4128516.
    ## # … with 116 more rows

#### Date partitioning

Many tables in BigQuery are *date partitioned*, i.e. ordered according
to timestamps of when the data were ingested. This provides a more
cost-effective way to query datasets. But, it necessitates a minor tweak
to how we query or manipulate this GFW data by date.

In native SQL we reference a special `_PARTITIONTIME` pseudo-column in
order to, say, filter by date. There is no exact **dplyr** translation
for this pseudo-column. But, we can specify it as a SQL variable in our
**dplyr** call using backticks:

``` r
effort %>%
  filter(
    `_PARTITIONTIME` >= "2016-01-01 00:00:00",
    `_PARTITIONTIME` < "2017-01-01 00:00:00"
    ) %>%
  group_by(flag) %>%
  summarise(total_fishing_hours = sum(fishing_hours, na.rm=T)) %>%
  arrange(desc(total_fishing_hours)) %>%
  collect()
```

    ## # A tibble: 121 x 2
    ##    flag  total_fishing_hours
    ##    <chr>               <dbl>
    ##  1 CHN             16882037.
    ##  2 TWN              2227341.
    ##  3 ESP              2133990.
    ##  4 ITA              2103310.
    ##  5 FRA              1525454.
    ##  6 JPN              1404751.
    ##  7 RUS              1313683.
    ##  8 GBR              1248220.
    ##  9 USA              1235116.
    ## 10 KOR              1108384.
    ## # … with 111 more rows

#### Global fishing effort in 2016

``` r
## Define the desired bin resolution in degrees (longitude/latitude)
resolution <- 1

globe <-
  effort %>%
  filter(
    `_PARTITIONTIME` >= "2016-01-01 00:00:00",
    `_PARTITIONTIME` <= "2016-12-31 00:00:00"
    ) %>%
  filter(fishing_hours > 0) %>%
  mutate(
    lat_bin = lat_bin/100,  ## Normalize data (not super important)
    lon_bin = lon_bin/100
    ) %>%
  mutate(  ## Find center of the 1-degree square bins, and round observations to the bin centers
    lat_bin_center = floor(lat_bin/resolution)*resolution + 0.5*resolution,
    lon_bin_center = floor(lon_bin/resolution)*resolution + 0.5*resolution
    ) %>%
  group_by(lat_bin_center, lon_bin_center) %>%
  summarise(fishing_hours = sum(fishing_hours, na.rm=T)) %>%
  collect()
```

``` r
globe %>% 
  filter(fishing_hours > 1) %>% 
  ggplot() +
  geom_tile(aes(x=lon_bin_center, y=lat_bin_center, fill=fishing_hours))+
  scale_fill_viridis_c(
    name = "Fishing hours (log scale)",
    trans = "log",
    breaks = scales::log_breaks(n = 5, base = 10),
    labels = scales::comma
    ) +
  labs(
    title = "Global fishing effort in 2016",
    subtitle = paste0("Effort binned at the ", resolution, "° level."),
    y = NULL, x = NULL,
    caption = "Data from Global Fishing Watch"
    ) +
  theme_ipsum(grid=F) +
  theme(axis.text=element_blank())
```

![](databases_files/figure-gfm/unnamed-chunk-32-1.png)<!-- -->

``` r
dbDisconnect(gfw_con)
```
