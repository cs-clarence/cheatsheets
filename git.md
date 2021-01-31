# Git-Cheat-Sheet
A cheat sheet for uncommon Git commands

## Git - The Three Trees
- Git always has three trees at the minimum:
    - Working Tree - this tree has all the untracked and tracked files in your repository, this is also where modifications first appear.
    - Index Tree - this is the proposed commit tree, where you add all the files that you want to commit. If a file reaches this tree, it's automatically tracked by git. It is also called the staging area.
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
| Command | Description |
| - | - |
| `git branch foo`                              | Create a new branch |
| `git branch -d foo`                           | Deletes a branch |
| `git branch --set-upstream-to repo/branch`    | Set an upstream branch in the current repository |
| `git switch foo`                              | Switch to a branch |
| `git switch -c\|--create foo`                 | Create and switch to a branch |
| `git checkout file.js`                        | Acts like `git reset --hard branch -- file.js`, replaces file in the three trees, does not do a merge, not working-tree safe |
| `git checkout foo`                            | Moves the HEAD to point to the commit that the branch is pointing to, also moves the three trees, it's more Working Tree safe because it also does a merge, use `git switch` instead |
| `git checkout -b foo`                         | To create and switch to a branch, use `git switch -c` instead |
| `git switch foo`                              | To switch to a branch |
| `git switch --create foo`                     | To create and switch to a branch |
| `git merge foo`                               | Merge branch into current branch 

## Pulling
| Command | Description |
| - | - |
| `git pull --rebase --prune`               | Get latest, rebase any changes not checked in and delete branches that no longer exist | 

## Staged Changes
| Command | Description |
| - | - |
| `git add file.txt`                            | Stage file |
| `git add --patch file.txt`                    | Stage some but not all changes in a file |
| `git mv file1.txt file2.txt`                  | Move/rename file |
| `git rm --cached file.txt`                    | Unstage file |
| `git rm --force file.txt`                     | Unstage and delete file |
| `git reset HEAD`                              | Reset changes in the index area and use HEAD as the source, use `git restore --staged` instead |
| `git reset --hard HEAD`                       | Unstage and delete changes in index/staging area and working directory |
| `git reset file.txt`                          | Unstange a file, more specifically restore the state of the file in the staging area with the one in the HEAD as a source, sames as `git reset --mixed file.txt`|
| `git reset eb34f -- file.txt`                 | Unstange a file, more specifically restore the state of the file in the staging area with the one in the commit eb34f as a source, sames as `git reset eb34f --mixed file.txt`|
| `git restore --staged files.txt`              | Restore changes to one or more files in the staging area, using the HEAD as a source |
| `git restore --staged --worktree files.txt`   | Restore changes to one or more files in the staging and working area, using the HEAD as a source |
| `git clean -f\|--force -d`                    | Recursively remove untracked files from the working tree |
| `git clean -f\|--force -d -x`                 | Recursively remove untracked and ignored files from the working tree |

## Changing Commits
| Command | Description |
| - | - |
| `git reset --soft 5720fdf`                    | Reset current branch but not working area and staging area to specified commit |
| `git reset --mixed 5720fdf`                   | Reset current branch, staging area but not the working area to the specified commit, `--mixed` is the default flag for every reset command |
| `git reset HEAD~1`                            | Reset the current branch and working area and staging area to the previous commit, same as `git reset --mixed` |
| `git reset --hard 5720fdf`                    | Reset current branch, staging area and working area to the specified commit |
| `git commit --amend -m "New message"`         | Change the last commit message |
| `git commit --amend --no-edit`                | Update the last commit but don't edit the message |
| `git commit --fixup 5720fdf -m "New message"` | Merge into the specified commit |
| `git revert 5720fdf`                          | Revert a commit |
| `git rebase --interactive [origin/main]`      | Rebase a PR (`git pull` first) |
| `git rebase --interactive 5720fdf`            | Rebase to a particular commit |
| `git rebase --interactive --root 5720fdf`     | Rebase to the root commit |
| `git rebase --continue`                       | Continue an interactive rebase |
| `git rebase --abort`                          | Cancel an interactive rebase |
| `git cherry-pick 5720fdf`                     | Copy the commit to the current branch |

## Compare
| Command | Description |
| - | - |
| `git diff`                                | See difference between working area and current branch |
| `git diff HEAD HEAD~2`                    | See difference between te current commit and two previous commits |
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
| `git show main`                         | Show the last commit in a branch |
| `git show 5720fdf`                        | Show named commit |
| `git blame file.txt`                      | See who changed each line and when |

## Stash
| Command | Description |
| - | - |
| `git stash push -m "Message"`             | Stash staged files, use `git stash save instead` |
| `git stash save "Optional Message"`       | Stash staged files |
| `git stash`                               | Stash staged files |
| `git stash --include-untracked`           | Stash working area and staged files |
| `git stash --keep-index`                  | Stash staged files, but don't remove them from staging area |
| `git stash show.txt`                      | Show stash summary for file | 
| `git stash list`                          | List stashes |
| `git stash list stash@{0}`                | Show information about the first stash |
| `git stash apply`                         | Moved last stash to working area |
| `git stash apply stash@{0}`               | Moved named stash to working area |
| `git stash apply --index`                 | Apply and stage the last stash |
| `git stash apply --index stash@{0}`       | Apply and stage the named stash |
| `git stash drop`                          | Delete the last stash | 
| `git stash drop stash@{0}`                | Delete the named stash | 
| `git stash pop`                           | Apply and delete the last stash | 
| `git stash clear`                         | Delete all the stash |
| `git stash --patch`                       | Interactively choose which portion of the modified changes do you want to stash |
| `git stash branch stashbranchname`        | Stash the staging area and create a new branch from it |

## Tags
| Command | Description |
| - | - |
| `git tag`                                            | List all tags |
| `git tag -a\|--annotate 0.0.1 -m\|--message "Message"` | Create a tag |
| `git tag -d\|--delete 0.0.1`                          | Delete a tag |
| `git push --tags`                                    | Push tags to remote repository |

## Remote
| Command | Description |
| - | - |
| `git remote -v`                           | List remote repositories |
| `git remote show origin`                  | Show remote repository details |
| `git remote add upstream <url>`           | Add remote upstream repository |
| `git fetch upstream`                      | Fetch all remote branches |
| `git rebase upstream/main`              | Refresh main branch from upstream |
| `git remote -v`                           | List remote repositories |
| `git push --tags`                         | Push tags to remote repository |

## Submodules
| Command | Description |
| - | - |
| `git submodule status`                    | Check status of all submodules |

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
[Github Git Cheat Sheet](https://training.github.com/downloads/github-git-cheat-sheet/)
[Git Visual Cheat Sheet](https://ndpsoftware.com/git-cheatsheet.html)
[Git Rebase In-Depth](https://git-rebase.io/)
