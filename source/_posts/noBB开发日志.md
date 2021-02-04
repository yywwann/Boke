---
title: noBB开发日志
date: 2020-10-26 17:46:58
tags:
categories:
---


记一些 noBB 云顶之弈挂机脚本中发生的一些事情。

github 链接：[https://github.com/yywwann/TacticsScript](https://github.com/yywwann/TacticsScript)

> 目前暂时还未开源，所以只有说明界面

![](https://images-cdn.shimo.im/mubdjpaKfcFklCky.png)

<!--more-->

因为之前用的脚本是用按键精灵做的，容易被官方检测，所以尝试用 python 模拟一下按键精灵的操作，于是就有了这个脚本。

目前群里 2000 人正在使用


# 主要技术栈

- pyhon
- pyautogui
- Pyqt5

-----

## 如何使桌面程序置顶

```python
MainWindow.setWindowFlags(QtCore.Qt.WindowStaysOnTopHint)
```

## 使用别人写好的 QSS 进行美化

我直接用 `qdarkstyle`

```pyhon
MainWindow.setStyleSheet(qdarkstyle.load_stylesheet_pyqt5())
```

## 布局 TIPS

布局原则上从内到外

## 图像识别

`pyautogui`使用的也是 `openCV2` 自带的图像识别

可以自定义可信度和匹配方式

## 检查win10 桌面分辨率和缩放

```python
        # 检查电脑分辨率是否正确
        user32 = ctypes.windll.user32
        gdi32 = ctypes.windll.gdi32
        dc = user32.GetDC(0)
        # widthScale = gdi32.GetDeviceCaps(dc, 8)  # 分辨率缩放后的宽度
        # heightScale = gdi32.GetDeviceCaps(dc, 10)  # 分辨率缩放后的高度
        width = gdi32.GetDeviceCaps(dc, 118)  # 原始分辨率的宽度
        height = gdi32.GetDeviceCaps(dc, 117)  # 原始分辨率的高度
        hdpi = gdi32.GetDeviceCaps(dc, 88)    # 缩放
```



