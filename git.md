# Git-Cheat-Sheet
A cheat sheet for uncommon Git commands

## Git - The Three Trees
- Git always has three trees at the minimum:
    - Working Tree - this tree has all the untracked and tracked files in your repository, this is also where modifications first appear.
    - Index Tree - this is the proposed commit tree, where you add all the changes that you want to commit. If a file reaches this tree, it's automatically tracked by git. It is also called the staging area.
    - HEAD Tree - this tree points to your current branch, which in turn points to a commit. So the HEAD transitively points to the commit that the current branch is pointing. The commit that is pointed by the HEAD will be the parent of your next commit.
- Git always compares these three trees to determine the changes in your files. The `git status` command will show modifications in the working tree that are not in the index.
- Once you stage the changes in the working tree, they get added to the index tree, which will make the working and index tree the same, but not the HEAD tree.
- Running `git status` again will show which files are in the index. Once you commit the changes in staging area, the commit is added to the branch which has all the changes you made.
- At newest commit's parent will be the previous commit that the HEAD is pointing to.
- The branch then will move it's pointer to the latest commit, along with the HEAD.
- Now HEAD points to the branch that points to the latest commit. This makes the working tree, index tree, and HEAD tree equivalent.
- Running `git status` will now show that there are no changes.

## Important Notes For Some Commands:
- `git checkout` will move the HEAD itself to a specified commit or branch.
- `git reset` will not move the HEAD but the branch that the HEAD is pointing to.

## Configuration

| Command | Description |
| - | - |
| `git config --global user.name "foo"`              | Set user name |
| `git config --global user.email "foo@example.com"` | Set user email |

## Branches
Branches are used to "branch off" from a commit. 
Branches are essentially pointers to the "tip commit" of the current branch.
What makes branches special is that if you create another commit to branch, the branch will automatically move it's pointer to that new commit.

| Command | Description |
| - | - |
| `git branch`                                  | List all the branches |
| `git branch -r`                               | List all remote branches |
| `git branch --merged`                         | List all merged branches |
| `git branch <branch-name>`                    | Create a new branch |
| `git branch -d <branch-name>`                 | Deletes a branch |
| `git branch --set-upstream-to repo/branch`    | Set an upstream branch in the current repository |
| `git checkout -b <new-branch>`                | Create and switch to a branch using HEAD as the base, you can also use `git switch -c` |
| `git checkout -b <new-branch> <source>`       | Create and switch to a branch using the specified source as the base |

## Manipulating the HEAD
HEAD is a pointer to a commit or a branch, we can move it to a source (commit, branch, tag, or HEAD itself) using the `git checkout` command. Moving the HEAD also will also update the contents of Working and Index Tree. Moving to a commit other than a tip commit of a branch will result in a state called `detached head`, this is normal when you're just inspecting the content of a specific commit.

| Command | Description |
| - | - |
| `git checkout <source>` | Move the head to a source, updating the content of working and index tree |

## Resetting Files in the Working Tree
`git checkout` also has other uses other than moving the HEAD which is resetting specific files in the Working Tree based on a source by providing paths. This is different than regular checkout because it doesn't move the HEAD and just affects the Working Tree.

| Command | Description |
| - | - |
| `git checkout <source> <files>` | Reset files in the working tree based on the source |

## Manipulating Branches
As previously mentioned, a branch is nothing more than a pointer to a commit that moves when a new commit is added. Being a pointer, we can change where it is pointing to using the `git reset` command. Unlike `git checkout`, which moves the HEAD tree itself, `git reset` moves the current branch itself to a source. Remember that the HEAD, when attached, also points to the current branch, so the effect is as if the HEAD also moved. The source can either be a branch, a commit, the HEAD, or a tag. Be very careful when using this because resetting a branch to a previous commit also effectively drops the commits after that previous commit. Reset can also cascade it's effects to the working tree and index tree.

`git reset` flags that can be used to specify which trees should be affected
- `--soft` - only affects the `HEAD`
- `--mixed` - only affects the `HEAD` and the staging/indexing area, this is the default when neither of the three flags is provided
- `--hard`- affects the `HEAD`, staging/indexing area, and the working tree

| Command | Description |
| - | - |
| `git reset --soft <source>`                           | Reset changes only in the HEAD tree based on the source |
| `git reset <source>` or `git reset --mixed <source>`  | Reset changes in the index and HEAD tree based on the source |
| `git reset --hard <source>`                           | Reset changes in working, index, and HEAD tree based on the source |

