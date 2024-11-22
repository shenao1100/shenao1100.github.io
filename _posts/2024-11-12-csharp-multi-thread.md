---
title: C#多线程入门
description: C#多线程常用操作
author: ShenNya
date: 2024-11-13 17:01:00 +0800
categories: [CSharp]
tags: [CSharp, 多线程, Thread, 并发, 并行]
pin: true
math: true
mermaid: true
---

# C#多线程入门

## 前言
> 注：Python3.13去掉了Gil，已经拥有了真正的多线程

在使用Python写了几年 **虚假的** 多线程之后，想使用C#进行真正的多线程操作

问题的起因是遇到了高并发场景，同时需要控制线程对唯一文件IO进行操作，于是就有了这篇文章

如果你想深入了解更多的线程操作和同步机制，请点[这里](/posts/csharp-multi-thread-advance/)

后期应该会写一个实战，~~但不是现在 [Soon(TM)]~~ 已经[写完了](/posts/csharp-multi-thread-download-file/)


## 最简单的Task

```csharp
using System;
using System.Threading.Tasks;

class Program
{
  static void Main()
  {
    Task.Run(() => DoWork(1));
    Task.Run(() => DoWork(2));
  }
  
  static void DoWork(int id)
  {
    Console.WriteLine($"{id} is started");
    Thread.Sleep(1000);
    Console.WriteLine($"{id} is done");
  }
}
```
## 异步

常用于UI交互，处理长时任务时不会造成线程假死

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        Console.WriteLine("开始下载...");
        await DownloadDataAsync();
        Console.WriteLine("下载完成");
    }

    static async Task DownloadDataAsync()
    {
        using (HttpClient client = new HttpClient())
        {
            string data = await client.GetStringAsync("https://example.com"); // 阻塞等待下载完成
            Console.WriteLine(data);
        }
    }
}

