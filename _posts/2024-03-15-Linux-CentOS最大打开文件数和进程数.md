---
layout:     post
title:      CentOS 最大打开文件数和进程数
date:       2024-04-15
author:     KevinShan
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Linux
---

golang 代码出现服务的文件句柄超出系统限制(too many open files)，这个是Linux系统中常见的错误，从字面意思上看就是说程序打开的文件数过多，不过这里的files不单是文件的意思，也包括打开的通讯链接(比如socket)，正在监听的端口等等，所以有时候也可以叫做句柄(handle)，这个错误通常也可以叫做句柄数超出系统限制。 引起的原因就是进程在某个时刻打开了超过系统限制的文件数量以及通讯链接数。

[Linux](https://so.csdn.net/so/search?q=Linux&spm=1001.2101.3001.7020) 系统对打开文件数和进程数有限制，默认限制为1024，它是一种简单有效的实现资源限制的方式。但当单进程的并发量较大时，1024的限制很容易超标，报告 `too many open files` 的错误。为了让系统能够支持更大的并发，就需要修改默认的限制数。

##### 1、查看最大打开文件数

```shell
ulimit -n
```

> 可以通过 `ulimit -a` 查看更多的系统限制值

##### 2、修改最大文件数与进程数

终端可以通过执行 `ulimit -HSn 10240` 命令的方式临时生效，这里介绍永久生效的方法

###### 修改 limits.conf

修改 `/etc/security/limits.conf` 文件，文件尾部增加以下配置

```
* soft nofile 655350 
* hard nofile 655350
* soft nproc  655350
* hard nproc  655350
* soft core   unlimited
* hard core   unlimited
```

重启服务器后，再通过 `ulimit -n` 查看是否生效

###### systemd 生效

如果使用 `systemd` 自启动服务，使用的systemctl 命令，**在高版本的CentOS等系统中，可能上述修改没有生效**，此时需要进一步修改：

修改 `/etc/systemd/system.conf` 与 `/etc/systemd/user.conf` 文件，文件尾部增加以下配置：

```
DefaultLimitCORE=infinity
DefaultLimitNOFILE=655350
DefaultLimitNPROC=655350
```

> 执行 `systemctl daemon-reload` 命令，让配置文件即时生效
> 针对每个服务，修改 /etc/systemd/system/xiugou.service  添加如下

```ini
LimitNOFILE=100000 
LimitNPROC=100000
```

```linux
修改配置文件以后，需要重新加载配置文件，然后重新启动相关服务。

# 重新加载配置文件
$ sudo systemctl daemon-reload

# 重启相关服务
$ sudo systemctl restart foobar
```

### 引用

1. [Linux-CentOS 最大打开文件数和进程数_centos最大进程数-CSDN博客](https://blog.csdn.net/aoshilang2249/article/details/110650318)
2. [Centos7之Systemd(Service文件)详解_centos 没有systemd 文件夹-CSDN博客](https://blog.csdn.net/Mr_Yang__/article/details/84133783)
3. [教你如何自定义systemd开机启动脚本在我们实际的工作过程中，经常需要为一些安装好的第三方软件或服务，在服务器重启时， - 掘金](https://juejin.cn/post/7254572372136509477)
4. [Centos7之Systemd(Service文件)详解_centos 没有systemd 文件夹-CSDN博客](https://blog.csdn.net/Mr_Yang__/article/details/84133783)
