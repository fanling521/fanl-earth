# Hadoop-HDFS

## 什么是HDFS

**HDFS**是分布式文件系统（**H**adoop **D**istributed **F**ile **S**ystem），用于存储文件，通过目录树来定位文件。适合一次写入，多次读出的场景，不支持修改文件。

**HDFS的架构**

![HDFS架构](assets/20190326142740.png)

HDFS中的读/写操作运行在块级。HDFS数据文件被分成块大小的块，这是作为独立的单元存储。默认块大小为**128** MB，块太小，增加寻址时间；块太大，增加内存消耗。

**优点**：

- 高容错性：自动保存数据副本
- 适合处理大数据且可部署在廉价服务器集群上，通过多级副本机制提高可靠性

**缺点**：

- 不适合做延时数据访问
- 对小文件存储不友好，且违反了设计目标
- 一个文件不允许多线程同时写，不支持文件随机修改，只能追加

### HDFS的组成

`HDFS`由**NameNode**、**DataNode**和**SecondaryNameNode**组成。

#### NameNode

`NameNode`维护文件和目录的文件系统树和元数据。管理`HDFS`的名称空间，配置副本策略，管理数据库的映射信息，处理客户端的读写请求。

#### DataNode

`DataNode`存储实际的数据块，接收`NameNode`命令并执行执行数据的读写操作。

#### SecondaryNameNode

分担`NameNode`的工作，合并`Fsimage`和`edits`，紧急情况下，恢复`NameNode`

####  Client

`Client`是客户端，主要的功能：

- 文件切割，讲文件分割成一个一个的块，然后上传
- 与`NameNode`交互，获取文件的位置信息，与`DataNode`交互，进行读写操作
- 提供一些命令管理和操作`HDFS`

## HDFS Shell操作

### 基本语法

`bin/hadoop fs [具体命令]`或者`bin/hdfs dfs [具体命令]`，`dfs`是`fs`的实现类。

  查看帮助`bin/hadoop dfs -help`

### 用户命令

```bash
# -ls: 显示目录信息
[fanl@centos7 ~]$ hdfs dfs -ls /
=========================================
Found 3 items
drwxr-xr-x   - fanl supergroup          0 2019-03-26 18:26 /opt
drwx------   - fanl supergroup          0 2019-03-26 18:14 /tmp
drwxr-xr-x   - fanl supergroup          0 2019-03-26 18:29 /user
# -mkdir：在HDFS上创建目录
[fanl@centos7 ~]$ hdfs dfs -mkdir -p /user/fanl/a/aa
# -cat：显示文件内容
[fanl@centos7 ~]$ hdfs dfs -cat /user/fanl/myfile/LeagueClientSettings.yaml
# -cp ：从HDFS的一个路径拷贝到HDFS的另一个路径
[fanl@centos7 ~]$ [fanl@centos7 ~]$ hdfs dfs -cp \ 
> /user/fanl/myfile/LeagueClientSettings.yaml \
> /user/fanl
# -mv：在HDFS目录中移动文件
[fanl@centos7 ~]$ hdfs dfs -cp /user/fanl/myfile.sh /user/fanl/input/myfile.sh
# -put：从本地文件系统中拷贝文件到HDFS路径去，下载是get
[fanl@centos7 ~]$ hdfs dfs -tail /user/fanl/myfile/LeagueClientSettings.yaml
# -tail：显示一个文件的末尾
[fanl@centos7 ~]$ hdfs dfs -tail /user/fanl/myfile/LeagueClientSettings.yaml
# -rm：删除文件或文件夹
[fanl@centos7 ~]$ hdfs dfs -rm -f -r /user/fanl/a
=======================================================
19/03/27 18:53:17 INFO fs.TrashPolicyDefault: Namenode trash configuration: Deletion interval = 0 minutes, Emptier interval = 0 minutes.
Deleted /user/fanl/a
```

### 管理员命令

查看帮助`bin/hadoop dfsadmin -help`

### RPC通信原理

`RPC`即为远程过程调用协议，通过网络从远程计算机程序上请求服务。

请求程序就是一个客户机，而服务提供程序就是一个服务器，首先，客户机调用进程发送一个有进程参数的调用信息到服务进程，然后等待应答信息。在服务器端，进程保持睡眠状态知道调用信息的到达为止。当一个信息到达，服务器调用进程接收答复信息，获得进程结果。

