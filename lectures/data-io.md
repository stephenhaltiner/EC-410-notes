# Data I/O


## CSVs
readr::read_csv()
readr::write_csv()

**data.table**
data.table::fread()
You can read multiple files and bind together

data.table::fwrite()

**vroom**
vroom() is the fastest way to read in data

## Other data formats

**fst**
fst::read_fst()
fst::write_fst()

.fst files are super efficient for storing dataframes, but only R can read them

**arrow (parquet and feather)**

works in both R and python, but needs to be installed
faster than most other methods for data I/O


**disk.frame**

allows you to manipulate data larger than your RAM

does this by working in chunks, temporarily saving excess as .fst



