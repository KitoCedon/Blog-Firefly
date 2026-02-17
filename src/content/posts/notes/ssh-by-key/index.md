---
title: 使用密钥连接SSH
published: 2025-11-07
description: 创建密钥,复制密钥,信任公钥,测试连接
category: 学习笔记 
tags: ["SSH"]
draft: false
---

## 1. 使用SSH连接到服务器的`对应用户`创建一对密钥(公钥和私钥)

```bash "-C <可选备注>"
ssh-keygen -C <可选备注>
```

一路回车，默认的ed25519加密就行。  
然后把公钥加入信任

```bash
cat id_ed25519.pub >> authorized_keys
```

## 2. 复制私钥到连接的客户端上

使用SFTP，物理拷贝等等都可以。
```bash title="使用SFTP" "服务器端口" "用户名@服务器IP(域名)"
sftp -P 服务器端口 用户名@服务器IP(域名)
sftp> lpwd # 确认本地接收目录
sftp> get ~/.ssh/id_ed25519
```

## 测试连接

```bash "服务器端口" "用户名@服务器IP(域名)"
ssh -vi id_ed25519 -p 服务器端口 用户名@服务器IP(域名)
```
