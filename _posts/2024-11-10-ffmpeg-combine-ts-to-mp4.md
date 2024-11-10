---
title: ffmpeg对ts文件合并并转码为MP4
description: ffmpeg常见操作，对ts文件合并并转码为MP4
author: ShenNya
date: 2024-11-10 18:12:00 +0800
categories: [ffmpeg]
tags: [爬虫, ffmpeg, Python]
#pin: true
math: true
mermaid: true
---


# ffmpeg对ts文件合并并转码为MP4

## 起因

在写视频网站爬虫时，处理m3u8到最后经常会指向一堆ts文件，如：
```
#EXTM3U
#EXT-X-TARGETDURATION:4
#EXT-X-ALLOW-CACHE:YES
#EXT-X-PLAYLIST-TYPE:VOD
#EXT-X-VERSION:3
#EXT-X-MEDIA-SEQUENCE:1
#EXTINF:4.000,
720.mp4/seg-1-v1-a1.ts
#EXTINF:4.000,
720.mp4/seg-2-v1-a1.ts
#EXTINF:4.000,
720.mp4/seg-3-v1-a1.ts
#EXTINF:4.000,
720.mp4/seg-4-v1-a1.ts
#EXTINF:4.000,
720.mp4/seg-5-v1-a1.ts
#EXTINF:4.000,
720.mp4/seg-6-v1-a1.ts
#EXTINF:4.000,
720.mp4/seg-7-v1-a1.ts
#EXTINF:4.000,
720.mp4/seg-8-v1-a1.ts
#EXTINF:4.000,
720.mp4/seg-9-v1-a1.ts
#EXTINF:3.403,
720.mp4/seg-10-v1-a1.ts
#EXT-X-ENDLIST
```

在下载ts文件后，需要按照顺序合并并转码为mp4才能方便使用和存储

## 解决方案

1. 首先在电脑上下载ffmpeg（略）
2. 创建一个格式正确的包含所有ts文件路径的列表文件，如：
```
file 'segment1.ts'
file 'segment2.ts'
file 'segment3.ts'
```
此处命名为`filelist.txt`
3. 使用ffmpeg合并并输出
```
ffmpeg -f concat -safe 0 -i filelist.txt -c copy output.mp4
```

## 简单脚本

### 将文件夹内的ts文件整合为列表

```bash
for file in *.ts; do echo "file '$file'" >> filelist.txt; done
```

### 使用Python调用ffmpeg

```python
import subprocess

command = [
    "ffmpeg",
    "-f", "concat",
    "-safe", "0",
    "-i", f"filelist.txt",
    "-c", "copy",
    f"output.mp4"
]
try:
    subprocess.run(command, check=True)
    count_index = 0
    print("合并完成")
except subprocess.CalledProcessError as e:
    print("合并失败：", e)
```
