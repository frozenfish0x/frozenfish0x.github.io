---
layout: 问题与解决
title: 访问百度自动跳转github
date: 2023-12-01 16:31:37
tags:
---



使用steam++加速github后浏览百度居然自动跳转GitHub、

这一下子就想到了dns劫持，或者ip劫持。要么就是我的本地localhost文件被更改。总之强制跳转很恶心。

```
于是端口链接。检查localhost文件。发现一切正常
然后用  
Ipconfig/flushdns 清除dns解析记录
然后清楚浏览器缓存。就没有该问题
```

没钱买魔法。真的难受

<!--more-->
