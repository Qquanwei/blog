---
layout: post
title: 在docker中使用emacs
---

在widnows下开发，使用docker的emacs-gui版本。

为什么要在docker中使用emacs？ 因为win下的emacs远远没有linux下的emacs顺滑。

# prepare

* docker-toolbox
* xming

xming是一个win下的x-server， 使用xming可以让docker内的程序连接外部的图形x环境进行程序的显示。


# 定义docker

使用ubuntu定制一个image

```(shell)
> docker image ubuntu
> docker run -it --name build-emacs ubuntu bash

>> // install emacs25
>> // install custom emacs configure
>> // install git/wget/curl/nano
>> // install ...
```

## 让容器显示中文字体

编辑`～/.profile`, 放入下面的配置

```(shell)
> export LANG=en_US.UTF-8
> export LC_ALL=en_US.UTF-8
```

## 给emacs添加中文输入法

因为在docker中没有了外部系统的输入法，所以需要在emacs中配置中文输入法。可以参考：[我的配置](https://github.com/Qquanwei/emacs)


配置完成之后将此容器commit成为一个镜像供以后使用

> docker commit build-emacs emacs

# 启动emacs

```(shell)
> docker create --name demacs -e DISPLAY=<HOSTIP>:0 -v m-project/:/git -w /git emacs emacs25
```

* `<HOSTIP>` 为宿主机的地址, 这里的环境变量指定了x-server的地址，这样启动emacs时就可以知道连接到哪里了。

```(shell)
> docker start demacs
```

![docker-emacs.gif]({{site.baseurl}}/content/images/docker-emacs.gif)
