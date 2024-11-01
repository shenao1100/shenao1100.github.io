---
title: Avalonia UI常用C#实现
description: 使用C#控制元素
author: ShenNya
date: 2024-11-01 21:44:00 +0800
categories: [Avalonia UI, CSharp]
tags: [Avalonia UI, CSharp, Axaml, UserControl, WinUI, FluentUI]
#pin: true
math: true
mermaid: true
---

# Avalonia UI常用C#实现

## 寻找控件

需在xaml中为相应控件设置x:Name

```csharp
_myButton = this.FindControl<Button>("MyButton");
```

## Margin

```csharp
_myButton.Margin = new Thickness(0, 10, 0, 0);
```

## 应用变形（Transfrom）

制作动画

```csharp
var scaleTransform = new ScaleTransform { ScaleX = 0.9, ScaleY = 0.9 };
_myButton.RenderTransform = scaleTransform;
```

## 应用动画 (Animation)

```csharp
var DropAnimation = new Animation
{
  Duration = TimeSpan.FromSeconds(0.25),
  Easing = new ExponentialEaseOut(),
  Children =
  {
      new KeyFrame
      {
          Cue = new Cue(0),
          Setters =
          {
              new Setter(UserControl.MarginProperty, new Thickness(0,100,0,0)),
              new Setter(UserControl.OpacityProperty, 0.0)
          }
      },
      new KeyFrame
      {
          Cue = new Cue(1),
          Setters =
          {
              new Setter(UserControl.MarginProperty, new Thickness(0)),
              new Setter(UserControl.OpacityProperty, 1.0)
          }
      }
  }
};
await DropAnimation.RunAsync(ContentPanel1);
```

