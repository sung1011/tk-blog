---
layout: post
title: "Nginx-param-and-sign"
tags:
 - nginx
description: nginx-param-and-sign
---
> 官方文档：http://nginx.org/en/docs/control.html

# nginx参数和信号

## nginx参数

COMMAND|ILLUSTRATE
---|---
-c |  </path/to/config>为 Nginx 指定一个配置文件，来代替缺省的。
-t | 不运行，而仅仅测试配置文件。nginx 将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件。
-v| 显示 nginx 的版本。
-V|显示 nginx 的版本，编译器版本和配置参数。

## nginx信号  

### Master进程能够接收并处理如下的信号：

COMMAND | ILLUSTRATE
--- | ---
`ERM, INT`|（快速退出，当前的请求不执行完成就退出）  
`QUIT`| （优雅退出，执行完当前的请求后退出）  
`HUP`| （重新加载配置文件，用新的配置文件启动新worker进程，并优雅的关闭旧的worker进程）  
`USR1`| （重新打开日志文件）  
`USR2`| （平滑的升级nginx二进制文件）  
`WINCH`| （优雅的关闭worker进程）  

### Worker进程也可以接收并处理一些信号：

COMMAND | ILLUSTRATE
--- | ---
`TERM, INT` | （快速退出）  
`QUIT` |（优雅退出）  
`USR1` |（重新打开日志文件）  

## nginx 启动、停止、重启命令

### nginx启动

```
sudo /usr/local/nginx/nginx     (nginx二进制文件绝对路径，可以根据自己安装路径实际决定)
```

### nginx从容停止命令，等所有请求结束后关闭服务
```
ps -ef |grep nginx
kill -QUIT  nginx主进程号
```

### nginx 快速停止命令，立刻关闭nginx进程

```
ps -ef |grep nginx
kill -TERM nginx主进程号
```

如果以上命令不管用，可以强制停止

```
kill -9 nginx主进程号
```



如果嫌麻烦可以不用查看进程号，直接使用命令进行操作
其中/usr/local/nginx/nginx.pid 为nginx.conf中pid命令设置的参数，用来存放nginx主进程号的文件
kill -信号类型(HUP|TERM|QUIT) cat /usr/local/nginx/nginx.pid
例如

```
kill -QUIT `cat /usr/local/nginx/nginx.pid`
```
### nginx重启命令

- nginx重启可以分成几种类型:

1.简单型，先关闭进程，修改你的配置后，重启进程。
```
kill -QUIT cat /usr/local/nginx/nginx.pid
sudo /usr/local/nginx/nginx
```
2.重新加载配置文件，不重启进程，不会停止处理请求
3.平滑更新nginx二进制，不会停止处理请求


### 用HUP信号使Nginx加载新的配置文件
```
kill -HUP {pid}
nginx -s reload
```
当Nginx接收到HUP信号的时候，它会尝试着去解析并应用这个配置文件，如果没有问题，那么它会创建新的worker进程，并发送信号给旧的 worker进程，让其优雅的退出。接收到信号的旧的worker进程会关闭监听socket，但是还会处理当前的请求，处理完请求之后，旧的 worker进程退出。如果Nginx不能够应用新的配置文件，那么仍将用旧的配置文件来提供服务。   

### 在线升级Nginx二进制文件

```
kill -USR2 `cat /usr/local/nginx/nginx.pid`
kill -WINCH `cat /usr/local/nginx/nginx.pid.oldbin`

# old worker全部退出并确认使用new nginx运行正确后 执行：
kill -QUIT `cat /usr/local/nginx/nginx.pid.oldbin`

# new nginx运行异常，欲恢复old nginx 执行：
kill -HUP `cat /usr/local/nginx/nginx.pid.oldbin`
# 或
kill -TERM `cat /usr/local/nginx/nginx.pid`

```

当你想升级Nginx到一个新的版本，增加或减少module的时候，你需要替换Nginx的二进制文件，你可以平滑的实现它，没有请求会丢失。  

首先，用新的二进制文件替换掉旧的，然后发送USR2信号给master进程。master进程会把自己的.pid文件重命名为.oldbin（例 如，/usr/local/nginx/logs/nginx.pid.oldbin），然后执行新的二进制文件，从而启动一个新的master进程和新的worker进程：

    PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
    33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
    33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
    33135 33126 nobody   0.0  1380 kqread nginx: worker process (nginx)
    33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
    36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
    36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)

在这个时候，有两个Nginx实例在运行，一起处理进来的请求。为了让旧的实例退出，你需要发送WINCH信号给旧的master进程，这样旧master进程的worker进程就会优雅的退出：

        PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
    33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
    33135 33126 nobody   0.0  1380 kqread nginx: worker process is shutting down (nginx)
    36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
    36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)

一段时间后，旧的worker进程都已经退出了，只有新的worker进程处理进来的请求：

    PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
    33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
    36264 33126 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
    36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)

这个时候你仍然可以通过以下几个步骤回滚到旧的服务，因为旧master进程并没有关闭其监听的socket： 发送HUP信号给旧的master进程，它会启动worker进程并且不需要重新加载配置文件 发送QUIT信号给新的master进程，让它优雅的终止其worker进程发送TERM信号给新的master进程，强制其退出 如果一些原因，新的worker进程没有退出，发送KILL信号给它们 当新的master进程退出之后，旧的master进程会删除其pid文件名中的后缀.oldbin，这样一切就又变成升级之前的样子。 如果一个升级已经成功，然后你想只保留新的server，那么发送QUIT信号给旧的master进程让新的server来提供服务：

    PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
    36264     1 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
    36265 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36266 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
    36267 36264 nobody   0.0  1364 kqread nginx: worker process (nginx)
