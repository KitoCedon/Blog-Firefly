---
title: 优化家庭服务器的Frp通信
published: 2026-02-18
decriptrion: 通过修改Frp使用协议，排查丢包问题，优化穿透链路等等提升带宽使用率
category: 网络运维
tags: ["Frp", "Nginx", "OpenWrt", "OpenClash", "TCP", "UDP"]
draft: true
---

## 前言

之前在使用Frp穿透服务器进行MineCraft联机时，在大范围tp或移动时，很容易出现服务器加载延迟和卡顿。  
而Teamspeak服务器也会同时出现语音卡顿和延迟的情况。  
我还一直以为是上行就那样了，直到使用rsync同步文件时，发现只有*14 KB/s*时，这绝对是哪里出问题了。

我是网络小白，中途借助了Claude,GPT等等大模型分析可能性，总算是排查出了一点问题来。Frp穿透也才算是真正有了带宽使用率。

## 问题解决

### 1. 测速

[中国科学技术大学测速网站](https://test.ustc.edu.cn/)先看一下速度  
上行87.3Mbps，下行53.3Mbps。

再通过*iperf3*测试连接服务器的TCP上行和下行和情况(如下)  
内网服务器传输时丢包率有差不多1.5%。对游戏来说延迟和卡顿已经肉眼可见了。

```bash title="内网服务器>>云服务器" "云服务器IP"
❯ iperf3 -c 云服务器IP
Connecting to host 云服务器IP, port 5201
[  5] local 192.168.3.3 port 37378 connected to 云服务器IP port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec  14.4 MBytes   121 Mbits/sec    0    734 KBytes       
[  5]   1.00-2.00   sec  10.9 MBytes  91.2 Mbits/sec    0   1.28 MBytes       
[  5]   2.00-3.00   sec  6.00 MBytes  50.3 Mbits/sec  1486   78.6 KBytes       
[  5]   3.00-4.00   sec  6.75 MBytes  56.6 Mbits/sec   20   86.9 KBytes       
[  5]   4.00-5.00   sec  6.88 MBytes  57.7 Mbits/sec   13   97.9 KBytes       
[  5]   5.00-6.00   sec  6.75 MBytes  56.6 Mbits/sec   24    108 KBytes       
[  5]   6.00-7.00   sec  5.50 MBytes  46.1 Mbits/sec   24   85.5 KBytes       
[  5]   7.00-8.00   sec  6.88 MBytes  57.7 Mbits/sec   18   89.6 KBytes       
[  5]   8.00-9.00   sec  6.88 MBytes  57.7 Mbits/sec   17   99.3 KBytes       
[  5]   9.00-10.00  sec  6.75 MBytes  56.6 Mbits/sec   29   99.3 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  77.6 MBytes  65.1 Mbits/sec  1631            sender
[  5]   0.00-10.02  sec  73.9 MBytes  61.9 Mbits/sec                  receiver

iperf Done.
```

```bash title="云服务器>>内网服务器" "云服务器IP"
iperf3 -R -c 云服务器IP
Connecting to host 云服务器IP, port 5201
Reverse mode, remote host 云服务器IP is sending
[  5] local 192.168.3.3 port 59424 connected to 云服务器IP port 5201
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec  9.25 MBytes  77.6 Mbits/sec                  
[  5]   1.00-2.00   sec  10.6 MBytes  89.1 Mbits/sec                  
[  5]   2.00-3.00   sec  3.25 MBytes  27.3 Mbits/sec                  
[  5]   3.00-4.00   sec  3.62 MBytes  30.4 Mbits/sec                  
[  5]   4.00-5.00   sec  6.00 MBytes  50.3 Mbits/sec                  
[  5]   5.00-6.00   sec  7.50 MBytes  62.9 Mbits/sec                  
[  5]   6.00-7.00   sec  4.25 MBytes  35.7 Mbits/sec                  
[  5]   7.00-8.00   sec  6.38 MBytes  53.5 Mbits/sec                  
[  5]   8.00-9.00   sec  8.62 MBytes  72.4 Mbits/sec                  
[  5]   9.00-10.00  sec  10.0 MBytes  83.8 Mbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.02  sec  71.5 MBytes  59.9 Mbits/sec  300            sender
[  5]   0.00-10.00  sec  69.5 MBytes  58.3 Mbits/sec                  receiver

iperf Done.
```

### 2. 更换Frp的传输协议

#### 测试不同的协议下的传输效率改善

以我rsync TCP协议传输*14 KB/s*的情况来测试(每一次都全量传输)  
- QUIC协议比TCP激进一点，但提升有限(43 KB/s)
- KCP协议在增加10%~20%流量消耗的情况下，在弱网环境能明显改善传输效率(2.4 MB/s)  

#### 给不同服务分别分配不同的Frp协议

把原先的一份Frp启动项复制成三份，分开控制不同服务

- *游戏服务*改用KCP协议
- *Web服务*改用QUIC协议
- *SSH服务*维持TCP协议

## 一些后记

在去年八月改用Frp时我尝试过了QUIC协议和KCP协议，发现怎么都连不上，还以为是云服务器本身有限制。  
这次优化才发现其实是一直没注意对应端口只开了TCP没开UDP(乌龙啊卧槽😭)。


## References

- [KCP Doc](https://github.com/skywind3000/kcp/blob/master/README.md)
- [QUIC Doc](https://docs.quic.cloud)
