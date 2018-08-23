## Task 1
1. _Squash_ the five relevant commits into one, with the commit message of "close #321"

This is done by command:
```bash
git rebase -i 5905dfd # Where 5905dfd is commit-id
```

2. How does `git log` look now?
```bash
b4e4776 (HEAD -> master) Close #321
5905dfd initial file
```
## Task 2

You have a tree looking like this:
```bash
* 78457b0 (HEAD -> master) Implement second part of feature
* 03b4f6c Fix bug
| * 693e505 (new-feature) Implement first part of feature
|/  
* 647ae98 Initial commit
```

1. Move the faulty commit `Implement second part of feature` from the `master` branch to the `new-feature` branch.
2. Copy the bugfix to your feature branch as well

**This can be done in two ways...**
### 1st solution

```bash
git log --oneline --graph --decorate --all # to get an overview over commits
git branch # to check which branch we are on
git checkout new-feature # to switch branch
git rebase master new-feature # to rebase master onto new-feature
```

There is a merge-conflict in file myapp.txt. Solve by changing the content to:
```text
Some code
Some other line of code
Another line of code
First part of new awesome feature
Second part of new feature
```

Now run:
```bash
git add myapp.txt && git rebase --continue # to add the solved conflict and to continue rebase
git checkout master # to checkout master
git revert HEAD # to revert the faulty commit
```

Log output - note that the sequence is the first/second part of the commit is not taken into account, since this not necessarily has to be in sequence in practice:
```bash
git log --oneline --graph --decorate --all
* 1c14ad2 (HEAD -> master) Revert "Implement second part of feature"
| * c1c5584 (new-feature) Implement first part of feature
|/  
* 8cee7fb Implement second part of feature
* b48ca85 Fix bug
* 6c56128 Initial commit
```

### 2nd solution
With cherry pick.. But this results in two "Fix bug" commits..

```bash
git checkout new-feature
git cherry-pick 03b4f6c
git cherry-pick 78457b0
error: could not apply 78457b0... Implement second part of feature
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git commit'

```

```bash
git status

On branch new-feature
You are currently cherry-picking commit 78457b0.
  (fix conflicts and run "git cherry-pick --continue")
  (use "git cherry-pick --abort" to cancel the cherry-pick operation)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   myapp.txt

no changes added to commit (use "git add" and/or "git commit -a")
```


Fix the conflict..
```bash
nano myapp.txt

Some code
Some other line of code
Another line of code
First part of new awesome feature
Second part of new feature
```

```bash
git add . && git commit -m "Implement second part of feature"
git checkout master 
git revert 78457b0
```

End result...
```bash
git log --oneline --decorate --graph --all

* f39ccc3 (HEAD -> master) Revert "Implement second part of feature"
* 78457b0 Implement second part of feature
* 03b4f6c Fix bug
| * d064d1d (new-feature) Implement second part of feature
| * a2699c8 Fix bug
| * 693e505 Implement first part of feature
|/  
* 647ae98 Initial commit

```

End result in the files are the same but, 1st solutions looks nicer in my opinion since the bug fix commit is only applied once..

## Task 3
You have developed a new feature, and wants to commit it to master:

```bash
* 1f37b43 (HEAD -> master) Add readme
| * a824913 (uppercase) Change greeting to uppercase
|/  
* b84f28f Add content to greeting.txt
* efb74a6 Add file greeting.txt
```

Instead of just merging the commit, you like a straight line of commits like the example below

```bash
* 47558e0 (HEAD -> uppercase) Change greeting to uppercase
* 1f37b43 (master) Add readme
* b84f28f Add content to greeting.txt
* efb74a6 Add file greeting.txt
```



* Show the commands in order to achieve this position of commits.

```bash
git checkout feature # to checkout feature branch
git rebase master feature # to rebase master onto the feature branch
```

The output of `git log --oneline --graph --decorate --all` looks like this:
```bash
* b8aa867 (HEAD -> feature) Change greeting to uppercase
* 454bd79 (master) Add readme
* 6697cc2 Add content to greeting.txt
* 3474aea Add file greeting.txt
```


if you would like to make the actual commit of the feature to master, run the following commands:
```bash
git checkout master # to checkout master
git merge feature # to merge the feature branch onto master
```

Log output - notice that the the master branch is one step ahead of the desired:
```bash
git log --oneline --graph --decorate --all
* 568d990 (HEAD -> master, feature) Change greeting to uppercase
* a954a59 Add readme
* 8ada123 Add content to greeting.txt
* e0cf376 Add file greeting.txt
```


* Why does the commit with message "Change greeting to uppercase" change sha, when the others do not?

```bash
This is because is because when you rebase the master onto the feature branch, you basically take the diff between master and feature and make apply changes in a new commit that gets applied at last. Since every commit has a unique SHA1 it is the case that it has changed in this scenario as well.
```

## Task 4

Look at the git graph and status of the created repository to find out the state.

### Task

* In words, describe what you are seeing in the repository. Use the git terminology when applicable.

By issuing `git log --oneline --graph --decorate --all` we can see from the output below, that there is two commits to master. The position of HEAD is at master.

```bash
git log --oneline --graph --decorate --all
* 1bb04d5 (HEAD -> master) Add bad content to afile.txt
* de8c255 first commit
```

By issuing `git status` we can see that there are modifications to the file `afile.txt` and that these are in the staging area (index). Also, there are new changes that are not yet added to the staging area (index) which only yet lives in the working area. To find the difference between the files run `git diff` that will give you the following output which we can conclude the last line "a sandbox change" was added:
```bash
diff --git a/afile.txt b/afile.txt
index 936a148..f9a947e 100644
--- a/afile.txt
+++ b/afile.txt
@@ -1,3 +1,4 @@
 first commit
 second (bad) commit
 a uncommitted change
+a sandbox change
```


Show the list of commands you need to do the following (in the given order).

* remove the work in progress using git.
* remove the staged change
* make master go back to the commit before the bad commit
* does git track bfile.txt? Reason your answer. 

To remove both work in progress and staged changes (index) issue the commands:
```bash
git checkout afile.txt # to remove changes in the working tree
git reset afile.txt # to remove changes in the staged area (index)
git checkout afile.txt # to remove changes in the working directory again
git status # to verify that the working tree is clean
```

To make master go to the commit before the bad commit, make a revert and apply it:
```bash
git revert 0865ba9 # insert the SHA1 of the commit to revert and apply the commit
```

Output of `git log --oneline --graph --decorate --all`:
```bash
* 0865ba9 (HEAD -> master) Revert "Add bad content to afile.txt"
* 277393e Add bad content to afile.txt
* 5ca14dd first commit
```

Git does NOT track `bfile.txt` because it is explicitly written in the file `.gitignore`