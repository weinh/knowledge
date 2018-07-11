# mongodb安装
## 单机部署
下载软件包，解压，配置环境变量
```shell
[root@localhost opt]# wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.6.0-rc7.tgz
[root@localhost opt]# tar -vxzf mongodb-linux-x86_64-3.6.0-rc7.tgz
```
安装部署之前简单说下mongodb的安全机制，默认mongodb是没有密码的，所以我们需要主动添加用户并设置密码，mongodb有两个特殊的数据库`admin`和`local`这下面的用户可以访问所有数据库，相当于超级管理员

那么我们先部署一个无密码的mongodb，创建超级管理员后再开启密码验证，手动创建下用于存储mongodb日志或mongodb数据的目录（mongodb不会自动创建），创建配置文件mongodb.conf（不开启验证）
```shell
[root@localhost opt]# mkdir -p  /opt/mongodb/db/logs
[root@localhost opt]# mkdir -p  /opt/mongodb/db/data
[root@localhost db]# vim mongodb.conf
```
配置文件内容如下
```yaml
systemLog:
   path: /opt/mongodb/db/logs/mongod.log
   logAppend: true
   destination: file
storage:
   dbPath: /opt/mongodb/db/data
net:
   port: 27017
   bindIpAll: true
processManagement:
   fork: true
#先关闭验证
#security:
#   authorization: enabled
```
然后启动
```shell
[root@localhost data]# mongod -f mongodb.conf
```
客户端链接，默认本机IP，27017端口
```shell
[root@localhost log]# mongo
```
建立一个管理员用户，如下
```shell
> use admin
switched to db admin
> db.createUser({
    user: "admin",
    pwd: "admin",
    roles: [{
        role: "userAdminAnyDatabase",
        db: "admin"
    }]
})
```
停止mongodb，开启验证，启动mongodb

客户端链接，创建自定义用户管理自定义数据库
```shell
[root@iZbp173lfys1bmejzp9g4fZ dev]# mongo -u admin -p admin --authenticationDatabase "admin"
> use test
switched to db test
> db.createUser({
    user: "test",
    pwd: "test",
    roles: [{
        role: "readWrite",
        db: "test"
    }]
})
```
切换自定义用户
```shell
[root@iZbp173lfys1bmejzp9g4fZ dev]# mongo -u test -p test --authenticationDatabase "test"
```
然后用户test，就能操作数据库test了
## 副本集部署
由于主从（`Master-Slave`）的方式已经过时了被副本集（`Replica Sets`）替代了，副本集除了使用主从方式部署，当主机挂掉后从机经过选举将会成为新的主机从而达到失败转移效果，新版本的mongodb的YAML配置文件已经不支持主从部署了，那么这里简单说明下副本集的部署

