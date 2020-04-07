# My first test repo

Hello world!

## How we will use Git

1. Create a repo on GitHub and initialize with a README.
2. `$ git clone REPOSITORY-URL` Clone the repo to your local machine. Preferably using an RStudio Project.
3. `$ git add -A` Stage any changes you make
4. `$ git commit -m "Helpful message"` Commit your changes
5. `$ git pull` Pull from GitHub
6. Fix any merge conflicts.
7. `$ git push` Push your changes to GitHub
  Repeat steps 3-7 (but especially steps 3 and 4) often.

### Tracking changes
* `$ git log` See the commit history (hit spacebar to scroll down or q to exit).
* `$ git status` See what has changed.

### Staging files
* `$ git add NAME-OF-FILE-OR-FOLDER` Stage ("add") a file or group of files.  
You can use wildcard characters to stage a group of files (e.g. sharing a common prefix). There are a bunch of useful flag options too:
* `$ git add -A` Stage all files.
* `$ git add -u` Stage updated files only (modified or deleted, but not new).
* `$ git add .` Stage new files only (not updated).



