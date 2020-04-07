# My first test repo

Hello world!

Another line of text.

Now there's a script file too.

No merge conflicts here.


## Git

### 4 main Git commands

1. **Stage (or "add"):** Tell Git that you want to add changes to the repo history (file edits, additions, deletions, etc.)
2. **Commit:** Tell Git that, yes, you are sure these changes should be part of the repo history.
3. **Pull:** Get any new changes made on the GitHub repo (i.e. the upstream remote), either by your collaborators or you on another machine.
4. **Push:** Push any (commited) local changes to the GitHub repo

### Git shell commmands

* Clone the repo.
`$ git clone REPOSITORY-URL`
* See the commit history (hit spacebar to scroll down or q to exit).
`$ git log`
* What has changed?
`$ git status`
* Stage ("add") a file or group of files.
`$ git add NAME-OF-FILE-OR-FOLDER`
+ You can use wildcard characters to stage a group of files (e.g. sharing a common prefix). There are a bunch of useful flag options too:
  - `$ git add -A` Stage all files.
  - `$ git add -u` Stage updated files only (modified or deleted, but not new).
  - `$ git add .` Stage new files only (not updated).

* Commit your changes.
`$ git commit -m "Helpful message"`
* Pull from the upstream repository (i.e. GitHub).
`$ git pull`
* Push any local changes that you've commited to the upstream repo (i.e. GitHub).
`$ git push`


