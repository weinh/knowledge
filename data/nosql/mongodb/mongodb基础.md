# mongodb
## 安装
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

## 基础命令
* db.`collectionName`.insert
> 插入文档，集合不存在会创建集合
* db.`collectionName`.find
> 查询集合所有文档
* db.`collectionName`.findOne
> 查询集合中一个文档
* db.`collectionName`.remove
> 根据条件删除文档，集合还存在，可能是空集合
* db.`collectionName`.update
> 根据条件更新文档，文档替换，修改器可以实现局部更新，还可以设置存在更新，不存在插入的功能（第三个参数控制），第四个参数控制更新多个文档
* db.`collectionName`.batchInsert
> 批量插入文档
* db.`collectionName`.drop
> 删除集合
* db.`collectionName`.save
> 参数含有_id调用upsert，否则调用insert
* db.getCollention
> 部分保留字或无效的js名称，可以使用该方式获取集合
* db.runCommand
> 执行相关命令


## 基本数据类型
* null
* 布尔型
* 数值
* 字符串
* 日期
```js
{
    "x":new Date()
}
```
* 正则表达式
```js
{
    "x":/D/
}
```
* 数组
* 内嵌文档
* 对象id
* 二进制数据
* 代码

## db.`collectionName`.update修改器详解
* $inc
> 累加数值，只能是数值类型
* $set
> 更新指定key，key不存在设置key
* $unset
> 删除key
* $setOnInsert
> 创建文档的同时赋值，后面的所有更新操作不会修改，比如创建时间很有效果，如果不存在ObjectId属性的_id

### 数组修改器
* $push
> 增加元素
* $each
> 批量增加元素
* $slice
> 固定长度的数组，必须是负整数，需要配合$each使用
* $sort
> 排序
* $ne
> 添加到集合，不会重复
* $addToSet
> 配合$each，可以批量添加到集合
* $pop
> 删除元素
* $pull
> 匹配文档，删除多个
* $
> 基于位置的修改器，定位符只匹配第一个文档

## db.runCommand命令详解
* getLastError获取最后一次执行情况
```js
db.runCommand({
  "getLastError":1
})
```
* collMod分配之后的快空间为2的幂，适用于经常打乱数据的集合
```js
db.runCommand({
  "collMod":`collectionName`,
  "usePowerOf2Sizes":true
})
```
* findAndModify获取并修改匹配的数据
```js
db.runCommand({
  "findAndModify":`collectionName`,
  "query":{},
  "sort":{},
  "update":{}
})
```
> findAndModify有很多可选字段
> 
> * query查询条件
> * sort排序
> * update修改器
> * remove删除文档，布尔类型，标识是否删除（和update只能存在一个）
> * new布尔类型，返回更新前还是更新后的文档，默认更新前的文档
> * fields指定文档中需要返回的字段
> * upsert布尔类型，true表示是upsert，默认false
>
> 如果没有找到匹配的文档，该命令将返回错误

## db.`collectionName`.find/findOne命令详解
* 第一个参数指定查询条件，默认{}查询全部
* 第二个参数，表示指定返回键`{"unshow":0,"show":1}`，`_id`默认存在，可以设置0进行隐藏
* 查询限制，查询文档必须是常量，也就是不能引用文档中的其他键的值

### 查询条件
* `$lt`，`$lte`，`$gt`，`gte`，`$ne`对应于，<，<=，>，>=，<>可以组合起来实现范围查询，查询类型要匹配，字段什么类型条件必须什么类型
```js
db.user.find("age": {
  "$gt": 18,
  "$lt": 30
})

start = new Date("2017-01-01")
db.user.find("regDate": {
  "$gte": start
})
```
* OR查询有两种方式，`$or`和`$in`，`$in`可以设置键匹配多个值，可以不同类型
```js
db.user.find("id": {
  "$in": [1, 2, "3"]
})
```
* `$nin`是和`$in`相反的操作，和给出的值都不匹配
* `$in`可以指定某个键等于这个值或那个值，那么`$or`可以指定这个键等于这个值或那个键等于那个值，可以嵌套子查询
```js
db.user.find({
  "$or": [{
    "name": "abc"
  }, {
    "age": 20
  }]
})
```
* `$not`，就是取非操作，比如在`$mod`中的使用，`$mod`的作用将查询值除第一个值，余数等于第二个值，如果能对应就匹配上
```js
db.user.find({
  "abc": {
    "$not": {
      "$mod": [5, 1]
    }
  }
})
```
* 大部分查询条件都是内层文档的键，除了`$and`，`$or`，`$nor`，都是外层文档的键，查询优化器不会对$and进行优化
```js
db.user.find({
  "$and": [{
    "x": {
      "$lt": 1
    }
  }, {
    "x": 4
  }]
})
```
* 当以上方式都行不通的时候就需要`$where`上场了，它可以执行任意的js脚本，应该在shell终端限制其使用，查询很慢，而且不能用索引
```js
db.user.find({
  "$where": function() {

  }
})
```

