# hadoop安装
## hadoop伪分布式安装
### 部署配置
新建hadoop用户，设置密码
```shell
[root@iZbp1amsik1dd70x6y5q62Z ~]# useradd hadoop
[root@iZbp1amsik1dd70x6y5q62Z ~]# passwd hadoop
```
修改用户权限，非生产环境简单点，权限放大点以免出现权限不足
```shell
[root@iZbp1amsik1dd70x6y5q62Z ~]# vim /etc/sudoers
```
修改内容如下
```shell
## Allow root to run any commands anywhere
root    ALL=(ALL)   ALL
hadoop  ALL=(root)      NOPASSWD:ALL
```
如果文件是自读的无法修改，将文件属性改下
```shell
[root@iZbp1amsik1dd70x6y5q62Z etc]# chmod u+w /etc/sudoers
```
切换用户
```shell
[root@iZbp1amsik1dd70x6y5q62Z etc]# su hadoop
[hadoop@iZbp1amsik1dd70x6y5q62Z etc]$
```
新增用于存放hadoop文件的目录，然后修改下权限赋值给hadoop
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z opt]$ sudo mkdir /opt/hadoop
[hadoop@iZbp1amsik1dd70x6y5q62Z opt]$ sudo chown -R hadoop:hadoop /opt/hadoop
```
下载文件hadoop文件到/opt/hadoop
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z hadoop]$ wget http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.8.4/hadoop-2.8.4.tar.gz
```
解压hadoop
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z hadoop]$ tar -vxf hadoop-2.8.4.tar.gz
```
添加环境变量，如果不切换用户需要使用sudo编辑环境变量
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z etc]$ sudo vim /etc/profile
```
添加内容
```shell
export HADOOP_HOME=/opt/hadoop/hadoop-2.8.4
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```
重新登陆或者执行以下命令，生效环境变量
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z etc]$ source /etc/profile
```
配置etc/hadoop目录下hadoop-env.sh，mapred-env.sh，yarn-env.sh文件的JAVA_HOME参数，修改内容
```shell
export JAVA_HOME=/opt/jdk1.8.0_172
```
配置etc/hadoop目录下core-site.xml
```xml
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://127.0.0.1:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/opt/hadoop/tmp/data</value>
        </property>