## HDFS Java API

### 环境准备

下载**Windows**版本的Hadoop压缩包，解压，配置环境变量`HADOOP_HOME`

新建maven工程，参考如下的`pom.xml`

```xml
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-common</artifactId>
	<version>${hadoop-version}</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-client</artifactId>
	<version>${hadoop-version}</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-hdfs</artifactId>
	<version>${hadoop-version}</version>
</dependency>
```

### HDFS的API

配置文件的优先级：客户端代码中设置的值 >配置文件>用户自定义配置 >hadoop的默认配置

```java
//配置文件
Configuration configuration = new Configuration();
```

实例化文件系统，FileSystem是一个实现了文件系统的抽象类，继承`org.apache.hadoop.conf.Configured`并实现了Closeable接口，可以适用多种文件系统。

```java
FileSystem fs=FileSystem.get(new URI("hdfs://centos7:9000"),conf,"fanl");
```

一些操作就可以查看对象`fs`中方法。

实例：定位读取文件

```java
//##########第一块############
//输入流
FSDataInputStream fis=fs.open(xxx);
//输出流
FileOutputStream fos=new FileOutputStream(xxx);
//每次读取1024B
byte[] buf = new byte[1024];	
for(int i =0 ; i < 1024 * 128; i++){
	fis.read(buf);
	fos.write(buf);
}
//##########第二块############
// 定位到128M
fis.seek(1024*1024*128)
```

## HDFS的原理⭐

### HDFS的数据流

Hadoop默认的存储格式如下

```
file_format:
  : SEQUENCEFILE
  | TEXTFILE    -- (Default, depending on hive.default.fileformat configuration)
  | RCFILE      -- (Note: Available in Hive 0.6.0 and later)
  | ORC         -- (Note: Available in Hive 0.11.0 and later)
  | PARQUET     -- (Note: Available in Hive 0.13.0 and later)
  | AVRO        -- (Note: Available in Hive 0.14.0 and later)
  | JSONFILE    -- (Note: Available in Hive 4.0.0 and later)
  | INPUTFORMAT input_format_classname OUTPUTFORMAT output_format_classname
```

#### HDFS写文件流程

1. 客户端分割文件，向`NameNode`请求上传文件，`NameNode`响应可以上传，返回节点信息
4. 客户端通过`FSDataOutputStream`模块请求`dn1`上传数据，`dn1`收到请求会继续调用`dn2`，逐个建立管道，然后逐级应答客户端
3. 客户端按**128MB**的块切分文件，发送第一块，向`dn1`以`packet`为单位传输数据，`dn1`收到就会传给`dn2`，依次传输，逐级应答，每写完一个块，返回确认信息，客户端会再次请求
7. 全部写完，关闭输入输出流，给`NameNode`发出完成信号

#### HDFS读文件流程

1. 客户端通过`Distributed FileSystem`模块向`NameNode`请求下载文件，`NameNode`通过查询元数据，找到对应的`DataNode`地址
2. 客户端通过`FSDataInputtStream`模块向对应的`DataNode`请求数据读取各个`block`
3. `DataNode`传输数据给客户端，客户端接收缓存然后合并，最后写入目标文件

### NameNode工作原理

`NameNode`被格式化之后，将在`${HADOOP_HOME}/data/tmp/dfs/name/current`目录中产生如下文件

```
fsimage_0000000000000000000
fsimage_0000000000000000000.md5
seen_txid
VERSION
```

- **fsimage文件：**`HDFS`文件系统元数据的一个**永久性的检查点**，其中包含`HDFS`文件系统的所有目录和文件`inode`的序列化信息
- **edits文件：**存放`HDFS`文件系统的所有更新操作的路径，文件系统客户端执行的所有写操作首先会被记录到`edits`文件中

#### 为什么要这样做

首先，如果存储在`NameNode`节点的磁盘中，因为经常需要进行随机访问，还有响应客户端请求，必然是效率过低。因此，元数据需要存放在内存中。但如果只存在内存中，一旦断电，元数据丢失，整个集群就无法工作了。因此产生在磁盘中备份元数据的`fsImage`。

