---
layout: single
title: "[BigData]WSL配置Hbase"
categories: [AI]
last_modified_at: 2024-03-15
excerpt: "在WSL2上配置Hbase伪分布式数据库"
header:
  teaser: https://static.runoob.com/images/mix/life-code-typography-hd-wallpaper-1920x1080-7168.jpg
---


## 在WSL2上配置Hbase伪分布式数据库

### 前置条件
#### ssh(加密网络协议)设置

略

#### jdk(java环境)设置

``openjdk version "1.8.0_392"``

#### Hadoop  

依赖关系略

正常拖曳解压,放在熟悉的目录就行.

#### 检查是否安装成功

sudo chown -R XXX ./hbase #赋予权限

XXX为能控制hadoop的用户名.可以是Hadoop账号,也可以是root/master.未赋权会permission de报错

 ``{hbase所在路径}/bin/hbase version ``
输出版本消息表示HBase已经安装成功

### 伪分布式配置

#### 0.熟悉目录 


```shell
~/hbase-2.2.2$ ls
CHANGES.md  LICENSE.txt  README.txt       bin   docs           lib
LEGAL       NOTICE.txt   RELEASENOTES.md  conf  hbase-webapps

:~/hbase-2.2.2/bin$ ls
considerAsDead.sh       local-regionservers.sh
draining_servers.rb     master-backup.sh
get-active-master.rb    region_mover.rb
graceful_stop.sh        region_status.rb
hbase                   regionservers.sh
hbase-cleanup.sh        replication
hbase-common.sh         rolling-restart.sh
hbase-config.cmd        shutdown_regionserver.rb
hbase-config.sh         start-hbase.cmd
hbase-daemon.sh         start-hbase.sh
hbase-daemons.sh        stop-hbase.cmd
hbase-jruby             stop-hbase.sh
hbase.cmd               test
hirb.rb                 zookeepers.sh
local-master-backup.sh
```


#### 1.配置./conf/hbase-env.sh。


```shell

export JAVA_HOME=/usr/lib/jvm/jdk1.8.0 #绝对路径

export HBASE_CLASSPATH=././hbase/conf #指定了HBase启动时应该加载的类路径。比如解析配置文件、连接ZooKeeper等。

export HBASE_MANAGES_ZK=true 
```


`HBASE_MANAGES_ZK` 配置项决定了 HBase 是否负责管理 ZooKeeper 集群。

- **设置为 `true`:** HBase 会自动启动、停止和管理 ZooKeeper 服务。
- **设置为 `false`:** HBase 依赖外部 ZooKeeper 服务，并假设该服务已在运行。

推荐将 `HBASE_MANAGES_ZK` 设置为 `true`，因为它简化了 HBase 的部署和管理。

#### 2.配置./conf/hbase-site.xml

**1. hbase.rootdir**
- **值:** `hdfs://localhost:9000/hbase`
- 指定了 HBase 数据存储的根目录。与单机指定tmp不同，这里将数据存储在 HDFS（Hadoop Distributed File System）中，也就是分布式文件系统。
- **hdfs://localhost:9000:** 这部分指定了 HDFS 的 Namenode 地址，`localhost` 表示本机，`9000` 是 HDFS 的默认端口。
- **/hbase:** 这部分指定了 HDFS 中用于存储 HBase 数据的目录名称。

**2. hbase.cluster.distributed**
- **值:** `true`
- 指定 HBase 集群的运行模式。
- **true:** 表示这是一个分布式集群，有多个 RegionServer 负责存储和处理数据。
- **false:** 表示这是一个单机运行的 HBase 实例，只有一个 RegionServer 处理所有数据。

**3. hbase.unsafe.stream.capability.enforce**

- **值:** `false` (**注意：不建议在生产环境中设置此属性为 false** )
- 用于强制启用 HBase 与底层存储系统的流功能交互。HBase 通常会检查底层文件系统是否支持这些流功能，以确保数据完整性。将此属性设置为 `false` 会绕过这些检查，可能导致数据丢失的风险。


```xml

<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://localhost:9000/hbase</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
</configuration>

```

### 启动Hbase

启动关闭Hadoop和HBase的顺序一定是： 启动Hadoop—>启动HBase—>关闭HBase—>关闭Hadoop.  ``bin/start-hbase.sh``  ``bin/stop-hbase.sh`` 

进入目录(或者加入环境变量了)``bin/start-hbase.sh`` 然后jps查看是否出现进程


#### 进入shell界面

``bin/hbase shell``

>   In HBase, data is stored in tables, which have rows and columns. This is a terminology overlap withrelational databases (RDBMSs), **but this is not a helpful analogy**.

hbase 与 sql 的区别:

