---
title: mysql 免安装配置学习
date: 2020-07-27
banner_img: /img/mysql.jpg
index_img: /img/mysql.png
tags: 
 - mysql-install
categories:
 - basic-component
 - mysql
---

## [MySQL5.7绿色版（免装版）的初始化和修改密码](https://www.cnblogs.com/jyiqing/p/6924062.html)



**1、下载**

（1）下载地址：https://dev.mysql.com/downloads/mysql/

（2）选择下载

![img](https://img2018.cnblogs.com/blog/1627739/201906/1627739-20190620001542784-897003254.png)

**2、配置环境变量**

（1）解压目录：D:\mysql-8.0.16-winx64

（2）配置环境变量

![img](https://img2018.cnblogs.com/blog/1627739/201906/1627739-20190620002109235-1598560339.png)

**3、添加配置文件**

（1）配置文件目录：D:\mysql-8.0.16-winx64

（2）配置文件名：my.ini

（3）文件内容：

```
[mysql]``# 设置mysql客户端默认字符集``default-character-``set``=utf8` `[mysqld]``# 绑定IPv4``bind-address=0.0.0.0``# 设置端口号``port=3306``# 设置mysql的安装目录，即解压目录``basedir=D:\\mysql-8.0.16-winx64``# 设置数据库的数据存放目录``datadir=D:\\mysql-8.0.16-winx64\\data``# 设置允许最大连接数``max_connections=200``# 设置允许连接失败次数``max_connect_errors=10``# 设置服务端的默认字符集``character-``set``-server=utf8``# 创建表使用的默认存储引擎``default-storage-engine=INNODB``# 使用“mysql_native_password”插件认证``default_authentication_plugin=mysql_native_password
```

**4、CMD命令窗口配置mysql**

（1）初始化mysql（记住随机密码）：**mysqld --initialize --console**

 ![img](https://img2018.cnblogs.com/blog/1627739/201906/1627739-20190620014312114-157191049.png)

随机密码为：%tbmq)f<;3jE

目录会多处data文件夹

（2）安装mysql服务：**mysqld --install mysql --defaults-file=****"D:\mysql-8.0.16-winx64\my.ini"**

**![img](https://img2018.cnblogs.com/blog/1627739/201906/1627739-20190620014706448-303230754.png)**

切换管理员身份运行

![img](https://img2018.cnblogs.com/blog/1627739/201906/1627739-20190620015148365-1323561363.png)

（3）启动mysql服务：**net start mysql**

**![img](https://img2018.cnblogs.com/blog/1627739/201906/1627739-20190620015241272-592406864.png)**

（4）登录mysql：**mysql -uroot -p**

密码为之前的随机密码：：%tbmq)f<;3jE

（5）修改密码：**ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';**

![img](https://img2018.cnblogs.com/blog/1627739/201906/1627739-20190620020223252-138085659.png)

退出重新登录

![img](https://img2018.cnblogs.com/blog/1627739/201906/1627739-20190620021250850-1721675715.png)

**5、其他命令**

（1）关闭服务：net stop mysql

（2）卸载服务：mysqld --remove