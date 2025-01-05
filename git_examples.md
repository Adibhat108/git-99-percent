
## Script Setup

```bash
SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"
```

This gets the directory where the current script resides, as long as the last element in the path is not a symlink. For more details, refer to [this explanation](https://stackoverflow.com/a/246128/2512141).

```bash
cd $SCRIPT_DIR
mkdir -p $SCRIPT_DIR/sandbox
```

If the `sandbox` directory contains unexpected content, stop the script:

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

List git configurations along with their sources:

```bash
git config --list --show-origin
```

Example outputs:
- `C:/Program Files/Git/mingw64/etc/gitconfig` (system)
- `C:/Users/$USER/.gitconfig` (global)
- `.git/config` (local)

List system configurations:

```bash
git config --list --show-origin --system
echo
```

List global configurations:

```bash
git config --list --show-origin --global
echo
```

List local configurations:

```bash
git config --list --show-origin --local
echo
```

Global gitconfig contents:

```bash
cat ~/.gitconfig
echo
```

List aliases:

```bash
git config --get-regexp alias
echo
```

Set or unset aliases:

```bash
git config --global alias.la log --all --graph --oneline --decorate
git config --global --unset alias.la
```

Manually add aliases using an editor:

```bash
vim ~/.gitconfig
[alias]
    la = log --all --graph --oneline --decorate
```

Set global username and email:

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
printf "cherry\n" >> fruits.txt
git add fruits.txt
printf "apricot\n" >> fruits.txt
printf "sourdough\n" >> breads.txt
mkdir other
printf "potato\n" >> other/vegetables.txt
git status --untracked-files=all
```

Run a trial cleanup:

```bash
git reset --hard HEAD
git clean -x -d -n
```

Perform the actual cleanup:

```bash
git clean -x -d -f
```

Unstage changes but keep them in the working directory:

```bash
printf "cherry\n" >> fruits.txt
git add fruits.txt
printf "apricot\n" >> fruits.txt
git reset --mixed HEAD
```

Revert the working directory to the staged state:

```bash
git checkout .
```

Revert a single file:

```bash
printf "cherry\n" >> fruits.txt
git checkout fruits.txt
```

Compare working directory changes:

```bash
printf "lemon\n" >> fruits.txt
git add fruits.txt
printf "lime\n" >> fruits.txt
git diff
```

Compare staged changes with the last commit:

```bash
git diff --cached
```

Compare the working directory with the last commit:

```bash
git diff HEAD
```

Remove a file from all states:

```bash
printf "shiitake\n" >> mushrooms.txt
git add -A
git commit -m 'added shiitake'
rm mushrooms.txt
git add -A
git commit -m 'removed mushrooms.txt'
```

Ignore a file while keeping it in the working directory:

```bash
printf "oat\n" >> milks.txt
git add -A
git commit -m 'added oat to milks.txt'
git rm --cached milks.txt
printf "milks.txt\n" >> .gitignore
git add -A
git commit -m 'moved milks.txt to gitignore'
git status
cat milks.txt
```

## Local Versioning Across Commits

Check which commit `HEAD` is currently pointing to:

```bash
git rev-parse HEAD
git rev-parse master
git log --all --oneline --graph --decorate
```

Add changes and create a commit, then compare two commits:

```bash
printf "strawberry\n" >> fruits.txt
git add -A
git commit -m 'added strawberry'
git diff HEAD^ HEAD
git diff HEAD~1 HEAD
git diff @~1 @
git diff @~1 @ -- fruits.txt
```

The `-- fruits.txt` option limits the diff output to that specific file. Compare two commits on the `master` branch:

```bash
git diff master~1 master
```

View the content of a file from a previous commit:

```bash
git show HEAD~2:fruits.txt > tmp.txt
cat tmp.txt
rm tmp.txt
```

View a file’s content at a specific date or time (you must specify the branch):

```bash
git show master@{2019-10-02}:fruits.txt
git show master@{"2019-10-02 20:39:04"}:fruits.txt
```

### Restoring the State of a Previous Commit

**Important Notes**: Avoid using `checkout` or `reset` to modify history. These commands can rewrite history and should be used with caution:

- [Explanation on Reset, Checkout, and Revert](https://www.atlassian.com/git/tutorials/resetting-checking-out-and-reverting)
- [Caution Against Git Reset](https://www.atlassian.com/git/tutorials/undoing-changes/git-revert)

The correct way to revert to a previous commit without rewriting history is to use `git revert`. For example:

1. Add several commits:

    ```bash
    printf "banana\n" >> fruits.txt
    git add -A
    git commit -m 'added banana'

    printf "kiwi\n" >> fruits.txt
    git add -A
    git commit -m 'added kiwi'

    printf "pineapple\n" >> fruits.txt
    git add -A
    git commit -m 'added pineapple'

    printf "mango\n" >> fruits.txt
    git add -A
    git commit -m 'added mango'
    ```

2. Revert all changes after the `banana` commit:

    ```bash
    git revert --no-commit HEAD~3..HEAD
    ```

    The `--no-commit` flag prevents automatic generation of individual commits for each reverted commit. Review the changes:

    ```bash
    cat fruits.txt
    git diff HEAD
    ```

    Commit the revert:

    ```bash
    git commit -m 'reverting all changes after banana'
    ```

---

## Local Branches

List all local branches:

```bash
git branch -l
```

- Use `-l` for local branches (default).
- Use `-a` for all branches.
- Use `-r` for remote branches.

Create a new branch:

```bash
git branch sweet_fruits
```

Switch to an existing branch:

```bash
git checkout sweet_fruits
```

Create a new branch and switch to it:

```bash
git checkout -b sour_fruits
```

Rename a branch:

```bash
git branch -m sweet_fruits sugary_fruits
git branch -l
git branch -m sugary_fruits sweet_fruits
git branch -l
```

## Modifying Working and Staged Changes

You can modify both the working and staged areas and later decide where to commit the changes:

```bash
printf "grape\n" >> fruits.txt
git add -A
printf "cantaloupe\n" >> fruits.txt
git status
```

### Switching Between Branches

Switch between branches to view their commit histories:

```bash
git checkout master
git log --all --oneline --graph --decorate
git checkout sweet_fruits
git log --all --oneline --graph --decorate
git checkout sour_fruits
git log --all --oneline --graph --decorate
```

### Committing Changes to a Specific Branch

Commit changes to the `sweet_fruits` branch:

```bash
git checkout sweet_fruits
git add -A
git commit -m 'added grape and cantaloupe'
git log --all --oneline --graph --decorate
```

### Diverged Branches

Create diverged commits by making changes in a different branch (`sour_fruits`):

```bash
git checkout sour_fruits
printf "grapefruit\n" >> fruits.txt
git add -A
git commit -m 'added grapefruit'
git log --all --oneline --graph --decorate
```

---

## Merging and Resolving Merge Conflicts

### Fast-Forward Merge

When the `master` branch is an ancestor of `sweet_fruits`, a fast-forward merge occurs:

```bash
git checkout master
git merge sweet_fruits
```

This operation updates the `master` branch pointer without creating a new commit object, so no commit message is needed. Check the updated commit graph:

```bash
git log --all --oneline --graph --decorate
```

### Handling Merge Conflicts

If there are conflicting changes (e.g., `fruits.txt` is modified in both `sweet_fruits` and `sour_fruits`), resolve conflicts manually:

```bash
git merge sour_fruits
```

To resolve the conflict:

1. Remove conflict markers (`<<<`, `===`, `>>>`) from `fruits.txt`:

    ```bash
    sed '/[<<<|>>>|===]/d' ./fruits.txt >> tmp.txt
    rm fruits.txt
    mv tmp.txt fruits.txt
    ```

2. Stage the resolved file and commit the merge:

    ```bash
    git add -A
    git commit -m 'resolved conflicts and merged sour_fruits'
    ```

Check the updated commit graph:

```bash
git log --all --oneline --graph --decorate
```

---

## Deleting Merged Local Branches

After merging, delete local branches that are no longer needed:

```bash
git branch -d sweet_fruits
git branch -d sour_fruits
```

## Deleting a Local Branch That Has Not Been Merged

To delete a branch that hasn’t been merged, switch to another branch first:

```bash
git checkout -b savory_fruits
printf "???\n" >> fruits.txt
git add -A
git commit -m 'cannot think of any savory fruits'
git checkout master
```

Attempt to delete the branch:

```bash
git branch -d savory_fruits
```

If the branch hasn’t been merged, you’ll get a warning. Force delete the branch:

```bash
git branch -D savory_fruits
git log --all --oneline --graph --decorate
```

**Note**: Deleting a branch does not delete the associated commits. You can locate them using `git reflog` and, if necessary, reattach a branch to recover them.

---

## Detached HEAD

A detached HEAD occurs when `HEAD` points directly to a commit instead of a branch reference. This can happen in several ways:

- Pointing `HEAD` to a past commit with no branch reference.
- Pointing `HEAD` to a commit where the branch reference was deleted.
- Pointing `HEAD` to a commit using a tag.

To fix a detached HEAD, simply check out a commit that has a branch reference.

### Example: Detached HEAD and Recovering Commits

Even in detached HEAD state, commits are possible, but they won’t have a branch reference. If you need to preserve changes:

1. Commit your changes.
2. Create a branch pointing to the new commit.

```bash
printf "watermelon\n" >> fruits.txt
git add -A
git commit -m 'added watermelon'
git tag -a watermelonTag -m 'tagging watermelon'

printf "honeydew\n" >> fruits.txt
git add -A
git commit -m 'added honeydew'

git checkout watermelonTag
# Detached HEAD warning
git checkout master
# Detached HEAD warning gone

git checkout watermelonTag
# Detached HEAD warning
printf "raisin\ncurrant\nprune\n" >> dried_fruits.txt
git add -A
git commit -m 'added raisin, currant, prune'
```

Create a branch to recover the new commit:

```bash
git checkout -b dried_fruits
```

Merge the branch into `master`:

```bash
git checkout master
git merge dried_fruits -m 'merged dried fruits'
```

Since this isn’t a fast-forward merge, a new commit object will be created, requiring a commit message.

---

## Taking Single-File Changes from Another Branch

To selectively apply changes from another branch to a single file:

1. Create a branch and make changes:

    ```bash
    git checkout -b dressings
    printf "balsamic\nred\n" >> vinaigrettes.txt
    printf "french\nitalian\n" >> sauces.txt
    git add -A
    git commit -m 'added balsamic, red, french, italian'
    ```

2. Switch back to `master` and checkout a single file from the other branch:

    ```bash
    git checkout master
    git checkout dressings -- vinaigrettes.txt
    git add -A
    git commit -m 'added balsamic, red'
    ```

## Remote Repositories

Git allows us to treat a repository on a local disk the same way as a repository hosted on a platform like GitHub. For demonstration purposes, we will create bare repositories on disk and pretend they are remote repositories.

**Note**: A bare repository does not have a working or staged area.

---

### Setting Up a Bare Remote Repository

Create a bare repository:

```bash
REMUP=$SCRIPT_DIR/sandbox/remoteUpstream
LOCA=$SCRIPT_DIR/sandbox/localA

mkdir -p $REMUP
cd $REMUP
git init --bare
```

---

### Creating a Local Repository and Associating it with the Remote

There are two ways to associate a local repository with a remote:

1. **Initialize Locally and Add Remote Manually:**

    ```bash
    # mkdir -p $LOCA
    # cd $LOCA
    # git init
    # ls -a
    # git remote add remUp $REMUP
    # # Note: We are using "remUp" instead of the conventional name "origin".
    ```

2. **Clone the Remote Repository:**

    ```bash
    cd $SCRIPT_DIR/sandbox
    git clone -o remUp $REMUP $LOCA
    # If -o <name> is omitted, the remote will default to "origin".
    cd $LOCA
    ```

**Key Differences**:

- Cloning a remote repository copies its content, if any, to the local repository.
- The default fetch/push location for the local `master` branch is automatically set when cloning.

---

### Pushing Content to the Remote Repository

Push some content to the remote repository:

```bash
printf "pinto\n" >> legumes.txt
git add -A
git commit -m 'added pinto'
printf "lima\n" >> legumes.txt
git add -A
git commit -m 'added lima'

# Push to the remote
git push
```

---

### Managing Remotes

List remotes:

```bash
git remote -v
git remote show remUp
```

Check which remote branch the local `master` branch is tracking:

```bash
git branch -avv
```

Rename a remote:

```bash
git remote rename remUp remFoo
git remote -v
git remote rename remFoo remUp
git remote -v
```

Add a new remote:

```bash
git remote add remBar ./../bar
# (Directory does not exist)
```

Delete a remote:

```bash
git remote remove remBar
```

---

### Upstream and Downstream Workflow

In most workflows, developers pull code from an upstream repository, edit it, and push the changes back. However, in some cases, it may be useful to pull code from an upstream repository, edit it, and then push it to a separate downstream repository. Below, we demonstrate this downstream workflow.

Create a downstream repository:

```bash
REMDN=$SCRIPT_DIR/sandbox/remoteDownstream
mkdir -p $REMDN
cd $REMDN
git init --bare
```

Add the downstream remote and push:

```bash
cd $LOCA
git remote add remDn $REMDN
git push remDn master
```

---

### Configuring a Branch to Push to Downstream Remote by Default

```bash
git branch -avv
git branch --set-upstream-to=remDn/master
git branch -avv
```

#### Notes on Terminology and Behavior:

- **Branch Default Remote**:
  - A branch can associate with only one default remote, configured using `--set-upstream-to`.
  - The default remote is used for `git fetch` and `git push` when no explicit remote is mentioned.

- **Fetch and Push URLs**:
  - A remote may have distinct fetch and push URLs, allowing configurations where:
    - A branch **pulls** from one repository and **pushes** to another.
    - This setup, however, breaks the convention of treating a remote as a single entity (i.e., one repository).
  
- **GitHub's Convention**:
  - Platforms like GitHub typically separate remotes for different fetch or push locations.

#### Related Stack Overflow References:
  - [Configure Different Fetch and Push URLs](https://stackoverflow.com/a/47959803/2512141)
  - [Set Different Push URL for Remote](https://stackoverflow.com/q/9257533/2512141)

---

### Summary:
1. A repository can have multiple branches.
2. Each branch has one default remote set with `--set-upstream-to`.
3. A remote consists of a **fetch URL** and a **push URL**.

---

### Demonstrating Default Push Behavior

```bash
printf "chickpea\n" >> legumes.txt
git add -A
git commit -m 'added chickpea'
git push
```

- Once the default upstream remote (`remDn`) is configured, you can push changes to it without specifying the remote explicitly in the `git push` command.

## Downstream demo is done, point master back to upstream by default
```bash
git branch --set-upstream-to=remUp/master
git push
```

## Add a branch, push it to remote, and list local and remote branches
```bash
git checkout -b are_these_beans
printf "green bean\nsoybean\nsnap peas\n" >> legumes.txt
git add -A
git commit -m 'added green bean, soybean, snap pea'
git branch -avv
git push
```

The command above will result in the error:  
**"fatal: No configured push destination"**  
This is because this is a new branch, so it doesn't have push configured yet.

```bash
git push remUp
```

This will produce the error:  
**"fatal: The current branch are_these_beans has no upstream branch."**

```bash
git push remUp are_these_beans
git branch -avv
git branch --set-upstream-to=remUp/are_these_beans
git branch -avv
```

## Delete a local branch and the remote branch it tracks
### An aside: This is the highest-voted Stack Overflow post I've ever seen: [SO Link](https://stackoverflow.com/q/2003505/2512141)
```bash
git checkout master
git push remUp --delete are_these_beans
```

The remote branch is deleted first (this allows tab completion for the remote branch).

```bash
git branch -D are_these_beans
```

## Remote repos with multiple users

### Create a second local repo
```bash
LOCB=$SCRIPT_DIR/sandbox/localB
cd $SCRIPT_DIR/sandbox
git clone $REMUP $LOCB
```

Here, no name argument is passed, so Git will create the remote with the default name, **"origin"**.

```bash
cd $LOCB
ls -a
git log --all --graph --oneline --decorate
cat legumes.txt
```

### Merging another user's changes
```bash
cd $LOCA
printf "mung\n" >> legumes.txt
git add -A
git commit -m 'added mung'
git push
```

```bash
cd $LOCB
git status
git log --all --graph --oneline --decorate
```

At this stage, **local B** is not yet aware that anything has changed in the upstream repo.

```bash
git fetch
```

Fetching makes the local repo aware of all changes in the remote repo, across all branches.

```bash
git status
git log --all --graph --oneline --decorate
```

Now, the changes are visible, and they can be merged into the local master branch.

```bash
git merge
```

### Merging another user's changes with conflict
```bash
cd $LOCA
printf "ricebean\n" >> legumes.txt
git add -A
git commit -m 'added ricebean'
git push
```

```bash
cd $LOCB
printf "lentil\n" >> legumes.txt
git add -A
git commit -m 'added lentil'
```

```bash
git fetch
git status
git log --all --graph --oneline --decorate
```

At this point, **repo localB** has diverged from `origin/master`.

```bash
git merge
```

A conflict will occur here. Use the following commands to resolve it:
```bash
sed '/[<<<|>>>|===]/d' ./legumes.txt >> tmp.txt;
rm legumes.txt
mv tmp.txt legumes.txt
git add -A
git commit -m 'resolved merge conflict'
```

```bash
git log --all --graph --oneline --decorate
git status
```

The status will indicate:  
**"Your branch is ahead of 'origin/master' by 2 commits."**  
The upstream repo and **repoA** are not aware of the resolved merge.

```bash
git push
```

```bash
cd $LOCA
git pull
```

Pulling performs a fetch and merge operation.

```bash
cat legumes.txt
```

### Adding a branch added by another user
```bash
cd $LOCA
git checkout -b grains
printf "millet\n" >> grains.txt
git add -A
git commit -m 'added millet'
git push remUp grains
```

```bash
cd $LOCB
git fetch
git branch -a
git log --all --graph --oneline --decorate
```

Although `origin/grains` is present, there is no local `grains` branch.

```bash
git checkout -b bGrains origin/grains
```

This creates a local branch based on the newly copied remote branch. Observe that the local branch name is not required to match the remote branch. However, by convention, the local branch name should match the remote.

```bash
git branch -m bGrains grains
```

Renames **bGrains** to **grains**.

### Cleaning up a branch deleted by another user
```bash
cd $LOCA
git checkout master
git push remUp --delete grains
git branch -D grains
```

```bash
cd $LOCB
git fetch
git branch -a
```

Notice that `origin/grains` is still visible. A simple fetch did not delete it.

```bash
git fetch --prune
git branch -a
```

Now, `origin/grains` is gone.

```bash
git checkout master
git branch -D grains
```

Finally, delete the local branch that was tracking the remote branch.

## Commands that can change published commit history

### Use these with caution
If working on a shared repository, changing and pushing commit history will break the repository for your collaborators.

```bash
cd $SCRIPT_DIR/sandbox/repoA
```

## Reset --soft
`git reset --soft` takes the branch tag that `HEAD` points to and moves it backwards in the graph without changing the working or staged state. If you reset with `--soft` and then commit, it effectively "squashes" multiple commits into a single commit.

```bash
printf "almond\n" >> nuts.txt
git add -A
git commit -m 'added almond'

printf "cashew\n" >> nuts.txt
git add -A
git commit -m 'added cashew'

printf "pecan\n" >> nuts.txt
git add -A
git commit -m 'added pecan'

git log --all --graph --oneline --decorate
git reset --soft HEAD~3
git commit -m 'squashed commits'
git log --all --graph --oneline --decorate
```

## Interactive rebase, squash
```bash
printf "celery\n" >> vegetables.txt
git add -A
git commit -m 'added celery'

printf "carrot\n" >> vegetables.txt
git add -A
git commit -m 'added carrot'

printf "plum\n" >> fruits.txt
git add -A
git commit -m 'added plum'

printf "tomato\n" >> vegetables.txt
git add -A
git commit -m 'added tomato'

printf "onion\n" >> vegetables.txt
git add -A
git commit -m 'added onion'
```

We wish to rebase such that the fruit commit comes first and the vegetable commits are squashed together. Unfortunately, interactive rebase cannot be done cleanly by script. The steps are described instead:  
1. Run:  
   ```bash
   git rebase -i HEAD~5
   ```  
2. In the editor:
   - Move `plum` to the top.
   - Leave `celery` as `pick`.
   - Change `carrot`, `tomato`, and `onion` action to `squash`.  
3. Finally, in the commit message editor:
   - Remove all lines and write a single commit line for the squashed vegetable commits:  
     **"add carrot, tomato, onion"**

## Amend
Amending affects only the most recent commit. It is useful if you forgot to include a file in the last commit or if you have a minor change you want to add to the last commit.

```bash
printf "papaya\n" >> fruits.txt
printf "broccoli\n" >> vegetables.txt
git add fruits.txt
git commit -m 'added papaya'
git log --all --graph --oneline --decorate
git add vegetables.txt
git commit --amend -m 'added papaya, broccoli'
```

If you do not want to change the commit message, use the argument `--no-edit`.

```bash
git log --all --graph --oneline --decorate
```

## Misc

### "*" vs "." vs "-A"
- **`git add *`**: This is intercepted by the shell, and Git gets a list of files in the current directory. The list will not include files that begin with `.`.
- **`git add .`**: This is not intercepted by the shell, and Git interprets it to mean all files in the current directory and its subdirectories.  
  [Related StackOverflow discussion](https://stackoverflow.com/q/26042390/2512141)
- **`git add -A`**: Adds all changes from the repository root down, regardless of where the command is called.

### "origin/master" vs "origin master"
- **`origin/master`**: A branch reference that refers to a local copy of a remote branch. It is used in local operations.  
- **`origin master`**: Used in commands that actually change the remote repository.

### Git checkout vs Git reset
- **`git checkout`**: Moves only `HEAD` and copies the contents of the addressed commit to the staged and working directory.  
- **`git reset`**: Moves `HEAD` and the tag that `HEAD` points to, then copies content to the staged and working directory, depending on the flags.  
  - Note: `git reset` can alter pushed history (use caution).  
  [Related StackOverflow discussion](https://stackoverflow.com/q/3639342/2512141)

### .gitignore Syntax
- A `/` at the beginning or middle of an ignore pattern makes the pattern use the location of the `.gitignore` file as the root.  
- Patterns without a `/` at the beginning or middle apply everywhere.  
- Patterns with `/` at the beginning or middle can apply anywhere if preceded by `**`.  
- Multiple `.gitignore` files can be used within a repository.

Examples:
```plaintext
*.txt       # Ignore all files that end with ".txt".
foo         # Ignore all files and directories named "foo".
foo/        # Ignore all directories named "foo".
/foo        # Ignore file or directory named "foo" in the repository root.
/foo/bar    # Ignore file or directory "bar" in the root directory "foo".
**/foo/bar  # Ignore all files and directories "bar" in any "foo" directory.
```

### Not Covered
- **Stashing**
- **Patching**
- **Switch and Restore**: Experimental functions added in Git 2.23 (Aug 2019).  
  - [GitHub blog highlights](https://github.blog/2019-08-16-highlights-from-git-2-23/)  
  - [InfoQ article](https://www.infoq.com/news/2019/08/git-2-23-switch-restore/)  
  - [Git documentation](https://git-scm.com/docs/git-restore)  
  - Functionality of `git checkout` is split:  
    - `switch` changes branches.  
    - `restore` reverts files.
- **Cherry Pick**: Was planned to be covered but deferred for deeper understanding.  
  [Blog series on cherry picking](https://devblogs.microsoft.com/oldnewthing/?p=98245)

## Create a Tag and Push to Remote
```bash
cd $LOCA
git checkout master
printf "tangerine\n" >> fruits.txt
git add -A
git commit -m 'added tangerine'
git push

git tag -a tangerineTag -m 'tagging tangerine'
git push --tags
git log --all --graph --oneline --decorate
git ls-remote --tags remUp
```
> Note: `git tag -r` might be a more natural choice for listing remote tags.

## Remove a Tag and Delete from Remote
```bash
git tag -d tangerineTag
git push --delete remUp tangerineTag
git log --all --graph --oneline --decorate
git ls-remote --tags remUp
```

## Script Cleanup
```bash
echo 'goodbye'
```
