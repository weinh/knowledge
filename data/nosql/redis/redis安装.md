# redis安装
## 单机
### 前期准备
下载解压安装包
```shell
[root@localhost opt]# wget http://download.redis.io/releases/redis-4.0.10.tar.gz
[root@localhost opt]# tar -xvf redis-4.0.10.tar.gz
```
安装依赖包
```shell
[root@localhost utils]# yum install gcc
```
### 安装
安装
```shell
[root@localhost redis-4.0.10]# make MALLOC=libc
[root@localhost redis-4.0.10]# make install
```
修改配置
```shell
[root@localhost utils]# vim redis.conf
# 修改绑定IP
bind 0.0.0.0
# 设置为守护进程
daemonize yes
# 开启AOF
appendonly yes
# 设置密码
requirepass 123456
[root@localhost utils]# vim redis_init_script.tpl
# 停止命令增加权限验证
$CLIEXEC -p $REDISPORT -a 123456 shutdown
```
执行部署脚本，那些提示都回车选择默认就好
```shell
[root@localhost utils]# ./install_server.sh
```
### 验证
```shell
[root@localhost redis]# redis-cli -a 123456
127.0.0.1:6379> info
```
