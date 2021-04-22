# 修改macOS Mysql的 root用户密码

# 修改macOS Mysql的 root用户密码

## 1. 关闭mysql服务

<img src="https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/20210422221252.png" style="zoom:80%;" />

## 2. 禁止mysql验证功能

```bash
mbp:~ mr.x$ cd /usr/local/mysql
# 输入苹果密码进入root用户
mbp:mysql mr.x$ sudo su
Password:
sh-3.2# cd bin/
sh-3.2# ./mysqld_safe --skip-grant-tables &
2021-04-22T14:09:36.6NZ mysqld_safe Logging to '/usr/local/mysql/data/mbp.lan.err'.
2021-04-22T14:09:36.6NZ mysqld_safe Starting mysqld daemon with databases from /usr/local/mysql/data
……
```

回车后mysql会自动重启（偏好设置中**mysql的状态会变成running**）（注意：设置完成以后就不能手动关闭了，除非开机重起）

## 3.刷新权限列表,重置新密码

普通用户执行`. /usr/local/bin/mysql`命令

```sql
mbp:bin mr.x$ ./mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 196
Server version: 5.7.28-log MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
mysql> flush PRIVILEGES;
Query OK, 0 rows affected (0.01 sec)

mysql> SET PASSWORD FOR 'root'@'localhost'='12345678'
    -> ;
Query OK, 0 rows affected (0.01 sec)

mysql>
mysql> exit
```

## 4. 验证

- 命令行工具

```shell
mbp:bin mr.x$ ./mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 251
Server version: 5.7.28-log MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. 
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| 用户               |
| mysql              |
| performance_schema |
| sys                |
| test01             |
+--------------------+
6 rows in set (0.01 sec)
```

- navicat 连接
![](https://raw.githubusercontent.com/SeaSoonKeun/Picture/main/Blog_Pic/20210423013520.png)
