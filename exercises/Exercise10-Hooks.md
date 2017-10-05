# Advanced Git
## Exercise Ten - Hooks

### Overview
In this exercise, we'll set up a pre-commit hook and see how this powerful tool might be used to keep low-quality code from getting into your repo.

### Prerequisite
You should have the [`advanced-git-exercises`](https://github.com/nnja/advanced-git-exercises)  repository cloned locally. Checkout the `exercise10` branch to begin:

```
$> git checkout exercise10
Switched to branch 'exercise10'
```

Note: This pre-commit hook script will only work in a Bash environment. Windows users can follow along but the script my not work as expected.

### Exercise
1. Copy the `pre-commit` script into your git hooks folder and make it executable. Try committing a shell (.sh) script to your repo with no shebang (#!) line at the top - your commit should fail. Try committing a script with a shebang line - your commit should succeed.

## Solution

### Step 1 - Set up a Pre-Commit Hook
Git has the ability to call arbitrary scripts at different points in time, such as pre-commit, post-merge, and pose-checkout, among others.

In our `exercise10` branch we should have a little bash script called `pre-commit`. This script will run before a commit - it will call git to get the names of any .sh script that's staged to be committed, then check the first two characters to see if they match `#!` (called a shebang line in bash). If this line doesn't exist, it will throw an error and not allow the change to be committed.

Move this script into your git hooks folder, make sure it's executable, then make a new shell script - without a #! line - and try to commit it:

```
$> cp pre-commit .git/hooks/pre-commit

$> chmod +x .git/hooks/pre-commit

$> echo "Bad bash script" > test_script.sh

$> git add test_script.sh

$> git commit -m "Adding a new test script"
No shebang found! Not allowed to commit!
```

Oh no! Let's fix our test script so that it has a valid #! line and try committing it again:

```
$> echo '#!/bin/bash\n Good bash script' > test_script.sh

$> git add test_script.sh

$> git commit -m "Adding a new test script"
[exercise10 6b346ab] Adding a new test script
 1 file changed, 1 insertion(+)
 create mode 100644 test_script.sh

```
Success! This was a very simple example, but it's easy to see how this can be extended to do proper linting or checking of code - or even run unit tests - before commit, to decrease the chances of bad code being checked-in.

#### End of Exercise Ten 