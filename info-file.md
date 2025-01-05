
# Script Setup

The following script setup commands are used to determine the directory of the current script and initialize the sandbox:

```bash
SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"
cd $SCRIPT_DIR
mkdir -p $SCRIPT_DIR/sandbox
```

This gets the directory where the current script resides, as long as the last element in the path is not a symlink. For more details, refer to [this explanation](https://stackoverflow.com/a/246128/2512141).

To ensure the sandbox directory does not contain unexpected content:

```bash
EXP_CONT='\(repoA\|localA\|localB\|remoteUpstream\|remoteDownstream\)'
UNEXP_CONT=$(ls sandbox/ | sed "/$EXP_CONT/d" | wc -l)
if [[ $UNEXP_CONT -gt 0 ]]
then
    echo 'found some unexpected content in sandbox dir.'
    echo 'did you already have a directory named sandbox?'
    echo 'this script will clean out sandbox. best to run it from an empty dir.'
    exit
fi
rm -rf $SCRIPT_DIR/sandbox/*
```

Ensure the `sandbox` directory is included in `.gitignore`:

```bash
if ! grep -q 'sandbox' ./.gitignore 2> /dev/null
then
    printf "sandbox\n" >> ./.gitignore
fi
```

## Git Config Files

To list git configurations along with their sources:

```bash
git config --list --show-origin
```

Example output:
- `C:/Program Files/Git/mingw64/etc/gitconfig` (system)
- `C:/Users/$USER/.gitconfig` (global)
- `.git/config` (local)

```bash
git config --list --show-origin --system
git config --list --show-origin --global
git config --list --show-origin --local
```

Contents of the global gitconfig:

```bash
cat ~/.gitconfig
```

List git aliases:

```bash
git config --get-regexp alias
```

To set an alias:

```bash
git config --global alias.la log --all --graph --oneline --decorate
```

To unset an alias:

```bash
git config --global --unset alias.la
```

To manually add an alias:

```bash
vim ~/.gitconfig
# [alias]
#     la = log --all --graph --oneline --decorate
```

Set username and email globally:

```bash
git config --global user.name "john smith"
git config --global user.email "jsmith@domain.com"
```

## Local Versioning Within a Single Commit

Initialize a git repository:

```bash
REPA=$SCRIPT_DIR/sandbox/repoA
mkdir -p $REPA
cd $REPA
git init
git status
```

Add changes to the staging area:

```bash
printf "apple\npear\npeach\n" >> fruits.txt
git add -A
```

Prefer using `-A` instead of `.` because `-A` captures all changes in the repository, while `.` only captures changes in the current directory.

Commit the staged changes:

```bash
git commit -m 'added apple, pear, peach'
```

Restore the working directory to the last committed state:

```bash
git reset --hard HEAD
git clean -x -d -f
```

Check differences between working, staged, and committed states:

```bash
git diff
git diff --cached
git diff HEAD
```

Remove a file from all states:

```bash
rm mushrooms.txt
git add -A
git commit -m 'removed mushrooms.txt'
```

Move a file to `.gitignore`:

```bash
git rm --cached milks.txt
printf "milks.txt\n" >> .gitignore
git add -A
git commit -m 'moved milks.txt to gitignore'
```

## Local Versioning Across Commits

To compare two commits:

```bash
git diff HEAD^ HEAD
git diff HEAD~1 HEAD
git diff @~1 @
```

See the state of a file in a previous commit:

```bash
git show HEAD~2:fruits.txt > tmp.txt
cat tmp.txt
rm tmp.txt
```

Revert to a previous state using a new commit:

```bash
printf "banana\n" >> fruits.txt
git add -A
git commit -m 'added banana'
```