## Resetting Files in the Index/Staging Tree
`git reset` has other use besides moving the branch, it can also reset files in the index tree using a source when provided with paths. Unlike the regular `git reset`, however, it does not move the branch to the source, it merely affects the files in the index tree. This use of reset can only use the `--mixed` flag because it wouldn't make sense to use the other flags otherwise.

| Command | Description |
| - | - |
| `git reset <source> <files>` or `git reset --mixed <source> <files>` | Reset the files in the index tree based on the source |

## Merging
Merging is the act of applying the changes of a `<source-branch>` into a destination branch (which is the current branch).
Merging can either be a `3-way merge` (AKA `merge commit`) or `fast-forwarded merge`, git will choose one of the two depending on the situation.

`3-way merge` creates a special commit called a `merge commit`. A merge commit is special because it has two parents commits which are the tip commits of the two branches being merged. If there are no conflicts between the two commits, a `merge commit` will be made on the current branch.

`before merge`
```
         master (destination)
           |
           v
 <- *a <- *b <- *e
           ^
            \
             <- *c <- *d
                       ^
                       |
                    feature (source)
```
`after merge`
```
                         master (destination)
                           |
                           v
 <- *a <- *b <- *e <----- *f (merge commit)
           ^             /
            \           v
             <- *c <- *d
                       ^
                       |
                    feature (source)
```
    
`fast-forward merge` is used when the destination branch has no other commits after the commit where the source branch is branched off of.
Git will take advantage of that by just moving pointer of the destination branch to the commit where the source branch is pointing to.

`before merge`
```
         master (destination)
           |
           v
 <- *a <- *b (no more commit after commit b, fast forward will be used when merging the feature branch)
           ^
            \
             <- *c <- *d
                       ^
                       |
                    feature (source)
```
`after merge`
```
 <- *a <- *b        master (destination, fast forwarded)
           ^           |
            \          v
             <- *c <- *d
                       ^
                       |
                    feature (source)
```
There are cases against both of types, fast forward looks cleaner but information about the git history is lost because of the fact that there will be no information about where the merge has happened due to the lack of a merge commit. The `--no-ff` flag will disable fast forward merges even if they're possible to do.

| Command | Description|
| - | - |
| `git merge <source-branch>`                               | Merge source into current branch |
| `git merge <source-branch> <destination-branch`           | Merge source into destination |
| `git merge --no-ff <source-branch>`                       | Merge source into current branch but don't use fast forward even if possible |

## Pulling and Fetching
Pulling and fetching are used to download commits and files from remote repositories. 
Even though the two are similar, they have different use-cases.
`git fetch` is the safer of the two because it only download the content from the remote repository into your local fetched content but not update your local branches so it has no effect on local development work. 
Git is able to do this because isolates fetched remote content from local content. 

`git pull` on the other hand also updates your local branches.
Pull is actually composed of two actions, a `git fetch` and a `git merge` (or `git rebase` if `--rebase` flag is provided).
Using the bare `git pull` command will target the current branch, fetch it's content from the remote version of the branch, then merge remote branch into the local branch.
The merge can either be a `3-way merge` or a `fast-forward merge` depending on the situation.
The `--rebase` flag can provided to use a rebase instead of a merge after fetching, in this case, the local branch will be rebased to the remote branch.

| Command | Description |
| - | - |
| `get fetch` | Fetches all content of origin (the default when no remote is provided) remote repository |
| `get fetch <remote>` | Fetches all content of the specified repository |
| `get fetch <remote> <branch>` | Fetches all content of specified remote branch repositories |
| `get fetch --all` | Fetches all content of every remote repositories into the local repository |
| `get pull` | Fetch the current branch from the remote repository and merge it into the local version |
| `get pull <remote>` | Fetch the current branch from the specified remote repository and merge it into the local version |
| `get pull <remote> <branch>` | Fetch the specified branch from the specified remote repository and merge it into the current local branch |
| `get pull --rebase` | Fetch the current branch from the remote repository and the local version to the remote version |
| `git pull --rebase --prune` | Get latest, rebase any changes not checked in and delete branches that no longer exist | 

## Staging Files
Staging is the act of preparing changes that should be committed and files to be tracked. Once a file is staged, git will automatically track it's changes by compare the file in the staging/index tree and the one from the working tree. Once changes are mod to tracked files, changes to them need to be readded using `git add` before you can commit them.

| Command | Description |
| - | - |
| `git add file.txt`                            | Stage file/changes to a file |
| `git add .`                                   | Stage all file/changes to files |
| `git add --patch file.txt`                    | Stage some but not all changes in a file |

