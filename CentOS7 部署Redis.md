---
title: CentOS7部署Redis
tags:
  - Redis
  - CentOS7
categories:
  - 学习笔记
date: 2018-02-14 18:04:57
---

# 操作系统 #
* CentOS Linux release 7.4.1708 (Core)

# 安装Redis #
+ Redis官网下载网址：[这里](http://redis.io/download)
```
# wget http://download.redis.io/releases/redis-4.0.2.tar.gz
# tar -zxvf redis-4.0.2.tar.gz
# cd redis-4.0.2
# make
```
# 配置Redis开机自启 #
1. 修改Redis配置文件，开启守护进程
```
# cd redis-4.0.2
# vi redis.conf
```
将
```
daemonize no
```
改为
```
daemonize yes
```
2. 编写脚本文件
```
# vi /etc/init.d/redis
```
尽量手写，内容
```
# chkconfig: 2345 10 90
# description: start and stop redis

PATH=/usr/local/bin:/sbin/usr/local/bin:/bin

REDISPORT=6379 # redis端口号

EXEC=/usr/local/bin/redis-server 
REDIS_CLI=/usr/local/bin/redis-cli
PIDFILE=/var/run/redis_6379.pid #redis pid绝对路径(默认路径)
CONF="/usr/local/redis/redis.conf" # redis配置文件绝对路径

case "$1" in
    start)
        if [ -f $PIDFILE ]
            then
                echo "$PIDFILE exists, process is already running or crashed."
            else
                echo "starting redis server..."
                $EXEC $CONF
        fi
        if [ "$?"="0" ]
            then
                echo "redis is running..."
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ] 
            then
                echo "$PIDFILE exists, process is not running"
            else
                PID = $(cat $PIDFILE)
                echo "stoping.."
                $REDIS_CLI -p $REDISPORT SHUTDOWN
                while [ -x $PIDFILE ]
                do
                        echo "waiting for redis to shutdown.."
                        sleep 1
                done
                echo "redis stopped"
        fi
        ;;
    restart|force-reload)
        ${0} stop
        ${0} start
        ;;
    *)
        echo "Usage: /etc/init.d/redis {start|stop|restart|force-reload}" >&2
        exit 1
esac
```
3. 赋予文件权限
```
# chmod 755 redis
```
4. 设置开机自启
```
# chkconfig redis on
```
# 允许远程访问Redis #
1. 修改redis.conf配置文件，将
```
bind 127.0.0.1
```
注释或改为
```
bind 0.0.0.0
```
修改Redis认证密码，将
```
# requirepass foobared
```
取消注释，并修改foobared为你的认证密码
2. 设置服务器端口允许被远程访问
```
# vi /etc/sysconfig/iptables
```
增加
```
-A INPUT -p tcp -m state --state NEW -m tcp --dprot 6379 -j ACCEPT
```
保存并退出
3. 连接并测试
```
# redis-cli -h ip -p port -a password
```
