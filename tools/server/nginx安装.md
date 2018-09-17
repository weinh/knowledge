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