</configuration>
```
创建配置中的临时目录，目录创建后注意拥有属性
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z hadoop]$ sudo mkdir -p /opt/hadoop/tmp/data
```
配置etc/hadoop目录下hdfs-site.xml，因为我们部署的是伪分布式环境，所以这里的值是1，表示只有一个节点
```xml
<configuration>
        <property>
                <name>dfs.replication</name>
                <value>1</value>
        </property>
</configuration>
```
格式化HDFS
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z hadoop-2.8.4]$ bin/hdfs namenode -format
```
验证该格式化是否成功，只需要查看core-site.xml配置的hadoop.tmp.dir目录是否产生了dfs文件夹

分别启动namenode，namenode，secondarynamenode
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z current]$ hadoop-daemon.sh start namenode
starting namenode, logging to /opt/hadoop/hadoop-2.8.4/logs/hadoop-hadoop-namenode-iZbp1amsik1dd70x6y5q62Z.out
[hadoop@iZbp1amsik1dd70x6y5q62Z current]$ hadoop-daemon.sh start datanode
starting datanode, logging to /opt/hadoop/hadoop-2.8.4/logs/hadoop-hadoop-datanode-iZbp1amsik1dd70x6y5q62Z.out
[hadoop@iZbp1amsik1dd70x6y5q62Z current]$ hadoop-daemon.sh start secondarynamenode
starting secondarynamenode, logging to /opt/hadoop/hadoop-2.8.4/logs/hadoop-hadoop-secondarynamenode-iZbp1amsik1dd70x6y5q62Z.out
```
可以通过Java自带的命令jps查看以上是否启动成功，还可以通过浏览器查看http://127.0.0.1:50070
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z bin]$ ./jps
4976 DataNode
5077 SecondaryNameNode
5141 Jps
4876 NameNode
```
配置mapred-site.xml，默认没有mapred-site.xml文件但是etc/hadoop目录下有mapred-site.xml.template模板
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z hadoop]$ cp mapred-site.xml.template mapred-site.xml
[hadoop@iZbp1amsik1dd70x6y5q62Z hadoop]$ vim mapred-site.xml
```
拷贝模板，编辑内容如下，表示MapReduce运行在YARN框架上，下面配置YARN
```xml
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
</configuration>
```
配置etc/hadoop目录下yarn-site.xml
```xml
<configuration>

<!-- Site specific YARN configuration properties -->
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
</configuration>

```
分别启动resourcemanager，nodemanager
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z sbin]$ yarn-daemon.sh start resourcemanager
starting resourcemanager, logging to /opt/hadoop/hadoop-2.8.4/logs/yarn-hadoop-resourcemanager-iZbp1amsik1dd70x6y5q62Z.out
[hadoop@iZbp1amsik1dd70x6y5q62Z sbin]$ yarn-daemon.sh start nodemanager
starting nodemanager, logging to /opt/hadoop/hadoop-2.8.4/logs/yarn-hadoop-nodemanager-iZbp1amsik1dd70x6y5q62Z.out
```
一样可以通过jps查看是否启动成功，还可以通过浏览器查看http://127.0.0.1:8088
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z bin]$ ./jps
4976 DataNode
5077 SecondaryNameNode
5637 NodeManager
4876 NameNode
5390 ResourceManager
5775 Jps
```
### 小测试
新增文件，上传到hdfs指定目录中
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z native]$ vim wc.input
[hadoop@iZbp1amsik1dd70x6y5q62Z native]$ hdfs dfs -mkdir -p /wordcountdemo/input
[hadoop@iZbp1amsik1dd70x6y5q62Z native]$ hdfs dfs -put /opt/hadoop/tmp/wc.input /wordcountdemo/input
```
文件内容
```shell
hadoop mapreduce hive
hbase spark storm
sqoop hadoop hive
spark hadoop
```
执行hadoop自带的例子
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z tmp]$ yarn jar /opt/hadoop/hadoop-2.8.4/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.4.jar wordcount /wordcountdemo/input /wordcountdemo/output
```
查看运行结果
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z tmp]$ hdfs dfs -ls /wordcountdemo/output
18/06/11 17:14:04 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 2 items
-rw-r--r--   1 hadoop supergroup          0 2018-06-11 17:12 /wordcountdemo/output/_SUCCESS
-rw-r--r--   1 hadoop supergroup         60 2018-06-11 17:12 /wordcountdemo/output/part-r-00000
[hadoop@iZbp1amsik1dd70x6y5q62Z tmp]$ hdfs dfs -cat /wordcountdemo/output/part-r-00000
18/06/11 17:16:38 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
hadoop  3
hbase   1
hive    2
mapreduce   1
spark   2
sqoop   1
storm   1
```
### 服务启停
停止hadoop服务
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z sbin]$ hadoop-daemon.sh stop namenode
stopping namenode
[hadoop@iZbp1amsik1dd70x6y5q62Z sbin]$ hadoop-daemon.sh stop datanode
stopping datanode
[hadoop@iZbp1amsik1dd70x6y5q62Z sbin]$ yarn-daemon.sh stop resourcemanager
stopping resourcemanager
[hadoop@iZbp1amsik1dd70x6y5q62Z sbin]$ yarn-daemon.sh stop nodemanager
stopping nodemanager
```
文中提到的启动停止脚本太过复杂，其中hadoop目录下有sbin目录有很多可用脚本，不过那些脚本需要免密登陆使用起来才更顺畅，配置免密
```shell
# 切换到hadoop用户，生成密钥
[root@iZbp1amsik1dd70x6y5q62Z ~]# su hadoop
[hadoop@iZbp1amsik1dd70x6y5q62Z root]$ cd ~
[hadoop@iZbp1amsik1dd70x6y5q62Z ~]$ ssh-keygen -t rsa
[hadoop@iZbp1amsik1dd70x6y5q62Z ~]$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
[hadoop@iZbp1amsik1dd70x6y5q62Z ~]$ cd .ssh/
[hadoop@iZbp1amsik1dd70x6y5q62Z .ssh]$ chmod 600 authorized_keys
```
然后就可以通过以下命令重启dfs和yarn
```shell
[hadoop@iZbp1amsik1dd70x6y5q62Z hadoop]$ stop-dfs.sh
[hadoop@iZbp1amsik1dd70x6y5q62Z hadoop]$ stop-yarn.sh
[hadoop@iZbp1amsik1dd70x6y5q62Z hadoop]$ start-dfs.sh
[hadoop@iZbp1amsik1dd70x6y5q62Z hadoop]$ start-yarn.sh
```