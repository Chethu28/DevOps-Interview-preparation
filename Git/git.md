# Git Interview Questions and Answers

This README contains a list of common Git interview questions along with their answers.



### 1. What is Git?

Git is also called as version control system and source code management.
It is used to track the files.
It will maintain the multiple versions of the same file.
Git is free and open source.

### 2. What are the benefits of using Git?

Some benefits of using Git include:
- Distributed version control: Each developer has their own copy of the repository, allowing for easier collaboration and offline work.
- Branching and merging: Git allows for easy creation of branches for new features or bug fixes, and seamless merging of changes between branches.
- Data integrity: Every change made to the codebase is tracked, providing a complete history of the project.
- Performance: Git is designed for speed, allowing for quick commits, branching, and merging operations.

### 3. What is a repository in Git?

A repository in Git is a collection of files and their history of changes. It contains all the files and metadata necessary to track the changes made to the project over time.

### 4. Explain the difference between Git and GitHub.

Git is a software                                          GitHub is a service
Git can be installed locally on the system                 GitHub is hosted on the web
Git has 3 types of repositories                            GitHub has 2 types of repositories
Git has a default branch called master                     GitHub has a default branch called main

### 5. What is a commit in Git?

A commit in Git is a snapshot of the repository at a particular point in time. It represents a set of changes made to the files in the repository, along with a commit message that describes the changes.

### 6. How do you revert a commit that has already been pushed and made public?

To revert a commit that has already been pushed and made public, you can use the `git revert` command followed by the hash of the commit you want to revert. This will create a new commit that undoes the changes made by the specified commit.



### 7. Explain the difference between 'git pull' and 'git fetch'.

`git pull` Git pull updates the current HEAD branch with the latest changes from the remote server.
    


`git fetch` The Git fetch command only downloads new data from a remote repository.


### 8. What is a merge conflict in Git?

A merge conflict in Git occurs when two branches have diverged and Git is unable to automatically merge the changes. This typically happens when the same part of a file has been modified in both branches.

### 9. How do you resolve a merge conflict in Git?

To resolve a merge conflict in Git, you need to manually edit the files to resolve the conflicting changes. After resolving the conflicts, you need to stage the changes and commit them to complete the merge.
    

### 10. What is git logs?

Git log is a history, we can see the list of every commit in our project along some
other information like who commit the file and when we commit the file.

### 11.What is git merge?
It allows us to get the code from one branch to another. This is useful when
developers work on the same code and want to integrate their changes before pushing
them up in a branch.

### 12.What is git diff?
It is used to show all the differences between any two commits or files within your
Git repository.

### 13.What is git stash?
It is used to save your changes but not record them in the Git repository.

### 14.What is git push?
Git push is used to share the local files into central repositories like GitHub and
Bitbucket.

### 15.What is git pull?
Git pull is the combination of git fetch and git merge
Git pull is used to get the commits from central repo to local repo

### 16.What is git tag?
Tagging is generally used to capture a point in history that is used for a marked
version release.

### 17. git merge vs git rebase
When you want to get the changes from one branch to another, we can do merge or
rebase
Merging preserves the history of both branches, while rebasing rewrites the history
of the branch being rebased.
Merging creates a new commit that has two parent commits, while rebasing applies
the changes from one branch onto another.
Merging is generally easier and safer, while rebasing can be more complex and
risky.



