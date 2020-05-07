# Using the Shell

[Navigation](#navigation)  
[List objects in a directory](#list-objects-in-a-directory)  
[Create files and directories](#create-files-and-directories)  
[Copy objects](#copy-objects)  
[Move (and rename)](#move-\(and-rename\))  
[Rename en masse](#rename-en-masse)  
[Wildcards](#wildcards)  
[Find](#find)  
[Counting text](#counting-text)  
[Reading text](#reading-text)  
[Preview text](#preview-text)  
[Find patterns](#find-patterns)  
[Manipulate text](#manipulate-text)  
[Sorting and removing duplicates](#sorting-and-removing-duplicates)  
[Redirect](#redirect)  
[Pipes](#pipes)  
[For loops](#for-loops)  
[Scripts](#scripts)  
[Permissions](#permissions)  
[Extras](#extras)  
[Further reading](#further-reading)  

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

### Redirect

Send output from the shell to a file using the redirect operator, `>`.  

* `echo "At first, I was afraid, I was petrified" > survive.txt`
    + `echo` prints text to the shell
    + `find survive.txt` to see that it now exists  
    
* to *append* text to an existing file, use `>>`.
    + `echo "'Kept thinking I could never live without you by my side" >> survive.txt`

### Pipes
You can send (i.e. "pipe") intermediate output to another command. This way, you can chain together a sequence of simple operations to create a more complex operation.  

* `cat -n sonnets.txt | head -n 100 | tail -n 10` returns the last 10 lines of the first 100 lines of the whole .txt file (i.e. lines 91-100)  

Use pipes to search through your Bash command history:

* `cat ~/.bash_history | grep head` searches history for "head"
    + Every shell command you type is stored in a `~/.bash_history` file.

### For loops

Basic syntax:  
> for i in LIST  
> do  
>   OPERATION \$i  
> done  

Or `for i in LIST; do OPERATION $i; done`

* the `$` sign indicates a variable in bash  
* e.g. `for i in {1..5}; do echo $i; done` returns `1; 2; 3; 4; 5`

* e.g. `for i in $(ls *day.csv); do tail -n +2 $i >> mealplan.csv; done`
    + The results are sorted alphabetically by default, but we can fix that in R.
    + Combining .csv files in the shell this way doesn't require loading all the files into RAM at the same time, which is what would happen if you tried to combine them in R or any other typical program.

### Scripts

Shell scripts have the `.sh` file extension.  

They start with a "shebang": `#!/bin/sh` (indicating which program to run the command with - here, a bash-compatible shell). Shebangs are typically ignored - note it begins with the comment character `#`.  

To run a script, type the relative path of the file. The path is needed, otherwise bash thinks the scriptfile.sh is an internal command.

* e.g. `./hello.sh` runs the script called `hello.sh`.

For editing scripts in the shell, the text editor [nano](www.nano-editor.org) is recommended.

### Permissions

There are two main user roles on a Unix system:  
1) Normal users  
2) Superuser (a.k.a. "root")  
This separation is a security feature. Superuser priveleges are required to install software and manage important parts of the filesystem.  
  
You could log in as the superuser, but that's terrible practice because then there are no safety checks for anything.

Instead, temporarily invoke superuser status with `sudo`.  

#### Changing Permissions
Use `chmod`.

**Example 1: rwxrwxrwx**

* Octal notation: `chmod -R 777 myfolder`
* Symbolic notation: `chmod -R a=rwx myfolder`  
  
**Example 2: rwxr-xr-x**

* Octal notation: `chmod -R 755 myfolder`
* Symbolic notation: `chmod -R u=rwx,g=rx,o=rx myfolder`  

Symbolic notation arguments:

* Users: `u` (user/owner), `g` (group), `o` (others), `a` (all)
* Permissions: `r` (read), `w` (write), `x` (execute)
* Changes: `+` (add permissions), `-` (remove permissions), `=` (set permissions)

Reassign ownership of a file/folder with `chown -R username myfolder`.

### Extras

* Use the [htop](https://hisham.hm/htop/) program for memory management
* Use [cron jobs](https://www.tecmint.com/create-and-manage-cron-jobs-on-linux/) for automating tasks

### Further reading

* [The Unix Shell](http://swcarpentry.github.io/shell-novice/) (Software Carpentery)
* [The Unix Workbench](https://seankross.com/the-unix-workbench/) (Sean Kross)
* [Data Science at the Command Line](https://www.datascienceatthecommandline.com/) (Jeroen Janssens)
* [Using AWK and R to parse 25tb](https://livefreeordichotomize.com/2019/06/04/using_awk_and_r_to_parse_25tb/) (Nick Strayer)
