---
title: C#多线程下载文件
description: C#多线程并发下载文件
author: ShenNya
date: 2024-11-14 14:11:00 +0800
categories: [CSharp]
tags: [CSharp, 多线程, Thread, 并发, 下载]
pin: true
math: true
mermaid: true
---

# C#多线程下载文件

## 前言

> 这段就算是简单多线程的实战了
> 
> 如果需要多线程入门，点[这里](/posts/csharp-multi-thread/)

日常中经常会遇到各种下载慢的情况，很多软件可以解决这个问题，如IDM、FDM、aria2c，甚至是某MC启动器PCL

本质上他们都是多线程下载，其主要思想是将文件**切片**

假设我们有大小为255byte的文件，一个人下就是从0下载直到255结束

这个时候，如果将文件分为多片，建立多个线程同时下载，就相当于有多个人在帮你下载同一个文件了，也就实现了加速（BT种子也是这样，也许以后数学好了我也会发一篇这个的笔记）



## 代码及用例
这里先贴代码：

从以下URL下载文件：
```
http://192.168.10.199/p/Local%20500G%20%E5%91%9C%E5%91%9C/SurfacePro3_Win10_18362_1902002_0.msi?sign=hwUl4yfX8aRrmAb-U0oqcyELDWUY_XxPzlFHXQuVjUk=:0
```
使用了浏览器的UA：
```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36 Edg/130.0.0.0
```
保存到`a.msi`

```csharp
using System;
using System.IO;
using System.Net.Http;
using System.Threading.Tasks;
using System.Threading;

public class Range
{
    public long from { get; set; }
    public long to { get; set; }
}

public class Downloader
{
    private readonly object _lock = new object();
    private FileStream FileStream {  get; set; }
    private int TotalThread {  get; set; }
    private int CurrentThread { get; set; }
    private string Url { get; set; }
    private string UserAgent { get; set; }

    private long downloadSize = 0;
    private int downloadProgress = 0;
    private long downloaded = 0;
    private long totalSize = 0;
    public Downloader(string url, string user_agent, string file_path, int thread)
    {
        TotalThread = thread;
        CurrentThread = 0;
        Url = url;
        UserAgent = user_agent;
        FileStream = new FileStream(file_path, FileMode.OpenOrCreate, FileAccess.ReadWrite);
    }
    public async Task<long> GetInfo()
    {
        using (HttpClient client = new HttpClient()){
            client.DefaultRequestHeaders.Add("User-Agent", UserAgent);
            using (HttpResponseMessage response = await client.GetAsync(Url))
            {
                response.EnsureSuccessStatusCode();

                totalSize = response.Content.Headers.ContentLength ?? 0;
                FileStream.SetLength(totalSize);
                Console.WriteLine(totalSize.ToString());
                return totalSize;
            }
        }
        return 0;
    }
    public List<Range> CalcRange()
    {
        var range = new List<Range>();
        long start = 0;
        if (totalSize > 0)
        {
            long average_size = totalSize / TotalThread;
            for (int i = 0; i < TotalThread; i++)
            {
                if (i == TotalThread - 1)
                {
                    range.Add(new Range
                    {
                        from = start,
                        to = totalSize
                    });
                }
                else
                {
                    range.Add(new Range
                    {
                        from = start,
                        to = start + average_size
                    });
                    start += average_size;
                }
            }
        }
        return range;
    }
    public async Task DownloadThread(Range range)
    {

        long pointer = range.from;
        using (HttpClient client = new HttpClient())
        {
            client.DefaultRequestHeaders.Add("Range", $"bytes={range.from}-{range.to}");
            client.DefaultRequestHeaders.Add("User-Agent", UserAgent);
            using (HttpResponseMessage response = await client.GetAsync(Url))
            {
                response.EnsureSuccessStatusCode();

                long? totalSize = response.Content.Headers.ContentLength;
                using (Stream contentStream = await response.Content.ReadAsStreamAsync())
                {
                    byte[] buffer = new byte[8192];
                    int bytesRead;
                    while ((bytesRead = await contentStream.ReadAsync(buffer, 0, buffer.Length)) > 0)
                    {
                        //await write 
                        downloaded += bytesRead;
                        lock (_lock)
                        {
                            Console.WriteLine("WriteTo: " + pointer.ToString() + " By: " + range.from.ToString());

                            FileStream.Seek(pointer, SeekOrigin.Begin);
                            FileStream.Write(buffer, 0, bytesRead);
                        }
                        pointer += bytesRead;
                        
                    }
                }
            }
        }
    }
    public void releaseFile()
    {
        Console.WriteLine("File has been relese");
        lock (_lock)
        {
            FileStream.Close();
        }
    }
    public async void Run()
    {
        long size = await GetInfo();
        Console.WriteLine("Getinfo complete, size: " + totalSize.ToString());
        //CountdownEvent countDown = new CountdownEvent(TotalThread);
        List<Task> tasks = new List<Task>();
        foreach (Range range in CalcRange())
        {
            Console.WriteLine("Thread start: " + range.from.ToString());
            tasks.Add(DownloadThread(range));
        }
        await Task.WhenAll(tasks);
        //countDown.Wait();
        releaseFile();
    }
}

class Program
{
    static void Main(string[] args)
    {
        Downloader download = new Downloader("http://192.168.10.199/p/Local%20500G%20%E5%91%9C%E5%91%9C/SurfacePro3_Win10_18362_1902002_0.msi?sign=hwUl4yfX8aRrmAb-U0oqcyELDWUY_XxPzlFHXQuVjUk=:0", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36 Edg/130.0.0.0", "a.msi", 32);
        download.Run();
        Console.ReadLine();
    }
}
```
## 明确需求

