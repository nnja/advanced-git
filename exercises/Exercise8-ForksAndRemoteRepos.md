# Advanced Git
## Exercise Eight - Forks And Remote Repos

### Overview
In this exercise, we'll try creating our own Github fork of the `advanced-git-exercises` repo that we've been working with. Then we'll rename the remotes and try using `git pull --rebase` to do a cleaner pull without adding merge commits.

### Prerequisite
You should have the [`advanced-git-exercises`](https://github.com/nnja/advanced-git-exercises)  repository cloned locally. Checkout the `master` branch:

```
$> git checkout master
Switched to branch 'master'
```

### Exercise:
1. Create your own Github fork of `https://github.com/nnja/advanced-git-exercises`.
2. Look at your git remotes. Rename your `origin` remote (nnja's copy) to `upstream`. Add your personal fork as the `origin` remote.
3. Nina will make a change to her copy of the repo. Make a different change to your local repo, then use `git pull --rebase` to merge them.

## Solutions

### Step 1 - Create your own Fork
In Github, go to `https://github.com/nnja/advanced-git-exercises` and create your own fork of this repo by clicking the Fork button in the top right corner. This should create a copy of the repo that belongs to you.

### Step 2 - Set up Remotes
Now, we want to set up our local repository so that your fork is the origin, but we'll keep Nina's version of the repository as a remote called `upstream`.

First, if you checked out `advanced-git-exercises` from Nina's Github, you should see something like this:

```
$> git remote -v
origin	git@github.com:nnja/advanced-git-exercises.git (fetch)
origin	git@github.com:nnja/advanced-git-exercises.git (push)
```

Let's rename the `origin` remote to `upstream`:

```
$> git remote rename origin upstream

$> git remote -v
upstream	git@github.com:nnja/advanced-git-exercises.git (fetch)
upstream	git@github.com:nnja/advanced-git-exercises.git (push)
```

Now, let's add a new remote - our fork of the repo. We'll call it origin. Substitute the url with the correct url for your fork. You can find the full url on the Github page for your fork, in the dropdown when you click "Clone or download"

```
$> git remote add origin git@github.com:mhenstell/advanced-git-exercises.git

$> git remote -v
origin	git@github.com:mhenstell/advanced-git-exercises.git (fetch)
origin	git@github.com:mhenstell/advanced-git-exercises.git (push)
upstream	git@github.com:nnja/advanced-git-exercises.git (fetch)
upstream	git@github.com:nnja/advanced-git-exercises.git (push)
```

There, now we should have our two remotes set up. This will allow us to push and pull changes to our personal fork, `origin`, while also allowing us to pull in changes from Nina's `upstream` version of the repo.

### Step 3 - Pull with Rebase
We should already be familiar with merging commits from a remote repo, but let's take a look at a handy option for pulling - `git pull --rebase`. As you might expect, this pulls down the changes from a remote, but instead of merging them, it rebases - any changes you've made are replayed on top of the remote's changes. This is especially useful if you're working on a continually changing codebase and don't want lots of unsightly merge commits in your history.

First let's checkout `master`. `master` is still set up to track `upstream/master`, so we'll want to change it to track our personal copy - `origin/master`

```
$> git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'upstream/master'.

$>git pull origin
From github.com:mhenstell/advanced-git-exercises
 * [new branch]      exercise10 -> origin/exercise10
 * [new branch]      exercise2  -> origin/exercise2
 * [new branch]      exercise3  -> origin/exercise3
 * [new branch]      exercise4  -> origin/exercise4
 * [new branch]      exercise5  -> origin/exercise5
 * [new branch]      exercise6  -> origin/exercise6
 * [new branch]      exercise7  -> origin/exercise7
 * [new branch]      exercise9  -> origin/exercise9
 * [new branch]      master     -> origin/master

$> git branch --set-upstream-to origin/master
Branch master set up to track local branch origin/master.
```

For this example, we'll need to have a new commit on both the `upstream` repo and our local repo. First Nina will make a change to her version of the repo:

**Don't follow the next set of instructions. These are the actions that Nina will take.**

```
# Nina does these steps:

$> echo "Change to upstream" > upstream_change.txt

$> git add upstream_change.txt

$> git commit -m "Change to the upstream repo"
[master d9d0989] Change to the upstream repo
 1 file changed, 1 insertion(+)
 create mode 100644 upstream_change.txt
 
# Push master to the upstream remote specifically, not origin

$> git push upstream master
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 313 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To git@github.com:nnja/advanced-git-exercises.git
   43388fe..d9d0989  master -> master

# Reset local repo to Initial commit

$> git reset --hard HEAD^
HEAD is now at 43388fe Initial commit
```

Now we'll make a new `feature` branch locally and add a change to it.:

```
$> git checkout -b `feature`

$> echo "Change to local repo" > local_change.txt

$> git add local_change.txt

$> git commit -m "Change to local repo"
[feature c5019be] Change to local repo
 1 file changed, 1 insertion(+)
 create mode 100644 local_change.txt
```

Now that we have a change in our local repo feature branch, and a change in our `upstream` repo, let's pull in the changes from the `upstream` repo without an unsightly merge commit in our feature branch:

```
$> git pull --rebase upstream master
From github.com:nnja/advanced-git-exercises
 * branch            feature     -> FETCH_HEAD
First, rewinding head to replay your work on top of it...
Applying: Change to local repo

$> git log --oneline
0aa7023 Change to local repo
d9d0989 Change to the upstream repo
43388fe Initial commit
```

Great! Use `git pull --rebase` frequently to keep your local fork up-to-date with a remote repo without merging. If you open a Pull Request, your history will be clean.

#### End of Exercise Eight