## Unstaging Files
Sometimes you need to remove file(s) from the staging/index tree. A common use-case for this is when adding a new entry to a .gitignore file;
once a file is staged, before adding it to .gitignore, you need to unstage it before before git to actually start ignoring it.

| Command | Description |
| - | - |
| `git rm --cached <files>`                    | Remove the file(s) from the index |
| `git rm <files>`                             | Remove the file(s) from the index and working directory, but will warn you if you have unsaved changes |
| `git rm --force <files>`                     | Force remove the file(s) from the index and working directory and disregarding any changes |

## Committing
Committing is the act of saving the staged changes into a "checkpoint" that you can go back to if you made a mistake in your current Working Tree.

Commits, in git, are like singly linked list because they have a reference to a commit before them called a "parent commit". 
The very first commit doesn't have a parent commit so it points to a null.
```
<- *a <- *b <- *c
```
In this visual, a, b, c are commits pointing to their respective parent commit.

| Command | Description |
| - | - |
| `git commit -m "New message"`                 | Commit the staged changes with a message |
| `git commit --amend -m "New message"`         | Update the last commit with the staged changes and a new message |
| `git commit --amend --no-edit`                | Update the last commit with the staged changes but don't edit the message |
| `git commit --fixup 5720fdf -m "New message"` | Merge into the specified commit |

## Cleaning the Working Tree

| Command | Description |
| - | - |
| `git clean -f\|--force -d`                    | Recursively remove untracked files from the working tree |
| `git clean -f\|--force -d -x`                 | Recursively remove untracked and ignored files from the working tree |

## Rebasing

| Command | Description |
| - | - |
| `git revert 5720fdf`                          | Revert a commit |
| `git rebase --interactive [origin/main]`      | Rebase a PR (`git pull` first) |
| `git rebase --interactive 5720fdf`            | Rebase to a particular commit |
| `git rebase --interactive --root 5720fdf`     | Rebase to the root commit |
| `git rebase --continue`                       | Continue an interactive rebase |
| `git rebase --abort`                          | Cancel an interactive rebase |

## Cherry-Picking Commits

| Command | Description|
| - | - |
| `git cherry-pick 5720fdf`                     | Copy the commit to the current branch |

## To Rebase or To Merge
There has a been discussion among git users as to which approach is better. The common advise is to never rebase anything branch that is public, public meaning there is more than one developer working on the branch. As for private branches, do what you prefer. 

## Compare

| Command | Description |
| - | - |
| `git diff`                                | See difference between working tree and HEAD tree |
| `git diff HEAD HEAD~2`                    | See difference between the current commit and two previous commits |
| `git diff main other`                     | See difference between two branches |

## View

| Command | Description |
| - | - |
| `git log`                                 | See commit list |
| `git log --patch`                         | See commit list and line changes |
| `git log --decorate --graph --oneline`    | See commit visualization |
| `git log --grep foo`                      | See commits with foo in the message |
| `git show HEAD`                           | Show the current commit |
| `git show HEAD^` or `git show HEAD~1`     | Show the previous commit |
| `git show HEAD^^` or `git show HEAD~2`    | Show the commit going back two commits |
| `git show main`                           | Show the last commit in a branch |
| `git show 5720fdf`                        | Show named commit |
| `git blame file.txt`                      | See who changed each line and when |

## Stash

Stashes are used **temporarily store your (staged and unstaged) changes** so that you can a clean working tree and work on something else.
Stashes are stack based and as such use `push` and `pop` operations.
| Command | Description |
| - | - |
| `git stash push -m "Message"`             | Stash changes |
| `git stash push -m "Message"` <files...>  | Stash changes but only from on specified files |
| `git stash list`                          | List stashes |
| `git stash list stash@{0}`                | Show information about the first stash |
| `git stash --include-untracked/-u`        | Stash changes including new (untracked/not `git add`ed) files |
| `git stash --keep-index`                  | Stash staged changes, but don't remove them from staging area |
| `git stash show.txt`                      | Show stash summary for file | 
| `git stash apply`                         | Moved last stash to working area |
| `git stash apply stash@{0}`               | Moved named stash to working area |
| `git stash apply --index`                 | Apply and stage the last stash |
| `git stash apply --index stash@{0}`       | Apply and stage the named stash |
| `git stash drop`                          | Delete the last stash | 
| `git stash drop stash@{0}`                | Delete the named stash | 
| `git stash pop`                           | Apply and delete the top most stash (combination of `apply` and `drop`) | 
| `git stash clear`                         | Delete all the stash |
| `git stash --patch`                       | Interactively choose which portion of the modified changes do you want to stash |
| `git stash branch <stash branch name>`    | Stash changes and create a new branch from it |

