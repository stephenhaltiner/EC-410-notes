# Using the Shell

### Navigation

* `pwd` to print the current working directory
* `cd` to change directory

Special symbols:

* `~` user's home folder
* `.` current directory
* `..` parent directory

Examples:

* `cd ~` brings you to the home directory
* `cd ../..` brings you back up two directories

### List objects in a directory

* `ls -lh` list (**l**ong format, **h**uman readable)

Interpreting `ls` output:

`drwxr-xr-x  3 stephenhaltiner  staff   102B Apr  9 11:27 Documents`

* First column denotes the object type:
    + `d` (directory or folder), `l` (link), or `-` (file)
* Next we see the read/write/execute (`r` / `w` / `x`) permissions for: 1) owner, 2) the owner's group, and 3) all other users.
    + `-` denotes missing permissions for a class of operations.
* `3` is the number of hard links to the object.
* Next is the identity of the object's owner and their group.
* Then come size, date and time of creation, and the object name.

### Create files and directories

Create directories with `mkdir`

* e.g. `mkdir testing`

Create (empty) files with `touch`

* e.g. `touch testing/test1.txt testing/test2.txt`

### Remove items

Remove files with `rm`

* e.g. `rm testing/test1.txt`
* e.g. `rm -rf testing/` deletes the directory and its contents

Arguments:

* `r` (recursive): removes a directory's contents as well
* `f` (force): skips the warning prompt ("Are you sure you want to delete?")

### Copy objects

`cp object path/copyname`

* `-f` will force an overwrite if there is already a file with the same name

### Move (and rename)

`mv object path/newobjectname`

* Note that "moving" an object to the same directory but with a new name is effectively the same as renaming it.

### Rename *en masse*

syntax is `rename pattern replacement file(s)`

* e.g. `rename -s csv TXT meals/monday.csv`
* note: in macOS homebrew, the `rename` command needs the `-s` flag

### Wildcards
Special characters used as placeholders. The two most important are:

* `*` replaces any number of characters
    + Useful for selecting a group of similarly-named files
    + `cp examples/*.sh examples/copies` copies everything with extension ".sh"
    + `rm examples/copies/*` deletes everything in the "copies" directory
    
* `?` replaces a single character
    + Useful for discriminating between similarly-named files
    + `ls examples/meals/??nday.csv` returns monday and sunday
    + `ls examples/meals/?onday.csv` returns monday

### Find

Locate files and directories based on a variety of criteria, from object properties to pattern matching.


* `find . -iname "monday.csv"` searches pwd (recursively) for matching files
* `find . -iname "*.txt"` searches pwd for any .txt files
    + `-iname` indicates a case-insensitive search by filename

* `find . -size +100K` finds files larger than 100 KB

### Counting text

* `wc` returns 1) lines of text, 2) number of words, and 3) number of characters

### Reading text

* read everything: `cat` (concatenate)
    + the `-n` flag shows line numbers
    
The `more` and `less` commands provide additional functionality over `cat`. For example, you can move through long text one page at a time.

* Move forward and back with the "f" and "b" keys, and quit with "q".

### Preview text

`head` and `tail` let you preview a specified number of rows, using the `-n` flag (default is 10).

* `head -n 5 sonnets.txt` prints the first 5 rows of text.
* `tail -n 3 sonnets.txt` prints the last 3 rows.
* `tail -n +3024 sonnets.txt` prints lines #3024 to the end.

### Find patterns
Use regular expression-type matching with `grep`

* e.g. `grep -n 'Shall I compare thee' sonnets.txt` returns the full line
* `-n` (number) returns the line number

You can search multiple files or folders too:

* `grep -Rli 'pasta' /meals`
* `-R` (recursive) and `l` (just list the files, don't print the output), `-i` (ignore case)

### Manipulate text
`sed` and `awk`

* `sed -i 's/Jack/Bill/g' nursery.txt` (`-i` is case-insensitive)

`## Jack and Jill`

`## Went up the hill`

becomes

`## Jack and Bill`

`## Went up the hill`

* `sed -e 's/\s/\n/g' < sonnets.txt | sort | uniq -c | sort -nr | head -10`
    + This uses spaces (`\s`) as delineators for splitting text into separate lines (`\n`)
    + It works a [little differently](https://unix.stackexchange.com/questions/13711/differences-between-sed-on-mac-osx-and-other-standard-sed) on macOS though.

### Sorting and removing duplicates

* `sort -u reps.txt` sorts the text file line-by-line and removes duplicate lines. Lines are displayed alphabetically by default.





