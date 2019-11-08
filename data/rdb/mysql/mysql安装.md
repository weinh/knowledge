# mysql安装
## 前期准备
下载解压安装包
```shell
[root@localhost opt]# wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.27-1.el7.x86_64.rpm-bundle.tar
[root@localhost opt]# tar -xvf mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar
```
## 安装
依次执行
```shell
[root@localhost opt]# rpm -ivh mysql-community-common-5.7.23-1.el7.x86_64.rpm
[root@localhost opt]# rpm -ivh mysql-community-libs-5.7.23-1.el7.x86_64.rpm
[root@localhost opt]# rpm -ivh mysql-community-client-5.7.23-1.el7.x86_64.rpm
[root@localhost opt]# rpm -ivh mysql-community-server-5.7.23-1.el7.x86_64.rpm
```
## 问题处理
本次安装过程中出现一些错误，包冲突的问题，需要先移除包再安装，移除包方法
```shell
[root@localhost opt]# yum remove mariadb-libs
[root@localhost opt]# yum install libaio
```
## 启动配置
启动服务
```shell
[root@localhost opt]# service mysqld start
```
5.7的mysql被设置了默认密码，查看密码并登录
```shell
[root@localhost opt]# grep "temporary password" /var/log/mysqld.log
[root@localhost opt]# mysql -u root -p
```
登录强制修改密码，然而密码有安全控制，修改部分配置，可以设置一个简单密码
```shell
mysql> set global validate_password_policy=0;
mysql> set global validate_password_length=1;
mysql> set global validate_password_mixed_case_count=2;
mysql> SET PASSWORD = PASSWORD("123456");
mysql> flush privileges;
mysql> grant all privileges on *.* to 'root'@'%' identified by '123456';
mysql> flush privileges;
```
退出再次登录，大功告成

## 非root安装
准备
```shell
[hsyq@jiaotou-40 mysql]$ rpm2cpio mysql-community-common-5.7.25-1.el6.x86_64.rpm | cpio -idvm
[hsyq@jiaotou-40 mysql]$ rpm2cpio mysql-community-client-5.7.25-1.el6.x86_64.rpm | cpio -idvm
[hsyq@jiaotou-40 mysql]$ rpm2cpio mysql-community-libs-5.7.25-1.el6.x86_64.rpm | cpio -idvm
[hsyq@jiaotou-40 mysql]$ rpm2cpio mysql-community-server-5.7.25-1.el6.x86_64.rpm | cpio -idvm
```
配置环境变量
```shell
[hsyq@jiaotou-40 ~]$ vim .bash_profile
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin:/home/hsyq/middleware/mysql/usr/bin:/home/hsyq/middleware/mysql/usr/sbin

export PATH
```
修改配置文件
```shell
[hsyq@jiaotou-40 etc]$ vim my.cnf
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
basedir=/home/hsyq/middleware/mysql/usr
datadir=/home/hsyq/middleware/mysql/var/lib/mysql
socket=/home/hsyq/middleware/mysql/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/home/hsyq/middleware/mysql/var/log/mysqld.log
pid-file=/home/hsyq/middleware/mysql/var/run/mysqld/mysqld.pid

secure-file-priv=/home/hsyq/middleware/mysql/var/lib/mysql-files

init-connect=\'SET NAMES utf8\'
collation_server=utf8_unicode_ci
character_set_server=utf
```
安装，启动，登录
```shell
[hsyq@jiaotou-40 ~]$ mysqld --defaults-file=/home/hsyq/middleware/mysql/etc/my.cnf --initialize --user=hsyq
[hsyq@jiaotou-40 ~]$ mysqld --defaults-file=/home/hsyq/middleware/mysql/etc/my.cnf --user=hsyq &
[hsyq@jiaotou-40 ~]$ mysql -u root -p
```