## Tags
Tags are kind of similar to branches, both are used to point to a commit.
Unlike branches, however, tags are immutable, you can't change the commit it's pointing to.
Tags are useuful 
| Command | Description |
| - | - |
| `git tag`                                                 | List all tags |
| `git tag -a\|--annotate 0.0.1 -m\|--message "Message"`    | Create a tag |
| `git tag -d\|--delete 0.0.1`                              | Delete a tag |
| `git push --tags`                                         | Push tags to remote repository |
| `git checkout <tag-annotation>`                           | Checkout to the specified tag |

## Remote
| Command | Description |
| - | - |
| `git remote -v`                                       | List remote repositories |
| `git remote show <remote>`                            | Show remote repository details |
| `git remote add <remote> <url>`                       | Add remote upstream repository |

## Submodules
- Submodules are a way to incorporate a git repository as a subdirectory in your repository.
- Git knows which are submodules in your repo.
- Submodules are added to your commit but are not tracked by git. The content inside are not added to the commit.
- When you add a submodule to your repository using `submodule add`, files are not pulled by default.

| Command | Description |
| - | - |
| `git clone --recurse-submodules <url> [name]`                 | Clone a repository, download it's contents, as well as the submodules inside it |
| `git submodule add <url> [name]`                              | Add a submodule to your repository, adds the submodule to the .gitmodules file which git tracks |
| `git submodule status`                                        | Check status of all added submodules |
| `git submodule sync`                                          | Syncs the submodule if there are any changes to the upstream repo such as URL changes |
| `git submodule init`                                          | Initialize the git configuration file with the added submodules |
| `git submodule update`                                        | Downloads the content of the submodule |
| `git submodule update --init [path to sm]`                    | Initalize the config file first then download the contents, if path to submodule is not specified, applies to all submodule |
| `git submodule update --recursive [path to sm]`               | Download content of the submodule as well as the content of the submodules inside them, if path to submodule is not specified, applies to all submodule |
| `git submodule update --init --recursive [path to sm]`        | Initialize the config file, download recursively, if path to submodule is not specified, applies to all submodule |
| `git submodule update --remote [path to sm]`                  | Pull the updates in remote tracking branches (assumes master branch is tracking branch), if path to submodule is not specified, applies to all submodule |
| `git pull --recurse-submodules`                               | Does a`git pull` then runs `submodule update --recursive` right after |
| `git push --recurse-submodules check`                         | Does a`git push` to your repository and submodules, the `check` option makes the the push fail if push on submodules fail |
| `git push --recurse-submodules on-demand`                     | Does a`git push` to your inside the submodule folders first then into the main repository, will not continue pushing the main if one push in a submodule fails |
| `git config --local submodule.<submodule-name>.branch <tracking branch>`                  | Set which branch to track in a submodule locally, not tracked by git |
| `git config --local -f .gitmodules submodule.<submodule-name>.branch <tracking branch>`   | Set which branch to track in a submodule locally and add reflect the change to .gitmodules file so other it can be tracked by git |
| `git submodule update --remote --merge [path to sm]`          | Fetch the updates of the submodule then merge, applies to all submodule if no path is specified  | 
| `git submodule update --remote --rebase [path to sm]`         | Fetch the updates of the submodule then rebase, applies to all submodule if no path is specified  | 

### NOTE:
- When using `git submodule update --recursive` to fetch updates from remote tracking branch, it best to add `--init` flag to be on the safe side as the newest commit might have added a new submodule which is needed to be initialized.
- You can treat submodules as their own repositories, you don't have to use the submodule commands, just go inside the submodule folders and manage them there.

- Pull submodules
  1. `git submodule sync`
  2. `git submodule init`
  3. `git submodule update`
- Change branch
  1. `cd /submodule`
  2. `git fetch origin <branch-name>`
  3. `git checkout <branch-name>`
  4. `cd /`

- Additional Sources
    - [Github Git Cheat Sheet](https://training.github.com/downloads/github-git-cheat-sheet/)
    - [Git Visual Cheat Sheet](https://ndpsoftware.com/git-cheatsheet.html)
    - [Git Rebase In-Depth](https://git-rebase.io)
    - [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)
    - [Git Merge](https://www.atlassian.com/git/tutorials/using-branches/git-merge)
