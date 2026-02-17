---
title: 使用HTTPS端口连接Github Repo
published: 2026-02-16
description: 软路由透明代理使用Fake-IP模式，使用HTTPS连接Github Repo
category: 实用技巧
tags: ["github", "fake-ip", "ssh"]
draft: false
---

## 前言

回家后使用软路由Fake-IP走的代理，发现git push时ssh端口无法使用(被软路由接管了)，如果把github加入直连规则，又会导致网页连接巨卡无比。

## 解决方法

**使用github提供的443端口连接。**

先测试一下443端口是否可行。  
如果出现`Hi USERNAME! You've successfully authenticated, but GitHub does not`就没问题。

```zsh
ssh -T -p 443 git@ssh.github.com

```

把443端口改为默认设置

```bash
# ~/.ssh/config

Host github.com
    Hostname ssh.github.com
    Port 443
    User git
```

最后去掉端口号测试一下

```zsh
ssh -T git@ssh.github.com

```

## Refenrences

- [Github Docs](https://docs.github.com/en/authentication/troubleshooting-ssh/using-ssh-over-the-https-port)
