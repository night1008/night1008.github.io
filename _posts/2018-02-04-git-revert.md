---
layout: post
keywords: git, revert, 撤销已修改的文件
description: how to revert git modified and unstaged file, Git如何撤销已修改的文件
title: Git如何撤销已修改的文件
comments: true
---

撤销已修改的文件，分三种情况

如果修改还没commit
```
revert modified file
git checkout -- $(git ls-files -m)

revert unstaged file
git clean -nf
git clean -df
```


如果修改已经commit，但还没push到origin，则执行
```
git reset HEAD~
```

如果已经push到origin，则执行
```
git reset --hard <revision_id_of_last_known_good_commit>
git push --force origin develop
```
