---
layout: post
title: 同步org文件
---

最近一直有一件无比头痛的事情，虽然我们的计算机性能都已经这么强大，但是我想用计算机来管理我的生活确是那么困难!!

我喜欢org来管理一切，虽然现在还无法做到100%的管理但是我相信会有那么一天的。下面说下我使用的跨平台同步org的方案



* windows

最近对windows好感度up，因为借住docker的力量使得开发非常有感觉。

- docker 下跑 dropbox 同步 dropbox-sync volume
- docker 下跑 emacs 挂载 dropbox-sync

* ios

- beorg



这种方案要比直接同步一个文件夹，然后让emacs去编辑那个文件夹要好很多。