刚刚说到多线程下载文件的核心是切片，那么就引申出了几个问题：

- 如何提前预知文件的大小
- 如何计算每个分片的大小并让服务器返回对应分片
- 如何将这些分片整合


## 预知大小

在HTTP传输中，正文会包含一个拥有许多信息的`头（Header）`，头里面一般会有一个叫`Content-Length`的值，这就是body的大小，也就是我们文件的大小

打开你的浏览器，按下`F12`打开开发者工具，选择网络，刷新一下网页，就可以看到`Response Headers`中包含有`Content-Length`

![alt text](../imgs/2024-11-14-csharp-multi-thread-download-file/image-1.png)

这个数值就是这个页面的大小

可以使用以下代码获取：
```csharp
long totalSize;
using (HttpClient client = new HttpClient()){
    using (HttpResponseMessage response = await client.GetAsync(Url))
    {
        response.EnsureSuccessStatusCode();
        totalSize = response.Content.Headers.ContentLength ?? 0;
        FileStream.SetLength(totalSize);
        Console.WriteLine("文件大小为: " + totalSize.ToString());
        return totalSize;
    }
}
return 0;
```
> 由于文件一般很大，文件大小最好用`long`存储
>
> 如果无法获取，有可能是这个页面不支持`Content-Length`或是未经授权，可以尝试`client.DefaultRequestHeaders.Add`语句添加`User-Agent`和`Host`
{: .prompt-info }

## 计算切片大小

这里我们简化以下，假设有255长度的文件，为了方便表示我们把他切成不等的两片：

![alt text](../imgs/2024-11-14-csharp-multi-thread-download-file/image.png)

可以看到第一段的范围为`0-63`

第二段范围为`64-255`

将这两段合并即为整个文件

这里的规则是：第一段以`0`为起点，`n`为终点，后一段以`n+1`为起点`m`为终点，以此类推，直到终点为文件总长度

---

既然要使用多线程，那么就要定义线程数，每个线程下载对应的部分，这里使用均分的方法：


```csharp

public class Range
{
    public long from { get; set; }
    public long to { get; set; }
}

private long totalSize = 0;           // 在这里定义文件大小
private int TotalThread {  get; set; }// 在这里定义线程数

public List<Range> CalcRange()
{
  var range = new List<Range>();
  long start = 0;
  if (totalSize > 0)
  {
      long average_size = totalSize / TotalThread;
      for (int i = 0; i < TotalThread; i++)
      {
          if (i == TotalThread - 1) // 最后一段要把剩余所有没下的都要下上
          {
              range.Add(new Range
              {
                  from = start,
                  to = totalSize
              });
          }
          else                    // 之前的直接均分
          {
              range.Add(new Range
              {
                  from = start,
                  to = start + average_size
              });
              start += average_size;
          }
      }
  }
  return range;
}
```

## 让服务器返回对应的文件块

