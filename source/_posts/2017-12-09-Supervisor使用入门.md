---
title: Supervisor使用入门
date: 2017-12-09 19:43:53
tags:
- Server 
- Tools
- 记录

categories:
- Server
- Tools
---

## 前言

之前写的一个TCP Server最近部署到生产环境，Server用的Python的Twisted框架，里面虽然有进程守护的功能，但是我之前一直用的Supervisor做后台进程管理，其比较好的是具有一个Web管理界面，可以可视化的控制后台进程的启停，我比较喜欢。

## 安装Supervisor

由于Supervisor是用python2开发，于是在服务器里就开了一个新的Python虚拟环境，安装了python2.7。直接 `	pip install supervisor ` 就好。

## 配置Supervisor

### 基本配置

```
# 生成配置文件
echo_supervisord_conf > /etc/supervisord.conf

# 生成后台进程配置文件文件夹
mkdir /etc/supervisord.d/

# 修改配置文件：
vi /etc/supervisord.conf
	
# 在文件最后添上以下命令，意思包括上面文件夹里的配置文件
[include]
files = /etc/supervisord.d/*.conf
```

### 配置Web远程控制

```
# 修改配置文件
vi /etc/supervisord.conf

# 把以下内容取消注释，建议把这三个都改调，防止端口被扫描
[inet_http_server]         ; inet (TCP) server disabled by default
port=0.0.0.0:9001        ; ip_address:port specifier, *:port for all iface
username=user              ; default is no username (open server)
password=123               ; default is no password (open server)
```

**注意：**  要想外网访问，还得记得在阿里云的安全组设置里打开对应的端口

### 配置Supervisor为Service

1. 查看好python2.7的路径

    ```
    which python2.7
    /usr/local/bin/python2.7
    ```

2. 添加service启动脚本,将python路径换成上文的路径
    
    ```
    cd /etc/init.d/
    vi supervisord
    ```

    将以下内容添加进去
    
    ```
    #!/bin/sh
    #
    # /etc/init.d/supervisord
    #
    # Supervisor is a client/server system that
    # allows its users to monitor and control a
    # number of processes on UNIX-like operating
    # systems.
    #
    # chkconfig: - 64 36
    # description: Supervisor Server
    # processname: supervisord
     
    # Source init functions
    . /etc/rc.d/init.d/functions
     
    prog="supervisord"
     
    prefix="/usr/local"
    exec_prefix="${prefix}"
    prog_bin="${exec_prefix}/bin/supervisord"
    PIDFILE="/var/run/$prog.pid"
     
    start()
    {
           echo -n $"Starting $prog: "
           ###注意下面这一行一定得有-c /etc/supervisord.conf   不然修改了配置文件根本不生效！
           daemon $prog_bin -c /etc/supervisord.conf --pidfile $PIDFILE
           [ -f $PIDFILE ] && success $"$prog startup" || failure $"$prog startup"
           echo
    }
     
    stop()
    {
           echo -n $"Shutting down $prog: "
           [ -f $PIDFILE ] && killproc $prog || success $"$prog shutdown"
           echo
    }
     
    case "$1" in
     
     start)
       start
     ;;
     
     stop)
       stop
     ;;
     
     status)
           status $prog
     ;;
     
     restart)
       stop
       start
     ;;
     
     *)
       echo "Usage: $0 {start|stop|restart|status}"
     ;;
     
    esac
    ```

3. 把启动脚本加入service启动项

    ```
    chmod +x /etc/init.d/supervisord
    chkconfig --add supervisord
    chkconfig supervisord on
    service supervisord start
    ```

## 配置要管理的进程 

在管理目录下新建配置文件：

```
cd /etc/supervisord.d/
vi xxx.conf
```

```
[program:xxx]
command=/usr/local/python3/bin/python3  /root/xxx.py
autostart=true                           ;是否随supervisor启动
autorestart=true                         ;是否在挂了之后重启，意外关闭后会重启，比如kill掉！
startretries=3                           ;启动尝试次数
stderr_logfile=/tmp/pileSocketServer.err.log        ;标准输出的位置
stdout_logfile=/tmp/pileSocketServer.out.log        ;标准错误输出的位置
```

这里更多的配置参数详见[Supervisor官网参数配置页](http://supervisord.org/configuration.html#program-x-section-values)

添加完配置文件后，重启Supervisor服务`service supervisord restart`就大功告成了。

注：我在实测中，虽然调用restart显示成功，但是Supervisor并没有启动成功。测试以后发现，我的环境需要Start两次才能成功，因此我每次重启Supervisor，只能stop->start->start.

```
service supervisord stop
service supervisord start
service supervisord start
```

## 番外

这个项目是暑期写的，这次回过头来看，重构的时候发现有些业务逻辑都忘了，因此感叹文档记录太重要了。

有了记录文档，以后就算忘了回过头来看也可以更快恢复上下文。

说这些就是提醒自己，笔记和博客都得勤记。

## 参考

- [Centos平台Supervisord全攻略](http://blog.51cto.com/youerning/1714627)
- [Supervisor官网文档](http://supervisord.org/configuration.html)


