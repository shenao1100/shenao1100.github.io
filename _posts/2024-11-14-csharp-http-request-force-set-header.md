---
title: C#强制设置Header
description: C# HttpClient强制设置Header
author: ShenNya
date: 2024-11-14 21:20:00 +0800
categories: [CSharp]
tags: [CSharp, HTTP, User-Agent, Header]
#pin: true
math: true
mermaid: true
---

# C#强制设置Header

## 前言

为什么要发这样一篇笔记？

因为我无语了

很早之前我就发现了某软件的UA并不合规，我不确定为什么，但是被卡在UA + 他们返回的错误信息根本不对后，我还是决定写这么一篇笔记
## 解决方案

使用`TryAddWithoutValidation`

这样可以跳过对Header的验证，直接设置对应的Header

```csharp
using (HttpClient client = new HttpClient()){
    //client.DefaultRequestHeaders.Add("User-Agent", UserAgent);
    client.DefaultRequestHeaders.TryAddWithoutValidation("User-Agent", UserAgent);
}
```

## User Agent合法格式

你可以在**Mozilla Web Docs**的[这里](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/User-Agent)找到官方对于UA的格式定义

User-Agent 请求标头是一个特征字符串，使得服务器和对等网络能够识别发出请求的用户代理的应用程序、操作系统、供应商或版本信息

### 通用格式：

```
<product> / <product-version> <comment>
```

| 字段                | 描述                                             |
| ------------------- | ------------------------------------------------ |
| \<product\>         | 产品标识符——其名称或开发代号                     |
| \<product-version\> | 产品版本号                                       |
| \<comment\>         | 零个或多个包含更多细节的注释。例如，子产品的信息 |


Web浏览器的格式为：

```
Mozilla/5.0 (<system-information>) <platform> (<platform-details>) <extensions>
```

示例：
```
Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:47.0) Gecko/20100101 Firefox/47.0
Mozilla/5.0 (Macintosh; Intel Mac OS X x.y; rv:42.0) Gecko/20100101 Firefox/42.0
```


## 某软件的UA

再来看看某软件的UA

```
netdisk;P2SP;3.0.20.63;netdisk;7.46.5.113;PC;PC-Windows;10.0.22631;WindowsXxxxxXxxGuanJia
```

> 兄啊，这啥啊，这啥也不是
{: .prompt-info }
> 兄啊，这啥啊，这啥也不是
{: .prompt-warning }
> 兄啊，这啥啊，这啥也不是
{: .prompt-danger }

HTTP协议是你家自研的？就算是，能不能改成这样？
```
netdisk/3.0.20.63 (Windows NT 10.0.22631; Win64; x64) WindowsXxxxxXxxGuanJia/7.46.5.113 P2SP/3.0.20.63
```
