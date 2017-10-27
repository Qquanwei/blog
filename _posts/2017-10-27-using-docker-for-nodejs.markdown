---
layout: post
title: windows下使用docker跑node程序
---

win下使用docker进行node的开发。

# prepare

* docker-toolbox

# install

下载好docker-toolbox后会自动安装上`Oracle VM VirtualBox`,`Docker Quickstart Terminal`。

启动`Docker Quickstart Terminal`会自动初始化虚拟机（前提是你的VT-x已经开启，而且没有被win10没有启用自带的virtual machine）。

在终端中安装必要的环境

```(shell)
> docker image pull node
```

下载好node镜像后基本上已经安装完毕了。但是有时候需要定制安装一些其他的程序包，例如一些全局npm程序等，这种情况可以直接基于debian的镜像进行定制。

## 定制node环境

```(shell)
> docker image pull debian
> docker run --name custom-node -it debian bash

>> # 双层提示符代表容器内shell操作
>> apt install wget
>> wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.6/install.sh | bash
>> nvm install stable
>> nvm use default
>> npm install webpack -g
>> ...
>> ...
>> exit

> docker commit custom-node my-node-env
> docker container rm custom-node
```

使用docker commit可以将容器提交成一个自定义的镜像，可以方便后续开发使用。自定义镜像内包含了自己安装的一系列程序包，配置信息等，可以push到dockerhub或内网的仓库中以供团队开发分享使用。


之后将会使用这个自定义镜像进行后续操作。

# 本地项目绑定

项目代码在windows系统上，因为我们可能想使用外部编辑器方便编辑，所以我们的docker仅仅用于执行命令使用，所有的文件都发生在本地。

假设本地目录为`e:/git/projectA`.

开发步骤为

```(shell)
> docker run --rm -p 8080:8080 -v /e/git/projectA:/app -w /app my-node-env npm install
```

`-v`参数为`--volume`的简写，用于挂载本地目录到容器内。注意windows风格的路径 e:/ 需要转换成/e/进行操作，否则绑定失败。
`--rm` 参数为当容器内的进程执行结束后自动删除容器(如果不指定，你每运行一个命令将会创建一个容器留在磁盘中，除非之后手动清理)。

```(shell)
> docker create --name site -v /e/git/projectA/:/app my-node-env npm start
```

这里使用create命令创建了一个容器，create与run不同点就是create后的容器不会立即执行，需要手动启动。仅此而已。另外这里也给这个容器绑定了一个name为`site` 方便我们日后的操作。因为这个容器是我们一直都需要用的，不需要自动清除它。

```(shell)
> docker start site   // 启动容器
> docker stop site    // 停止容器
> docker restart site // 重启容器
```

上面三个命令就是今后将会一直执行的操作。可以很方便地启动暂停。由于container没有被自动清除，所以之后你可以在任何时候启动这个容器。（完全不用担心端口被占用的问题，stop/restart会很好地释放这些资源）


## 小技巧

* 因为不仅仅是这个容器的启动与执行，有时候添加依赖和执行其他的程序也需要用到容器，每次都需要输一段非常长的代码去执行很不方便。例如这个`> docker run --rm -v /e/git/projectA:/app -w /app my-node-env npm install` 我想执行`npm install somePackage`怎么办?

可以给自己的shell自定义函数来解决，自动绑定目录, 或者自动绑定端口。由于我使用emacs的eshell，可以很方便地添加一个函数

```(lisp)
(defun eshell/drun (&rest args)
  "Run docker command in current directory
drun: image command [options]
options
  -p port set docker expose port
"
  (let ((curoptions nil)
        (options (make-hash-table))
        (command "")
        (flags (format "--rm -v %s:/app -w /app" (eshell/wpwd))))
    (dolist (item args)
      (if (and
           (stringp item)
           (s-starts-with-p "-" item))
          (setq curoptions item)
        (progn
          (if curoptions
              (puthash curoptions item options)
            (setq command (s-concat command " " item)))
          (setq curoptions nil))))
    (maphash
     (lambda (key value)
       (setq flags (s-concat
                    flags " "
                    (cond
                     ((string= "-p" key) (if (and
                                              (stringp value)
                                              (s-contains-p ":" value))
                                             (s-concat "-p" " " value)
                                           (format "-p %d:%d" value value)))
                     ))))
     options)
    (insert (format "docker run %s %s" flags command))
    (eshell-send-input)
    (insert "\n")))
```

这样就只需要每次下面这样就好了, 很方便。
```
> drun node npm install
```


* 为什么在localhost:8080上访问不到应用程序?

首先需要弄明白一点，docker并不是真正跑在windows上的，目前之后linux才原生支持资源虚拟化，所以非linux平台都是使用虚拟机上的Linux系统跑docker-deamon。所以真正的端口绑定发生在 `Linux虚拟机 <--> Linux虚拟机内的容器`, 你去请求Windows的localhost:8080是请求不到的。

可以使用`docker-machine`来很方便地获取到Linux虚拟机的ip，直接访问虚拟机地址:端口就好。

关于docker-machine建议了解一下 (地址)[https://docs.docker.com/machine/concepts/#default-base-operating-systems-for-local-and-cloud-hosts]
