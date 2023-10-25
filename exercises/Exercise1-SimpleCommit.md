# Advanced Git
## Exercise One - Under The Hood of a Simple Commit

<!-- This is a comment that I have added to test my first push request to a foreign project -->

### Overview
In this exercise, we'll create a simple commit, and then peek under the hood at the objects stored in our `.git` folder to gain some insight into how things work.

### Prerequisite
If you have a Mac with `brew` set up, install `tree`. This makes it easy to visualize the contents of your `.git` folder.

### Exercise
1. Create a new folder and initialize it as a git repo
2. Create a file, stage it, and commit it to your new repo
3. Look at your `.git` folder, using `tree` if you have it
4. Inspect the objects in your `.git/objects` folder using `git cat-file`. See if you can find the tree, blob, and commit objects for your recent commit.
5. Look at your `.git/HEAD` and `.git/refs/heads/master` files and see if you can figure out where these references are pointing to.

## Solutions

### Step 1 - Initialize the Repo
Create a new sample project folder. Run `git status` to see that it is not yet a git repository. Use `git init` to initialize it as a repository.

```
$> mkdir -p ~/projects/sample

$> cd ~/projects/sample

$> git status
fatal: Not a git repository (or any of the parent directories): .git

$> git init
Initialized empty Git repository in /Users/nnja/projects/sample/.git/
```

### Step 2 - First Commit
Create a new document, stage it for a commit, then commit it to your repository.

```
$> echo 'Hello World!' > hello.txt

$> git add hello.txt

$> git commit -m "Initial commit"
[master (root-commit) aceb9e8] Initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 hello.txt
```

### Step 3 - View the .git Folder
Using `tree`, look in your `.git/objects` folder, you should now see three objects, represented by long SHA1 hashes. These represent the tree, blob, and commit that we created in the last step.

```
$> tree .git

.git
├── COMMIT_EDITMSG
├── HEAD
├── config
├── description
├── index
├── info
│   └── exclude
├── logs
│   ├── HEAD
│   └── refs
│       └── heads
│           └── master
├── objects
│   ├── 43
│   │   └── 388fee19744e8893467331d7853a6475a227b8
│   ├── 58
│   │   └── 1caa0fe56cf01dc028cc0b089d364993e046b6
│   ├── 98
│   │   └── 0a0d5f19a64b4b30a87d4206aade58726b60e3
│   ├── info
│   └── pack
└── refs
    ├── heads
    │   └── master
    └── tags
```

### Step 4 - Inspect the Objects:
Note: The SHA1 hash for your commit will be different than the one displayed here. The SHA1 hash for your `blob` and `tree` will be the same as mine, as long as the content is the same.

One of the objects should be a tree object. The tree contains the filename `hello.txt` and a pointer to the blob.

```
$> git cat-file -t 581caa
tree

$> git cat-file -p 581caa
100644 blob 980a0d5f19a64b4b30a87d4206aade58726b60e3	hello.txt
```

The blob object, pointed to by the tree, contains the contents of the file `hello.txt`

```
$> git cat-file -t 980a0d5
blob

$> git cat-file -p 980a0d5
Hello World!
```

The commit object contains a pointer to the tree, along with metadata for the commit, such as the author and commit message.

```
$> git cat-file -t 43388f
commit

$> git cat-file -p 43388f
tree 581caa0fe56cf01dc028cc0b089d364993e046b6
author Nina Zakharenko <nina@nnja.io> 1507168309 -0700
committer Nina Zakharenko <nina@nnja.io> 1507168309 -0700

Initial commit
```

Because this is our very first commit, it doesn't have a parent. The next commit we make will point to our initial commit as the parent. 

### Step 5 - Look at refs

Let's look under the hood at our `HEAD` variable. `HEAD` is just git's pointer to "where you are now," usually referring to the current branch. More on this later. We can see that right now, it points to our current branch - `master`

Now, if we look at our `master` reference, we can see that it points to the latest commit.

```
$> cat .git/HEAD
ref: refs/heads/master

$> cat .git/refs/heads/master
43388fee19744e8893467331d7853a6475a227b8
```
`43388f...` is the hash of the commit we saw in the last step. You can confirm this by running `git log`

```
$> git log --oneline
43388f Initial commit
```

Git stores references in the `.git/refs/heads/` directory, and the `HEAD` pointer in `.git/HEAD`

We can verify this by creating a new branch.

```
$> git branch new_branch
```

The `git branch` command will create a new branch without switching to it.

Now, if we look in `.git/refs`, we'll see two branches. The `master` branch, which is created by default, and `new_branch`.

```
$> tree .git/refs
.git/refs
├── heads
│   ├── master
│   └── new_branch
└── tags

```

#### End of Exercise One