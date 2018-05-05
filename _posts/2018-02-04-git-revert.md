---
layout: post
keywords: git, revert, 撤销已修改的文件
description: how to revert git modified and unstaged file, Git如何撤销已修改的文件
title: Git如何撤销已修改的文件
comments: true
---

{{ page.title }}
<p class="meta">04 Feb 2018</p>
<hr>

```
revert modified file
git checkout -- $(git ls-files -m)

revert unstaged file
git clean -nf
git clean -df
```
