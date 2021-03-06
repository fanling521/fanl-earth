# Hive 数据操作

## Hive的数据类型

### 基本数据类型

| Hive数据类型 | Java数据类型 | Hive数据类型 | Java数据类型 |
| ------------ | ------------ | ------------ | ------------ |
| TINYINT      | byte         | FLOAT        | float        |
| SMALINT      | short        | DOUBLE       | double       |
| INT          | int          | STRING       | string       |
| BIGINT       | long         | TIMESTAMP    | /            |
| BOOLEAN      | boolean      | BINARY       | /            |

对于Hive的String类型相当于数据库的varchar类型，该类型是一个可变的字符串，不过它不能声明其中最多能存储多少个字符，理论上它可以存储2GB的字符数。

### 复杂数据类型

Hive有三种复杂数据类型ARRAY、MAP 和 STRUCT。ARRAY和MAP与Java中的Array和Map类似，而STRUCT与C语言中的Struct类似，它封装了一个命名字段集合，复杂数据类型允许任意层次的嵌套。

```sql
create table test(
name string,
friends array<string>,
children map<string, int>,
address struct<street:string, city:string>
)
row format delimited fields terminated by ','
collection items terminated by '_'
map keys terminated by ':'
lines terminated by '\n';
```

**字段解释**：
`row format delimited fields terminated by ','`  -- 列分隔符
`collection items terminated by '_'`  	               --MAP STRUCT 和 ARRAY 的分隔符(数据分割符号)
`map keys terminated by ':'`				          -- MAP中的key与value的分隔符
`lines terminated by '\n';`					    -- 行分隔符

## DDL数据定义

### 创建数据库

