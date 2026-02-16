---
title: "手动引导ArchLinux并修复GRUB"
published: 2025-10-06
description: 手动引导ArchLinux并修复GRUB
category: 系统运维
tags: ["grub", "linux", "arch linux", "btrfs"]
draft: false
---

## 前言

起因是宿舍突然断电了, 来电后启动发现直接进入了GRUB控制台。  
推测是断电导致的GRUB分区故障, 着手修复一下。

我的系统为ArchLinux，文件系统为Btrfs，boot为单独一个分区，Btrfs 子卷结构为：`@`、`@home`、`timeshift-btrfs`.  
接下来的实际操作不一定适用于所有文件系统，请备好🧠，有取舍地阅读。

## 实际操作

### 设置GRUB pager

```bash
set pager=1
```

### 查看硬盘以及分区

```bash
# 查看硬盘以及分区
ls

# Output 输出可能有很多, 这里只给出大概输出样例
# e.g.
(hd0) (hd0,gpt1) (hd0,gpt2) (hd2) (hd2,gpt1) (hd2,gpt2) (hd3) (hd3,gpt1) (hd3,gpt2) (hd3,gpt3)
```

#### 继续查看分区以确定boot分区位置

```bash
# 查看分区内容
# etc.
ls (hd0,gpt1)/

# 最后确定boot分区在(hd3,gpt3)

# Output 样例输出中应有类似
vmlinuz-linux initramfs-linux.img # etc.
```

#### 再使用 set 确认 GRUB 的 root与prefix

```bash
# 查看GRUB参数
set

# 如果需要修改
# 设置需要的参数
set root=(hd3,gpt1)
set prefix=(hd3,gpt1)/grub
```

```bash
#Output
...
root='(hd3,gpt1)'
prefix='(hd3,gpt1)/grub'
...
```

#### 最后加载内核与初始文件系统

因为是Btrfs文件系统，需要指定子卷 bootflags=subvol=@

```bash
# 可以在grub.cfg中翻一翻, 很容易就能找到需要的UUID
cat /grub/grub.cfg
```

```bash
# 加载Kernel内核
# root为自己的系统根分区
linux /vmlinuz-linux root=/dev/nvme2n1p3 rw rootflags=subvol=@

# 当然也可以使用UUID的方式, UUID是唯一的.
linux /vmlinuz-linux root=UUID=根分区的UUID rw rootflags=subvol=@

# 加载初始文件系统
initrd /initramfs-linux.img

# 引导
boot
```

### 重新设置GRUB引导

进入操作系统后就是重新设置引导

```zsh
# 安装引导
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch
# 生成配置文件
grub-mkconfig -o /boot/grub/grub.cfg
```

但在生成配置文件阶段遇到报错：只读的文件系统  
尝试重新挂载后仍会报错。  
看来需要修复一下boot分区

```zsh
# 查看boot分区
mount | grep boot

# Output
# 发现为 read only
/dev/nvme2n1p1 on /boot type vfat (ro,...,errors=remount-ro)
```

#### 修复boot分区

```zsh
# 需要 dosfstools
sudo pacman -S dosfstools

# 确认boot分区位置
lsblk

# 检查并修复boot分区
# 覆盖原有备份, 按照引导来修复就好
# /dev/...记得替换为自己的boot分区
sudo fsck.vfat -v /dev/nvme2n1p1
```

修复完后即可[生成GRUB配置文件](/posts/system-ops/manually-boot-archlinux-fix-grub/#重新设置grub引导)  
最后 reboot 测试一下是否正常引导  

```zsh
reboot
```
