---
title: C#多线程深入
description: C#多线程线程操作和同步机制
author: ShenNya
date: 2024-11-13 17:25:00 +0800
categories: [CSharp]
tags: [CSharp, 多线程, Thread, 并发, 并行]
#pin: true
math: true
mermaid: true
---
# 前言

多线程的东西实在是太多了，一篇文章说不完

本文将包括更多线程操作和同步机制

- Semaphore/SemaphoreSlim：控制同时访问资源的线程数
- CountdownEvent：等待多个线程任务完成
- ManualResetEvent / AutoResetEvent：线程间信号通知
- Barrier：多线程协调同步，适合分阶段执行
- Task：高级并发控制，支持任务链、异步操作
- Dataflow Blocks：数据流处理，适合复杂的并发任务传递
- ReaderWriterLockSlim：读写锁，优化多读少写场景
- Concurrent Collections：线程安全集合，支持线程间通信

你可以在[这里](/posts/csharp-multi-thread/)找到多线程入门

> 此页面由于东西太多尚未完工
{: .prompt-info }
## 事件（Event）同步机制

**ManualResetEvent / AutoResetEvent**：用于在线程之间发信号，让一个或多个线程等待另一个线程的信号。ManualResetEvent 可以手动重置信号，而 AutoResetEvent 每次发送信号后会自动重置

```csharp
AutoResetEvent autoEvent = new AutoResetEvent(false);

void Thread1()
{
    Console.WriteLine("线程1等待信号...");
    autoEvent.WaitOne(); // 阻塞，直到信号被设置
    Console.WriteLine("线程1收到信号，继续执行");
}

void Thread2()
{
    Console.WriteLine("线程2准备发送信号...");
    autoEvent.Set(); // 设置信号，释放等待的线程
}

```

## 障碍器(Barrier)

**Barrier** 是一种多线程协调机制，让一组线程在执行到指定点时等待其他线程，然后再一起继续执行。适合需要分阶段执行的多线程任务

```csharp
Barrier barrier = new Barrier(3, (b) =>
{
    Console.WriteLine("所有线程都已到达屏障点，继续执行下一阶段");
});

for (int i = 0; i < 3; i++)
{
    Task.Run(() =>
    {
        Console.WriteLine("线程到达屏障点");
        barrier.SignalAndWait(); // 线程到达屏障点
        Console.WriteLine("线程继续执行下一阶段");
    });
}
```
## 读写锁（ReaderWriterLockSlim）
**ReaderWriterLockSlim**是一种读写锁，允许多个线程同时读数据（不冲突），但写入时会阻塞其他读写操作。适合读操作频繁但写操作较少的场景

```csharp
ReaderWriterLockSlim rwLock = new ReaderWriterLockSlim();

void ReadData()
{
    rwLock.EnterReadLock();
    try
    {
        // 读取操作
    }
    finally
    {
        rwLock.ExitReadLock();
    }
}

void WriteData()
{
    rwLock.EnterWriteLock();
    try
    {
        // 写入操作
    }
    finally
    {
        rwLock.ExitWriteLock();
    }
}

```
## 线程间通信（Thread Communication）

通过 **线程安全队列**（如 ConcurrentQueue）、**任务通道（Channel）**等，可以实现线程之间的数据传递和通信

(有点类似Avalonia中的`Dispatcher`)

```csharp
ConcurrentQueue<int> queue = new ConcurrentQueue<int>();
queue.Enqueue(1);
queue.TryDequeue(out int result);
```
