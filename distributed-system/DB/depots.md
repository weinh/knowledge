# 分库
## myCat安装
```
[root@localhost opt]# wget http://dl.mycat.io/1.6.5/Mycat-server-1.6.5-release-20171117203123-linux.tar.gz
[root@localhost opt]# tar -xvf Mycat-server-1.6.5-release-20171117203123-linux.tar.gz
```
mycat也需要JDK，那么安装一个1.8的JDK，然后修改下配置文件wrapper.conf
```
wrapper.java.additional.5=-XX:MaxDirectMemorySize=512M
```
因为是虚拟机内存设置的小点

启动myCat，通过日志查看启动结果
```
[root@localhost bin]# ./mycat start
Starting Mycat-server...
[root@localhost bin]# tail -f ../logs/wrapper.log
```
然后链接myCat，可以通过工具，也可以通过命令行，链接方式和mysql一样，默认用户名密码root/123456，默认端口8066
