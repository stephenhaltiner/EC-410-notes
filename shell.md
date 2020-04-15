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


