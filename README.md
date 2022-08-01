# git-99-percent
99% of the Git commands you'll need at work, demonstrated in a single script (bitbucket.org/bitpusher16)
##########
# contents

* contents
* notes
* script setup
* git config files
* local versioning within a single commit
* local versioning across commits
* local branches
* remote repos
* remote repos with multiple users
* commands that can change published commit history
* misc
* script cleanup

## notes


* this script creates repos in a sandbox directory and uses them for demos.
* add an exit command anywhere to stop the script and examine repo state.
* rerun the script freely. it cleans up the sandbox before each run.
* the sandbox dir is ignored so created files are not added to any uber repo.

## terminology

* working, staged, committed <=> filesystem, index, commit graph
* ref => a branch ref or a tag ref

## direction of git diff

* "git diff commitA commitB", where commitA predates commitB, 
* will show the changes required to transform commitA into commitB.
* similarly, when diffing within a single commit 
* (e.g., diff staged and committed),
* git shows the changes needed to transform older state into newer state.
* in this case, committed is "older", 
* so git diff shows how to transform committed into staged.
	
## examining repo state

* suggest to set the git la alias described and run it often.
* it's an easy way to visualize HEAD, branches, tags, and the commit graph.
