```python
#数据可视化matplotlib和pandas学习
#条形图、折线图、数据清洗处理、数据统计
#统计票房前5 ,并绘制出 条形图
#axis=0对横轴操作，axis=1对纵轴操作
```


```python
#包导入
import pandas as pd
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
```


```python
#数据展示
import os
for dirname, _, filenames in os.walk('.'):
    for filename in filenames:
        print(os.path.join(dirname, filename))
```


```python
#数据读取
piaofang_data = pd.read_csv('train.csv')
pf_data=piaofang_data.loc[:,['id','title','revenue']]
print(pf_data)
```

# 数据处理


```python
piaofang_data.isnull().any(axis=0) #查看每一列是否有nan
```


```python
#pd.dropna()如果想直接去掉有 NaN 的行或列, 可以使用 dropna
pf_drop=piaofang_data
pf_drop.dropna(
    axis=1,     # 0: 对行进行操作; 1: 对列进行操作
    how='any'   # 'any': 只要存在 NaN 就 drop 掉; 'all': 必须全部是 NaN 才 drop 
    ) 
```


```python
#pd.fillna()如果是将 NaN 的值用其他值代替, 比如代替成 0:
pf_drop=piaofang_data
pf=pf_drop.fillna(value=0)
pf.isnull().any(axis=0) #查看每一列是否有nan
```


```python
#合并数据
#定义资料集
df1 = pd.DataFrame(np.ones((3,4))*0, columns=['a','b','c','d'])
df2 = pd.DataFrame(np.ones((3,4))*1, columns=['a','b','c','d'])
#concat横向合并
res = pd.concat([df1, df2], axis=1)
print(res)
#承上一个例子，并将index_ignore设定为True
res = pd.concat([df1, df2], axis=1, ignore_index=True)
print(res)
```


```python
#join (合并方式) 
#join='outer'为预设值，因此未设定任何参数时，函数默认join='outer'。
#此方式是依照column来做纵向合并，有相同的column上下合并在一起，
#其他独自的column个自成列，原本没有值的位置皆以NaN填充。
#定义资料集
df1 = pd.DataFrame(np.ones((3,4))*0, columns=['a','b','c','d'], index=[1,2,3])
df2 = pd.DataFrame(np.ones((3,4))*1, columns=['b','c','d','e'], index=[2,3,4])

#纵向"外"合并df1与df2
res = pd.concat([df1, df2], axis=0, join='outer')

print(res)
#纵向"内"合并df1与df2
res = pd.concat([df1, df2], axis=0, join='inner')

#打印结果
print(res)

#重置index并打印结果
res = pd.concat([df1, df2], axis=0, join='inner', ignore_index=True)
print(res)
```


```python
#append (添加数据)
#定义资料集
df1 = pd.DataFrame(np.ones((3,4))*0, columns=['a','b','c','d'])
df2 = pd.DataFrame(np.ones((3,4))*1, columns=['a','b','c','d'])
df3 = pd.DataFrame(np.ones((3,4))*1, columns=['a','b','c','d'])
s1 = pd.Series([1,2,3,4], index=['a','b','c','d'])

#将df2合并到df1的下面，以及重置index，并打印出结果
res = df1.append(df2, ignore_index=True)
print(res)
```


```python
#依据一组key合并
#定义资料集并打印出
left = pd.DataFrame({'key': ['K0', 'K1', 'K2', 'K3'],
                             'A': ['A0', 'A1', 'A2', 'A3'],
                             'B': ['B0', 'B1', 'B2', 'B3']})
right = pd.DataFrame({'key': ['K0', 'K1', 'K2', 'K3'],
                              'C': ['C0', 'C1', 'C2', 'C3'],
                              'D': ['D0', 'D1', 'D2', 'D3']})

print(left)
print(right)
#依据key column合并，并打印出
res = pd.merge(left, right, on='key')
print(res)
```


```python
#数据整合
top_5 = pf_data.sort_values(by=['revenue'],ascending=False).head(5)
print(top_5)
```


```python
#画条形图
plt.barh(top_5['title'],top_5['revenue'],align='center')
plt.title('票房top_5') 
plt.ylabel('电影') 
plt.xlabel('票房')
plt.show()
```


```python
#画折线图
plt.plot(top_5['title'],top_5['revenue'])
plt.plot(top_5['title'],top_5['revenue'],'.','r')
plt.title('票房top_5') 
plt.ylabel('电影') 
plt.xlabel('票房')
plt.show()
```

# 环境搭建

## 关闭防火墙：
```shell
ufw disable
```