### 特定类型的查询
#### null类型数据
查询的特殊处理，null可以查询到值为null的文档，还会查到name这个键不存在的文档
```js
db.user.find({
  "name": null
})

db.user.find({
  "name": {
    "$exists": true,
    "$eq": null
  }
})
```
#### 查询数组
* 包含元素
```js
db.user.insert({
  "name": ["lili", "lucy", "tom"]
})

db.user.find({
  "name": "lili"
})
```
* $all用来匹配多个元素，不分顺序
```js
db.user.insert({
  "group": ["lili", "lucy", "tom"]
})
db.user.insert({
  "group": ["lili",  "tom"]
})
db.user.insert({
  "group": ["lili", "lucy"]
})

db.user.find({
  "group": {
    "$all": ["lili", "tom"]
  }
})
```
* 精确匹配，内容一样，顺序一样
```js
db.user.insert({
  "name": ["lili", "lucy", "tom"]
})

db.user.find({
  "name": ["lili", "lucy", "tom"]
})
```
* 特定位置的元素key.index，下标从0开始
```js
db.user.insert({
  "name": ["lili", "lucy", "tom"]
})

db.user.find({
  "name.1": "lucy"
})
```
* $size查询特定长度的数组
```js
db.user.find({
  "name": {
    "$size": 3
  }
})
```
* $slice可以返回某个键匹配的数组元素的一个子集，以下例子的效果就是返回，角色下面的10个用户
```js
db.role.user.find({}, {
  "user": {
    "$slice": 10
  }
})
```
* $ 返回一个匹配的数组元素，如果角色下面的用户有两个都叫tom，只返回第一个
```js
db.role.user.find({
  "user.name": "tom"
}, {
  "user.$": 1
})
```
* 数组和范围查询的相互作用
```js
db.test.insert({
  "x": 5
})
db.test.insert({
  "x": 15和
})
db.test.insert({
  "x": 25
})
db.test.insert({
  "x": [5, 25]]
})

db.test.find({
  "x": {
    "$gt": 10,
    "$lt": 20
  }
})
//这种方式能够查询到"x": 15和"x": [5, 25]]，因为5符合"$lt": 20，25符合"$gt": 10

//可以通过
db.test.find({
  "x": {
    "$elemMatch": {
      "$gt": 10,
      "$lt": 20
    }
  }
})
//该方式查询不到任何结果，因为"x": 15不是数组

//当如果x键建立过索引可以使用
db.test.find({
  "x": {
    "$gt": 10,
    "$lt": 20
  }
}).min({
  "x": 10
}).max({
  "x": 20
})
//可以查询出"x": 15
```

#### 查询内嵌文档
* 如果查询一个完整的文档必须精确匹配
```js
db.blog.insert({
  "content": "mongodb",
  "comments": [{
    "author": "tom",
    "score": 3,
    "comment": "good"
  }, {
    "author": "june",
    "score": 6,
    "comment": "ok"
  }]
})

db.blog.find({
  "comments": {
    "author": "tom",
    "score": 3,
    "comment": "good"
  }
})
```
* 使用`.`表示进入文档的意思，可以根据内嵌文档特定的键进行查询，所以如果内容包含`.`需要做全局替换
```js
//查询无结果
db.blog.find({
  "comments.author": "tom",
  "comments.score": 4
})

//查询有结果
db.blog.find({
  "comments.author": "tom",
  "comments.score": 6
})
```
* $elemMatch将限定条件进行分组，仅当需要一个内嵌文档的多个键同时满足要求
```js
db.blog.find({
  "comments": {
    "$elemMatch": {
      "author": "tom",
      "score": 3
    }
  }
})
```

