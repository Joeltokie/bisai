# Ubuntu16.04 spark 集群安装

## 1、设备

```shell
172.16.21.2 master 
172.16.21.3 node1
172.16.21.4 node2  
```

将上述映射关系分别保存到 每台机器的 /etc/hosts 文件中。

```shell
sudo chmod 777 /etc/hosts
vim /etc/hosts
```

## 2、创建 master 用户，配置免密登录（已经搭建好 hadoop 可以略过）

```shell
sudo adduser master
```

设置好密码后一路回车即可。

然后配置免密登录：

[Ubuntu16.04下配置ssh免密登录 - 紫轩弦月 - 博客园](https://www.cnblogs.com/ALittleMoreLove/p/9455407.html)

## 3、安装 JDK（已经搭建好 hadoop 可以略过）

给所有的机器都安装好 jdk，配置好环境变量：

```shell
vi ~/.bashrc

```
export JAVA_HOME=/usr/local/jdk1.8.0_181
export JRE_HOME=${JAVA_HOME}/jre

export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

```

source ~/.bachrc 后可以验证：

```shell
java -version 

```
java version "1.8.0_181"
Java(TM) SE Runtime Environment (build 1.8.0_181-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.181-b13, mixed mode)
```

```

## 4、spark具体安装步骤

### 4.1 下载 Spark 安装包

到 spark 官方页面下载 spark：[http://spark.apache.org/downloads.html](http://spark.apache.org/downloads.html)

或者百度网盘下载链接: https://pan.baidu.com/s/1kNHRaCvu9fDT5o7MvSTFkA  密码: wi1i


将下载好的安装包上传到` master：172.16.21.2`解压（我是在 /usr/local 下解压的）:

```shell
sudo tar -zxvf spark-2.4.7-bin-hadoop2.7.tgz 
#解压后更改文件夹名称
mv spark-2.4.7-bin-hadoop2.7 spark
```

### 4.2 配置 spark

```shell
cd /home/spark/spark/conf/
mv spark-env.sh.template spark-env.sh
vi spark-env.sh
```

在该文件下添加：

```shell
export JAVA_HOME=/usr/local/jdk1.8.0_181 #(这个 jdk 所在路径是你电脑上的。和我的不一定一样)
export SPARK_MASTER_IP=master
export SPARK_MASTER_PORT=7077
```

SPARK_MASTER_IP 指明主节点。

```cpp
mv slaves.template slaves
vi slaves
```

在该文件中添加子节点所在的位置（Worker节点）：

```shell
node1
node2
```

### 4.3 节点复制

配置好 spark 后，将整个 spark 目录拷贝到其他节点上（注意节点的路径必须和master一样，否则master启动集群回去从节点中对应目录中去启动 work，不一致会报 No such file or directory）。

```shell
sudo scp -r /usr/local/spark linda@node1:
sudo scp -r /usr/local/spark linda@node2:
```

### 4.4 启动节点

接着就可以启动了：在 master 节点，执行 spark 目录下 sbin 目录下的 start-all.sh 脚本即可：

```shell
linda@master:/usr/local/spark/sbin$ ./start-all.sh
```

启动完成后用 `jps`命令查看主节点上有 Master 进程，其他子节点上有 Work 进程

```shell
linda@master:/usr/local/spark/sbin$ jps
81650 Master
81871 Jps
```


# Ubuntu16.04安装并配置hive-2.3.7

Apache-hive-2.3.7-bin.tar.gz的**下载地址**：  
链接：https://pan.baidu.com/s/1m48jHcykJc2OEMv9Uvr5Bw  
提取码：qfji

## 1、安装hive2.3.7

```shell
sudo tar -zxvf spark-2.4.7-bin-hadoop2.7.tgz
sudo mv spark-2.4.7-bin-hadoop2.7 spark
```

设置环境变量

```shell
sudo vim /etc/profile

export HIVE_HOME=/usr/local/hive
export PATH=$PATH:$HIVE_HOME/bin

```

执行source /etc/profile命令，刷新环境变量

```shell
hive
```

查看版本信息

```shell
hive --version
```

<img src="file:///Users/linda/Library/Application%20Support/marktext/images/2020-11-24-20-30-24-image.png" title="" alt="" data-align="left">

## 2、配置hive并进入hive

进入hive的安装目录的conf文件夹；

```shell
cd /usr/local/hive/conf
```

新建hive-site.xml文件；

```shell
sudo vim hive-site.xml
```

打开hive-site.xml文件并添加如下内容；

```shell
<configuration>
  <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://master:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=true</value>
      <description>JDBC connect string for a JDBC metastore</description>    
  </property>
  <property> 
      <name>javax.jdo.option.ConnectionDriverName</name> 
      <value>com.mysql.jdbc.Driver</value> 
      <description>Driver class name for a JDBC metastore</description>     
   </property>               

   <property> 
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>hive</value>
      <description>username to use against metastore database</description>
   </property>
   <property>  
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>hive</value>
      <description>password to use against metastore database</description>  
   </property>
</configuration>
```

把下载目录下的mysql驱动包(链接：`https://pan.baidu.com/s/1XhaTsZUJ1lItVMiyiVywuA  
提取码：4fp8)`添加到HIVE_HOME的lib目录

初始化元数据

```shell
linda@master:/usr/local/hive/bin$ ./schematool -dbType mysql -initSchema
```

```shell
Metastore connection URL:        jdbc:mysql://master:3306/hive?createDatabaseIfNotExist=true&useSSL=true
Metastore Connection Driver :    com.mysql.jdbc.Driver
Metastore connection User:       hive
org.apache.hadoop.hive.metastore.HiveMetaException: Failed to get schema version.
Underlying cause: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException : Communications link failure

The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.
SQL Error code: 0
Use --verbose for detailed stacktrace.
*** schemaTool failed ***
```

启动hive

```shell
linda@master:/usr/local/hive/bin$ hive

Logging initialized using configuration in jar:file:/usr/local/hive/lib/hive-common-2.3.7.jar!/hive-log4j2.properties Async: true
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
hive> 

```

这时可以看见hive进入成功，然后退出hive环境可以使用exit;命令**（千万别少了分号）**。

参考文章：[（超详细+亲测有效）Ubuntu18.0.4安装并配置hive-2.3.7_Z_r_s的博客-CSDN博客](https://blog.csdn.net/Z_r_s/article/details/105875353)





```python

```
