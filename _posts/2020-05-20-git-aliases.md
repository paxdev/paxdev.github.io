---
  title: Git Aliases
  categories:
    - git
  tags:
    - git
    - productivity
---

# Create a shortcut
To save yourself a bit of typing you can create a Git Alias for frequently used commands, e.g.

```shell
git config --global alias.create-branch 'checkout -b'
```

I can now type something like

```shell
git create-branch git-aliases
```

and I will create and switch to a new branch called `git-aliases`.

Under the hood, `git` has updated my `.gitconfig` and added an `alias` section as follows

```
[alias]
	create-branch = checkout -b
```

# Get the name of the current branch
I can never remember the syntax for getting the name of the current branch: 

```shell
git config --global alias.current-branch 'rev-parse --abbrev-ref HEAD'
```

So now `git current-branch` will return the name of the current branch.


# Chaining commands
Of course this in and of itself is of limited use except for shortening common commands or giving them more memorable names.

However, if you add a `!` at the beginning of your `alias` you are telling `git` to start a `Shell` where you can start using `bash`.

* **Committing everything**

  I pretty much always want to commit all of my unstaged files - otherwise how do I know if I've missed something crucial from my commit?

  ```shell
  git config --global alias.commit-all '!git add -A && git commit -m'
  ```

  I can now type, e.g. `git commit-all "add a commit-all alias"`.

  Git will expand the command in place so that anything I type after the alias is treated as part of the command. In the above example it's as though I had typed `commit -m "my commit message"`.

* **Cleaning up and starting afresh**

  When I've finished working on a branch I want to go back to `master`, get latest and clean up any stray non-tracked files. 

  ```shell
  git config --global alias.get-clean-master '!git checkout master && git pull && git clean -xfd -e .vs/'
  ```

  Usage `git get-clean-master`.

  Note the `-e .vs/` ignores a folder Visual Studio takes an exclusive lock on, so some files cannot be deleted without closing Visual Studio. Omit this part if you don't use Visual Studio.

* **Getting latest from `master` into branch**
  
  When I'm working in a branch and someone has updated `master` I want to pull their changes in to my branch.

  The simplest way to do that is with 

  ```shell
  git config --global alias.get-latest '!git fetch origin && git merge origin master'
  ```

  Then `git get-latest` will fetch the latest changes from `origin` and merge them into my branch. The problem with this approach is that my local copy of `master` is still behind `origin`, so when I switch back I'll still need to update.
  
  To make sure my local `master` is updated with the changes too, a more complex command is.
  
  ```shell
  git config --global alias.get-latest '!branch=$(git current-branch) && git checkout master && git pull && git checkout $branch && git merge master --no-edit'
  ```

  So now, from a branch you can type `git get-latest` and it will switch to `master`, do a `git pull`, switch back to your branch merging from `master` and accepting the default commit message

  Note how we have managed to reuse our earlier alias `git current-branch`. Nice!

  Note also the use of single `'` quotes above. If I had used double `"` quotes then `git` would have evaluated what's inside the double quotes, i.e. the `$(git current-branch)` and `$branch`. At the point of execution (i.e. when creating the alias) they would have evaluated to `null` and I would have got unexpected blank spaces.

* **Starting a GitHub Pull Request**

  When you've finished working on your branch and you want to create a Pull Request:

  ```shell
  git config --global alias.pull-request '!branch=$(git current-branch) && git remote get-url origin | sed \"s/\.git$/\/pull\/new\/$branch/\" | start $(cat)'
  ```

  Usage `git pull-request`

  This will assemble the correct `GitHub` URL to create a Pull Request and launch it in the default browser. **Caveat this is only tested on Windows!**

* **Starting a GitLab Merge Request**

  The version of the above for GitLab is:

  ```shell
  git config --global alias.merge-request '!branch=$(git current-branch | sed \"s/\//%2f/g\") && git remote get-url origin | sed \"s/\.git$//\" | sed \"s/$/\/-\/merge_requests\/new?merge_request%5Bsource_branch%5D=/\" | sed \"s|$|${branch}|\" | start $(cat)'
  ```

  Note that handily GitHub and GitLab use differt names *Pull-* and *Merge-* request so you can have both commands globally if you happen to use both.

  For more information, check out [this excellent article from Phil Haack](https://haacked.com/archive/2014/07/28/github-flow-aliases/) 