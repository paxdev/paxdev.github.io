---
  title: Working with Git Remote Origin
  categories:
    - git
  tags:
    - git
---

To list the current remote origin use

```
git remote show origin
```

You can easily change the remote origin by using

```
git remote set-url origin {your-git-url}
```

Note that cloning a repo does not necessarily set the remote and you may need to do it manually.

When you do a `git push` you will push the whole repo to the new origin maintaining the full history.