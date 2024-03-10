---
layout: single
title: "[BigData]WSL配置Hadoop"
categories: [AI]
last_modified_at: 2024-03-10
excerpt: "记WSL2上配置Hadoop伪分布式集群的过程"
header:
  teaser: https://static.runoob.com/images/mix/life-code-typography-hd-wallpaper-1920x1080-7168.jpg
---


##  在WSL2上配置Hadoop伪分布式集群

### ssh(加密网络协议)设置

SSH 进行免密登录:

Hadoop通常部署在多个节点上，这些节点之间需要进行通信和数据传输。为了方便管理和操作，**需要在各个节点之间建立免密登录**。Hadoop的启动和运行依赖于SSH服务。Hadoop NameNode、DataNode、 ResourceManager等组件都需要通过SSH进行通信。

```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa # 生成 SSH 密钥对,分别保存
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys # 公钥内容追加到授权密钥文件
chmod 0600 ~/.ssh/authorized_keys # 确保只有文件所有者可读写
```

### jdk(java环境)设置

``openjdk version "1.8.0_392"``

不做赘述 , WSL使用``sudo apt install openjdk-8-jdk-headless ssh pdsh`` (Hadoop 集群管理经常需要在所有节点上执行相同的操作，pdsh 可以简化这一过程。) 主力系统直接用原本的环境变量即可.

查看绝对路径: ``java -verbose``

### Hadoop 安装

拖曳下载好的hadoop-x.x.x.tar.gz文件到主目录(或wegt) 使用`tar xvaf hadoop-x.x.x.tar.gz` 解压

cd 进入hadoop-3.1.3 目录.开始配置

### 伪分布式配置

#### 0.熟悉目录 
``tree <Hadoop-...>`` 或者``ls -r ``递归查看

(避免在非图形化中因为盲目粘贴代码导致错误的环境变量新建而非正确写入目标位置)

#### 1.修改etc/hadoop/core-site.xml

- 指定 Hadoop 的默认文件系统是 HDFS。
- 指定 HDFS 的 Namenode 地址和端口号。
```xml
<configuration>  
    <property>  
        <name>fs.defaultFS</name>  
        <value>hdfs://localhost:9000</value>  
    </property>  
</configuration>
```

#### 2.修改etc/hadoop/hdfs-site.xml

- `dfs.replication` 属性: 指定 HDFS 数据块的副本系数。(每个数据块DN存储几个副本)
- `value` 元素: 指定 `dfs.replication` 属性的值，本例中为 `1`。(测试环境或资源有限)
```xml
<configuration>  
    <property>  
        <name>dfs.replication</name>  
        <value>1</value>  
    </property>  
</configuration>
```

#### 3.修改etc/hadoop/mapred-site.xml

- 确保YARN框架用于资源管理，并且MapReduce应用程序在集群上运行时可以访问必要的库。
- 确保设置了`$HADOOP_MAPRED_HOME`环境变量。
```xml
<configuration>
     <property>
         <name>mapreduce.framework.name</name>
         <value>yarn</value>
     </property>
     <property>
         <name>mapreduce.application.classpath</name>
         <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
     </property>
</configuration>
```

#### 4.修改etc/hadoop/hadoop-env.sh

- **JAVA_HOME:** 指定 Java 虚拟机的安装路径。Hadoop 需要 Java 才能运行，因此必须设置此变量。
- **HDFS_NAMENODE_USER:** 指定 HDFS Namenode 进程运行的用户。
- **HDFS_DATANODE_USER:** 指定 HDFS Datanode 进程运行的用户。...后略.
```bash
export JAVA_HOME=/usr/lib/jvm/java-8-xxx # 上文提过的路径
export HDFS_NAMENODE_USER=root # 或者当前用户名
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```
---
伪分布式模式下，NameNode、DataNode、Secondary NameNode、ResourceManager 和 NodeManager 等组件都在同一台机器上运行，涵盖了 Hadoop 集群的基本功能.

注意 : 

- 伪分布式模式仅用于学习和测试，不适用于生产环境。
- 在生产环境中，建议将 Hadoop 组件部署在不同的机器上，以提高可靠性和性能。

