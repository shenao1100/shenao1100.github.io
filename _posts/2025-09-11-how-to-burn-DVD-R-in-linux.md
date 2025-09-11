---
title: 如何在Linux中烧录DVD-R
description: 如题
author: ShenNya
date: 2025-09-11 23:14:00 +0800
categories: [Linux]
tags: [Linux, DVD]
---

# 确定设备

通常是`srX`, 这里是`sr0`

# 工具

安装

```bash
sudo apt-get install dvd+rw-tools mkisofs
```
# 打包文件

```bash
mkisofs -V "disc_name" -J -r -o output.iso /data
```

# 刻录

> proxmox挂载的光驱是只读格式，除非你可以透传整个SATA控制器否则请在物理机上执行

```bash
growisofs -dvd-compat -Z /dev/sr0=./out.iso
```
信息如下
```
Executing 'builtin_dd if=./out.iso of=/dev/sr0 obs=32k seek=0'
/dev/sr0: "Current Write Speed" is 8.2x1352KBps.
     294912/15802368 ( 1.9%) @0.0x, remaining 4:22 RBU  46.3% UBU   0.0%
    9175040/15802368 (58.1%) @1.9x, remaining 0:06 RBU  19.8% UBU  99.2%
builtin_dd: 7728*2KB out @ average 1.1x1352KBps
/dev/sr0: flushing cache
/dev/sr0: updating RMA
/dev/sr0: closing disc
/dev/sr0: reloading tray
```
刻完了

## 刻录失败

尝试降低速度：

```bash
growisofs -speed=4 -dvd-compat -Z /dev/sr0=./out.iso
```

或者换张盘

