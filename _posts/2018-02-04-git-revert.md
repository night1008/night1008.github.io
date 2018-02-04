---
layout: post
keywords: git,revert
description: how to revert git modified and unstaged file
title: how to revert git modified and unstaged file
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