1. 数据类型：关系数据库采用关系模型，具有丰富的数据类型和存储方式，HBase则采用了更加简单的数据模型，它把数据存储为未经解释的字符串；
2. 数据操作：关系数据库中包含了丰富的操作，其中会涉及复杂的多表连接。HBase操作则不存在复杂的表与表之间的关系，只有简单的插入、查询、删除、清空等，因为HBase在设计上就避免了复杂的表和表之间的关系；
3. 存储模式：关系数据库是基于行模式存储的。HBase是基于列存储的，每个列族都由几个文件保存，不同列族的文件是分离的；
4. 数据索引：关系数据库通常可以针对不同列构建复杂的多个索引，以提高数据访问性能。HBase只有一个索引——行键，通过巧妙的设计，HBase中的所有访问方法，或者通过行键访问，或者通过行键扫描，从而使得整个系统不会慢下来；
5. 数据维护：在关系数据库中，更新操作会用最新的当前值去替换记录中原来的旧值，旧值被覆盖后就不会存在。而在HBase中执行更新操作时，并不会删除数据旧的版本，而是生成一个新的版本，旧有的版本仍然保留；
6. 可伸缩性：关系数据库很难实现横向扩展，纵向扩展的空间也比较有限。相反，HBase和BigTable这些分布式数据库就是为了实现灵活的水平扩展而开发的，能够轻易地通过在集群中增加或者减少硬件数量来实现性能的伸缩


列式存储是HBase的一个重要特性，它与传统的行式存储有所不同。

- 表名：`student_info`

- 列族：
    - `info`：包含学生的姓名、年龄和性别等信息
    - `score`：包含学生的各科成绩信息

列族是HBase中组织数据的基本单元，它可以看作是一组相关列的集合。列族通常是根据数据的访问模式和查询需求进行设计的。

#### 创建表：

`create 'student_info', 'info', 'score'`

上述命令创建了一个名为 `student_info` 的表，包含两个列族：`info` 和 `score`。

#### 插入数据：

```sql

put 'student_info', 'row_key', 'info:name', 'Alice'
put 'student_info', 'row_key', 'info:age', '20'

```

行键为 `row_key` | exit即可退出shell 

---


### HBase Java API编程

> 确保 hdfs和hbase进程已启动, hbase/lib下的JAR均已导入

代码如下:


```java

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;

public class ExampleForHBase {
    public static Configuration configuration;
    public static Connection connection;
    public static Admin admin;

    public static void main(String[] args) throws IOException {
        init(); // 初始化HBase连接
        createTable("student", new String[]{"score"}); // 创建名为"student"的表，包含一个名为"score"的列族
        insertData("student", "kasumi", "score", "English", "99"); // 向表中插入数据
        insertData("student", "kasumi", "score", "Math", "99");
        insertData("student", "kasumi", "score", "Computer", "99");
        getData("student", "kasumi", "score", "English"); // 从表中获取数据
        close(); // 关闭HBase连接
    }

    public static void init() {
        configuration = HBaseConfiguration.create(); // 创建HBase配置对象
        configuration.set("hbase.rootdir", "hdfs://localhost:9000/hbase"); // 设置HBase数据存储根目录
        try {
            connection = ConnectionFactory.createConnection(configuration); // 创建HBase连接
            admin = connection.getAdmin(); // 获取HBase管理器
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void close() {
        try {
            if (admin != null) {
                admin.close(); // 关闭HBase管理器
            }
            if (connection != null) {
                connection.close(); // 关闭HBase连接
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void createTable(String myTableName, String[] colFamily) throws IOException {
        TableName tableName = TableName.valueOf(myTableName); // 表名
        if (admin.tableExists(tableName)) { // 判断表是否已经存在
            System.out.println("Table already exists!");
        } else {
            TableDescriptorBuilder tableDescriptor = TableDescriptorBuilder.newBuilder(tableName); // 创建表描述符
            for (String str : colFamily) { // 遍历列族
                ColumnFamilyDescriptor family = ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes(str)).build(); // 创建列族描述符
                tableDescriptor.setColumnFamily(family); // 将列族添加到表描述符中
            }
            admin.createTable(tableDescriptor.build()); // 创建表
            System.out.println("Table created successfully!");
        }
    }

    public static void insertData(String tableName, String rowKey, String colFamily, String col, String val) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName)); // 获取表对象
        Put put = new Put(Bytes.toBytes(rowKey)); // 创建Put对象，指定行键
        put.addColumn(Bytes.toBytes(colFamily), Bytes.toBytes(col), Bytes.toBytes(val)); // 添加列值
        table.put(put); // 插入数据
        table.close(); // 关闭表对象
    }

    public static void getData(String tableName, String rowKey, String colFamily, String col) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName)); // 获取表对象
        Get get = new Get(Bytes.toBytes(rowKey)); // 创建Get对象，指定行键
        get.addColumn(Bytes.toBytes(colFamily), Bytes.toBytes(col)); // 指定要获取的列族和列
        Result result = table.get(get); // 获取数据
        byte[] value = result.getValue(Bytes.toBytes(colFamily), Bytes.toBytes(col)); // 获取指定列的值
        if (value != null) {
            System.out.println("Value: " + Bytes.toString(value)); // 打印值
        } else {
            System.out.println("No such column data!");
        }
        table.close(); // 关闭表对象
    }
}
```

### 后记.END

[Hbase 中文文档](https://hbase.org.cn/)


> 在同一个硬件上运行多个 HMaster 实例在生产环境中是没有意义的，就像运行伪分布式集群对于生产没有意义一样。
