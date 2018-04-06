# Advanced Git
## Exercise Three - References

### Overview
In this exercise, we'll take a look at our references (`refs`) and create some lightweight and annotated tags. Then we'll make a dangling commit from a "detached HEAD" state and learn why this isn't a great idea.

### Prerequisite
You should have the [`advanced-git-exercises`](https://github.com/nnja/advanced-git-exercises) repository cloned locally. Checkout the `exercise3` branch to begin:

```
$> git checkout exercise3
Switched to branch 'exercise3'
```

### Exercise
1. Check the value of your `HEAD` variable (hint: look in `.git`) and confirm you're pointed at the `exercise3` branch.
2. Use `show-ref` to look at your other heads.
3. Create a lightweight tag and confirm that it's pointing at the right commit.
4. Create an annotated tag, and use `git-show` to see more information about it.
5. Get into a "detached HEAD" state by checking out a specific commit, then confirm that your HEAD is pointing at this commit rather than at a branch.
6. Make a new commit, then switch branches to confirm that you're leaving a commit behind.

## Solutions
### Step 1 - Where's your HEAD?
Assuming you checked out the `exercise3` branch in Step 0, your `HEAD` should be pointing to `exercise3`. You can corroborate this with `git branch`:

```
$> cat .git/HEAD
ref: refs/heads/exercise3

$> git branch
  exercise2
* exercise3
  master
```

### Step 2 - Where are your refs?
Use `git show-ref` to see which commits your HEADs are pointing at. You should see one for every branch you have, as well as every remote branch we've interacted with. Yours may look slightly different.

```
$> git show-ref --heads
43388fee19744e8893467331d7853a6475a227b8 refs/heads/exercise2
e348ebc1187cb3b4066b1e9432a614b464bf9d07 refs/heads/exercise3
43388fee19744e8893467331d7853a6475a227b8 refs/heads/master
43388fee19744e8893467331d7853a6475a227b8 refs/remotes/origin/exercise2
e348ebc1187cb3b4066b1e9432a614b464bf9d07 refs/remotes/origin/exercise3
43388fee19744e8893467331d7853a6475a227b8 refs/remotes/origin/master
```

You can see for yourself that our `master` branch is pointing to our "Initial commit"

```
$> git cat-file -p 43388fee19744e8893467331d7853a6475a227b8
tree 581caa0fe56cf01dc028cc0b089d364993e046b6
author Nina Zakharenko <nina@nnja.io> 1507168309 -0700
committer Nina Zakharenko <nina@nnja.io> 1507168309 -0700

Initial commit
```

Whereas our `exercise3` branch is pointing to our newer commit from Exercise 2:

```
$> git cat-file -p e348ebc1187cb3b4066b1e9432a614b464bf9d07
tree cbcdf5dda7853d595fe0b1942cb0d1d72eb910f3
parent 43388fee19744e8893467331d7853a6475a227b8
author Nina Zakharenko <nina@nnja.io> 1507168872 -0700
committer Nina Zakharenko <nina@nnja.io> 1507168872 -0700

Testing the emergency git-casting system
```


### Step 3 - Lightweight Tags:
Lightweight tags are simply named pointers to a commit. Make a new tag, then confirm that it points to the correct commit using `show-ref`:

```
$> git tag my-exercise3-tag

$> git show-ref --tags
e348ebc1187cb3b4066b1e9432a614b464bf9d07 refs/tags/my-exercise3-tag
```
Our current HEAD, `38708c...` has now been tagged as `my-exercise3-tag`. 

You can also do a reverse lookup using `git tag --points-at`:

```
$> git tag --points-at e348ebc
my-exercise3-tag
```

### Step 4 - Annotated Tags:
Annotated tags serve the same function as regular tags, but they also store additional metadata:

```
$> git tag -a "exercise3-annotated-tag" -m "This is my annotated tag for exercise 3"

$> git show exercise3-annotated-tag
tag exercise3-annotated-tag
Tagger: Nina Zakharenko <nina@nnja.io>
Date:   Wed Oct 4 19:12:19 2017 -0700

This is my annotated tag for exercise 3

commit e348ebc1187cb3b4066b1e9432a614b464bf9d07
Author: Nina Zakharenko <nina@nnja.io>
Date:   Wed Oct 4 19:01:12 2017 -0700

    Testing the emergency git-casting system

diff --git a/hello.txt b/hello.txt
index 980a0d5..b31a35b 100644
--- a/hello.txt
+++ b/hello.txt
@@ -1 +1,2 @@
 Hello World!
+This is a test of the emergency git-casting system.
```

Using `git show`, we can see all of the pertinent information about our `exercise3-annotated-tag`. We see the tag metadata at the top - who made the tag and when, as well as the tag message. Below that, we see the commit that was tagged, and then the diff between the tagged commit and its parent.

### Step 5 - Detached HEAD
Now we're going to venture into a "detached HEAD" state. Use `git checkout` to checkout the latest commit directly. You'll get a scary-looking warning about your HEAD being detached. You can confirm this by looking at `.git/HEAD` and seeing that it's now pointing to a commit hash, instead of `refs/heads/exercise3`

```
$> git log --oneline
e348ebc Testing the emergency git-casting system
43388fe Initial commit

$> git checkout e348ebc
Note: checking out 'e348ebc'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at e348ebc... Testing the emergency git-casting system

$> cat .git/HEAD
e348ebc1187cb3b4066b1e9432a614b464bf9d07
```

### Step 6 - Create a Dangling Commit
Even though our `HEAD` is now pointing at a specific commit - instead of a branch or tag - we can still make commits. Go ahead and make a new commit, then confirm that our `HEAD` is now pointing at this new commit:

```
$> echo "This is a test file" > dangle.txt

$> git add dangle.txt

$> git commit -m "This is a dangling commit"
[detached HEAD 9bdea9e] This is a dangling commit
 1 file changed, 1 insertion(+)
 create mode 100644 dangle.txt
 
$> git log --oneline
9bdea9e This is a dangling commit
38708c1 Testing the emergency git-casting system
aceb9e8 Initial commit

$> cat .git/HEAD
9bdea9e5b47e6e4b8453a43a657d5e292fd9b3b5
```

But wait. Because our new commit - `60c4f0e` in this case - was made in a detached `HEAD` state, it doesn't have any references pointing to it. It's not part of a branch, and has no tags. This is called a Dangling Commit. You'll see this warning if you try to switch branches:

```
$> git checkout exercise3
Warning: you are leaving 1 commit behind, not connected to
any of your branches:

  9bdea9e This is a dangling commit

If you want to keep it by creating a new branch, this may be a good time
to do so with:

 git branch <new-branch-name> 9bdea9e
```

Here, git is warning you that you're leaving this commit dangling. If you wish, you may create a new branch that points to this commit. Git does a periodic garbage collection and will eventually delete any commits that don't have a reference pointing to them.

#### End of Exercise Three