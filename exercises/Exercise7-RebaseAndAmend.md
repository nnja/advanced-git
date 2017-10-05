# Advanced Git
## Exercise Seven - Rebase and Amend

### Overview
In this exercise, we'll practice amending commits, then we'll try a normal rebase and an interactive rebase.

### Prerequisite 
You should have the [`advanced-git-exercises`](https://github.com/nnja/advanced-git-exercises)  repository cloned locally. Checkout the `exercise7` branch:

```
$> git checkout exercise7
Switched to a new branch 'exercise7'
```

### Exercise
1. Make a commit, then practice using the `--amend` option to make another change to the previous commit.
2. Make two non-conflicting changes to two different branches. Rebase one branch onto the other.
3. Make another change to your current branch. Use an interactive rebase (`git rebase -i`) to rebase the two branches. Try squashing your two commits and rewording the message during the rebase.


## Solutions

### Step 1 - Amend a Commit
Create two new files, `first.txt` and `second.txt`, then commit the first but not the second:

```
$> echo "This is the first file" > first.txt

$> echo "This is the second file" > second.txt

$> git add first.txt

$> git commit -m "Committing two new files"
[exercise7 8adf686] Committing two new files
 1 file changed, 1 insertion(+)
 create mode 100644 first.txt
```

Oh no, we forgot to include the second file in our commit. No worries, as long as we haven't pushed our commit to a remote repository (like origin), we can amend the last commit to include the other file.

```
$> git add second.txt

$> git commit --amend

# This will open your editor, and let you amend your commit message

[exercise7 b4952f3] Committing two new files
 Date: Wed Oct 4 23:06:48 2017 -0700
 2 files changed, 2 insertions(+)
 create mode 100644 first.txt
 create mode 100644 second.txt
 
# Confirm that we've committed both files now
```

There we go, we've fixed the commit to contain both `first.txt` and `second.txt`. Notice that the SHAs are different between the original commit (`8adf686`) and the amended commit (`b4952f3`). Commits can't be edited, so a new commit with the changed data was created and the old commit was replaced.

### Step 2 - Set up for a Rebase
Let's get things set up for a simple rebase demo. Checkout `master`, and let's pretend that we have a new feature branch, called `exercise7-2`.

```
$> git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.

$> git checkout -b exercise7-2
Switched to a new branch 'exercise7-2'

$> git log --oneline
43388fe Initial commit
```

Now let's create a new feature and commit it to our feature branch:

```
$> echo "New feature" > feature.txt

$> git add feature.txt

$> git commit -m "Adding a new feature"
[exercise7-2 edaa170] Adding a new feature
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt
```

At the same time, let's pretend that our `master` branch has continued to evolve while we were working on this feature:

```
$> git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.

# The double arrow >> means 'append' to the file instead of overwrite.
$> echo "Master has continued to change" >> hello.txt

$> git add hello.txt

$> git commit -m "Master has continued to change"
[master a2c699b] Master has continued to change
 1 file changed, 1 insertion(+)
```

Switching back to our feature branch. It's a good idea to periodically merge in the `master` branch, to keep things up to date and minimize the number of conflicts when the feature branch is eventually merged into `master`. Instead of creating unsightly merge commits though, let's use rebase to replay our feature commits on top of `master`'s commits.

```
$> git checkout exercise7-2
Switched to branch 'exercise7-2'

$> git rebase master
First, rewinding head to replay your work on top of it...
Applying: Adding a new feature

# Check that the changes to master are incorporated into our feature branch

$> git log --oneline
e83fafa Adding a new feature
a2c699b Master has continued to change
43388fe Initial commit
```

**Tip:** When working on a feature branch that's likely to conflict, I prefer to rebase from master often and fix conflicts as they come up. This way, I'm not stuck with a huge disastrous merge full of conflicts when I'm done with my feature and ready to merge it back to master.


### Step 3 - Interactive Rebase
Let's set up our feature branch for a very simple interactive rebase. Add another new feature and commit it:

```
$> echo "Another new feature" > another_feature.txt

$> git add another_feature.txt

$> git commit -m "Adding another new feature"
[exercise7-2 6449351] Adding another new feature
 1 file changed, 1 insertion(+)
 create mode 100644 another_feature.txt
```

Now we have two new commits on top of `master`. 

```
# passing -n 3 to log lets us see the last 3 commits

$> git log -n 3 --oneline
8470d04 (HEAD -> exercise7-2) Adding another new feature
64db08a Adding a new feature
ce8865e (master) Master has continued to change
```

When we're done with our feature, we want to clean these commits up by combining them using `squash`, and changing the commit message using `reword`.

Start the interactive rebase by passing in one commit *before* the one you want to start rebasing from. In this case, we want to rebase commit `64db08a` and the commit after it.

We have some options for which ref to use. All the options below will get the same result:

- `HEAD~2` - 2 commits before `HEAD`
- `64db08a^` - points to one commit back - parent `ce8865e`
- `ce8865e` - the actual commit before the one we want to rebase from 
- `master` - since `master` points to `ce8865e`

```
$> git rebase -i HEAD~2
```

Your editor will open and you should see the following:

```
pick edaa170 Adding a new feature
pick 6449351 Adding another new feature

# Rebase a2c699b..6449351 onto a2c699b (2 command(s))
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

The two feature commits we added to `exercise7-2` are listed in top-down order, and they default to pick, meaning the commits will be used as-is. Let's change the first one to `reword`, and the second to `squash`. This will re-open the editor twice - once to rename the first commit, and again to squash, or combine, the two commits.

In the first editor, change the commit message "Adding a new feature" to "Adding two new features"

In the second editor, delete the second message and leave only "Adding two new features"

You should see something similar:

```
detached HEAD dd693ff] Adding two new features
 Date: Wed Oct 4 23:54:35 2017 -0700
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt
[detached HEAD 03de89d] Adding two new features
 Date: Wed Oct 4 23:54:35 2017 -0700
 2 files changed, 2 insertions(+)
 create mode 100644 another_feature.txt
 create mode 100644 feature.txt
Successfully rebased and updated refs/heads/exercise7-2.
```

Now, take a look at your git log:

```
$> git log --oneline
03de89d Adding two new features
a2c699b Master has continued to change
43388fe Initial commit
```

Like magic, our two new features have been squashed into one commit, we've reworded the commit message to show this, and our changes are sitting neatly on top of the change from the `master` branch.

**Tip:** When working on a feature branch, commit early and often to minimize the likelihood of losing work. It'll also make back tracking easier. Once I'm ready to merge my changes, I do an interactive rebase to squash commits where needed, and take the opportunity to write better commit messages.

#### End of Exercise Seven