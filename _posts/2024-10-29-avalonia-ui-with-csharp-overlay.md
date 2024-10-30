---
title: Avalonia UI实现提示弹窗、输入弹窗
description: 使用异步处理的弹窗，代码上符合直觉
author: ShenNya
date: 2024-07-03 02:17:00 +0800
categories: [Avalonia UI, CSharp]
tags: [Avalonia UI, CSharp, Axaml, UserControl]
#pin: true
math: true
mermaid: true
---

# Avalonia UI实现提示弹窗、输入弹窗


> 此笔记尚未完成
{: .prompt-info }

## 带有输入框的弹窗

DialogOverlay.axml
```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
             mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
             x:Class="Netko.DialogOverlay">
	<Grid Background="{DynamicResource CatalogBaseLowColor}" HorizontalAlignment="Stretch" VerticalAlignment="Stretch">
		<Grid HorizontalAlignment="Center" VerticalAlignment="Center">
			<Border Background="{DynamicResource CatalogChromeMediumColor}" x:Name="BorderBackground" CornerRadius="7" Padding="0" Margin="0" Opacity="1" BoxShadow="0 0 3 1 #20000000"/>
			<StackPanel>
				<Label Margin="7" x:Name="Message">Explain</Label>
				<TextBox Margin="10 0" Width="200" BorderThickness="0" x:Name="Input"/>
				<Separator Margin="0 5" Padding="0"></Separator>

				<Button HorizontalAlignment="Right" Margin="5" Click="Send">Proceed</Button>
			</StackPanel>

		</Grid>

	</Grid>
</UserControl>

```

DialogOverlay.cs

```csharp
using Avalonia;
using Avalonia.Controls;
using Avalonia.Markup.Xaml;
using System.Threading.Tasks;

namespace Netko;

public partial class DialogOverlay : UserControl
{
    private TaskCompletionSource<string> _taskCompletionSource;
    public DialogOverlay()
    {
        InitializeComponent();
    }
    public Task<string> ShowDialog(string message)
    {
        Message.Content = message;
        _taskCompletionSource = new TaskCompletionSource<string>();
        this.IsVisible = true;
        return _taskCompletionSource.Task;
    }

    private void Send(object? sender, Avalonia.Interactivity.RoutedEventArgs e)
    {
        _taskCompletionSource?.SetResult(Input.Text);
        this.IsVisible = false;
    }
}
```


## 仅仅是提示框

MessageOverlay.axaml
```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
             mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
             x:Class="Netko.MessageOverlay">
	<Grid Background="{DynamicResource CatalogBaseLowColor}" HorizontalAlignment="Stretch" VerticalAlignment="Stretch">
		<Grid HorizontalAlignment="Center" VerticalAlignment="Center">
			<Border Background="{DynamicResource CatalogChromeMediumColor}" x:Name="BorderBackground" CornerRadius="7" Padding="0" Margin="0" Opacity="1" BoxShadow="0 0 3 1 #20000000"/>
			<StackPanel>
				<Label Margin="7">Notice</Label>
				<Label Margin="10">Login expired, please relogin</Label>
				<Separator Margin="0 5" Padding="0"></Separator>

				<Button HorizontalAlignment="Right" Margin="5" Click="Close">Understand</Button>
			</StackPanel>

		</Grid>
		
	</Grid>
</UserControl>

```

MessageOverlay.cs
```csharp
using Avalonia;
using Avalonia.Controls;
using Avalonia.Input;
using Avalonia.Markup.Xaml;

namespace Netko;

public partial class MessageOverlay : UserControl
{
    public MessageOverlay()
    {
        InitializeComponent();
    }
    private void Close(object? sender, Avalonia.Interactivity.RoutedEventArgs e)
    {

        this.IsVisible = false;

    }
}
```
