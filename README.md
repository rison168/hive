### 一、Hive的基本概念

#### 1.1 介绍

Hive: 由Facebook开源用于解决海量结构化日志的数据统计。

Hive是基于Hadoop的一个数据仓库工具，可以将结构化数据文件映射为一张表，并提供类SQL查询功能。

本质是：将HQL转化为MapReduce程序。

![image-20210818162451440](pic/image-20210818162451440.png)

1）Hive 处理的数据存储在HDFS

2)  Hive 分析数据底层的默认实现是MapReduce

3)  执行程序运行在Yarn上。

#### 1.2 Hive的优缺点

* 优点
  1. 操作接口采用类Sql语法，提供快速开发能力.
  2. 避免了去写MapReduce，减少开发人员的学习成本。
  3. Hive的执行延迟比较高，因此Hive常用于数据分析，对实时要求不高的场合。
  4. Hive优势在处理大数据，对于处理小数据没有优势，因为Hive的执行延迟比较高。
  5. Hive支持用户自定义函数，用户可以根据自己的需求来实现自己的函数。
* 缺点
  1. Hive的HQL表达能力有限
     * 迭代式算法无法表达
     * 数据挖掘方面不擅长
  2. Hive的效率比较低
     * Hive自动生成MapReduce作业，通常情况下不够智能化
     * Hive调优比较困难，粒度比较粗

#### 1.3 Hvie架构原理

![image-20210818163903498](pic/image-20210818163903498.png)