## 用户创建：
```shell
useradd hadoop
passwd 
vim /etc/sudoers 添加权限。  adduser hadoop sudo
```

## 配置jdk:

解压命令：tar -zxvf jdk-8u144-linux-x64.tar.gz -C /opt  
获取当前路径：pwd    
编辑环境变量文件：sudo vi /etc/profile
写入以下数据到profile文件中：
export JAVA_HOME=/opt/module/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin
重启服务：source /etc/profile 
查看java版本：java -version 

# 2、修改主机名字和ip映射

## 2.1、修改主机名

vi /etc/hostname

## 2.2、修改 ip 映射

nmcli dev show 查看本机网络详细地址

sudo vim /etc/hosts

  10.211.55.8	master
  10.211.55.9	node1
  10.211.55.10 node2

## 2.3 配置静态Ip
sudo vim /etc/network/interfaces

ifconfig 查看本机网卡名字

iface lo inet loopback
auto enp0s5
iface enp0s5 inet static
address 10.211.55.8
netmask 255.255.255.0
gateway 10.211.55.1
dns-nameservers 10.211.55.1

sudo ifdown enp0s5
sudo ifup enpos5


# **3 安装并配置ssh**

开始配置ssh之前，先确保**三台机器**都装了ssh。输入以下命令查看安装的ssh。

```shell
#查看ssh安装包情况
dpkg -l | grep ssh  

#查看是否启动ssh服务
ps -e | grep ssh
```

#ssh安装

```shell
sudo apt-get install openssh-server 
```
#开启ssh

sudo service ssh start

#生成公钥密钥

```shell
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
```
#将公钥加入到已认证的key中：

```shell
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

③将.ssh目录复制到node1和node2两台虚拟机去。（其实只用复制authorized_keys到另外两台虚拟机下的.ssh目录下就行了，但是另外两台还没创建.ssh目录，因此为了方便直接把.ssh目录复制过去）

```shell
scp - r ~/.ssh linda@node1:/home/linda
```

上面代码中 linda 是指自己的帐户名 node1是自己命名的。不清楚的话看下面图片


# 5、jdk 安装自行百度（JDK1.8）

## 5.1导入和解压jdk到/opt

tar -zxvf ***** -C /opt/
## 5.2 配置环境变量

pwd 获取路径

sudo vi /etc/profile

#JAVA_HOME

export JAVA_HOME=/opt/module/jdk1.8.0_144

export PATH=$PATH:$JAVA_HOME/bin


source /etc/profile

# 6.安装hadoop

tar -zxf hadoop-2.7.1.tar.gz

mv hadoop-2.7.1 hadoop

sudo chmod -R 777 /usr/local/hadoop

scp -r /usr/loacl/hadoop linda@node1:~/

ssh 从机 例如(ssh node1/node2)

sudo mv hadoop /usr/local


## 6.1配置环境变量

##HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-2.7.2

export PATH=$PATH:$HADOOP_HOME/bin

export PATH=$PATH:$HADOOP_HOME/sbin

现在这之前需要在`/hadoop`下创建tmp文件夹及其子目录：

```shell
 sudo mkdir -p tmp/dfs
$ sudo mkdir data
$ sudo mkdir name
```


**① 添加Java-home到hadoop-env.sh**

`./bin/hadoop version`运行这个命令可能会出现下面的情况：

```shell
Error: JAVA_HOME is not set and could not be found.
```

解决办法：修改`/etc/hadoop/hadoop-env.sh`中设JAVA_HOME。  
应当使用绝对路径。

```shell
export JAVA_HOME="这个地是你第5步配置时的java 地址"
```

**ps：不知道自己 java 路径的想想第五步配置的 java路径**

**② 添加Java-home到yarn-env.sh**

```shell
export JAVA_HOME="这个地是你第5步配置时的java 地址"

```

**③ 添加slave主机名到slaves**

```shell
master
node1
node2
```

**④ 修改 core-site.xml**

```shell
<configuration>
       <property>
                <name>fs.defaultFS</name>
                <value>hdfs://master:8020</value>
       </property>
       <property>
               <name>hadoop.tmp.dir</name>
               <value>file:/usr/local/hadoop/tmp</value>
               <description>Abase for other temporary   directories.</description>
       </property>
</configuration>

```

**注意： **

A. `hdfs://adminserver:8020` 的`adminserver`是主机名，和之前的hostname对应。8020是端口号，注意不要占用已用端口就可以。  
B. `file:/usr/local/hadoop-2.7.2/tmp` 指定到刚刚创建的tmp文件夹.  
C. 所有的配置文件< name >和< value >节点处不要有空格，否则会报错!!

