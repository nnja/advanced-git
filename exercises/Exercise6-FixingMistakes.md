# Advanced Git
## Exercise Six - Fixing Mistakes

### Overview
In this exercise, we'll practice reverting a file and cleaning our repo. Then we'll take a deeper look at `git reset` and `git revert` to go back in time.

### Prerequisite
You should have the [`advanced-git-exercises`](https://github.com/nnja/advanced-git-exercises)  repository cloned locally. Checkout the `exercise6` branch:

```
$> git checkout exercise6
Switched to a new branch 'exercise6'
```

### Exercise
1. Make bad changes to a file, then use `git checkout` to fix it. Use `git checkout` to reset your file back to an earlier point in time.
2. Use `git clean` to remove untracked files from your repo. Remember to use `--dry-run` first.
3. Stage a change and then use `git reset` to unstage it. Use `git reset --hard` to reset your branch back pointer, staging area, and working area to an earlier commit. Use "mixed mode" to reset your branch back to an earlier commit, then use `ORIG_HEAD` to reset your branch back to where you were.
4. Practice using `git revert` to safely revert a commit with a new commit.

## Solutions

### Step 1 - Undo changes in your working area with `git checkout -- <file>`
We all make mistakes. Let's make one now:

```
$> echo "Bad data" > hello.template

$> cat hello.template
Bad data
```

Oops, we messed up our `hello.template`. No worries, the mistake hasn't been committed yet, so we'll just check out the last committed version of `hello.template` from the repository, overwriting the working area:

```
$> git checkout -- hello.template

$> cat hello.template
[greeting] [noun]!
This is a test of the emergency git-casting system.
```

By default, if you don't pass arguments to `git checkout`, it assumed you meant to use `HEAD`.
**Note:** Remember that `git checkout --` is a destructive operation.

And, all is right again. Let's say we want to check out a file from a specific point in time. For example, we want to see what `hello.txt` was like before it was templatized:

```
# Let's find the commit where hello.txt was renamed to hello.template

$> git log --name-status --follow --oneline hello.template
4b2b90e Replacing greeting with tokens for i18n
R073    hello.txt       hello.template
fec9e7b Changing Hello to Hola
M       hello.txt

# Now let's checkout hello.txt from one commit before then

$> git checkout fec9e7b -- hello.txt

$> cat hello.txt
Hola World!
This is a test of the emergency git-casting system.
```

As expected, git has restored a snapshot of our `hello.txt` file from the commit `fec9e7b Changing Hello to Hola`. First, git copied the staging area from that commit. Next it copied the file from the updated staging area into our working area. Because of this, our `hello.txt` file appears added to the staging area. We don't really want to keep it, so let's reset it to unstage it and clear the staging area.

Then let's go ahead and delete our `hello.template` file:

```
$> git reset HEAD hello.txt

$> git rm hello.template
rm 'hello.template'

$> git status
On branch exercise6
Your branch is up-to-date with 'origin/exercise6'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	deleted:    hello.template

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	hello.txt

$> git commit -m "Deleting hello.template"
[exercise6 713f6a1] Deleting hello.template
 1 file changed, 2 deletions(-)
 delete mode 100644 hello.template
```

Later on, we decide that deleting `hello.template` was an accident. Let's track down where we deleted it, and bring it back:

```
$> git log --diff-filter=D --oneline -- hello.template
713f6a1 Deleting hello.template

# Ah, it was deleted at 713f6a1. Let's use the carrot (^) syntax to checkout hello.template from one commit before that

$> git checkout 713f6a1^ -- hello.template

$> git status
On branch exercise6
Your branch is ahead of 'origin/exercise6' by 1 commit.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   hello.template

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	hello.txt

$> cat hello.template
[greeting] [noun]!
This is a test of the emergency git-casting system.
```

Excellent, we now have our `hello.template` file back in our staging area and working area.

### Step 2 - Clean your Repo
We should still have this old `hello.txt` file sitting around, cluttering up our repo. We can use `git clean` to blow out anything that isn't tracked by git. We'll do a dry run first, just to be safe:

```
$> git clean --dry-run
Would remove hello.txt

$> git clean -f
Removing hello.txt
```

All clean!

### Step 3 - Git Reset
But wait, we never recommitted our `hello.template` file after we deleted it. It should still be staged for commit:

```
$> git status
On branch exercise6
Your branch is ahead of 'origin/exercise6' by 1 commit.
  (use "git push" to publish your local commits)
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   hello.template
```

Let's use `git reset` to unstage it. By default, if you don't specify an argument to `git reset`, it'll assume you meant `HEAD`.

```
$> git reset -- hello.template

$> git status
On branch exercise6
Your branch is ahead of 'origin/exercise6' by 1 commit.
  (use "git push" to publish your local commits)
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	hello.template
```

We can see that `hello.template` is now untracked, because it was deleted in the latest commit. Remember that using `git reset` on a file instead of a commit works with `mixed` mode. That means it updates the copy in the staging area, but it keeps the copy in the working area. 

Along with unstaging the file from `HEAD`, you can also reset individual files in the staging area to a specific point in time.

```
# Let's find the commit before we deleted hello.template

#> git log --oneline
713f6a1 Deleting hello.template
4b2b90e Replacing greeting with tokens for i18n

# Let's remove the untracked copy of hello.template
$> rm hello.template

$> cat hello.template
cat: hello.template: No such file or directory

# Reset hello.template back to 4b2b90e
# This will update the staging area with 'hello.template' from that commit.

$> git reset 4b2b90e -- hello.template

$> git status
On branch exercise6
Your branch is ahead of 'origin/exercise6' by 1 commit.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   hello.template

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	deleted:    hello.template
	

# But git reset won't update the working area.
$> cat hello.template
cat: hello.template: No such file or directory

# At this point, we could commit the file back into our repository if we wanted to.
```

But let's say we really screwed something up, and want to reset everything (the working area and the staging area) back to the latest commit we made. Warning: this will overwrite tracked files and could cause you to lose work.

```
$> git reset --hard HEAD
HEAD is now at 713f6a1 Deleting hello.template
```

What happens if you make a mistake using `git reset`? Git keeps a copy of your `HEAD` in a variable called `ORIG_HEAD`, to help you get back to where you want to be.

```
$> git log -2 --oneline
713f6a1 Deleting hello.template
4b2b90e Replacing greeting with tokens for i18n

# Reset our repo back to 4b2b90e

$> git reset 4b2b90e

$> git log -1 --oneline
4b2b90e Replacing greeting with tokens for i18n

$> git reset ORIG_HEAD

$> git log -1 --oneline
713f6a1 Deleting hello.template
```

And there we are, back at the commit where we deleted the file `hello.template`

### Step 4 - Git Revert
Let's say we want to undo deleting `hello.template`, but don't want to alter history. Reverts can don't look as nice in your history, but are a safer option when working with collaborators.

```
$> git log -1 --oneline
713f6a1 Deleting hello.template

# Let's revert the last commit. 

$> git revert 713f6a1

# This will open your editor for you to write a commit message for the reverting commit

[exercise6 9b2c3b3] Revert "Deleting hello.template"
 1 file changed, 2 insertions(+)
 create mode 100644 hello.template
```

There, we've successfully brought `hello.template` back from the dead, and we have a revert commit to show others exactly what happened. Plus, we didn't have to change history, so this is a good method to use if the changes you want to revert have already been pushed to your origin.

#### End of Exercise Six