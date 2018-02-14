---
title: 分布式部署pyspider
tags:
  - 学习笔记
  - pyspider
categories:
  - pyspider
date: 2018-02-14 16:24:30
---

# 环境 #
+  CentOS Linux release 7.4.1708 (Core) * 2
+  Python 2.7.5
+  pip 9.0.1

# 目标 #
+ 在2台服务器上分布式部署PySpider爬虫框架，各服务器中PySpider组件如下：

|主机 |scheduler | fetcher | processor|webui|
| :------:| :------: | :------: | :------: | :------: |
|Master |√|√|√|√|
|Slave ||√|√||

# 安装并配置Redis #
+ 配置Master，上一篇文章 →_→ [传送门](https://zjhdota.github.io/2018/02/14/CentOS7%20%E9%83%A8%E7%BD%B2Redis/)

# 安装并配置MySQL #
+ 配置Master，上一篇文章 →_→ [传送门](https://zjhdota.github.io/2018/02/14/CentOS7%20%E9%83%A8%E7%BD%B2MySQL/)
# 安装并配置PhantomJS #
+ [PhantomJS官网](http://phantomjs.org/download.html)
```
# wget https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2
# tar -jxvf phantomjs-2.1.1-linux-x86_64.tar.bz2
```
创建软连接，根据实际情况(绝对路径)
```
# ln -s /home/guest/phantomjs-2.1.1-linux-x86_64/bin/phantomjs /usr/bin/
```
+ 测试
```
# phantomjs -v
```

# 安装并配置PySpider #
+ 配置Master、Slave
1. 安装Pyspider 
```
# pip install pyspider
```
2. 安装redis-py
+ [PyPI](https://pypi.python.org/pypi/redis)
```
# pip install redis 
```
3. 安装mysql-connector
+ [PyPI](https://pypi.python.org/pypi/mysql-connector/)
```
# pip install mysql-connector==2.1.6
```
4. 编写conf.json(PySpsider的启动配置文件)
+ 主要参数说明：
user 为用户名，password为密码，ip为地址，port为端口号，
requirepass为Redis认证密码。
+ Master和Slave
```
{
  "taskdb": "mysql+taskdb://user:password@ip:port/taskdb",
  "projectdb": "mysql+projectdb://user:password@ip:port/projectdb",
  "resultdb": "mysql+resultdb://user:password@ip:port/resultdb",
  "message_queue": "redis://:requirepass@ip/15",
  "phantomjs-proxy": "127.0.0.1:25555",
  "webui": {
    "username": "admin",
    "password": "admin",
    "need-auth": true
  }
}

```

5. 编写启动脚本文件run.py
+ Master
```
#schedular
nohup pyspider -c config.json scheduler  >> scheduler.log 2>&1 &

#phantomjs
nohup pyspider -c config.json phantomjs  >> phantomjs.log 2>&1 &

#fetcher ( 这里可以限定组件的数量 )
for ((i=1; i<=1; i++)); do

nohup pyspider -c config.json --phantomjs-proxy="127.0.0.1:25555" fetcher >> fetcher.log 2>&1 &

pyspider -c config.json processor >> processor.log 2>&1 &

pyspider -c config.json result_worker >> result_worker.log 2>&1 &

done

#webui
pyspider -c config.json webui >> webui.log 2>&1 &
```
+ 授予脚本权限
```
# chmod 755 run.sh
```
+ Slave
```
#phantomjs
nohup pyspider -c config.json phantomjs  >> phantomjs.log 2>&1 &

#fetcher ( 这里可以限定组件的数量 )
for ((i=1; i<=1; i++)); do

nohup pyspider -c config.json --phantomjs-proxy="127.0.0.1:25555" fetcher >> fetcher.log 2>&1 &

pyspider -c config.json processor >> processor.log 2>&1 &

pyspider -c config.json result_worker >> result_worker.log 2>&1 &

done
```
+ 授予脚本权限
```
# chmod 755 run.sh
```

6. 编写停止脚本文件kill.py
+ Master和Slave
```
# sudo ps -ef|grep pyspider|grep -v grep | awk {'print $2'}| xargs kill
```
+ 授予脚本权限
```
# chmod 755 kill.sh
```

7. 开启并测试
```
# ./run.py
```

# 参考资料 #
[PySpider中文网](http://www.pyspider.cn/book/pyspider/Deployment-6.html)
[PySpider官方文档](http://docs.pyspider.org/en/latest/Deployment/)