1. 用户接口：Client

   CLI(hive shell）、JDBC(java 访问Hive)、WEBUI(浏览器访问Hive)

2. 元数据：Metastore

   元数据包含：表名、表所属的数据库、表的拥有者、列/分区字段、表的类型（是否外部表）、表的数据所在目录等；默认存储在自带的derby数据库，推荐使用Mysql存储MetaStore

3. Hadoop

   使用HDFS进行存储，使用MapReduce进行计算。

4. 驱动器：Driver

   * 解析（Sql Parser）：将Sql字符串转换成抽象语法树AST,这一步一般都用第三方工具库完成，比如antlr: 对AST进行语法分析。比如表是否存在、字段是否存在、sql语义是否有误。
   * 编译器（Physical Plan）: 将AST编译生成逻辑执行计划
   * 优化器（Query Optimzer）: 对逻辑执行计划进行优化。
   * 执行器（Execution）: 把逻辑执行计划转换成可以运行的物理计划。对于Hive来说，就是MR/Spark。

   ![image-20210818164904226](pic/image-20210818164904226.png)

Hive 通过给用户提供一系列交互接口，接收到用户的指令（SQL），使用自己的Driver，结合元数据（MetaStore）,将这些指令翻译成MapReduce,提交到Hadoop中执行，最后，将执行的结果输出到用户交互接口。

#### 1.4Hive和数据库比较

由于Hive采用了类似于SQL的查询语言（HQL），因此很容易将Hive理解为数据库，其实从结构上来看，Hive和数据除了拥有类似的查询语言，再无类似之处。本节将从多个方面来阐述Hive和数据库的差异。数据库可以用Online的应用中，但是Hive是为了数据仓库而设计的。清楚这一点有助于从应用角度理解Hive的特性。

1. 查询语言

   由于SQL被广泛的应用在数据仓库中，因此，专门针对Hive的特性设计了类SQL的查询语言HQL，熟悉SQL开发的开发者可以很方便的使用Hive进行开发。

2. 数据存储位置

   Hive是建立在Hadoop之上的，所有的Hiv数据都是存储咋HDFS中的。而数据库则可以将数据保存在块设备或者本地文件系统中。

3. 数据更新

   由于Hive是针对数据仓库应用设计的，而数据仓库的内容读多写少的。因此，Hive中不建议对数据的改写，所有的数据都是在加载的时候确定的。而数据库中的数据通常是要经常修改的，因此可以使用insert into ... values添加数据，使用update...set修改数据。

4. 索引

   Hive在加载数据的过程中不会对数据进行任何处理，甚至不会对数据进行扫描，因此也没有对数据中的某些key建立索引。Hive要访问数据满足条件的特定值时。需要暴力扫描整个数据，因此访问延迟较高。由于MapReduce的引入，Hive可以并行访问数据，因此即使没有索引，对于大数据量访问。Hive仍然可以体现出优势。数据库中，通常会针对一个或者及格列建立索引，因此对于少量的特定条件数据的访问，数据库可以有很高的效率，较低的延迟。由于数据访问延迟较高，决定了Hive不适合数据在线查询。

5. 执行

   Hive中大多数的查询的执行是通过Hadoop提供MapReduce来实现的。而数据库通常有自己的执行引擎。

6. 执行延迟

   Hive在查询数据的时候，由于没有索引，需要扫描整个表，因此延迟较高。另外一个导致Hive执行延迟高的因素是MapReduce框架。由于MapReduce本身具有较高的的延迟，因此在利用MapReduce执行Hive查询时，也会有较高的延迟。相对的，数据库的执行延迟较低。当然，这个低是有条件的，即数据规模较小，当数据规模导到超过数据库的处理能力的时候，Hive的并行计算显然能体现出优势。

7. 扩扩展性

   由于Hive是建立在Hadoop之上的，因此Hive的扩展性是和Hadoop的扩展性是一致的。

8. 数据规模

   由于Hive建立在集群上并可以利用MapReduce进行并行计算，因此可以支持很大的规模的数据；对应的，数据库可以支持的数据规模较小。

### 二、 Hive安装

#### 2.1 Hive安装地址

**hive官网地址**

http://hive.apache.org/

**文档查看地址**

https://cwiki.apache.org/confluence/display/Hive/GettingStarted

**下载地址**

http://archive.apache.org/dist/hive/

#### 2.2 Hive安装部署

1. hive 安装及配置

   * 把 apache-hive-1.2.1-bin.tar.gz 上传到 linux 的/opt/software 目录下

   * 解压 apache-hive-1.2.1-bin.tar.gz 到/opt/module/目录下面

   * 修改 apache-hive-1.2.1-bin.tar.gz 的名称为 hive

   * 修改/opt/module/hive/conf 目录下的 hive-env.sh.template 名称为 hive-env.sh

   * 配置 hive-env.sh 文件

     * 配置 HADOOP_HOME 路径

       export HADOOPHOME=/opt/module/hadoop-2.7.2

     * 配置 HIVE_CONF_DIR 路径

       export HIVECONFDIR=/opt/module/hive/conf

2. Hadoop 集群配置

   * 必须要启动hdfs和yarn

     ~~~shell
     [atguigu@hadoop102 hadoop-2.7.2]$ sbin/start-dfs.sh
     [atguigu@hadoop103 hadoop-2.7.2]$ sbin/start-yarn.sh
     ~~~

     

   * 在HDFS上创建/tmp和/user/hive/warehouse 两个目录并修改他们的同组权限可写。（可不操作，系统会自动创建）

     ~~~shell
     [atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -mkdir /tmp
     [atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -mkdir -p /user/hive/warehouse
     [atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -chmod g+w /tmp
     [atguigu@hadoop102 hadoop-2.7.2]$ bin/hadoop fs -chmod g+w 
     /user/hive/warehouse
     ~~~

3. Hive基本操作

   * 启动hive

     ~~~mysql
     bin/hive
     ~~~

   * 查看数据库

     ~~~mysql
     show databases;
     ~~~

   * 打开默认数据库

     ~~~mysql
     use default;
     ~~~

   * 显示default数据库中的表

     ~~~mysql
     show tables;
     ~~~

   * 创建一张表

     ~~~mysql
     create table student(id int, name string);
     ~~~

   * 显示数据库中几张表

     ~~~mysql
     show tables;
     ~~~

   * 向表中插入数据

     ~~~mysql
     insert into student values(1000, "ss");
     ~~~

   * 查看表结构

     ~~~mysql
     desc student;
     ~~~

   * 查询表中数据

     ~~~mysql
     select * from student;
     ~~~

   * 退出hive

     ~~~mysql
     quit;
     ~~~

#### 2.3 将本地导入Hive

需求：

将本地/opt/module/data/student.txt 这个目录下的数据导入到hive的student(id int, name string)

1. 数据准备

   在/opt/module/data这个目录下准备数据

   * 在/opt/module/目录下创建data

     > mkdir data

   * 在/opt/module/datas/目录下创建student.txt文件添加数据

     ~~~shell
     touch student.text
     vi student.txt
     10001 zhang
     10002 li
     10003 zhao
     ~~~

2. Hive实际操作

   ~~~mysql
   # 启动hive
   bin/hive
   # 显示数据库
   show database;
   # 使用default数据库
   use default;
   # 显示default数据库中的表
   show tables;
   # 删除已经创建的表
   drop table student;
   # 创建student表，并声明文件分隔符‘\t’
   creat table student(id int, name string) ROW formatdelimited fields terminated by '\t';
   # 加载/opt/module/data/student.txt文件到student数据库表中
   load data local inpath '/opt/module/data/student.txt' into table student;
   # Hive查询结果
   select * from student;
   ~~~

3. 遇到的问题

   再打开一个客户端窗口启动hive,会产生java.sql.SQLException异常。

   ~~~shell
   Exception in thread "main" java.lang.RuntimeException: 
   java.lang.RuntimeException:
   Unable to instantiate
   org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClien
   t
    at 
   org.apache.hadoop.hive.ql.session.SessionState.start(Session
   State.java:522)
    at 
   org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:677)
    at 
   org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:621
   )
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native 
   Method)
    at 
   sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAcce
   ~~~

原因是metastore默认存储在自带的derby数据库中，推荐使用MySQL存储Metastore;

#### 2.4 Mysql安装

1. 查看mysql是否安装，如果安装了就卸载mysql
2. 解压
3. 配置

#### 2.5 Hive元数据配置到Mysql

**启动拷贝**

1. 在/opt/software/mysql-libs目录下解压mysql-connector-java-5.1.27.tar.gz驱动包

   ~~~shell
   [root@hadoop102 mysql-libs]# tar -zxvf mysql-connector-java-5.1.27.tar.gz
   ~~~

2. 拷贝mysql-connector-java-5.1.27-bin.jar 到/opt/module/hive/lib

   ~~~shell
   [root@hadoop102 mysql-connector-java-5.1.27]# cp /opt/software/mysql-libs/mysql-connector-java-5.1.27/mysql-connector-java-5.1.27-bin.jar/opt/module/hive/lib/
   ~~~

**配置MetaStore 到Mysql**

1. 在/opt/module/hive/conf目录下创建一个hive-site.xml

   ~~~mysql
   [atguigu@hadoop102 conf]$ touch hive-site.xml
   [atguigu@hadoop102 conf]$ vi hive-site.xml
   ~~~

2. 根据官方文档配置参数，拷贝数据到hive-site.xml文件中

   https://cwiki.apache.org/confluence/display/Hive/AdminManual+MetastoreAdmin

   ~~~xml
   <?xml version="1.0"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   <configuration>
   <property>
    <name>javax.jdo.option.ConnectionURL</name>
    
   <value>jdbc:mysql://hadoop102:3306/metastore?createDatabaseI
   fNotExist=true</value>
    <description>JDBC connect string for a JDBC 
   metastore</description>
   </property>
   <property>
       <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC 
   metastore</description>
   </property>
   <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
    <description>username to use against metastore 
   database</description>
   </property>
   <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>000000</value>
    <description>password to use against metastore 
   database</description>
   </property>
   </configuration>
   ~~~

3. 配置完毕后，如果启动hive异常，可以重新启动虚拟机。（重启后，别忘了启动hadoop集群）

**多窗口启动Hive测试**

1. 先启动MySql

   ~~~shell
   [atguigu@hadoop102 mysql-libs]$ mysql -uroot -p000000
   ~~~

2. 再次打开多个窗口，分别启动hive

   ~~~shell
   mysql> show databases;
   +--------------------+
   | Database |
   +--------------------+
   | information_schema |
   | mysql |
   | performance_schema |
   | test |
   +--------------------+
   [atguigu@hadoop102 hive]$ bin/hive
   ~~~

3. 启动hive后，回到MySql窗口查看数据库，显示增加了metastore数据库

   ~~~shell
   mysql> show databases;
   +--------------------+
   | Database |
   +--------------------+
   | information_schema |
   | metastore |
   | mysql |
   | performance_schema |
   | test |
   +--------------------+
   ~~~

   **HiveJDBC访问**

   1. 启动hiveserver2服务

      ~~~shell
      [atguigu@hadoop102 hive]$ bin/hiveserver2
      ~~~

   2. 启动beeline

      ~~~shell
      [atguigu@hadoop102 hive]$ bin/beeline
      Beeline version 1.2.1 by Apache Hive
      beeline>
      ~~~

   3. 连接hiveserver2

      ~~~shell
      beeline> !connect jdbc:hive2://hadoop102:10000（回车）
      Connecting to jdbc:hive2://hadoop102:10000
      Enter username for jdbc:hive2://hadoop102:10000: atguigu（回车）
      Enter password for jdbc:hive2://hadoop102:10000: （直接回车）
      Connected to: Apache Hive (version 1.2.1)
      Driver: Hive JDBC (version 1.2.1)
      Transaction isolation: TRANSACTION_REPEATABLE_READ
      0: jdbc:hive2://hadoop102:10000> show databases;
      +----------------+--+
      | database_name |
      +----------------+--+
      | default |
      | hive_db2 |
      +----------------+--+
      
      ~~~

**Hive 常用命令**

~~~SHELL
[atguigu@hadoop102 hive]$ bin/hive -help
usage: hive
-d,--define <key=value> Variable subsitution to apply 
to hive
 commands. e.g. -d A=B or --define 
A=B
 --database <databasename> Specify the database to use
-e <quoted-query-string> SQL from command line
-f <filename> SQL from files
-H,--help Print help information
 --hiveconf <property=value> Use value for given property
 --hivevar <key=value> Variable subsitution to apply 
to hive
 commands. e.g. --hivevar A=B
-i <filename> Initialization SQL file
-S,--silent Silent mode in interactive 
shell
-v,--verbose Verbose mode (echo executed SQL 
to the console)
~~~

1. "-e" 不进入hive的交互窗口执行sql语句

   ~~~shell
   bin/hive -e "select id from student;"
   ~~~

2. “-f”执行脚本中sql语句

   ~~~shell
   touch touch hivef.sql
   # 文中写入正确的sql语句
   select * from student;
   # 执行文件中的sql
   bin/hive -f /opt/module/datas/hivef.sql
   # 执行文件转给的语句并将结果写入到文件中
   bin/hive -f /opt/module/datas/hivef.sql > /opt/module/datas/hive_result.txt
   
   ~~~

**Hive 其他命令操作**

1. 在hive cli 命令窗口如何查看hdfs文件系统

   ~~~shell
   dfs -ls / ;
   ~~~

2. 在hive cli命令窗口如何查看本地文件系统

   ~~~shell
   ！ ls /opt/module/datas;
   ~~~

3. 查看hive中输入的所有历史命令

   * 进入到当前用户的根目录/root 或 /home/rison
   * 查看.hivehistory文件

   ~~~shelL
   cat .hivehistory
   ~~~

**Hive常见属性配置**

1. hive 数据仓库位置

   * Default 数据仓库的最原始位置是hdfs的：/user/hive/warehouse路径下。

   * 在仓库目录下，没有对默认的数据库default创建文件夹。如果某张表default数据库，直接在数据仓库目录下创建一个文件夹。

   * 修改default数据仓库原始位置（将hive-default.xml.template如下配置信息拷贝到hive-site.xml文件中）

     ~~~xml
     <property>
     <name>hive.metastore.warehouse.dir</name>
     <value>/user/hive/warehouse</value>
     <description>location of default database for the 
     warehouse</description>
     </property
     ~~~

     配置同组用户有执行权限

     ~~~shell
     bin/hdfs dfs -chmod g+w /user/hive/warehouse
     ~~~

2. 查询后信息显示配置

   * 在hive-site.xml文件中添加如下配置信息，就可以实现显示当前数据库，以及查询表的头信息配置。

     ~~~xml
     <property>
     <name>hive.cli.print.header</name>
     <value>true</value>
     </property>
     <property>
     <name>hive.cli.print.current.db</name>
     <value>true</value>
     </property>
     ~~~

   * 重新启动hive,对比配置前后差异。

     ![image-20210818205952646](pic/image-20210818205952646.png)

**Hive 运行日志信息配置**

1. Hive的log默认存放在/tmp/atgugu/hive.log目录下

2. 修改hive的log存放到/opt/module/hive/logs

   * 修改/opt/module/hive/conf/hive-log4j.properties.template改为hive-log4j.properties

   * 在hive-log4j.properties文件中修改log存放位置

     hive.log.dir = /opt/moduel/hive/logs



**参数配置方式**

1. 查看当前所有的配置信息

   ~~~shell
   hive>set;
   ~~~

2. 参数的配置三种方式

   * 配置文件的方式

     默认配置文件：hive-default.xml

     用户自定义配置文件：hive-site.xml

     注意：用户自定义配置会覆盖默认配置，另外，hive也会读入Hadoop的配置，因为Hive是作为Hadoop的客户端启动的，Hive的配置会覆盖hadoop的配置。配置文件的设定对本机启动的进程都有效。

   * 命令行参数方式

     启动Hive时，可以在命令行添加-hiveconf param=value来设定参数。

     例如：

     ~~~python
     [atguigu@hadoop103 hive]$ bin/hive -hiveconf mapred.reduce.tasks=10;
     ~~~

     注意：仅仅对本次hive启动有效

     查看参数设置：

     ~~~shell
     hive(default)> set mapred.reduce.tasks;
     ~~~

   * 参数声明方式

     可以在HQL中使用set关键字来设定参数

     ~~~SHELL
     hive(default)> set mapred.reduce.tasks=100;
     ~~~

     注意：仅仅对本次的hive启动有效

   上述三种设定方式的优先级依次递增，即配置文件<命令行<参数声明。注意某些系统级的参数，例如log4j相关设定，必须用前两种方式设定，因为那些参数的读取在会话建立之前就已经完成了。

   

### 三、Hive数据类型

#### 3.1 基本类型

![image-20210818211750824](pic/image-20210818211750824.png)

![image-20210818211915603](pic/image-20210818211915603.png)

对于Hive的string类型相当于数据库的varchar类型，该类型是一个可变的字符串，不过她可能声明其中最多能存储多少个字符，理论上可以存储2GB的字符数。

#### 3.2 集合类型

![image-20210818212158337](pic/image-20210818212158337.png)

Hive有三种复杂数据类型Array/map和struct。array和map与java中的array和map类似。strcut与语言的struct类似，她封装了一个命名字段集合，复杂数据类型允许任意层次的嵌套。

**案例**

1. 假设某表如下一行，我们用哪个JSON格式来表示其数据结构。在Hive下访问格式为

   ~~~json
   {
    "name": "songsong",
    "friends": ["bingbing" , "lili"] , //列表 Array, 
    "children": { //键值 Map,
    "xiao song": 18 ,
    "xiaoxiao song": 19
    }
    "address": { //结构 Struct,
    "street": "hui long guan" ,
    "city": "beijing" 
    }
   }
   ~~~

   

2. 基于上述数据结构，我们在Hive创建对应的表，并导入数据。

   ~~~shell
   songsong,bingbing_lili,xiao song:18_xiaoxiao song:19,hui long 
   guan_beijing
   yangyang,caicai_susu,xiao yang:18_xiaoxiao yang:19,chao 
   yang_beijing
   ~~~

   注意：map/struct/array里的元素都可以用同一个字符表示，这里用“_”

3. Hive上创建测试表test

   ~~~mysql
   create table test(
      name string,
      friends array<string>,
      children map<string, int>,
      address struct<street:string, city:string>
   )
   row format delimited fields terminated by ','
   collection items terminated by '_'
   map keys terminated by ':'
   lines terminated by '\n'
   ~~~

   字段解释：
   row format delimited fields terminated by ',' -- 列分隔符
   collection items terminated by '_' --MAP STRUCT 和 ARRAY 的分隔符(数据分割
   符号)
   map keys terminated by ':' -- MAP 中的 key 与 value 的分隔符
   lines terminated by '\n'; -- 行分隔符

4. 导入文本数据到测试表

   ~~~shell
   load data local inpath "/opt/module/datas/test.txt" into table test;
   ~~~

   

5. 访问三种集合列里的数据，以下分别是Array,map,Struct的访问方式

   ~~~shell
   select friends[1],children["xiao song"],address.city from test
   where name = "songsong";
   OK
   _c0 _c1 city_
   lili 18 beijing
   Time taken: 0.076 seconds, Fetched: 1 row(s)
   ~~~



#### 3.2 类型转化

Hive 的原子数据类型是可以进行隐式转换的，类似于 Java 的类型转换，例如某表达式
使用 INT 类型，TINYINT 会自动转换为 INT 类型，但是 Hive 不会进行反向转化，例如，
某表达式使用 TINYINT 类型，INT 不会自动转换为 TINYINT 类型，它会返回错误，除非使
用 CAST 操作。

1. 隐式类型转换规则如下

   * 任何整数类型都可以隐式地转换为一个范围更广的类型，如tinyint可以转换为int,int可以转换为bigint
   * 所有整数类型、float和string类型都可以隐式地转换为Double.
   * Tinying、smallint、int都可以转换为float
   * boolean 类型不可以转换为任何其他类型。

2. 可以使用cast操作显示数据类型转换

   例如cast ('1' AS INT )将把字符串‘1’转换成整数1；如果强制转换类型失败，如执行cast('X' AS INT),表达式返回NULL。

   

