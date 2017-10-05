# Advanced Git
## Exercise Five - History and Diffs

### Overview
In this exercise, we'll practice making a good commit, take a look at some of the interesting command line arguments for `git log`, use `git show` to get more information about a commit, then take a quick look at `git branch`.

### Prerequisite
You should have the [`advanced-git-exercises`](https://github.com/nnja/advanced-git-exercises)  repository cloned locally. Checkout the `exercise5` branch:

```
$> git checkout exercise5
Switched to a new branch 'exercise5'
```

### Exercise
1. Practice creating a well-crafted commit - look at the format given on the slides for help.
2. Use `git log` to find commits created since yesterday. Rename a file and use the `--name-status` and `--follow` options to `git log` to track down when the file was renamed, and what it used to be called. Use `--grep` to search within commit messages, and `--diff-filter` to find renamed and modified files from `git log`.
3. Use `git show` to get more information about a specific git hash.
4. Try the `--merged` and `--no-merged` options to `git branch` to see which branches have been merged into `master` (or not).

## Solutions

### Step 1 - Make a Good Commit
We've all made commits with short, silly, or otherwise unhelpful messages. Let's practice making a solid commit message for use in this example.

```
# Change your `hello.txt` to say [greeting] [noun]!

$> cat hello.txt
[greeting] [noun]!
This is a test of the emergency git-casting system.

# Rename hello.txt to hello.template

$> git mv hello.txt hello.template

$> git add hello.template

$> git commit

# This will open your text editor
# Type the following...
Replacing greeting with tokens for i18n

Currently, hello.txt contains both Spanish and English.
Let's replace Hola with a [greeting] token, and Mundo
with a [noun] token. That way, we can localize hello.txt for
any language!

# Save and exit your editor

[exercise5 4b2b90e] Replacing greeting with tokens for i18n
 1 file changed, 1 insertion(+), 1 deletion(-)
```

### Step 2 - Git Log
Let's take a look at our new commit using `git log`. First we'll see how to see all commits in the log since yesterday:

```
$> git log --since="yesterday"
commit 4b2b90ec4f526139ca9c81e22174ebf5b9c56b52
Author: Nina Zakharenko <nina@nnja.io>
Date:   Wed Oct 4 20:46:45 2017 -0700

    Replacing greeting with tokens for i18n

    Currently, hello.txt contains both Spanish and English.
    Let's replace Hola with a [greeting] token, and Mundo
    with a [noun] token. That way, we can localize hello.txt for
    any language!
```

Things can get tricky when trying to track down changes to files when they've been renamed. Here we see how to use `git log` to find the commit where `hello.txt` was renamed to `hello.template`. The `--follow` command will continue following the file backward in history, showing its filename for every commit:

```
$> git log --name-status --follow --oneline hello.template
4b2b90e Replacing greeting with tokens for i18n
R073    hello.txt       hello.template
fec9e7b Changing Hello to Hola
M       hello.txt
afa34a6 Changing World to Mundo
M       hello.txt
e348ebc Testing the emergency git-casting system
M       hello.txt
43388fe Initial commit
A       hello.txt
```

Ever spent too much time trying to track down a commit in Github? Use `git log --grep` to quickly find a commit. We can chain this with other flags. Let's find all the internationalization commits that the author `Nina` has added in the last two weeks:

```
$> git log --grep=i18n --author=nina --since=2.weeks
commit 4b2b90ec4f526139ca9c81e22174ebf5b9c56b52
Author: Nina Zakharenko <nina@nnja.io>
Date:   Wed Oct 4 20:46:45 2017 -0700

    Replacing greeting with tokens for i18n

    Currently, hello.txt contains both Spanish and English.
    Let's replace Hola with a [greeting] token, and Mundo
    with a [noun] token. That way, we can localize hello.txt for
    any language!
```

We can use `--diff-filter` to find commits where files have been renamed:

```
$> git log --diff-filter=R --find-renames
4b2b90e Replacing greeting with tokens for i18n
```

Or to find commits where files have been modified:

```
$> git log --diff-filter=M --oneline
fec9e7b Changing Hello to Hola
afa34a6 Changing World to Mundo
e348ebc Testing the emergency git-casting system
```

### Step 3 - Git Show
Now that we've mastered `git log`, how do we actually see what happened in a commit? Let's use `git show` to find out.

```
$> git log --grep=i18n --oneline
4b2b90e Replacing greeting with tokens for i18n

# Let's see the full commit and diff for 4b2b90e

$> git show 4b2b90e
commit 4b2b90ec4f526139ca9c81e22174ebf5b9c56b52
Author: Nina Zakharenko <nina@nnja.io>
Date:   Wed Oct 4 20:46:45 2017 -0700

    Replacing greeting with tokens for i18n

    Currently, hello.txt contains both Spanish and English.
    Let's replace Hola with a [greeting] token, and Mundo
    with a [noun] token. That way, we can localize hello.txt for
    any language!

diff --git a/hello.template b/hello.template
new file mode 100644
index 0000000..a6c57ac
--- /dev/null
+++ b/hello.template
@@ -0,0 +1,2 @@
+[greeting] [noun]!
+This is a test of the emergency git-casting system.
diff --git a/hello.txt b/hello.txt
deleted file mode 100644
index 7018e35..0000000
--- a/hello.txt
+++ /dev/null
@@ -1,2 +0,0 @@
-Hola Mundo!
-This is a test of the emergency git-casting system.

# Show the files changed in 4b2b90e

$> git show 4b2b90e --stat --oneline
4b2b90e Replacing greeting with tokens for i18n
 hello.template | 2 ++
 hello.txt      | 2 --
 2 files changed, 2 insertions(+), 2 deletions(-)
```

### Step 4 - Git Branch
Let's say you're working on a complicated codebase with a `master` branch and lots of feature branches. Some of your coworkers forget to clean up their branches when they're done (we're all guilty). Which branches have been merged into `master` and can be cleaned up? Which branches haven't been merged yet? If you've been following along, yours may look different.

```
$> git branch --merged master
  exercise2
  exercise4
  master

$> git branch --no-merged master
  exercise3
* exercise5
  mundo
```

### End of Exercise Five