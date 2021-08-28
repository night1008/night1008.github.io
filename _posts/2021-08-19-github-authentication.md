---
layout: post
keywords: github, authentication
description: 修改github认证方式
title: 修改github认证方式
comments: true
---

晚上想推代码到 github，发现报了如下错误，

```
remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
remote: Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ for more information.
fatal: 无法访问 'https://github.com/night1008/diary.git/'：The requested URL returned error: 403
```

原来是用账号密码进行推送的，但 github 调整了策略，相关信息可见 [Token authentication requirements for Git operations](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/)

感觉使用 personal access token 需要定期更换，比较麻烦，所以采用 ssh 的方式。

生成 SSH 密钥，具体可看 [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

```
ssh-keygen -t ed25519 -C "your_email@example.com"

eval "$(ssh-agent -s)"
```

```
open ~/.ssh/config

添加以下内容

Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

修改代码库 remote url 为 ssh 形式，

```
git remote set-url origin git@github.com:night1008/diary.git

git remote -v

git push
```