**⑤ 修改 hdfs-site.xml**

```shell
<configuration>
       <property>
                <name>dfs.namenode.secondary.http-address</name>
               <value>master:50090</value>
       </property>
     <property>
             <name>dfs.namenode.name.dir</name>
             <value>file:/usr/local/hadoop/tmp/dfs/name</value>
       </property>
      <property>
              <name>dfs.datanode.data.dir</name>
              <value>file:/usr/local/hadoop/tmp/dfs/data</value>
       </property>
       <property>
               <name>dfs.replication</name>
               <value>3</value>
        </property>
        <property>
                 <name>dfs.webhdfs.enabled</name>
                  <value>true</value>
         </property>
</configuration>

```

**⑥ 修改 mapred-site.xml**  
如果文件夹下没有这个文件，需要先执行`sudo cp mapred-site.xml.template mapred-site.xml`复制一份，然后再执行`sudo vim mapred-site.xml`命令进行修改。

```shell
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>master:10020</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>master:19888</value>
        </property>
</configuration>

```

**⑦ 修改yarn-site.xml**

```shell
<configuration>  
  <property>  
    <name>yarn.nodemanager.aux-services</name>  
    <value>mapreduce_shuffle</value>  
  </property> 
  <property>  
    <name>yarn.resourcemanager.scheduler.address</name>  
    <value>adminserver:8030</value>  
  </property> 
  <property>  
    <name>yarn.resourcemanager.address</name>  
    <value>master:8032</value>  
  </property>
  <property>  
    <name>yarn.resourcemanager.resource-tracker.address</name>  
    <value>adminserver:8031</value>  
  </property> 
  <property>  
    <name>yarn.resourcemanager.admin.address</name>  
    <value>master:8033</value>  
  </property> 
  <property>
     <name>yarn.resourcemanager.webapp.address</name>
     <value>master:8088</value>
  </property> 
</configuration>  

```

### 7. 启动hadoop集群

**① 初始化namenode**

```shell
# 在主机上操作
$ hadoop namenode -format
```

**注意：首次运行需要执行初始化，之后不需要。**  
成功运行，应该返回`Exitting with status 0`，提示`Shuting down Namenode at adminserver/xxx.xxx.xxx.xx(adminserver的IP地址）`，具体结果如下图所示：

```shell
20/11/23 18:21:29 INFO namenode.FSNamesystem: Retry cache on namenode is enabled
20/11/23 18:21:29 INFO namenode.FSNamesystem: Retry cache will use 0.03 of total heap and retry cache entry expiry time is 600000 millis
20/11/23 18:21:29 INFO util.GSet: Computing capacity for map NameNodeRetryCache
20/11/23 18:21:29 INFO util.GSet: VM type       = 64-bit
20/11/23 18:21:29 INFO util.GSet: 0.029999999329447746% max memory 889 MB = 273.1 KB
20/11/23 18:21:29 INFO util.GSet: capacity      = 2^15 = 32768 entries
20/11/23 18:21:29 INFO namenode.FSImage: Allocated new BlockPoolId: BP-996203936-172.16.21.2-1606184489915
20/11/23 18:21:29 INFO common.Storage: Storage directory /usr/local/hadoop/tmp/dfs/name has been successfully formatted.
20/11/23 18:21:30 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
20/11/23 18:21:30 INFO util.ExitUtil: Exiting with status 0
20/11/23 18:21:30 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************

```

**② 启动Hadoop的守护进程（NameNode, DataNode, ResourceManager和NodeManager等）**

- 首先启动NameNode、SecondaryNameNode、DataNode
  
  ```shell
  # On Master
  $ start-dfs.sh    
  # 此时master节点上面运行的进程有：NameNode、SecondaryNameNode  
  # 此时slave节点上面运行的进程有：DataNode  
  ```

运行`start-dfs.sh` 的结果如下，可以发现`NameNode、SecondaryNameNode、DataNode`均已按照集群最初的设计，在相应的主机上启动

- 启动ResourceManager、NodeManager
  
  ```shell
  # On Master
  $ start-yarn.sh
  # YARN 是从 MapReduce 中分离出来的，负责资源管理与任务调度。YARN 运行于 MapReduce 之上，提供了高可用性、高扩展性  
  # 此时master节点上面运行的进程有：NameNode、SecondaryNameNode、ResourceManager
  # slave节点上上面运行的进程有：DataNode、NodeManager  
  
  ```

- 启动JobHistoryServer
  
  ```shell
  # On Master
  $ mr-jobhistory-daemon.sh start historyserver
  # master节点将会增加一个JobHistoryServer 进程
  ```

