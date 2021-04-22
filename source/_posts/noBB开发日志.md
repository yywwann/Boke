---
title: noBB开发日志
date: 2020-10-26 17:46:58
tags:
categories:
---


记一些 noBB 云顶之弈挂机脚本开发中发生的一些事情。

github 链接：[https://github.com/yywwann/TacticsScript](https://github.com/yywwann/TacticsScript)

> 目前暂时还未开源，所以只有说明界面

<!--more-->

# 2021年02月05日

因为之前用的脚本是用按键精灵做的，容易被官方检测，所以尝试用 python 模拟一下按键精灵的操作，于是就有了这个脚本。

目前群里 1931+994+200 人【2021年02月05日】

v1 还是黑框框， 极简功能
v2 花了一晚上使用 pyqt5 画了一个能用的界面
v3 美化了一下界面
v4 多标签页，自定义阵容，配合服务器统计真实使用人数

------

目前 noBB 宇宙主要分三个部分，脚本 noBB，登录中心 noBB-drf, 数据监测 noBB-web-back

- 主要技术栈

  |            | noBB                                               | noBB-drf                                                     | noBB-web-back                                         |
  | ---------- | -------------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
  | 主要技术栈 | python<br />pyqt5<br />pyautogui                   | django<br />mysql<br />django-rest-framework<br />nginx<br />docker | django<br />sqlite3<br />docker<br />nginx<br />redis |
  | 主要功能   | 自动进行云顶之弈挂机                               | 对使用脚本的用户进行验证和使用统计                           | 对脚本用户数据进行可视化展示                          |
  | 后续目标   | 自动登录游戏<br />更多分辨率支持<br />棋子优先上场 | 换种通信方式                                                 |                                                       |



------

`noBB` 使用 `pyinstaller` 打包，可以在 2g2 核的 win7 虚拟机中和 lol 一起运行.

`noBB-drf`和`noBB-web-back`都通过 docker 部署在腾讯云 1g2 核的学生机上,目前还凑合能用,2 月 5 日活动开启第一天大概 300 人在用.

之前`noBB-drf`还是使用 sqlite3 数据库的, 但是有一天晚上突然发生了莫名的连接失败, 打开 nginx 日志一看,全是 504 错误.![](https://docimg9.docs.qq.com/image/9HoCFGkusAQCt83h1NanyQ?w=2081&h=669)

>  **[error]** 7#7: *25602 upstream timed out (110: Connection timed out) while reading response header from upstream, 

想了一下, 还是把数据库换成靠谱点的`mysql`好了

发生了这种事情以后, 我开始关注 nginx 日志, 发现有很多记录是访问我不存在的网页的, 可能是什么脚本在暴力扫描

所以我百度研究了一下, 弄了个`fail2ban`来检测 nginx 日志, 把进程访问 404 的 ip 给 ban 掉.

------



3月 8日凌晨, 服务器内存满了,nginx 直接 499

重启 django 就好了, 应该是 django 内存泄漏.

重启也没用

吧 gunicorn -w 参数改成 3, 加了一些 nginx 配置, 看起来好了一些.