```
## 并发

强调任务交替执行，没有控制任务在哪个核心上执行，也不一定所有任务都会同时执行

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

class Program
{
    static async Task Main(string[] args)
    {
        List<Task> tasks = new List<Task>();

        for (int i = 1; i <= 5; i++)
        {
            int taskId = i;
            tasks.Add(Task.Run(() => DoWorkAsync(taskId)));
        }

        await Task.WhenAll(tasks);

        Console.WriteLine("所有任务已完成！");
    }

    static async Task DoWorkAsync(int id)
    {
        Console.WriteLine($"任务{id}开始");
        await Task.Delay(1000 * id);  // 模拟异步操作
        Console.WriteLine($"任务{id}完成");
    }
}
```
是不是感觉并发和并行在代码上很相似？你可以在[这里](#并行并发异步的区别)找到区别

## 并行

充分利用多核CPU，确保多个任务能够同时执行

> 注：Parallel本身是基于线程池的执行工具 (基于ThreadPool调度管理线程) ，你可以在[这里](#线程池)找到有关线程池的信息


```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("并行计算开始");

        // 使用 Parallel.For 并行计算
        Parallel.For(1, 11, i =>
        {
            int result = i * i;
            Console.WriteLine($"数字 {i} 的平方是: {result} (线程 {Task.CurrentId})");
        });

        Console.WriteLine("并行计算结束");
    }
}

```

是不是感觉并发和并行在代码上很相似？你可以在[这里](#并行并发异步的区别)找到区别

## 并行、并发、异步的区别

> 这里把异步一起提到是因为很多人接触图形界面编程时更熟悉异步，而并不了解他

### 区别

**并行**

多个任务真正同时执行。并行通常需要多核处理器或多台计算机。每个任务运行在独立的核心或处理器上 

**并发**

多个任务在同一时间段内交替执行。例如，仅单核处理但可以在不同任务之间快速切换，给人一种“同时进行”的感觉。

**异步**

异步是一种*编程模型*，指的是任务不需要等待其他任务完成就可以继续执行。异步任务通常会在等待某个操作（如 I/O 操作或网络请求）完成时释放控制权，让程序可以去做其他事情，等到操作完成后再继续

---

某种程度上说，异步是并发的一种实现方式，我认为原因如下：
- 异步是一种并发模型
- 异步本质上是一种非阻塞的任务处理
- 并发强调交替进行，而并行则是同时进行，异步是通过非阻塞的方式实现并发的
- 并发是一种任务管理思想，可以通过多线程、多进程、协程等方式来实现，而异步是其中一种方式

因此 **并发 ≠ 并行**，而 **异步 ⊂ 并发**

由此我们可以得到下面的表格 (理论上异步不应该在表格里)：

| 方式 | 类型           | 实现方式                                      | 特点                       |
| ---- | -------------- | --------------------------------------------- | -------------------------- |
| 并行 | 多进程         | 在多个处理器或核心上运行多个进程              | 并行依赖硬件               |
|      | 多线程         | 多核 CPU 上运行多个线程，每个线程执行不同任务 |                            |
| 并发 | 多线程并发     | 同一个进程中创建多个线程，交替执行            | 并发是一种任务管理方式     |
|      | 多进程并发     | 通过多进程方式交替执行多个任务                |                            |
|      | 协程           | 在一个线程内非阻塞地切换任务                  |                            |
| 异步 | 事件异步       | 使用事件触发任务执行                          | 异步是一种非阻塞的编程模型 |
|      | Async/Await    | 非阻塞的 I/O 操作                             |                            |
|      | Future/Promise | 表示即将完成的操作                            |                            |


## 线程阻塞

> 线程阻塞适合那些比较强调`顺序`的操作，如等待用户输入, 定时任务, 等待另一个线程完成某项任务后才能继续执行等
>
> 他与线程锁的区别你可以在[这里](#线程锁和线程阻塞的区别)看到
{: .prompt-info }

```csharp
using System;
using System.Threading;

class Program
{
    // 创建一个 ManualResetEvent 实例，初始状态为未触发（即阻塞状态）
    private static ManualResetEvent resetEvent = new ManualResetEvent(false);

    static void Main(string[] args)
    {
        Console.WriteLine("主线程启动。");

        // 启动一个新线程
        Thread workerThread = new Thread(Worker);
        workerThread.Start();

        Console.WriteLine("按任意键解除阻塞...");
        Console.ReadKey();

        // 解除阻塞，使工作线程继续
        resetEvent.Set();

        // 等待工作线程结束
        workerThread.Join();

        Console.WriteLine("主线程结束。");
    }

    // 工作线程的执行方法
    static void Worker()
    {
        Console.WriteLine("工作线程启动，等待主线程解除阻塞...");

        // 阻塞，等待 ManualResetEvent 被触发
        resetEvent.WaitOne();

        Console.WriteLine("工作线程继续运行...");

        // 模拟一些工作
        Thread.Sleep(2000);

        Console.WriteLine("工作线程结束。");
    }
}

```

## 线程加锁

> 线程锁适合需要确保`数据一致性`的操作，如下面的文件IO操作，他与线程阻塞的区别你可以在[这里](#线程锁和线程阻塞的区别)看到
{: .prompt-info }

```csharp
using System;
using System.IO;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    private static readonly object fileLock = new object();
    private static string filePath = "example.txt";

    static async Task Main(string[] args)
    {
        // 启动多个线程同时写入文件
        Task task1 = Task.Run(() => WriteToFile("线程1的数据"));
        Task task2 = Task.Run(() => WriteToFile("线程2的数据"));
        
        await Task.WhenAll(task1, task2);
        
        Console.WriteLine("文件写入完成。");
    }

    static void WriteToFile(string content)
    {
        lock (fileLock) // 加锁，确保只有一个线程能进入
        {
            using (StreamWriter writer = new StreamWriter(filePath, true)) // 追加模式
            {
                writer.WriteLine(content);
            }
        }
    }
}
```


## 线程锁和线程阻塞的区别

|          | 线程锁                                                        | 线程阻塞                     |
| -------- | ------------------------------------------------------------- | ---------------------------- |
| 定义     | 控制线程对资源的`独占`访问<br/>(同时只能有一个线程在访问资源) | 让线程暂停执行，等待条件满足 |
| 目的     | 保护共享资源的完整性                                          | 控制线程执行的顺序           |
| 触发条件 | 线程试图访问已上锁的资源                                      | 等待某个条件、资源、时间     |
| 影响     | 同时只允许一个线程访问资源                                    | 当前线程停止运行，释放CPU    |


## 线程池

**目的**: 管理和复用线程，减少频繁创建和销毁线程带来的开销

适用于 I/O 密集型任务、短时间大量的的并行任务、频繁执行的任务

> ***在C#中，线程的创建、销毁和复用都是自动管理的, 这会引出下面的问题***
{: .prompt-info }

> **线程池中的线程数量是有限的**，默认会根据系统的 CPU 核心数自动调节, 你可以通过 `ThreadPool.SetMinThreads` 和 `ThreadPool.SetMaxThreads` 设置最小和最大线程数
>
> **线程池中的线程由系统自动管理**，开发者无法直接控制线程的启动和终止。这适合短时间任务，但不适合需要精确控制线程生命周期的场景
>
> **线程池主要设计用于处理短小的任务**，不适合长时间运行的任务。长时间运行的任务可能会占用线程池中的线程，导致其他任务无法及时获得线程执行。
{: .prompt-warning }


有三种方法可以使用线程池：

### ThreadPool.QueueUserWorkItem

```csharp
using System;
using System.Threading;

class Program
{
    static void Main()
    {
        // 提交任务到线程池
        ThreadPool.QueueUserWorkItem(DoWork, "任务1");
        ThreadPool.QueueUserWorkItem(DoWork, "任务2");

        Console.WriteLine("任务已提交到线程池");
        Console.ReadLine(); // 防止程序立即退出
    }

    static void DoWork(object state)
    {
        string taskName = state as string;
        Console.WriteLine($"{taskName} 正在执行，线程ID: {Thread.CurrentThread.ManagedThreadId}");
        Thread.Sleep(1000); // 模拟任务耗时
        Console.WriteLine($"{taskName} 执行完成");
    }
}
```

### Task

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main()
    {
        // 使用 Task 提交任务到线程池
        Task task1 = Task.Run(() => DoWork("任务1"));
        Task task2 = Task.Run(() => DoWork("任务2"));

        await Task.WhenAll(task1, task2);

        Console.WriteLine("所有任务完成");
    }

    static void DoWork(string taskName)
    {
        Console.WriteLine($"{taskName} 正在执行，线程ID: {Thread.CurrentThread.ManagedThreadId}");
        Task.Delay(1000).Wait(); // 模拟任务耗时
        Console.WriteLine($"{taskName} 执行完成");
    }
}
```

### Parallel

此方法在[上文](#并行)有提到过

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        // 使用 Parallel.For 并行执行任务
        Parallel.For(0, 5, i =>
        {
            Console.WriteLine($"任务 {i} 正在执行，线程ID: {Thread.CurrentThread.ManagedThreadId}");
            Task.Delay(500).Wait(); // 模拟任务耗时
            Console.WriteLine($"任务 {i} 执行完成");
        });

        Console.WriteLine("所有任务完成");
    }
}
```

## 取消线程

在需要让某一线程停止工作时，由于可能会发生资源泄露、不一致(反正就是会出问题)一般不推荐销毁线程，解决方法是使用**取消请求**

```csharp
using System;
using System.Threading;

class Program
{
    static void Main()
    {
        // 创建取消信号源
        CancellationTokenSource cts = new CancellationTokenSource();

        // 启动线程并传入取消令牌
        Thread thread = new Thread(() => DoWork(cts.Token));
        thread.Start();

        // 等待一段时间再取消线程
        Thread.Sleep(2000);
        Console.WriteLine("请求取消线程...");
        cts.Cancel();

        // 等待线程完成
        thread.Join();
        Console.WriteLine("线程已终止");
    }

    static void DoWork(CancellationToken token)
    {
        Console.WriteLine("线程开始执行...");
        for (int i = 0; i < 10; i++)
        {
            // 检查是否有取消请求
            if (token.IsCancellationRequested)
            {
                Console.WriteLine("取消请求已收到，线程即将退出...");
                return;
            }

            // 模拟工作
            Console.WriteLine($"正在处理数据 {i}...");
            Thread.Sleep(1000);
        }
        Console.WriteLine("线程正常结束");
    }
}

```

## 终止线程

> ***在C#中，终止线程可能会导致不一致的状态，因此在.NET Core和.Net5之后已被弃用***
{: .prompt-danger }

由于已经弃用且不推荐使用，这里只说一下方法名
```csharp
Thread.Abort()
```

## 如何使用并发和并行

在Web开发中，常会听到人们在强调高并发而非高并行

我们就以这个为例子来讲一下什么时候用并发，什么时候用并行

### 工作负载

工作负载常为**I/O密集型**和**CPU密集型**任务

### I/O密集型

大部分 Web 请求的操作包括数据库查询、文件读取、网络通信等，主要是**I/O 操作**而非**复杂的计算**

在处理这些操作时，处理的等待时间通常是瓶颈，而非计算能力

因此 Web 开发更关注如何处理大量的并发请求，以便在多个请求之间有效利用资源，而不是提高单个请求的执行速度

### CPU密集型

科学计算、图像处理等任务需要充分利用CPU多核性能，这样可以利用多个核心运行多个任务来加速计算结果

### I/O密集型任务常用并发，CPU密集型任务常用并行

由此可见，像Web服务这种对算力要求更低，但是涉及到很多请求的**I/O密集型**任务，使用**并发**是个更好的选择，在单个核心内能够同时处理更多的任务

在高并发环境中，资源的复用和高效调度是关键。现代 Web 服务器倾向于使用线程池、协程等机制来控制资源消耗。如果每个请求都创建一个线程，系统资源会迅速耗尽；相反，通过并发编程模型，可以让少量的线程处理大量的请求，提升服务器的资源使用效率

---

像科学计算这种需要大量算力的**CPU密集型**任务，使用**并行**是个更好的选择，可以充分利用多核CPU的全部性能
