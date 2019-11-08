# nginx安装
## 前期准备
下载解压安装包
```shell
[root@localhost opt]# wget http://nginx.org/download/nginx-1.14.0.tar.gz
[root@localhost opt]# tar -vxf nginx-1.14.0.tar.gz
```
## 安装
```shell
[root@localhost nginx-1.14.0]# ./configure
```
提示报错，安装依赖包
```shell
[root@localhost nginx-1.14.0]# yum install pcre pcre-devel
[root@localhost nginx-1.14.0]# yum install zlib zlib-devel
```
```shell
[root@localhost nginx-1.14.0]# make && make install
```
启动
```shell
[root@localhost sbin]# ./nginx
```
## 非root用户下安装
安装依赖，指定安装目录（其他目录没有权限）
```shell
[hsyq@jiaotou-40 pcre-8.40]$ ./configure --prefix=/home/hsyq/middleware/pcre-8.40
[hsyq@jiaotou-40 pcre-8.40]$ make && make install
[hsyq@jiaotou-40 zlib-1.2.11]$ ./configure --prefix=/home/hsyq/middleware/zlib-1.2.11
[hsyq@jiaotou-40 zlib-1.2.11]$ make && make install
```
安装nginx，指定安装目录，指定依赖模块源码目录
```shell
[hsyq@jiaotou-40 nginx-1.16.0]$ ./configure --prefix=/home/hsyq/middleware/nginx-1.16.0 --with-pcre=/home/hsyq/soft/pcre-8.40 --with-zlib=/home/hsyq/soft/zlib-1.2.11
[hsyq@jiaotou-40 nginx-1.16.0]$ make && make install
```