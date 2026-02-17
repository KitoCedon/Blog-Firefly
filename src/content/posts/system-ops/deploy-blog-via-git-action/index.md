---
title: 通过GitAction自动部署Blog到家庭服务器
published: 2026-02-17
description: 设置GitAction部署Blog到家庭服务器
category: 系统运维
tags: ["Github", "Blog"]
draft: false
---

## 概述

服务器网络拓扑图大概为:  
`云服务器-frps` -> `PVE-OpenWrt-frpc` -> `PVE-OpenWrt-nginx` -> `PVE-Ubuntu`

---

## 具体操作

### 复制一份并修改[Firefly主题自带的deploy.yml](https://github.com/CuteLeaf/Firefly/blob/master/.github/workflows/deploy.yml)

```diff lang="yml" title=".github/workflows/rsync-deploy.yml"
- name: Deploy to Pages Branch
+ name: Deploy to Home Server

on:
  # 每次推送到 `master` 分支时触发这个"工作流程"
  # 如果你使用了别的分支名，请按需将 `master` 替换成你的分支名
  push:
    branches: [ master ]
  # 允许你在 GitHub 上的 Actions 标签中手动触发此"工作流程"
  workflow_dispatch:

# 需要写入权限来推送到pages分支
permissions:
  contents: write

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout your repository using git
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Setup pnpm
        uses: pnpm/action-setup@v4
        with:
          version: 9.14.4
          run_install: false
          
      - name: Install dependencies
        run: pnpm install --no-frozen-lockfile
      
      - name: Build site
        run: pnpm run build

      - name: Create .nojekyll file
        run: touch dist/.nojekyll
      
-      - name: Deploy to pages branch
-        uses: JamesIves/github-pages-deploy-action@v4
-        with:
-          branch: pages # 部署到pages分支
-          folder: dist # Astro默认构建输出目录
-          clean: true # 清理目标分支中的旧文件
+      - name: Deploy to home server
+        uses: burnett01/rsync-deployments@v8
+        with:
+          switches: -avzr --delete
+          path: dist/
+          remote_path: ${{ secrets.REMOTE_PATH }} # ex: /var/www/html/
+          remote_host: ${{ secrets.REMOTE_HOST }} # ex: example.com
+          remote_port: ${{ secrets.REMOTE_PORT }} # ex: 22
+          remote_user: ${{ secrets.REMOTE_USER }} # ex: ubuntu
+          remote_key: ${{ secrets.REMOTE_PRIVATE_KEY }}
```

### 配置Ubuntu

#### 1. 创建一个受限的新用户专门用于部署

```bash
su root #切换为root
adduser --comment "Blog-Deployer" blogdeploy  

# output
info: Adding user `blogdeploy' ...
info: Selecting UID/GID from range 1000 to 59999 ...
info: Adding new group `blogdeploy' (1001) ...
info: Adding new user `blogdeploy' (1001) with group `blogdeploy (1001)' ...
info: Creating home directory `/home/blogdeploy' ...
info: Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
info: Adding new user `blogdeploy' to supplemental / extra groups `users' ...
info: Adding user `blogdeploy' to group `users' ...
```
#### 2. 创建blog专用的目录

```diff lang="bash" {"检查权限是否正确rwxr-xr-x": 3-5}
mkdir -p /var/www/blog
chown -chR blogdeploy:blogdeploy /var/www/blog

ls -l /var/www
chmod +x /var/www/blog
```

#### 3. SSH密码登录`对应用户`创建密钥



```bash
ssh-keygen -C "blog-git-action"

# output
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/blogdeploy/.ssh/id_ed25519): 
Created directory '/home/blogdeploy/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/blogdeploy/.ssh/id_ed25519
Your public key has been saved in /home/blogdeploy/.ssh/id_ed25519.pub
```

> [!TIP] 按需锁定密码登录
> ```bash
> passwd -l blogdeploy
> ```

## 设置Github Repository secrets

- `REMOTE_PATH` 部署目录
- `REMOTE_HOST` 连接主机
- `REMOTE_PORT` 连接端口
- `REMOTE_USER` 连接用户
- `REMOTE_PRIVATE_KEY` 连接私钥

## 测试

尝试手动启动一下workflow，看看是否能够部署

## References
- <https://github.com/marketplace/actions/rsync-deployments-action>
- [Github Doc #use-secrets](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets)