和`Response Headers`中含有文件大小的`Content-Length`一样，你也可以在`Request Headers`中定义想要文件的哪一部分

具体操作为，在`Request Headers`中定义：
```
Range: bytes=0-63
```
使用C#：
```csharp
Range range = new Range{from=0, to=63};
using (HttpClient client = new HttpClient())
{
  client.DefaultRequestHeaders.Add("Range", $"bytes={range.from}-{range.to}");
  using (HttpResponseMessage response = await client.GetAsync(Url))
  {
      response.EnsureSuccessStatusCode();
      ...
  }
}
```

这里我们使用**流式传输 (Stream)**的方式下载文件块

如果不使用流式传输，那么整个请求会进入*假死*状态，直到下载完毕整个文件块再执行后面的代码

而使用了流式传输，我们就可以接收一点文件（如8192字节的文件）后去完成些别的任务（如计算下载进度、下载速率等），之后重复这一步骤直到文件块接收完毕

代码如下：
```csharp


public async Task DownloadThread()
{
  Range range = new Range{from=0, to=63}; // 使用之前写的Range
  string UserAgent = "....."  //这里为了通过验证填入UA
  long pointer = range.from;              
  using (HttpClient client = new HttpClient())
  {
      client.DefaultRequestHeaders.Add("Range", $"bytes={range.from}-{range.to}");//在Request Headers中定义下载范围
      client.DefaultRequestHeaders.Add("User-Agent", UserAgent);          //设置UA
      using (HttpResponseMessage response = await client.GetAsync(Url))
      {
          response.EnsureSuccessStatusCode(); //确保状态码无误
          long? totalSize = response.Content.Headers.ContentLength;             //获取这部分文件块的大小
          using (Stream contentStream = await response.Content.ReadAsStreamAsync())   //使用流式传输
          {
              byte[] buffer = new byte[8192];   // 新建buffer，用来存储流式传输每次下载的部分文件
              int bytesRead;
              while ((bytesRead = await contentStream.ReadAsync(buffer, 0, buffer.Length)) > 0) //获取最多8192字节的内容，获取到的内容长度存入bytesRead
              {
                  downloaded += bytesRead;    //已经下载完毕的大小，用来计算下载进度
                  lock (_lock)      //使用线程锁，之后会提到
                  {
                      Console.WriteLine("WriteTo: " + pointer.ToString() + " By: " + range.from.ToString());
                      FileStream.Seek(pointer, SeekOrigin.Begin);   // 将文件指针移动到需要写入数据的地方
                      FileStream.Write(buffer, 0, bytesRead);       // 将获取到的那点数据写入
                  }
                  pointer += bytesRead;
                  
              }
          }
      }
  }
}
```

## 多线程管理

由于这是个典型的**I/O密集型操作**，我们使用并发的方式进行处理

这里的想法是：
1. 向服务器获取文件大小
2. 创建一个文件实例，让所有线程都使用这一文件实例进行文件写入操作
3. 计算每个线程下载的文件块大小
4. 建立多个线程同时下载文件，使用线程池自动管理
5. 等待所有文件块下载完毕，此时所有下载线程关闭，释放文件块

这里使用`Task`来对线程池进行自动管理

代码如下：
```csharp
public async void Run()
{
    long size = await GetInfo();      // 获取文件大小，获取完毕进入下一步
    Console.WriteLine("Getinfo complete, size: " + totalSize.ToString());

    // 你也可以使用倒计时器countDown对线程是否全部关闭做出判断，但是Task更加简单好用
    //CountdownEvent countDown = new CountdownEvent(TotalThread);   

    List<Task> tasks = new List<Task>();      // 这个列表用来存储、等待 所有的下载线程
    foreach (Range range in CalcRange())      // 计算下载大小并拉起线程进行下载
    {
        Console.WriteLine("Thread start: " + range.from.ToString());
        tasks.Add(DownloadThread(range));     // 将线程加入列表
    }
    await Task.WhenAll(tasks);                // 等待所有线程下载完毕后退出
    //countDown.Wait();
    releaseFile();                            // 释放文件资源
}
```

## 改进

这里只是一个简单的示例，你还可以改进这段代码，增加速度、进度的计算，下载时的程序回调

使用**线程阻塞**实现暂停，**取消线程**实现取消下载文件等等