当在内存中的元数据更新时，如果同时更新`fsImage`，就会导致效率过低，但如果不更新，就会发生一致性问题，一旦`NameNode`节点断电，就会产生数据丢失。因此，引入`edits`文件(只进行追加操作，效率很高)。每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到`edits`中。这样，一旦`NameNode`节点断电，可以通过`fsImage`和`edits`的合并，合成元数据。

但是，如果长时间添加数据到`edits`中，会导致该文件数据过大，效率降低，而且一旦断电，恢复元数据需要的时间过长。因此，需要定期进行`fsImage`和`edits`的合并，如果这个操作由`NameNode`节点完成，又会效率过低。因此，引入一个新的节点`SecondaryNamenode`，专门用于`fsImage`和`edits`的合并。

#### NameNode的工作过程

`NameNode`启动后，会进入30秒的等待时间，此时处于安全模式，所谓的安全模式就是只能执行相关读取操作。此时，开始完成两件事：

**第一件事**

接受`DataNode`的心跳和块状态报告，心跳为每3秒发送一次，用来标记是否存活，而块的状态报告主要是用来发送`NameNode`节点下各个块的状态，默认每一小时发送一次，之后`NameNode`就会根据自身的元数据来比对`DataNode`发送的所有块报告的数据是否匹配来判断各个`DataNode`是否正常。

**第二件事**


1. 第一次启动`NameNode`，将创建`fsimage`和`edits`文件，如果不是第一次启动，先滚动`edits`并生成一个空的`edits.inprogress`，然后加载`fsimage`和`edits`到内存，当然，也会合并`fsimage`和`edits`
2. 客户端发起对**元数据**进行**增删改**的请求（查询元数据的操作不会被记录在`edits`中）
3. `NameNode`编辑日志先记录操作日志，更新滚动日志
4. `NameNode`在内存中对数据进行增删改

#### 2NN的工作过程

其主要工作是合并日志，详细的内容如下：

1. 询问`NameNode`是否需要`CheckPoint`（*默认1小时，或者每分钟检查操作数达到一定数值*）。直接带回`NameNode`是否检查结果
2. `NameNode`将滚动前的编辑日志和镜像文件拷贝到`Secondary NameNode`
3. `Secondary NameNode`加载编辑日志和镜像文件到内存，并合并，生成新的镜像文件`fsimage.chkpoint`
4. 生成新的镜像文件`fsimage.chkpoint`，`NameNode`将`fsimage.chkpoint`重新命名成`fsimage`

 **oiv**查看fsimage文件

```bash
[fanl@centos7 current]$ hdfs oiv -p XML -i fsimage_0000000000000000437 -o ./xx.xml
```

**oev**查看edits文件

```bash
[fanl@centos7 current]$ hdfs oev -p XML -i edits_0000000000000000438-0000000000000000439 -o ./xx2.xml
```

### DataNode工作原理

1. 一个数据块在`DataNode`上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳
2. `DataNode`启动后向`NameNode`注册，通过后，周期性（1小时）的向`NameNode`上报所有的块信息
3. 心跳检测是每3秒一次，心跳返回结果带有`NameNode`给该`DataNode`的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个`DataNode`的心跳，则认为该节点不可用

## HDFS其他特性

### 集群间数据复制

```bash
hadoop distcp hdfs://haoop102:9000/user/hello.txt hdfs://hadoop103:9000/user/hello.txt
```

### HDFS的机架感知

默认是没有机架感知的，需要配置，版本需要>2.7.2

1. 第一个副本在Client所处的节点上，如果客户端在集群外，随机选一个
2. 第二个副本和第一个副本位于相同机架，随机节点
3. 第三个副本位于不同机架，随机节点

### NameNode故障处理

**方法1**：将`SecondaryNameNode`中数据拷贝到`NameNode`存储数据的目录

**方法2**：使用`-importCheckpoint`选项启动`NameNode`守护进程，从而将`SecondaryNameNode`中数据拷贝到NameNode目录中。

### NameNode多目录

NameNode的本地目录可以配置成多个，且每个目录存放内容相同，增加了可靠性。

在`hdfs-site.xml`中配置

```xml
<property>
    <name>dfs.namenode.name.dir</name>
<value>file:///${hadoop.tmp.dir}/dfs/name1,file:///${hadoop.tmp.dir}/dfs/name2</value>
```

删除`data`和`logs`，重新格式化`NameNode`

