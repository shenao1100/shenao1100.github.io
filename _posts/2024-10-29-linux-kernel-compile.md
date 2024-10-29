---
title: Linux kernel compile
description: Examples of text, typography, math equations, diagrams, flowcharts, pictures, videos, and more.
author: NTGshen
date: 2019-08-08 11:33:00 +0800
categories: [Linux]
tags: [Linux, Compile]
pin: true
math: true
mermaid: true
---


# 编译第一个Linux内核

## 准备
- 前往 https://www.kernel.org/ 下载kernel源码
- 完整的Linux环境 `Linux ntg-make-env 5.15.0-91-generic #101~20.04.1-Ubuntu SMP Thu Nov 16 14:22:28 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux`
- 更新软件包

## 1. 安装依赖
安装以下软件包

```sudo apt-get install libncurses5-dev build-essential openssl flex bison libssl-dev libzstd-dev zstd libelf-dev```

## 2.解压源码

使用以下命令解压：

```sudo tar -xvJf linux-x.xx.xxx.tar```

## 3.设置参数

1. 清理

```
make mrproper
make clean
```

2. 进入配置菜单

```
make menuconfig
```

3. 修改并保存为 `.config` 后退出

4. 修改不存在的pem文件

```
vim .config
```

使用 `/` 搜索以下字符串并将值改为空
```
CONFIG_SYSTEM_TRUSTED_KEYS="debian/canonical-certs.pem"
CONFIG_SYSTEM_REVOCATION_KEYS="debian/canonical-revoked-certs.pem"
```
改为
```
CONFIG_SYSTEM_TRUSTED_KEYS=""
CONFIG_SYSTEM_REVOCATION_KEYS=""
```

## 4.编译

使用16线程编译

```
make -j16 bzImage && make -j16 modules && make modules_install && make install
```

## 5.应用新内核

1. 复制内核
```
cp arch/x86/boot/bzImage /boot/
cp /System.map /boot/
```

2. 建立引导内存盘
```
mkinitramfs -o /boot/initrd.img-x.xx.xxx-your-version
```

3. 更新引导
```
update-initramfs -c -k x.xx.xxx-your-version
update-grub2
```

4. 修改启动顺序
```
grep menuentry /boot/grub/grub.cfg
```

若第一项为新的内核，则不需要修改，若不是，使用 `vim /boot/grub/grub.cfg` 将代码块移至前面

5. 应用更改，重启
```
update-grub
reboot
```

重启后需关闭UEFI安全引导，否则会报错签名错误无法进入系统
