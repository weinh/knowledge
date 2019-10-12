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
```
退出再次登录，大功告成