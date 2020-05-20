---
  title: Git Aliases
  categories:
    - git
  tags:
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


# Chaining commands
Of course this in and of itself is of limited use except for shortening common commands or giving them more memorable names.
However, if you add a `!` at the beginning of your `alias` you are telling `git` to start a `Shell` where you can start using `bash`.

* Committing everything

  I pretty much always want to commit all of my unstaged files - otherwise how do I know if I've missed something crucial from my commit?

  ```shell
  git config --global alias.commit-all '!git add -A && git commit -m'
  ```

  I can now type, e.g. `git commit-all "add a commit-all alias"`.

  Git will expannd the command in place so that anything I type after the alias is treated as part of the command. In the above example it's as though I had typed `commit -m "my commit message"`.

* Getting clean `master`.

  I often find I want to go back to `master`, get latest and just clean up. E.g. after I've finished working on a branch.

  ```shell
  git config --global alias.get-clean-master '!git checkout master && git pull && git clean -xfd -e .vs/'
  ```

  You can now use `git get-clean-master`.

  Note the `-e .vs/` is a Visual Studio only workaround for some files that Visual Studio takes an exclusive lock on and cannot be deleted without closing Visual Studio. Omit this part if you don't use Visual Studio.

* Getting latest from `master`
  
  When I'm working in a branch and someone has updated `master` I want to pull their changes in to my branch. 
  I normally want to make sure my local `master` is updated with the changes too so I don't have to remember to do that later.
  
  ```shell
  git config --global alias.get-latest '!branch=$(git rev-parse --abbrev-ref HEAD) && git checkout master && git pull && git checkout $branch && git merge master --no-edit'
  ```

  So now, from a branch you can type `git get-latest` and it will switch to `master`, do a `git pull`, switch back to your branch merging from `master` and accepting the default commit message

  Note the use of single `'` quotes above. If I had used double `"` quotes then `git` would have evaluated what's inside the double quotes, i.e. the `$(git rev-parse --abbrev-ref HEAD)` and `$branch`. At the point of execution they would have evaluated to `null` and I would have got unexpected blank spaces.
