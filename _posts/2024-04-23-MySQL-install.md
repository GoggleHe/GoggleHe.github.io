---
layout: post
title: Linux安装MySQL
categories: [MySQL]
description: Linux安装MySQL
keywords: MySQL
---

## Linux安装MySQL

### 1、下载安装包

https://downloads.mysql.com/archives/community/

### 2、安装openssl-devel

```bash
yum install openssl-devel
```

### 3、安装RPM包

```bash
rpm -ivh mysql-community-common-8.2.0-1.el7.x86_64.rpm

rpm -ivh mysql-community-client-plugins-8.2.0-1.el7.x86_64.rpm

rpm -ivh mysql-community-libs-8.2.0-1.el7.x86_64.rpm

rpm -ivh mysql-community-libs-compat-8.2.0-1.el7.x86_64.rpm

rpm -ivh  mysql-community-devel-8.2.0-1.el7.x86_64.rpm

rpm -ivh mysql-community-client-8.2.0-1.el7.x86_64.rpm

rpm -ivh  mysql-community-icu-data-files-8.2.0-1.el7.x86_64.rpm

rpm -ivh  mysql-community-server-8.2.0-1.el7.x86_64.rpm
```

### 4、启动MySQL服务

```bash
systemctl start mysqld
```

检查`ps -aux | grep mysql`

### 5、连接MySQL

```bash
#查询临时密码
cat /var/log/mysqld.log | grep password
#连接MySQL
 mysql -u root -p
```

### 6、卸载

```bash
#关闭MySQL服务
systemctl stop mysqld
#查询安装的MySQL服务
rpm -qa | grep -i mysql
#卸载查询出来的所有服务
rpm -e mysql-* --nodeps
#删除MySQL的数据存放目录
rm -rf /var/lib/mysql/
#删除MySQL的配置文件备份
rm -rf /etc/my.cnf.rpmsave
```

