---
title: Nginx对单页面网页的反代问题
description: Nginx对单页面网页的反代可能引起的404错误
author: ShenNya
date: 2025-10-12 23:45:00 +0800
categories: [前端]
tags: [Nginx, 前端]
---

## 描述

最近在写学校社团的网站，网站时单页应用

在一次分享链接时发现只要不是从根路径`https://example.com/`进入网站，直接点击链接`https://example.com/somePath`时Nginx会报错`404`

## 原因与解决方法

Nginx原配置如下：

```
server {
        listen       8924;
        listen       [::]:8924;
        server_name  _;
        root         /var/www/ita;
        index        index.html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
 
        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```


Nginx在反代的过程中一旦碰到了带有Path的情况，默认会从对应的目录加载对应的html或index文件

而我的单页应用则是所有内容都由主页交给`vue`的`router`处理，主页负责解析url加载对应的页面

当用户访问`/somePath`时，我的对应路径并没有文件，所以会返回404

此时只需要加入一行找不到文件自动退回到主页`/index.html`即可，`router`会自动处理需要呈现的页面

于是加入

```
location / {
                try_files $uri $uri/ /index.html;
        }
```

变成

```
server {
        listen       8924;
        listen       [::]:8924;
        server_name  _;
        root         /var/www/ita;
        index        index.html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
                try_files $uri $uri/ /index.html;
        }

 
        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

问题解决