### 服务器端脚本
一个重点，需要防止脚本注入，类似SQL注入的风险

### 游标
可以将查询的结果放在游标中，而不是通过终端直接打印显示内容，可以通过`hasNext()`检查是否还有值，通过`next()`去下个值，还能通过forEach直接迭代

通过find()方法并没有直接查询数据库，而是在真正获取数据的时候才查询，所以可以在执行之前可以给查询附加一些选项比如排序，限制返回数量，几乎每个方法都返回游标本身
```js
db.user.find().sort("age": 1).limit(10).skip(10)
db.user.find().sort("age": 1).skip(10).limit(10)
db.user.find().limit(10).skip(10).sort("age": 1)
```
这些查询效果都是等价的，而且还没有开始进行查询，等调用hasNext()方法的时候才查询

* limit控制查询文档数量
* skip忽略一定数量的文档，略过过多的文档将出现性能问题
> 要解决这个问题，可以在查询上下功夫，上一个查询结果作为下一个查询的条件，使用偏移量的形式查询
* sort指定按那个字段排序参数是键值对，键是字段名，值1（升序）或-1（降序），可以指定多个键，会依次排序
* 随机获取文档，最方便的最方便的做法就是，取一个0-文档总量之间的随机数，然后通过skip和limit来获取一个
> 前面说过skip可能很慢，而且获取总量如果有条件的查询也可以比较慢，那么可以使用下面的方式变通下
```js
db.user.insert({
  "name": "tom",
  "wight": Math.random()
})
var random = Math.random()
var result = db.user.findOne({
  "wight": {
    "$gt": random
  }
})
if (result == null) {
  result = db.user.findOne({
    "wight": {
      "$lt": random
    }
  })
}
```
* `$maxscan`指定本次查询中扫描文档数量的上限，可以减少查询耗时，可是可能部分文档扫描不到
* `$min`，`$max`，文档必须和索引的键完全匹配
* 获取游标后，然后循环更新（大量），那么同一个文档可能多次被访问到
> 问题的原因是，更新的时候文档突然变大，原先的位置无法存放，放到了文档最后，那么就会被再次访问
> 
> 可以通过快照解决，db.user.find().snapshot()，快照的查询很慢，慎用，

* 游标对于服务器而言就是资源，如果游标迭代完毕会被回收，如果游标超过一定时间没有使用也会被回收，也可以设置超时不被回收直到迭代完成，资源有限如果游标太多没有回收将会资源耗尽

### 数据库命令
数据库命令是通过db.runCommand命令下执行的，可以通过db.listCommands命令查看所有的数据库命令

数据库命令其实是在$cmd集合上进行查询，只不过是使用特殊的方式执行命令，而不是查询处理，部分命令（比如shutdown）只能在admin库中执行才有效，如果需要在普通库执行特殊命令，需要使用db.adminCommand的方式执行

## 索引
* db.`collectionName`.ensureIndex命令创建索引，创建索引比较慢的话，可以通过db.currentOp()查看创建进度
* db.`collectionName`.find().explain()命令可以查看查询效率
* 索引的建立使查询变得快速了，可是对于插入，更新，删除都会有性能的影响，所以应该建立有必要是索引，特别是那些常用的查询字段
* 索引对于排序非常快，不过需要首先使用索引字段排序才有意义
```js
db.users.find().sort({
  "age": 1,
  "username": 1
})
//如果索引是建在username上那么，是没有太大意义的，所以引入了复合索引的概念
db.users.ensureIndex({
  "age": 1,
  "username": 1
})
```
* 使用覆盖索引，一个索引包含用户请求的所有字段，索引包含数组字段，那么永远也无法覆盖查询
* 隐式索引

### $操作符如何使用索引
* $where或$exists无法使用索引
* 如果使用稀疏索引就不能使用$exists
* $ne可以使用索引，但不有效
* 