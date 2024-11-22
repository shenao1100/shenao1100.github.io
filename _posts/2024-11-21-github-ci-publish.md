---
title: Github Action入门
description: 使用Gihub Action创建Workflow进行自动构建+自动部署
author: ShenNya
date: 2024-11-21 00:01:00 +0800
categories: [CI]
tags: [Workflow, CI, 自动构建]
pin: true
math: true
mermaid: true
---

# Github Action入门

> 此笔记尚未完成
{: .prompt-info }

本文将学习使用Gihub Action创建Workflow进行自动构建+自动部署

仅仅是个入门，深挖请移步[官方文档](https://github.blog/enterprise-software/ci-cd/build-ci-cd-pipeline-github-actions-four-steps/)

## 这是什么？

在我们的工作流中经常会遇到需要适配不同平台(Windows, Linux, MacOS...)对软件进行构建后发布

如果自己手动去Build后发布，那么需要在不同的虚拟机间切换、拉取代码、构建、发布

极其繁琐

这时我们就需要一个能够**自动构建，自动部署**的工具

隆重介绍：**CI/CD 持续集成和持续部署工具**

**Github Action**就是一个可以在代码的不同生命周期(如push请求)自动执行任务的CI/CD工具

下面将以最简单的C#程序为例子 了解这个workflow的配置

## 创建Gtihub Actions 工作流

Github Actions靠你的repo中`.github/workflows`中的文件定义

### 创建一个最简单的C#项目

创建一个hello world项目，并配置git

```bash
dotnet new console -n MyTestApp
cd MyTestApp
git init
git remote add origin https://github.com/yourusername/your-repository.git
git add .
git commit -m "Initial commit"
git push origin main
```

### 配置Github Actions

在repo中创建`.github/workflows`目录
```bash
mkdir -p .github/workflows
```


