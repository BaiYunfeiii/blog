---
title: Docker简单网络配置
date: 2019-03-26 10:58:51
tags:
---

## 总览
1. Host Network
2. Brigde Network
3. Overlay Network

## Host Network
与主机共享网络资源，与其他采用Host Network的机器存在资源竞争

PS：Host可以绑定多个IP

## Bridge
连接到同一个Bridge的容器，可以通过容器名访问彼此

Default Bridge 与 User-defined Bridge 的区别

## Overlay
主要与Docker swarm结合使用，用于多个Host上的容器互相通信