### 使用Hadoop

#### 0.熟悉进程

使用jps查看:**列出正在运行的 Java 进程**。

**1. NameNode:**

- **含义:** HDFS 文件系统的核心组件，负责管理 HDFS 文件系统的元数据，包括文件和目录的名称、位置、权限等信息。(一台)
- **作用:**
    - 存储和管理 HDFS 文件系统的元数据
    - 处理客户端对文件系统的操作请求，例如创建文件、删除文件、重命名文件等
    - 维护 HDFS 文件系统的块位置信息
    - 负责数据块的复制和均衡

**2. DataNode:**

- **含义:** HDFS 文件系统的存储节点，负责存储 HDFS 文件系统的数据块。
- **作用:**
    - 存储 HDFS 文件系统的数据块
    - 响应来自 NameNode 的指令，例如读取数据块、写入数据块、复制数据块等
    - 监控数据块的健康状况

**3. ResourceManager:**

- **含义:** YARN 资源管理框架的核心组件，负责管理 Hadoop 集群中的资源，并将其分配给各个应用程序。(一台)
- **作用:**
    - 监控集群资源的使用情况
    - 根据应用程序的需求分配资源
    - 负责应用程序的启动、运行和监控

**4. NodeManager:**

- **含义:** YARN 资源管理框架的节点代理，负责在每个节点上管理资源并执行来自 ResourceManager 的指令。
- **作用:**
    - 监控节点资源的使用情况
    - 向 ResourceManager 汇报资源信息
    - 接收并执行来自 ResourceManager 的指令，例如启动容器、杀死容器等


#### 1.**格式化 HDFS Namenode**

```bash
bin/hdfs namenode -format
```

- 由 HDFS Namenode 进程运行。
- HDFS DataNode 进程尚未启动。
- 时间具体取决于 HDFS 文件系统的规模。

#### 2.**启动 HDFS 的守护进程**

```bash
sbin/start-dfs.sh
```

jps:将会出现NameNode、DataNode 和 Secondary NameNode(辅助的nn,定期备份)。

此时可以去host:9870查看web界面

#### 3.启动yarn守护进程

```bash
sbin/start-yarn.sh
```
jps:会加入ResourceManager 和 NodeManager。共同管理 Hadoop 集群中的资源。

8088查看YARN资源管理界面,可查看集群,节点,应用程序..资源使用情况.

### 体验分布式计算 MapReduce 

- 创建 HDFS 用户目录
```bash
bin/hdfs dfs -mkdir /user  
bin/hdfs dfs -mkdir /user/用户名
```

- 任务输入文件

```bash
bin/hdfs dfs -mkdir input  
hadoop fs -put test.txt /input # 示例:写一点hello world
```

- 运行统计文本词频
```bash
bin/hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-x.x.x.jar wordcount /input /output
```
ResourceManager 会将最终结果保存到指定的输出路径 `/output` 中。

应熟悉/share/目录.

- 查看结果

```bash
bin/hdfs dfs -cat /output/part-r-00000 # 命名便是如此
```

可以自行查看或者webUI

### 合理结束进程

```bash
sbin/stop-all.sh 停止所有的Hadoop守护进程。包括NameNode、 Secondary NameNode、DataNode、ResourceManager、NodeManager

sbin/start-dfs.sh 启动Hadoop HDFS守护进程NameNode、SecondaryNameNode、DataNode

sbin/stop-dfs.sh 停止Hadoop HDFS守护进程NameNode、SecondaryNameNode和DataNode

sbin/hadoop-daemons.sh start namenode 单独启动NameNode守护进程

sbin/hadoop-daemons.sh stop namenode 单独停止NameNode守护进程

sbin/hadoop-daemons.sh start datanode 单独启动DataNode守护进程

```
后略

### 后记.END

本篇以学习记录为主 . 因为不是虚拟机 , 忘了啥就很难回滚(times备份也很占地方)了.除非熟悉或者纯吝啬资源,否则***不推荐使用子系统和主系统***

另外, 真分布式谨慎使用格式化命令，多次格式化后会导致 NameNode 的 clusterID 和 DataNode 的 clusterID 不匹配，无法启动 DataNode.