演示三个节点的副本集，设备有限建三组目录：
```shell
[root@hsipcc mongodb]# mkdir -p /opt/mongodb/r-s/node1/data
[root@hsipcc mongodb]# mkdir -p /opt/mongodb/r-s/node1/logs
[root@hsipcc mongodb]# mkdir -p /opt/mongodb/r-s/node2/data
[root@hsipcc mongodb]# mkdir -p /opt/mongodb/r-s/node2/logs
[root@hsipcc mongodb]# mkdir -p /opt/mongodb/r-s/node3/data
[root@hsipcc mongodb]# mkdir -p /opt/mongodb/r-s/node3/logs
```
每个节点的配置文件分别放到节点目录下内容如下：node1/mongod.conf 
```yaml
systemLog:
   path: /opt/mongodb/r-s/node1/logs/mongod.log
   logAppend: true
   destination: file
storage:
   dbPath: /opt/mongodb/r-s/node1/data
net:
   port: 27017
   bindIpAll: true
processManagement:
   fork: true
replication:
   replSetName: testSet
```
node2/mongod.conf 
```yaml
systemLog:
   path: /opt/mongodb/r-s/node2/logs/mongod.log
   logAppend: true
   destination: file
storage:
   dbPath: /opt/mongodb/r-s/node2/data
net:
   port: 27027
   bindIpAll: true
processManagement:
   fork: true
replication:
   replSetName: testSet
```
node3/mongod.conf 
```yaml
systemLog:
   path: /opt/mongodb/r-s/node3/logs/mongod.log
   logAppend: true
   destination: file
storage:
   dbPath: /opt/mongodb/r-s/node3/data
net:
   port: 27037
   bindIpAll: true
processManagement:
   fork: true
replication:
   replSetName: testSet
```
然后分别启动，这三个节点，到这里位置，节点启动完成，不过还不知道那个节点是主，那个节点是从，那个节点是裁判，需要进行以下设置
```shell
# 登陆node1设置为主机
[root@hsipcc r-s]# mongo
> rs.initiate()
testSet:SECONDARY> rs.conf()
testSet:PRIMARY>
# 上面执行完后，发现已经变成了主机，接下来添加从机
testSet:PRIMARY> rs.add("192.168.37.248:27027")
testSet:PRIMARY> rs.add("192.168.37.248:27037")
# 这里好像必须要填IP而不能是127.0.0.1，虽然127.0.0.1也绑定了对应端口
# 登录到从节点，设置可读属性
[root@hsipcc ccrm]# mongo -port 27027
testSet:SECONDARY> db.getMongo().setSlaveOk()
主机新增一个集合插入一条数据，从机查看下结果看是否同步了
```
下面测试下故障转移，主机通过`rs.conf()`可以看到成员列表，其中有个属性priority表示优先级0-100，数字越大优先级越高，默认为1，0的话不会参与选举，设置其中一个从机priority为2，进行故障转移
```shell
[root@hsipcc r-s]# mongo
testSet:PRIMARY> cfg = rs.conf()
testSet:PRIMARY> cfg.members[2].priority = 2
testSet:PRIMARY> rs.reconfig(cfg)
# 再次查看下第三个节点的优先级为2
# 登陆第三个节点
[root@hsipcc ccrm]# mongo -port 27037
testSet:PRIMARY> 
# 可以看到node3已经成为了主机，再次启动node1看看
[root@hsipcc r-s]# mongod -f /opt/mongodb/r-s/node1/mongod.conf
[root@hsipcc r-s]# mongo
testSet:SECONDARY>
# 可以看到node1变成了从机，到这里故障转移演示成功，为了顺利的进行故障转移，节点数必须为奇数，如果数据节点是偶数那么可以增加裁判节点
```
到目前位置mongodb还是裸奔的状态，我们需要增加验证机制
```shell
# 先登陆主机添加超级管理员
[root@hsipcc ccrm]# mongo -port 27037
testSet:PRIMARY> use admin
switched to db admin
testSet:PRIMARY> db.createUser({
...     user: "admin",
...     pwd: "admin",
...     roles: [{
...         role: "userAdminAnyDatabase",
...         db: "admin"
...     }]
... })
Successfully added user: {
  "user" : "admin",
  "roles" : [
    {
      "role" : "userAdminAnyDatabase",
      "db" : "admin"
    }
  ]
}
testSet:PRIMARY> 
# 创建一个密钥文件，注意这里生成的密钥的参数是21，尽量使用3的倍数，防止出现非法字符导致认证失败
[root@hsipcc r-s]# openssl rand -base64 21 > keyfile
[root@hsipcc r-s]# chmod 600 keyfile
[root@hsipcc r-s]# cat keyfile 
z6WVlpvb1UCOHhDtqdpi0UtVBmla
# 然后修改下几个节点的配置文件，统一增加如下内容
security:
   keyFile: /opt/mongodb/r-s/keyfile
   authorization: enabled
```
重启下各个节点，使用密码登陆主节点
```shell
# 创建test数据管理员
[root@hsipcc ccrm]# mongo -port 27037 -u admin -p admin --authenticationDatabase "admin"
testSet:PRIMARY> db.createUser({
...     user: "test",
...     pwd: "test",
...     roles: [{
...         role: "readWrite",
...         db: "test"
...     }]
... })
# 使用test登陆数据库，对test进行操作
[root@hsipcc ccrm]# mongo -port 27037 -u test -p test --authenticationDatabase "test"
testSet:PRIMARY> use test
switched to db test
testSet:PRIMARY> db.c1.find()
{ "_id" : ObjectId("5b2233cc6f2dba7a1aa0222d"), "name" : "hi" }
# 登陆子节点验证同步效果
[root@hsipcc ccrm]# mongo -port 27027 -u test -p test --authenticationDatabase "test"
testSet:SECONDARY> use test
# 这里需要执行设置可读操作，重启后读操作会报错需要再次执行
testSet:SECONDARY> db.setSlaveOk()
testSet:SECONDARY> db.c1.find()
{ "_id" : ObjectId("5b2233cc6f2dba7a1aa0222d"), "name" : "hi" }
```
## 分片集群部署