# Advanced Git
## Exercise Nine - Advanced Tools

### Overview
In this exercise, we'll take a look at some of the advanced features of `git grep`, we'll learn how to "cherry-pick" commits, then we'll take a look at `git blame` and `git bisect`.

## Prerequisite
You should have the [`advanced-git-exercises`](https://github.com/nnja/advanced-git-exercises)  repository cloned locally. Checkout the `exercise9` branch to begin:

```
$> git checkout exercise9
Switched to branch 'exercise9'
```

### Exercise
1. Use `git grep` to search for a string in your git repo. Try using arguments for `git grep` to print line numbers and group results by file. Use the `--cached` option to see the difference between grepping your working area and your staging area.
2. Try to cherry-pick a commit from one branch into the `exercise9` branch.
3. Use `git blame` to see who touched a file. Delete a file and commit your change. Use `git blame` again to blame a file from an earlier point in time, before it was deleted.
4. Start a `git bisect` session and try to find which commit introduced the word "emergency" into `hello.txt` 

## Solutions

### Step 1 - Grepping with Git
We should have three new code files in our `exercise9` branch. Let's use git's built in, super-fast grep functionality to search our code, looking for the word "Python":

```
$> git grep -e "Python"
python_code.py:# This is a Python file
python_code.py:    print("Welcome to Python!")
```

Neat, but the output is somewhat confusing. Let's use some options to clean it up:

```
$> git grep --line-number --heading --break -e "Python"
python_code.py
1:# This is a Python file
4:    print("Welcome to Python!")
```

Now we have matches broken up with line numbers and grouped by file, so it's a little easier to read. 

Let's make a change, and use `git grep` again. Because `python_code.py` is tracked, `git grep` will pick up the change. Try `git grep --cached` - this will only search the version in the staging area, so the new change will be ignored. Then stage the file and try again.

```
$> echo "More Python code" >> python_code.py

$> git grep --line-number -e "Python"
python_code.py:1:# This is a Python file
python_code.py:4:    print("Welcome to Python!")
python_code.py:5:More Python code

$> git grep --line-number --cached -e "Python"
python_code.py:1:# This is a Python file
python_code.py:4:    print("Welcome to Python!")

# No line number 5!

$> git add python_code.py

$> git grep --line-number --cached -e "Python"
python_code.py:1:# This is a Python file
python_code.py:4:    print("Welcome to Python!")
python_code.py:5:More Python code

# There it is!
```

### Step 2 - Cherry Picking
Let's reset our `python_code.py` to avoid errors when changing branches:

```
$> git checkout python_code.py
```

Now, say we have changes in a specific commit in another branch that we'd like to bring into our current branch, without merging everything from the other branch. We should have two commits in our `exercise9` branch:

```
$> git log --oneline
88f6e28 Adding bash, python, and java code examples
43388fe Initial commit
```

Run `git log` on the exercise3 branch. We're looking for the commit hash of the commit "Testing the emergency git-casting system":

```
$> git log exercise3 --oneline
e348ebc Testing the emergency git-casting system
43388fe Initial commit

```

The commit we're looking for is `e348ebc` in this copy of the repo (yours may be different). Make sure we're on the `exercise9` branch, and cherry-pick it.

```
$> git cherry-pick e348ebc
[exercise9 331024e] Testing the emergency git-casting system
 Date: Wed Oct 4 19:01:12 2017 -0700
 1 file changed, 1 insertion(+)

$> git log --oneline
331024e Testing the emergency git-casting system
88f6e28 Adding bash, python, and java code examples
43388fe Initial commit
```

Great - as we can see from `git log`, the commit "Testing the emergency git-casting system" was merged into our branch `exercise9`. You'll noticed the cherry-picked commit is on top - unlike if we had rebased our other changes on top of it.

### Step 3 - Git Blame
Say you come across some questionable code. How could we tell who the last person to touch it was? `git blame` of course:

```
$> git blame hello.txt
^43388fe (Nina Zakharenko 2017-10-04 18:51:49 -0700 1) Hello World!
331024e3 (Nina Zakharenko 2017-10-04 19:01:12 -0700 2) This is a test of the emergency git-casting system.
```

`git blame` also has some useful arguments, such as ignoring whitespace (-w), detecting moved or copied lines (-M), and detecting moved or copied lines from other files in the commit (-C)

What if a file was deleted, can we still blame it? Of course, you can `git blame` any file from any point in time. Let's delete `java_code.java` and then `blame` it:

```
$> git rm java_code.java

$> git commit -m "Who uses Java anyway?"
[exercise9 b8e1a56] Who uses Java anyway?
 1 file changed, 7 deletions(-)
 delete mode 100644 java_code.java
 
# Let's find the commit where java_code.java was deleted

$> git log --diff-filter=D -- java_code.java
commit b8e1a5692b0ecf1c3a01bce59e640287d0d298f8
Author: Nina Zakharenko <nina@nnja.io>
Date:   Thu Oct 5 12:19:16 2017 -0700

    Who uses Java anyway?
    
# Your commit hash will be different than mine, so take note of it.

# Now that we have the commit where it was deleted, let's git blame it from one commit before then (using the ^ syntax) 

$> git blame b8e1a5692b0ecf1c3a01bce59e640287d0d298f8^ -- java_code.java
88f6e286 (Nina Zakharenko 2017-10-05 11:31:34 -0700 1) // This is a Java file
88f6e286 (Nina Zakharenko 2017-10-05 11:31:34 -0700 2)
88f6e286 (Nina Zakharenko 2017-10-05 11:31:34 -0700 3) public class HelloWorld {
88f6e286 (Nina Zakharenko 2017-10-05 11:31:34 -0700 4)    public static void main(String[] args) {
88f6e286 (Nina Zakharenko 2017-10-05 11:31:34 -0700 5)       System.out.println("Привет от Java!");
88f6e286 (Nina Zakharenko 2017-10-05 11:31:34 -0700 6)    }
88f6e286 (Nina Zakharenko 2017-10-05 11:31:34 -0700 7) }
```

`git blame` also accepts line numbers or regular expressions, if you want to limit your blaming to a range of lines or specific function, rather than blaming the entire file.

### Step 4 - Git Bisect
`git bisect` is a really useful function for determining where in history something changed, especially when given a large timeframe. Maybe a bug was introduced last month, but going through every commit since then would be too time-consuming. Say we want to find out where the line "This is a test of the emergency git-casting system." was added to our `hello.txt` file. First we need to know a commit range - we know it's present in our current commit, and we know it wasn't in our Initial commit, so let's start a `git bisect` session with those start and end points:


```
$> git log --oneline
git log --oneline
b8e1a56 Who uses Java anyway?
331024e Testing the emergency git-casting system
88f6e28 Adding bash, python, and java code examples
43388fe Initial commit

$> git bisect start b8e1a56 43388fe
Bisecting: 0 revisions left to test after this (roughly 1 step)
[331024e3b1c500b4a30e5975636399bb6542d5f4] Testing the emergency git-casting system

# Let's test the file...

$> cat hello.txt
Hello World!
This is a test of the emergency git-casting system.

# The line is there, so we'll mark this as Bad. Git now moves backward in time...

$> git bisect bad
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[88f6e2864bd0829c71654f1d19096f436a66ce07] Adding bash, python, and java code examples

$> cat hello.txt
Hello World!

# No "emergency" string, so we'll mark this as Good

$> git bisect good
331024e3b1c500b4a30e5975636399bb6542d5f4 is the first bad commit
commit 331024e3b1c500b4a30e5975636399bb6542d5f4
Author: Nina Zakharenko <nina@nnja.io>
Date:   Wed Oct 4 19:01:12 2017 -0700

    Testing the emergency git-casting system

:100644 100644 980a0d5f19a64b4b30a87d4206aade58726b60e3 b31a35bc9c5ae5aff4a0f76f7834cc2428408050 M	hello.txt
```

Excellent - with a simple test - git has helped us figure out that the offending string was introduced in the `331024e...` commit. We can even perform manual tests to see if our commit is good - like loading a webpage. `bisect` is even more powerful with automated tests. By using `git bisect run` with an automated test - such as a unit test or even a simple shell script - we can quickly and easily find bugs introduced into codebases with complex history. 
#### End of Example Nine