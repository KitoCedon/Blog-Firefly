---
title: "ImmortalWrt-Wifi功能配置"
published: 2026-02-12
description: 配置 ImmortalWrt Wifi AP
category: 网络运维
tags: ["MT7922", "OpenWrt"]
draft: false
---

## 前言

寒假把台式机带回了家里,接入了上游PVE虚拟机下的软路由,但由于PVE的物理主机没有无线功能,导致无法使用LocalSend或串流。我自己房间也刚好缺个无线路由,直接就干了。

## 具体操作

### 选择硬件

参考[References](/posts/network-ops/configure-immortalwrt-wifi/#references)中的#wi-fi-cards-and-the-ap-mode  
我选择了MT7922(Mini PCI-e转PCI,蓝牙和Wifi需要分别接)
> [!WARNING] WARNING
> MT7922作为笔记本的批发卡, AP性能较差, 只建议连接设备在三个及以下时使用。
> 如果你有更多的连接需求，可以考虑QCA9882。

### 安装硬件,确认宿主机检测到

```bash
lspci -nn
```

### 设置PVE直通

确认被ImmortalWrt VM正确检测到

```bash
opkg update && opkg install pciutils
lspci -nn
dmesg | tail -n 50
dmesg | grep wifi
```

有以下类似内容那就对了  
`00:10.0 Network controller [0280]: MEDIATEK Corp. MT7922 802.11ax PCI Express Wireless Network Adapter [14c3:0616]`

可能会有IOMMU(确保这个无线网卡PCI在单独分组)和VFIO问题(屏蔽宿主机驱动,我没遇到)  
IOMMU在Add Device时注意一下就好,VFIO在Add Device时应该也是自动设置了  
勾选**All Function**和**ROM-bar**(如果OpenWrt不稳定可以尝试关闭**ROM-bar**)  
_如果需要直通蓝牙功能则还需要添加USB Device._

### 安装对应固件和内核模块(重启),以及需要的一些opkg包

```bash
opkg update
opkg list | grep mt7922
opkg list | grep mt7921
opkg install kmod-mt7921e kmod-mt7922-firmware mt7922bt-firmware wireless-tools luci-i18n-wifischedule-zh-cn
modprobe kmod-mt7921e
wifi config
reboot
```

MT7922和MT7921都是同一个内核模块(kmod-mt7921e),但固件是不同的,别装错了

### 进入WebUI设置Wifi

设置SSID和密钥,模式,带宽,信道  
以下是我自己的设置

- 模式: AX

- 带宽: 5Ghz (我只在自己房间内使用,2.4Ghz穿透性更好)

- 通道宽度: 80Mhz (2.4G一般用20Mhz)

- 信道: Auto (也可以扫描一下附近邻居的信道情况手动设置)

- 网络: LAN

- 加密: WPA2-PSK

- 密钥: 你自己设

### 添加延迟启动wifi脚本

重启ImmortalWrt VM时发现就算启用了Wifi也不会自动打开(每次重启后都需要手动启用)  
把dmesg给GPT看了一下,推测是在VM下,ImmortalWrt的MT7922驱动加载较慢,导致netifd无法启用  
加个启动脚本延迟重新启动一下wifi即可.

```bash
#!/bin/sh /etc/rc.common
# /etc/init.d/wifi_fix

START=99

start() {
    sleep 15
    wifi reload
    wifi up
}
```

```bash
chmod +x /etc/init.d/wifi_fix
/etc/init.d/wifi_fix enable
```

### Enjoy!

## References

- [OpenWrt Fourm #wi-fi-cards-and-the-ap-mode](https://forum.openwrt.org/t/solved-wi-fi-cards-and-the-ap-mode/151462)

- [OpenWrt Forum #how-to-enable-wifi-in-openwrt-and-does-luci-support-wifi](https://forum.openwrt.org/t/how-to-enable-wifi-in-openwrt-and-does-luci-support-wifi/154084/8)

- [OpenWrt docs #basic\_wifi](https://openwrt.org/zh/docs/guide-quick-start/basic_wifi)

- [OpenWrt docs #network-wifi-basic](https://openwrt.org/zh/docs/guide-user/network/wifi/basic)
