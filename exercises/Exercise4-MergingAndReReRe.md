# Advanced Git
## Exercise Four - Merging and ReReRe

### Overview
In this exercise, we'll take a look at the fast-forward merge, learn how to make a non-fast-forward merge, then learn how to use git's Reuse Recorded Resolution (ReReRe) functionality to automate complex merges.

### Prerequisite
You should have the [`advanced-git-exercises`](https://github.com/nnja/advanced-git-exercises)  repository cloned locally. Checkout the `exercise4` branch:

```
$> git checkout exercise4
Switched to branch 'exercise4'
```

### Exercise
1. Merge the `exercise3` branch into `exercise4`, and look at the git log.
2. Use `git reset --hard HEAD^` to reset your `exercise4` branch back one commit, then use the `--no-ff` option to `git merge` to merge `exercise3` again. Look at the git log, how is it different from the last step?
3. Make two conflicting changes to `hello.txt` in two different branches.
4. Enable ReReRe, then merge one branch into the other.
5. Backup again with `git reset --hard`, then attempt the merge again. Notice how ReReRe automatically resolves the conflict for you.

## Solutions


### Step 1 - Fast-Forward Merge
You've probably done plenty of git merges. Here we're going to practice making a merge commit without "fast-forwarding."

On the `exercise4` branch, you should be on the commit labeled "Initial commit." Let's say `exercise3` is our feature branch - we have a new commit in `exercise3` that we want to merge into `exercise4` - "Testing the emergency git-casting system." Let's try a regular merge:

```
$> git checkout exercise4

$> git merge exercise3
Updating 43388fe..e348ebc
Fast-forward
 hello.txt | 1 +
 1 file changed, 1 insertion(+)
```

Easy-peasy. But wait, look at the history:

```
$> git log --oneline
e348ebc Testing the emergency git-casting system
43388fe Initial commit
```

We see our new commit, `e348ebc`, with it's parent, `43388fe`, our initial commit. But there's nothing to let us know that `e348ebc` was merged into `exercise4` from the `exercise3` branch. This is because this was a "fast-forward" merge, meaning that there were no changes made to `exercise4` - the pointer for `exercise4` was simply updated via "fast-forward" to point to our new commit, `e348ebc`.

Fast forward commits happen when the base branch (the one you created your branch on) hasn't changed after you branched from it. The history is linear.

You'll usuaully see a fast-forward happening on the master branch after merging in a feature, like this:

![before ff](https://user-images.githubusercontent.com/2030983/31261121-f747408a-aa17-11e7-8377-e009019707a0.png)

![after-ff](https://user-images.githubusercontent.com/2030983/31261122-f75375d0-aa17-11e7-9218-4c7030acc137.png)

### Step 2 - Non-Fast-Forward Merge

In a more complicated codebase, merge commits can be very important for determining when certain features or changes were merged in. Let's go back and tell git to make a merge commit even when one isn't strictly necessary. Use `git reset --hard HEAD^` to back things up to the last commit.

```
$> git reset --hard HEAD^
HEAD is now at 43388fe Initial commit
```

There, `exercise4` is back to pointing at "Initial commit." Now use `git merge --no-ff`. Note: the editor will open and ask you to write a message for this merge.

```
$> git merge --no-ff exercise3
Merge made by the 'recursive' strategy.
 hello.txt | 1 +
 1 file changed, 1 insertion(+)
```

There, now we have a merge commit in place to tell us that `e348ebc` was merged into `exercise4` from the `exercise3` branch, along with who, when, and optionally why it was merged. Use the `--graph` argument to `git log` to see this more clearly:

```
$> git log --graph
*   commit 7ea8b01a763b19037ab79b17e6a54a41b60d88e2
|\  Merge: 43388fe e348ebc
| | Author: Nina Zakharenko <nina@nnja.io>
| | Date:   Wed Oct 4 19:21:05 2017 -0700
| |
| |     Merge branch 'exercise3' into exercise4
| |
| * commit e348ebc1187cb3b4066b1e9432a614b464bf9d07
|/  Author: Nina Zakharenko <nina@nnja.io>
|   Date:   Wed Oct 4 19:01:12 2017 -0700
|
|       Testing the emergency git-casting system
|
* commit 43388fee19744e8893467331d7853a6475a227b8
  Author: Nina Zakharenko <nina@nnja.io>
  Date:   Wed Oct 4 18:51:49 2017 -0700

      Initial commit
```

The picture is much clearer now: The `exercise4` branch diverged with the addition of "Testing the emergency git-casting system", then the `exercise3` branch, which included this new commit, was merged back into `exercise4` with the message "Merge branch `exercise3`"

![no-ff](https://user-images.githubusercontent.com/2030983/31261123-f756a890-aa17-11e7-8ab9-501af4e44373.png)

### Step 3 - Setting up for a Conflict
Here's where things get a little tricky. Let's set up a merge conflict, and use git's Reuse Recorded Resolution function to resolve it automatically for us. This is super useful in situations where you get a large number of repeated merge conflicts, such as when you're refactoring a codebase while others are still making changes.

Let's set things up so that we're changing the same line from two different branches, creating a merge conflict. You should still be on the `exercise4` branch. Let's create a new branch, called `mundo`.

```
$> git branch
  exercise2
  exercise3
* exercise4
  master

$> git checkout -b mundo
Switched to a new branch 'mundo'
```

Now, edit `hello.txt` so that instead of "Hello World!" it says "Hello Mundo!" and create a new commit.

```
$> git branch
  exercise2
  exercise3
  exercise4
  master
* mundo

$> # Edit your hello.txt to say "Hello Mundo!"

$> git status
On branch mundo
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   hello.txt

no changes added to commit (use "git add" and/or "git commit -a")

$> git add hello.txt

$> git commit -m "Changing World to Mundo"
[mundo afa34a6] Changing World to Mundo
 1 file changed, 1 insertion(+), 1 deletion(-)
```

Now, go back to our `exercise4` branch, where `hello.txt` should still read "Hello World!" Create a commit that changes this to "Hola World!"

```
$> git checkout exercise4
Switched to branch 'exercise4'

$> # Edit hello.txt to say "Hola World!"

$> git add hello.txt

$> git commit -m "Changing Hello to Hola"
[exercise4 fec9e7b] Changing Hello to Hola
 1 file changed, 1 insertion(+), 1 deletion(-)
```

### Step 4 - Enable ReReRe and Merge
Before trying to merge, let's enable git's ReReRe functionality. Then, let's try to merge the `mundo` branch into `exercise4` as normal.

```
$> git config rerere.enabled true

$> git merge mundo
Auto-merging hello.txt
CONFLICT (content): Merge conflict in hello.txt
Recorded preimage for 'hello.txt'
Automatic merge failed; fix conflicts and then commit the result.
```

You can see, as expected, we have a merge conflict that must be resolved manually. However, you should notice a new line: `Recorded preimage for 'hello.txt'`

Run `git rerere diff` to see what's going on:

```
$> git rerere diff
--- a/hello.txt
+++ b/hello.txt
@@ -1,6 +1,6 @@
-<<<<<<<
-Hello Mundo!
-=======
+<<<<<<< HEAD
 Hola World!
->>>>>>>
+=======
+Hello Mundo!
+>>>>>>> mundo
 This is a test of the emergency git-casting system.
```

This shows us the current state of the resolution - what we started with to resolve and what we've resolved it to. 

Resolve the conflict in `hello.txt` by changing it to say "Hola Mundo!" and run `rerere diff` again:

```
$> git rerere diff
--- a/hello.txt
+++ b/hello.txt
@@ -1,6 +1,2 @@
-<<<<<<<
-Hello Mundo!
-=======
-Hola World!
->>>>>>>
+Hola Mundo!
 This is a test of the emergency git-casting system.
```

Now things should be clearer - when git sees "Hello Mundo!" on one side of a merge, and "Hola World!" on the other, it will resolve it to "Hola Mundo!" Mark it as resolved and commit it:

```
$> git add hello.txt

$> git commit -m "Merging in mundo branch"
[exercise4 ff91b70] Merging in mundo branch
```

### Step 5 - Back up and Merge Again
Now let's back up to just before the last commit, and try the merge again - this time with the help of ReReRe:

```
$> git reset --hard HEAD^
HEAD is now at fec9e7b Changing Hello to Hola

$> git merge mundo
Auto-merging hello.txt
CONFLICT (content): Merge conflict in hello.txt
Resolved 'hello.txt' using previous resolution.
Automatic merge failed; fix conflicts and then commit the result.

$> cat hello.txt
Hola Mundo!
This is a test of the emergency git-casting system.
```

This time, our merge still failed, but the conflict was resolved automatically, per the line that says `Resolved 'hello.txt' using previous resolution.` There is no need to resolve the conflict manually, as we can see that our `hello.txt` now correctly says "Hola Mundo!" All we need to do is stage `hello.txt` and commit it.

#### End of Exercise Four