创建一个数据库，数据库在HDFS上的默认存储路径是/user/hive/warehouse/*.db

```sql
hive(default)> create database school;
-- 标准写法
hive(default)> create database if not exists school;
-- 显示数据库
hive(default)> show databases;
-- 过滤选择数据库
hive(default)> show databases like 'sch*';
-- 查看数据库的详细信息
hive(default)> desc database school;
hive(default)> desc database extended school;
-- 切换数据库
hive(default)> use school;
```

### 修改数据库

用户可以通过以下命令为数据库增加属性值，其他元数据信息是不可修改。

```sql
hive(default)> alter database school set dbproperties('owner'='fanl');
OK
Time taken: 0.08 seconds
hive(default)> desc database extended school;
OK
db_name	comment	location	owner_name	owner_type	parameters
school		hdfs://centos7:8020/user/hive/warehouse/school.db	fanl	USER	{owner=fanling}
Time taken: 0.018 seconds, Fetched: 1 row(s)
```

### 删除数据库

删除一个空数据库，非空数据库

```sql
hive(default)> drop database fanling;
-- 删除前判断是否存在
hive(default)> drop database if exists fanling;
-- 非空强制删除
hive(default)> drop database fanling cascade;
```

### 创建表

`Hive`默认情况下会将这些表的数据存储在由配置项`hive.metastore.warehouse.dir`所定义的目录的子目录下。 

当我们删除一个管理表时，`Hive`也会删除这个表中数据。管理表不适合和其他工具共享数据。

```sql
-- 普通创建表
hive(default)> create table if not exists school(sid int,sname string) row format delimited fields terminated by '\t';
-- 普通创建表指定数据位置
hive(default)> create table if not exists school(sid int,sname string) row format delimited fields terminated by '\t' location '/user/hive/warehouse/default.db/school';
-- as 复制表的结构和数据
hive(default)> create table if not exists student2 as select sid,sname from student;
-- like 复制表的结构
hive(default)> create table if not exists student3 like student;
```

#### 管理表和外部表

默认创建的表都是所谓的**管理表**，有时也被称为*内部表*。因为这种表，`Hive`会（或多或少地）控制着数据的生命周期。

删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉。

> （1）创建外部表

```sql
hive (default)> create external table if not exists emp(empno int,ename string,job string,mgr int,hiredate string,sal double,comm double,deptno int) 
              > row format delimited fields terminated by '\t';
OK
Time taken: 3.542 seconds
```

> （2）查看表的类型

```sql
hive (default)> desc formatted emp;
Table Type:         	EXTERNAL_TABLE
```

> （3）管理表和外部表的转化

注意：('EXTERNAL'='TRUE')和('EXTERNAL'='FALSE')为固定写法，区分大小写

```bash
hive (default)> alter table emp set tblproperties('EXTERNAL'='TRUE');
hive (default)> alter table emp set tblproperties('EXTERNAL'='FALSE');
```

#### 分区表

简单的说，Hive中的分区就是分目录，**支持多级分区。**

> （1）创建分区表，在原来的基础上添加分区字段

```sql
hive (default)> create table student_part(id int,name string) partitioned by(month string) row format delimited fields terminated by '\t';
```

> （2）导入数据到各个分区

```sql
hive (default)> load data local inpath '/home/fanl/student.txt' into table student_part partition(month='201901');
hive (default)> load data local inpath '/home/fanl/student.txt' into table student_part partition(month='201902');
```

> （3）查看HDFS上文件的变，多出了2个按照分区规则的目录

```sql
hive (default)> dfs -ls /user/hive/warehouse/student_partition;
Found 2 items
drwxrwxr-x   - fanl supergroup          0 2019-04-09 17:51 /user/hive/warehouse/student_partition/month=201901
drwxrwxr-x   - fanl supergroup          0 2019-04-09 17:51 /user/hive/warehouse/student_partition/month=201902
```

（4）查看分区的数据

```sql
hive (default)> select * from student_partition where month='201901';
-- 可以使用union all 联合查询多个分区数据
```

（5）为已经存在的表新增分区和将分区删除掉，单个和多个都可以。

```sql
hive (default)> alter table student add partition(month='201901'),partition(month='201902');
-- 查看表有多少分区
hive (default)> show partitions student_partition;
OK
partition
month=201901
month=201902
Time taken: 0.113 seconds, Fetched: 2 row(s)

hive (default)> alter table student drop partition(month='201901'),partition(month='201902');
```

（6）修复分区表，修复元数据

```bash
hive (default)> msck repair table student_partition;
```

#### 修改表

（1）重命名表

```sql
hive (default)> alter table student rename to student_new;
```

（2）增加/删除/替换表的列

```sql
-- 添加列
hive (default)> alter table student_new add columns(sex string);
-- 更新列
hive (default)> alter table student_new change column sex sex2 int;
-- 替换表中所有字段
hive (default)> alter table student_new replace columns(sid int, sname string);
```

#### 删除表

```sql
hive (default)> drop table stundent_new;
-- Truncate只能删除管理表，不能删除外部表中数据
hive(default)> truncate table student;
```

## DML数据操作

### 数据导入

基础语法：

​	load data [local] inpath '路径' [overwrite] into table student [partition(partcol1=val1,…)];

（1）本地文件系统加载

```sql
hive (default)> load data local inpath '/home/fanl/student.txt' into table student;
```

（2）HDFS上加载数据

```sql
hive (default)> load data inpath '/user/fanl/student.txt' into table student;
```

（3）覆盖已经存在的数据

```sql
hive (default)> load data [local] inpath 'path' overwrite into table student;
```

（4）import从HDFS上导入数据，格式要和导出的一致

```sql
hive (default)> import table student from '/home/fanl/student';
```

也可以通过查询语句导入数据，不再说明。

使用`as select`，查看本章节的【创建表】

### 数据导出

（1）使用insert，配置到目录即可。

```sql
-- 将查询的数据导出到本地
hive (default)> insert overwrite local directory '/home/fanling/student' select * from student
-- 将查询的数据导出到HDFS
hive (default)> insert overwrite directory '/user/fanl/student' select * from student;
-- 格式化导出
hive (default)> insert overwrite [local] directory '/home/fanling/student' row format delimited fields terminated by '\t' select * from student
```

（2）Hadoop直接下载文件

```bash
hive (default)> dfs -get 'user/hive/warehouse/xxx' 'xxxx'
```

（3）Hive Shell导出

```sql
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/hive -e 'select * from student' > /home/fanl/student/t.txt
```

（4）export导出到HDFS上的目录

```sql
hive (default)> export table student to '/home/fanl/student';
```

**注意**：将在**Sqoop教程**中再次说明导入/导出数据。

### Hive的查询

基本查询的语法和MySQL是一样的，如select、update、limit、between and，仅说明不同的地方。

> （1）like 和rlike

like中 `*`表示多个任意字符，`_`表示一个任意字符

rlike是Hive的扩展，使用Java的正则表达式匹配。

```sql
-- 查看emp表中薪水含有2的数据
hive (default)> select * from emp where sal rlike '[2]';
```

> （2）排序

**全局排序 order By**

全局排序 1个reducer

```sql
-- asc 升序 默认
-- desc 降序
```

每个Reducer内部排序Sort By

```sql
-- 设置Reducer的个数
hive (default)> set mapreduce.job.reduces=3;
hive (default)> select * from emp sort by sal;
hive (default)> insert overwrite local directory '/home/fanl/emp' select * from emp sort by sal;
```

**分区排序 distribute by**

结合sort by使用，**注意**：Hive要求distribute by语句要写在sort by语句之前。

**cluster by** 

当distribute by和sort by字段相同时，可以使用cluster by方式，但是排序只能是升序排序。

> （3）行转列

`CONCAT(string A/col, string B/col…)`：返回输入字符串连接后的结果，支持任意个输入字符串;
`CONCAT_WS(separator, str1, str2,...)`：它是一个特殊形式的 `CONCAT()`。第一个参数剩余参数间的分隔符。分隔符可以是与剩余参数一样的字符串。如果分隔符是 NULL，返回值也将为 NULL。这个函数会跳过分隔符参数后的任何 NULL 和空字符串。分隔符将被加到被连接的字符串之间;
`COLLECT_SET(col)`：函数只接受基本数据类型，它的主要作用是将某字段的值进行去重汇总，产生array类型字段。

> （4）列转行

explode(col)：将Hive一列中复杂的array或者map结构拆分成多行。

用法：`lateral view udtf(expression)` 表别称AS 显示的列名

解释：用于和split, explode等UDTF一起使用，它能够将一列数据拆成多行数据，在此基础上可以对拆分后的数据进行聚合。

### 分桶和抽样查询

#### 分桶

分区针对的是数据的存储路径；分桶针对的是数据文件。创建分桶表时，数据通过子查询的方式导入

```sql
create table stu_buck(id int, name string)
clustered by(id) 
into 4 buckets
row format delimited fields terminated by '\t';
```

```bash
hive (default)> set hive.enforce.bucketing=true;
hive (default)> set mapreduce.job.reduces=-1;
hive (default)> insert into table stu_buck
select id, name from stu;
```

#### 分桶抽样查询

对于非常大的数据集，有时用户需要使用的是一个具有代表性的查询结果而不是全部结果。Hive可以通过对表进行抽样来满足这个需求。

```bash
hive (default)> select * from stu_buck tablesample(bucket 1 out of 4 on id);
```

**注**：tablesample是抽样语句，语法：`TABLESAMPLE(BUCKET x OUT OF y)`

y必须是table总bucket数的倍数或者因子。hive根据y的大小，决定抽样的比例。例如，table总共分了4份，当y=2时，抽取(4/2=)2个bucket的数据，当y=8时，抽取(4/8=)1/2个bucket的数据。

x表示从哪个bucket开始抽取，如果需要取多个分区，以后的分区号为当前分区号加上y。

## Hive的函数

### 内置函数

```bash
hive> show functions;
# 查看函数的描述
hive> desc function upper;
# 查看函数的详细描述、用法
hive> desc function extended upper;
```

### 高级函数

（1）case when

（2）窗口函数

### UDF函数

根据用户自定义函数类别分为以下三种：UDF（一进一出）、UDAF（多进一出）、UDTF（一进多出）

实现步骤：

（1）继承`org.apache.hadoop.hive.ql.UDF`

（2）需要实现`evaluate`函数；`evaluate`函数支持重载

```java
    public Text evaluate(Text field) {
        if (field == null) {
            return null;
        } else {
            return new Text(field.toString().toLowerCase());
        }
    }
```

（3）在hive的命令行窗口创建函数，添加jar

```bash
hive(default)> add jar /home/fanl/lib/hivedemo-1.0.0.jar;
```

（4）创建临时函数

```bash
hive(default)> create [temporary] function mylowwer as 'com.fanling.hivedemo.MyUDF';
hive(default)> select mylowwer(ename) from emp;
hive(default)>
```

## 文件存储格式

Hive支持的存储数的格式主要有：`Textfile`、`Sequencefile`、`Orc`、`Parquet`

TEXTFILE和SEQUENCEFILE的存储格式都是基于行存储的，ORC和PARQUET是基于列式存储的。

在实际的项目开发当中，Hive表的数据存储格式一般选择：orc或parquet，压缩方式一般选择snappy，lzo。

## 相关问题

（1）Hive中常见的分析函数，像row_number()，rank()，dense_rank()

（2）Hive如何统计每周一，每月第一天，周，每月（第一天：trunc(日期,‘MM’)，最后一天：last_day(日期)），计算周次的用自定义函数（基姆拉尔森计算公式）

（3）Hive处理Json，用哪些函数
