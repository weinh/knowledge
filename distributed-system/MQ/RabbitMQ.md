# RabbitMQ
## 安装
跳过，因为虚拟机以及有了下次需要按照再补上一个文档
## 配置
```
[root@localhost opt]# rabbitmqctl add_user admin 123456
Creating user "admin" ...
[root@localhost opt]# rabbitmqctl set_user_tags admin administrator
Setting tags for user "admin" to [administrator] ...
[root@localhost opt]# rabbitmq-plugins enable rabbitmq_management
```
