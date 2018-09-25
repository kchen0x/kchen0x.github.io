title: AWS EC 主机无法通过 PHP 连接 RDS MySQL
date: 2016/12/02 09:24:07
updated: 
tags:
    - AWS
    - PHP
    - RDS
    - MySQL
    - WordPress
---

 ## 问题描述

 > 当我们选择 AWS EC 作为服务器主机，而 AWS RDS 作为 MySQL 服务器时，如果通过 PHP 访问 MySQL 可能会出现 「Error establishing a database connection」错误，这是使用 WordPress 建站时常见的数据库连接错误。

 <!--more-->

 ## 解决方案

 一、确认自己的安全组权限

 在 RDS 中需要为自己的数据库配置安全组权限，也即 `3306` 端口需要向 EC 主机开放：

![SG](http://data.kchen.cc/mac:rt67ujbt678ijbt78ijbty.png-960.jpg)

其中，选择类型为 `MYSQL/Aurora`，这样可以开启 `3306` 端口，把来源定义为我的 IP 以及 EC 主机所在安全组。（只需要键入主机名，AWS 可以自动适配）

二、关闭安全增强式 Linux（SELinux）

AWS EC 默认开启了 SELinux 是使得 PHP 服务器不能访问外部 MySQL 的元凶。使用 `root` 用户修改 EC 主机 `/etc/selinux/config` 文件中：

```
SELinux = disabled
```

重启 EC 主机，重新开启 httpd 服务即可。