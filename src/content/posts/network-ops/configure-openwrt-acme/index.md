---
title: "OpenWrt ACME 配置"
published: 2025-10-10
description: 更换OpenWrt ACME为Github版本，手动重新配置
category: 系统运维
tags: ["ACME", "OpenWrt"]
draft: false
---

## 前言

今天凌晨登陆Cloud时，发现Firefox跳出了证书警告, 检查证书详情发现刚好昨天到期。奇怪的是之前配置的ACME居然没正常renew, 感觉OpenWrt的ACME有点年久失修了, 打算使用Github的官方方式.

仅作记录，一步一步按照官方文档来很简单，有手就行😋

## 思路

使用ACME的 自动验证(DNS API) 来签发证书.

## 实际操作

### 登陆OpenWrt UI管理界面, 删除acme以及相应的依赖包.

### 按照[Doc](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)一步一步设置

```
# 安装
curl https://get.acme.sh | sh -s email=my@example.com
```

查看是否在对应用户下有了相应文件夹。  
至于alias, 我就不设置了, 毕竟设置一次后cornd能自动renew。

```
ls -al

# Output
drwxr-x--- 6 root     root          4096 Oct 11 02:32 .
drwxr-xr-x   20 root     root          4096 Jul 22 03:36 ..
drwx------ 8 root     root          4096 Oct 11 03:53 .acme.sh
drwxr-xr-x    2 root     root          4096 Jul 20 21:27 frpc
drwxr-xr-x    2 root     root          4096 Jun 28 09:56 nginx
# Output End

# Change Directory
cd .acme.sh/
```

此时OpenWrt WebUI的计划任务下应该已经能看到多了一行

```
# crontab
56 8 * * * "/root/.acme.sh"/acme.sh --cron --home "/root/.acme.sh" > /dev/null
```

### 设置默认的CA证书签发机构--[Doc](https://github.com/acmesh-official/acme.sh/wiki/Server)

```
# 设置为letsencrypt
./acme.sh --set-default-ca --server https://acme-v02.api.letsencrypt.org/directory
```

### 在[DNS API Doc](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)找到自己需要的DNS服务商, 根据文档说明操作即可

由于之前签发过，Cloudflare有对应的API，直接refresh token获取一个新的即可  

```
# CF_Token和CF_Account_ID, example.com替换为自己的即可
# 以下摘自官方文档

# For a single domain or multiple domains within the same Cloudflare account
export CF_Token="zfNp-Xm0VhSaCNun7dkLzwnw0UN7FNjaMurUZ8vf"
export CF_Account_ID="763eac4f1bcebd8b5c95e9fc50d010b4"
./acme.sh --issue --dns dns_cf -d example.com -d '*.example.com'
```

由于我自己有两个域名，国内的是在TencentCloud  
根据[Doc](https://github.com/acmesh-official/acme.sh/wiki/dnsapi2#160-use-tencentcloud-dnspod-api)操作即可

```
# 以下摘自官方文档

export Tencent_SecretId="<Your SecretId>"
export Tencent_SecretKey="<Your SecretKey>"
./acme.sh --issue --dns dns_tencent -d example.com -d *.example.com
```

## 复制证书到对应位置

```
./acme.sh --install-cert -d cedon.org  \
--key-file /etc/nginx/conf.d/key.pem  \
--fullchain-file /etc/nginx/conf.d/cert.pem  \
--reloadcmd "service nginx reload"
```

```
./acme.sh --install-cert -d cedon.cn  \
--key-file /etc/nginx/conf.d/key-cn.pem  \
--fullchain-file /etc/nginx/conf.d/cert-cn.pem  \
--reloadcmd "service nginx reload"
```

## 测试证书是否正常

查看ACME管理下的证书

```
./acme.sh --list
```

登陆 [www.cedon.cn](http://www.cedon.cn) 和 [blog.cedon.org](http://blog.cedon.org) ,打开小锁头查看详情✅