**注意：多次重启以后，一定要删除每个节点上的logs、tmp目录，并重新创建tmp目录**

**③ 查看mater和slave主机上进程**

- 通过在命令行运行jps指令，可以看到当前运行在adminserve节点上的守护进程：<img src="file:///Users/linda/Library/Application%20Support/marktext/images/2020-11-24-11-17-20-image.png" title="" alt="" width="220">
- 通过在命令行运行jps指令，查看在master、node1、node2节点上的守护进程：

<img title="" src="file:///Users/linda/Library/Application%20Support/marktext/images/2020-11-24-11-18-16-image.png" alt="" width="413" data-align="center">

缺少任一进程都表示出错。另外还需要在 namenode1 节点上通过命令 `hdfs dfsadmin -report` 查看 DataNode 是否正常启动，如果 Live datanodes 不为 0 ，则说明集群启动成功。例如我这边一共有 2 个 Datanodes：

```shell
linda@master:~$ hdfs dfsadmin -report
Configured Capacity: 59982336000 (55.86 GB)
Present Capacity: 49591455744 (46.19 GB)
DFS Remaining: 49591074816 (46.19 GB)
DFS Used: 380928 (372 KB)
DFS Used%: 0.00%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0

-------------------------------------------------
Live datanodes (3):

Name: 172.16.21.4:50010 (node2)
Hostname: node2
Decommission Status : Normal
Configured Capacity: 19994112000 (18.62 GB)
DFS Used: 126976 (124 KB)
Non DFS Used: 3461103616 (3.22 GB)
DFS Remaining: 16532881408 (15.40 GB)
DFS Used%: 0.00%
DFS Remaining%: 82.69%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Mon Nov 23 19:18:52 PST 2020


Name: 172.16.21.3:50010 (node1)
Hostname: node1
Decommission Status : Normal
Configured Capacity: 19994112000 (18.62 GB)
DFS Used: 126976 (124 KB)
Non DFS Used: 3461124096 (3.22 GB)
DFS Remaining: 16532860928 (15.40 GB)
DFS Used%: 0.00%
DFS Remaining%: 82.69%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Mon Nov 23 19:18:52 PST 2020


Name: 172.16.21.2:50010 (master)
Hostname: master
Decommission Status : Normal
Configured Capacity: 19994112000 (18.62 GB)
DFS Used: 126976 (124 KB)
Non DFS Used: 3468652544 (3.23 GB)
DFS Remaining: 16525332480 (15.39 GB)
DFS Used%: 0.00%
DFS Remaining%: 82.65%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Mon Nov 23 19:18:52 PST 2020

```

也可以通过 Web 页面看到查看 DataNode 和 NameNode 的状态：[http://master:50070/]()。(namenode1换成ip) 我的虚拟机不是界面的。自行测试

## 8、执行分布式实例

1】首先创建 HDFS 上的用户目录：

```shell
hdfs dfs -mkdir -p /user/hadoop/input
```

2】将 /usr/local/hadoop/etc/hadoop 中的配置文件作为输入文件复制到分布式文件系统中：

```shell
hdfs dfs -put /usr/local/hadoop/etc/hadoop/*.xml input
```

3】执行实例

```shell
hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar grep /user/hadoop/input /user/hadoop/output 'dfs[a-z.]+'
```

4】查看结果

```text
root@namenode1:/usr/local/hadoop/etc/hadoop# hdfs dfs -cat /user/hadoop/output/*
1   dfsadmin
1   dfs.replication
1   dfs.namenode.secondary.http
1   dfs.namenode.name.dir
1   dfs.datanode.data.dir
root@namenode1:/usr/local/hadoop/etc/hadoop#
```

## 9、启动关闭

启动全部

```shell
start-all.sh
```

关闭hadoop

```text
stop-all.sh
```

# 错误情况

## 1、ExitCodeException exitCode=1

```shell
root@namenode1:~# groupadd supergroup
root@namenode1:~# usermod -a -G supergroup root
root@namenode1:~# hdfs dfs -chmod 770 /user
```

## 2、Ubuntu关闭防火墙

```shell
root@datanode2:/usr/local/hadoop# ufw disable
防火墙在系统启动时自动禁用
root@datanode2:/usr/local/hadoop#
```

## 3、关于 Call From master/172.27.0.5 to master:8020 failed on connection exception: 问题解决

删除再新建你core-site.xml里设置路径的tmp文件夹、dfs文件夹、data文件夹、name文件夹、hadoop根目录logs文件夹.然后再执行下面的代码。前提是自己的配置时没有问题的

```shell
hdfs namenode -format
```



```python

```
