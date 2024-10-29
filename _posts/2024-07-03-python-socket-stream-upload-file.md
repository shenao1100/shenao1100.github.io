---
title: Python使用socket完成流式分片上传文件
description: Python使用socket完成流式分片上传文件
author: ShenNya
date: 2024-07-03 02:17:00 +0800
categories: [Python]
tags: [Python, Socket, HTTP, 流式传输]
#pin: true
math: true
mermaid: true
---

# Python使用socket完成流式分片上传文件

## 起因

有一个非常大的文件需要上传到网站，直接全部读取文件会导致内存占用过大、POST请求无法完成等问题，所以想到了类似stream方式上传文件的解决方法

在查找了诸多解决方案后，发现都不适合我的问题，他们大多都使用了MultipartEncoder来解决，但是这会导致上传的data中带有boundary，看上去像这样：

源文件内容：

```
nihao
```

上传文件的内容：

```
--66cc99952f5f4bffbd2c282f3aac4baf
Content-Disposition: form-data; name="file"

nihao
--66cc99952f5f4bffbd2c282f3aac4baf--
```

在查找了很多方案无果后，想到了直接建立socket，这样就可以直观的控制data的读取和上传了

## 解决方案

首先先来看下URL，我这里使用的是PUT方法：

```
http://127.0.0.1/api/fs/put
```

这里我们想要向127.0.0.1这个远程主机建立连接，对/api/fs/put这个资源路径进行操作

所以来建立一个这样的套接字：

```python
def stream_upload(host, path, header:dict, src_path):
    tcp_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 建立TCP套接字
    tcp_socket.connect((host, 80))
    # 连接至远程主机的80端口（没有指定端口就是80）
    body = f'PUT {path} HTTP/1.1\r\n'
    # 这一部分定义了请求的方式（PUT/GET/POST...）和请求的资源路径（传入的path）
```
由于我们有在请求中使用header，所以在后面加上header
```python
    for key in header.keys():
        body += f'{key}:{header[key]}\r\n'
    # 在最后加\r\n，表示header已经加完了
    body += f'\r\n'
```
之后将这一部分发送
```python
    tcp_socket.send(body.encode())
```

然后就来处理data的问题，上传文件时读1024就发送1024，直到把文件读完
```python
     with open(src_path, 'rb') as file_stream:
        #打开文件
        chunk = file_stream.read(1024)
        #先读1024
        while chunk:
            tcp_socket.send(chunk)
            #读到的1024大小的文件就会被发送
            chunk = file_stream.read(1024)
            #如果没读完就接着读，读完会返回None跳出循环
```

收尾(由于返回的body很死，所以偷个懒检测body里有没有成功的200字样)

```python
    result = tcp_socket.recv(8192).decode()
    print(result)
    if '"code":200' in str(result):
        return True
    else:
        return False
```

## 最终代码

```python
def stream_upload(host, param, header:dict, src_path):
    tcp_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    tcp_socket.connect((host, 80))
    body = f'PUT {param} HTTP/1.1\r\n'
    for key in header.keys():

        body += f'{key}:{header[key]}\r\n'
    # end header
    body += f'\r\n'
    tcp_socket.send(body.encode())

    with open(src_path, 'rb') as file_stream:
        chunk = file_stream.read(1024)
        while chunk:
            tcp_socket.send(chunk)
            chunk = file_stream.read(1024)
    # end data
    result = tcp_socket.recv(8192).decode()
    print(result)
    if '"code":200' in str(result):
        return True
    else:
        return False
```

使用(带有header，向http://127.0.0.1/api/fs/put上传./test.zip文件)：
```
header = {
    'Accept': 'application/json, text/plain, */*',
    'Accept-Encoding': 'gzip, deflate',
    'Authorization': '',
}
stream_upload("127.0.0.1", '/api/fs/put', header, "./test.zip")
```
## DONE!

想要实现查看进度只需要在read方面计算已经上传的大小，凌晨两点了就先不写了
