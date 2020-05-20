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

* Getting latest from `master`
  
  When I'm working in a branch and someone has updated `master` I want to pull their changes in to my branch. 
  I normally want to make sure my local `master` is updated with the changes too so I don't have to remember to do that later.
  
  ```shell
  git config --global alias.get-latest '!branch=$(git rev-parse --abbrev-ref HEAD) && git checkout master && git pull && git checkout $branch && git merge master --no-edit'
  ```

  So now, from a branch you can type `git get-latest` and it will switch to `master`, do a `git pull`, switch back to your branch merging from `master` and accepting the default commit message
* fdfd