### Hadoop的安全模式

`NameNode`启动时，首先将镜像文件（`Fsimage`）载入内存，并执行编辑日志（`edits`）中的各项操作。一旦在内存中成功建立文件系统元数据的映像，则创建一个新的`Fsimage`文件和一个空的编辑日志。此时，`NameNode`开始监听`DataNode`请求。这个过程期间，`NameNode`一直运行在安全模式，即`NameNode`的文件系统对于客户端来说是只读的。

系统中的数据块的位置并不是由`NameNode`维护的，而是以块列表的形式存储在`DataNode`中。在系统的正常操作期间，`NameNode`会在内存中保留所有块位置的映射信息。在安全模式下，各个`DataNode`会向`NameNode`发送最新的块列表信息，`NameNode`了解到足够多的块位置信息之后，即可高效运行文件系统。

如果满足“最小副本条件”，`NameNode`会在30秒钟之后就退出安全模式。

所谓的最小副本条件指的是在整个文件系统中99.9%的块满足最小副本级别（默认值：dfs.replication.min=1）。在启动一个刚刚格式化的HDFS集群时，因为系统中还没有任何块，所以`NameNode`不会进入安全模式。

（1）`hdfs dfsadmin -safemode get`        （功能描述：查看安全模式状态）

（2）`hdfs dfsadmin -safemode enter`    （功能描述：进入安全模式状态）

（3）`hdfs dfsadmin -safemode leave`     （功能描述：离开安全模式状态）

（4）`hdfs dfsadmin -safemode wait`      （功能描述：等待安全模式状态）

### 如何保证数据完整性

DataNode节点保证数据完整性的方法：

1. 当`DataNode`读取`Block`的时候，它会计算`CheckSum`
2. 如果计算后的`CheckSum`，与`Block`创建时值不一样，说明`Block`已经损坏
3. 客户端读取其他`DataNode`上的`Block`
4. `DataNode`在其文件创建后周期验证`CheckSum`

### HDFS的容错方式

1. 副本
2. 心跳检测

### 掉线时限参数设置

NameNode不会立即把出现问题的节点判定为死亡，要经过一段时间，这段时间叫做超时时长。

超时时长=2\*heartbeat.recheck.interval（毫秒）+10\*dfs.heartbeat.interval（秒）

### 服役新的节点

克隆一样的Hadoop目录，删除data和logs目录，直接启动守护进程，即可加入。

`start-balancer.sh` 集群的再平衡

### 退役旧的节点

#### 白名单hosts

添加到白名单的主机节点，都允许访问NameNode，不在白名单的主机节点，都会被退出。

（1）etc/hadoop目录下创建dfs.hosts文件，添加白名单主机名

（2）hdfs-site.xml配置文件中增加dfs.hosts属性

（3）分发文件，刷新NameNode`hdfs dfsadmin -refreshNodes`

（4）更新ResourceManager`yarn rmadmin -refreshNodes`

（5）如果数据不均衡，可以用命令实现集群的再平衡

#### 黑名单强制退役

在黑名单上面的主机都会被强制退出

（1）`etc/hadoop`目录下创建`dfs.hosts.exclude`文件，添加黑名单主机名

（2）`hdfs-site.xml`配置文件中增加`dfs.hosts.exclude`属性

（3）刷新`NameNode`、刷新`ResourceManager`

（4）等待数据复制完毕后，该主机上关闭进程

注意：如果副本数是3，服役的节点小于等于3，是不能退役成功的，需要修改副本数后才能退役，允许白名单和黑名单中同时出现同一个主机名称

### DataNode多目录

DataNode也可以配置成多个目录，每个目录存储的数据不一样。即：数据不是副本

```xml
<property>
	<name>dfs.datanode.data.dir</name>
   	<value>file:///${hadoop.tmp.dir}/dfs/data1,
       file:///${hadoop.tmp.dir}/dfs/data2</value>
</property>
```

## 相关问题

（1）怎么用命令查看、删除、移动、拷贝HDFS上的文件？

（2）简述NameNode启动过程

（3）如何实现HDFS文件下载到服务器磁盘？

（4）HDFS的DataNode保存哪些数据，进行简单描述

（5）HDFS的存储机制是什么及如何保证数据安全性？





































































































































































































































































































































