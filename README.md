# My first repo!

Hello world!

Another line of text.

Now there's a script file too.

No merge conflicts here.




# Git shell commmands

Clone the repo
`$ git clone REPOSITORY-URL`

See the commit history (hit spacebar to scroll down or q to exit).
`$ git log`

What has changed?
`$ git status`

Stage ("add") a file or group of files.
`$ git add NAME-OF-FILE-OR-FOLDER`

You can use wildcard characters to stage a group of files (e.g. sharing a common prefix). There are a bunch of useful flag options too:

* Stage all files.
  `$ git add -A`
* Stage updated files only (modified or deleted, but not new).
  `$ git add -u`
* Stage new files only (not updated).
  `$ git add .`

Commit your changes.
`$ git commit -m "Helpful message"`

Pull from the upstream repository (i.e. GitHub).
`$ git pull`

Push any local changes that you've commited to the upstream repo (i.e. GitHub).
`$ git push`


