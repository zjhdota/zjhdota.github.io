---
title: Centos7 部署MySQL
tags:
  - MySQL
  - CentOS7
categories:
  - 学习笔记
date: 2018-02-14 18:02:55
---

# 操作系统 #
* CentOS Linux release 7.4.1708 (Core)

# 下载并安装MySQL
+ [MySQL RPM 包下载地址](https://dev.mysql.com/downloads/repo/yum/)
1. 从官网下载MySQL RPM包
```
# wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```
2. 校验PRM是否正确
```
# md5sum mysql57-community-release-el7-11.noarch.rpm
```
3. 安装RPM
```
# rpm -ivh mysql57-community-release-el7-11.noarch.rpm
```
4. 安装MySQL
```
# yum install mysql-server
```
# 配置MySQL #
1. 将MySQL加入守护进程
```
# systemctl start mysqld
```

2. 检测MySQL服务是否启动
```
# netstat -tuln | grep 3306
```

3. 获取root的默认密码
```
# grep 'temporary password' /var/log/mysqld.log
```

4. 配置MySQL服务器
```
# mysql_secure_installation
```
5. 进入交互式界面进行MySQL配置
+ 是否重置root密码
+ 规则：输入一个包含至少一个大写字母，一个小写字母，一个数字和一个特殊字符的新的8个字符的密码。
+ ![](http://upload-images.jianshu.io/upload_images/7995580-04e7e2afcd759105.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ 是否删除匿名用户
+ ![](http://upload-images.jianshu.io/upload_images/7995580-8e8fe9ac1d08dc0c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ 是否删除test服务器
+ ![](http://upload-images.jianshu.io/upload_images/7995580-eebe08f0d36211a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ 是否重设对表的权限
+ ![](http://upload-images.jianshu.io/upload_images/7995580-c207d5ad25d50844.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6. 测试
```
# mysqladmin -u root -p version
```
7. 设置开机自启
```
# systemctl enable mysqld.service
```
# 建立用户 #
1. 登录
```
# mysql -uroot -p
```

2. 建立用户
```
mysql> CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
```
3. 授权
```
mysql> GRANT ALL ON *.* TO 'username'@'localhost' ;
```
4. 刷新缓存
```
mysql> FLUSH PRIVILEGES;
```
# 允许远程登录 #
1. 授权
```
mysql> GRANT ALL ON *.* TO 'username'@'%' IDENTIFIED BY 'password';
```
2. 查看用户信息
```
mysql> use mysql;
mysql> select user,host,authentication_string from user;
```
3. 配置iptables
```
# vi /etc/sysconfig/iptables
```
+ 加入
```
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
```
4. 测试
```
# mysql -u username -p password -h ip -P port
```
# 参考资料 #
+ [How To Install MySQL on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-centos-7)
