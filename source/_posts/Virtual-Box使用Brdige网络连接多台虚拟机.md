---
title: Virtual Box使用Brdige网络连接多台虚拟机
date: 2019-03-27 10:52:32
tags:
---

1. Settings > Network, Attach To "Bridge Adapter", Name "eth0"
2. Set ip address for each VM.
```
ip address add 192.168.0.1/24 dev enp0s3
```
3. Set ip address for Host, which is in the same sub network with VMs
```
ip address add 192.168.0.254/24 dev en0
```