---
title: Linux 常用操作
date: 2019-04-22 14:49:19
tags:
    - Linux
    - DevOps
category:
    - DevOps
---

## 时区操作

### 修改时区

```
sudo unlink /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

使用`ls /usr/share/zoneinfo`中查看所有支持的时区

> 参考 https://unix.stackexchange.com/questions/110522/timezone-setting-in-linux