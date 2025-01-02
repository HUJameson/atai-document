# CentOS7.9环境下MySQL8.4的安装

这篇文档旨在记录MySQL的安装过程。

参考资料：https://blog.csdn.net/zp8126/article/details/137084854

以上参考资料来自互联网，该参考资料是MySQL8.0的安装过程，和MySQL8.4的安装稍有不同。不同之处如下：

1）没有遇到GPG密钥验证问题。

2）给用户设置密码时遇到错误：plugin 'mysql_native_password' is not loaded（后文附有解决办法）

MySQL 8.4的全部安装过程如下：

Step 1: 检查本机是否已经安装了mysql/mariadb，如果是则先卸载。（如果是全新环境，可以跳过）

检查命令

# rpm -qa | grep mysql

# rpm -qa | grep mariabd

卸载命令

# rpm -e --nodeps 已经安装程序名称

Step 2：检查本机是否已经安装了wget，如果没有则先安装。

检查命令

# wget -V

安装命令

# yum install -y wget

Step 3：修改yum源为阿里云

先备份

# mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak

再下载阿里云yum源文件

# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

清理yum缓存，重新生成

# yum clean all

# yum makecache

Step 4：安装mysql源

下载mysql源，到当前目录（一般是～），来源：https://dev.mysql.com/downloads/repo/yum/

检查本机是否已经有

# rpm -qa -i mysql

下载命令

# wget http://dev.mysql.com/get/mysql84-community-release-el7-1.noarch.rpm

验证命令

# ls

安装mysql源

# yum localinstall -y mysql84-community-release-el7-1.noarch.rpm

验证命令

# rpm -qa |grep mysql

# yum repolist enabled | grep mysql

Step 5：安装和设置mysql

安装命令

# yum install -y mysql-community-server

验证命令

# rpm -qa |grep mysql

启动mysql

# systemctl start mysqld

设置开机启动

# systemctl enable mysqld

重新加载配置文件

# systemctl daemon-reload

开启3306端口

因为环境是阿里云ECS，所以需要在该ECS所在的安全组中，为3306增加入方向规则

Step 6：为mysql的root用户设置密码和权限

查看自动生成的临时密码

# cat /var/log/mysqld.log | grep password

尝试登录

# mysql -uroot -p （有密码登录）

修改mysql的root用户的密码（密码的要求看提示）

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '######';

此时会遇到错误：plugin 'mysql_native_password' is not loaded

通过如下方式解决

退出mysql

mysql> quit

在mysql配置文件（/etc/my.cnf  [mysqld]）增加：mysql_native_password=ON

# vim /etc/my.cnf

重启服务器，登录mysql，重复修改mysql的root用户的密码的过程，然后重新登录mysql验证

添加远程访问的权限（3条命令）

mysql> create user 'root'@'%' identified with mysql_native_password by '######';
mysql> grant all privileges on *.* to 'root'@'%' with grant option;
mysql> flush privileges;

远程登录验证（从mac笔记本的终端，或者从Navicat图形界面）

经过以上6个步骤，MySQL 8.4 在CentOS 7.9上安装成功。


