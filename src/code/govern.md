# **大数据治理之血缘系统**





 

## 一、需求背景

元数据与数据治理，是每一个大数据平台必须考虑的实际情况，这也是企业级数据湖的重要部分。面临的问题包括：怎样梳理数据的时间线变化？任务、表、字段等数据，如何进行复用、清理、热度统计？怎样对表、字段等进行权限控制、任务治理以及上下游依赖影响分析？

数据的产生、加工融合、流转流通，到最终消亡，数据之间自然会形成一种关系。我们借鉴人类社会中类似的一种关系来表达数据之间的这种关系，称之为数据的血缘关系。这篇文章介绍大数据与区块链研发中心开发的血缘系统，旨在阐述分享血缘系统的实现。

 

## 二、技术选型

目前的开源解决方案有2种，WhereHows系统和Atlas系统。

1、 Atlas 的介绍

一个Apache实现的元数据存储系统，通过kafka或者REST API对外连接元数据产生系统。

核心有3个主要模块：ingest/export负责元数据的接入或导出；type System涉及到entities的概念；Graph Engine是一个图数据库的引擎；底层的图数据库是Titan。

再底层的存储，有2个，元数据一般用HBase数据库，索引使用Solr存储。当然，可以分别配置修改成BerkeleyDB和ElasticSearch。支持表级血缘，字段级支持不完善。

 

2、 WhereHows的介绍

WhereHows是LinkedIn开源的元数据治理方案。Azkaban调度器抓取job执行日志，也就是Hadoop的JobHistory，Log Parser后保存DB，并提供REST查询。WhereHows太重，需要部署Azkaban等调度器，以及只支持表血缘，功能局限。

 

3、对比总结

​    调研过WhereHows，其所依赖组件非常多，而且github上的项目，文档滞后非常严重，编译会遇到非常多的问题，对于一个开源项目，这是致命伤，对使用者非常不友好。而且

其依赖Azkaban，而我们团队正在逐步下线Azkaban。再其不支持字段级别的血缘。

而Atlas 也是一个庞大的系统，需要依赖titan，solr等。而我们尝试去修改其元数据存储组件为HBase，但Atlas却只支持非常低级别的Hbase。ElasticSearch也是同样的问题，支持的版本太低。而大数据团队为该系统再维护2个低级别的HBase和ElasticSearch，成本太大。

基于以上分析，我决定自行开发一个血缘系统。支持表级别和字段级别的血缘关系，图形数据库使用Neo4j，同时采用HBase当辅助存储。同时打通大数据团队的ETL系统和接入分发系统Databus。

 

## 三、整体设计

下图为血缘系统的整体设计图：

![img](https://cdn.nlark.com/yuque/0/2020/png/1608572/1592309906701-1b9ad041-9728-45b8-a7dc-95de962c5452.png)

 

1、 组件解释

（1）   Neo4j：一个图形数据库，所有的血缘关系都存储于Neo4j

（2）   LineageMgr：解析Hive Sql的组件，并且得到血缘关系。

（3）   LineageAccessor：通过Restful Api服务，暴露血缘关系的数据访问。

（4）   ETL：大数据平台的ETL平台

（5）   Databus：大数据平台的接入分发平台

 

2、实现步骤

LineageMgr服务通过Antlr 解析器（org.apache.hadoop.hive.ql.parse. ParseDriver使用Antlr），解析存储于HDFS的已经成功执行的Hive Sql，得到Hive Sql的抽象语法树（AST），通过深度遍历语法树，采集Hive Sql中包含的源数据表和目标数据表，将其存储入neo4j图形数据库，同时将详细的元数据存储于HBase。这样存储分离，neo4j发挥了自己的图形存储的能力，而HBase互补了neo4j节点和边不能存储过长信息的问题。

 

通过分析ETL和Databus系统中数据库，将总共6种数据源引入血缘关系的系统。同时，针对neo4j的特定属性建立索引，优化查询的Cypher语句。通过Dubbo框架暴露该血缘关系服务的功能。

 

 

## 四、技术实现

 

整个系统的难点和工作量，在于对Hive Sql 的深度遍历，采集相应的血缘信息。当采集到了需要的血缘关系和业务数据，剩下的就是把数据存入Neo4j图形数据库，对数据建立合适的索引，即可保证Neo4j的写入和读取速度。

以下阐述一下主要开发步骤。

 

1、 筛选出所有需要考虑的Hive Sql的类型，并分类考虑。

读操作，比如 SELECT fld FROM tbl，是不会产生血缘关系的；只有写操作，才会产生血缘关系。

经过查看Hive Sql 的语法文档，总结出主要有3种情况产生血缘关系。

1） CREATE TABLE target AS SELECT col1,col2 FROM source1;

CREATE TABLE yh123_cp LIKE zdyh.yh123;

2）INSERT OVERWRITE TABLE dstTbl SELECT age_3 FROM third;

3）INSERT INTO TABLE dstTbl SELECT age_3 FROM third;

 

比如，1）的第一个Hive Sql会产生的表级别血缘关系为：source1  target

字段级别血缘关系为：source1.col1  target.col1，source1.col2  target.col2

 

当然，为了便于阐述，这是极度简化后的Hive Sql。实际的线上Hive Sql中，会包含很多关键字：比如WHERE, JOIN…ON, UNION, AS等。考虑到这些关键字，给遍历采集增加了一定的难度。

 

2、 抽象语法树（AST）的结构说明

先通过一棵AST的例子，讲解一下TOKEN的概念。Hive Sql如下：

CREATE TABLE dst_table AS

SELECT pv.*, u.gender, u.age

FROM user u JOIN page_view pv ON (pv.userid = u.id);

 

这句Hive Sql对应的AST如下图：

![img](https://cdn.nlark.com/yuque/0/2020/jpeg/1608572/1592309906857-e3ca3d69-1e1c-43f7-9ff4-c14d2ccd9409.jpeg)

 

其中，

**TOK_CREATETABLE**：代表create table 操作

**TOK_QUERY**：以此TOKEN为根的子树，代表的含义为 select XXX from YYY.

**TOK_FROM**：代表从哪个表读取数据

**TOK_INSERT**：一般会有2个子节点。TOK_DESTINATION，TOK_SELECT。表示TOK_SELECT表示select 出来的字段；TOK_DESTINATION代表写入的终点表，如果其子节点为TOK_TMP_FILE，而非表名，则代表这是一个select操作而已，没有写入。

**TOK_TABREF**：描述表名的节点，当有2个子节点时，代表Hive Sql中使用了别名，如果只有1个子节点，则使用原表名。

 

3、 抽象语法树（AST）的规律总结

 

3.1表级别血缘关系

\1)   INSERT INTO/OVERWRITE 的AST，必然存在以下路径：TOK_INSERT => TOK_DESTINATION => TOK_TAB

\2)   TOK_INSERT 只有一个 TOK_TAB，即为 dst_table，其 src_table 为 同级的 TOK_FROM.

\3)   多重插入表现为：多个 TOK_INSERT, 一个 TOK_FROM

\4)   TOK_FROM 可以包含子句

\5)   TOK_UNION 看成，多个 TOK_QUERY 并列；即，src_table 会有几个

 

如果仅仅考虑表级别的血缘关系，只需要收集TOK_FROM下面的TOK_TABREF节点，把真是表名，别名数据采集到；然后再在TOK_DESTINATION节点下采集到终点表。那么血缘关系就可以得到。

 

3.2字段级别的血缘关系

当需要采集字段级别的血缘关系时，像表级别那样只考虑几个特定节点，不能满足需求，需要考虑更多的节点。这让整个解析增加了很大的难度，需以新的角度看待AST。

 

\1)   走通用的思考路线，因为Hive Sql存在子查询的操作。每个子查询都是一个”Hive Sql”，而且子查询理论上可以嵌套很多层。如果用特殊的方法解析了一层子查询，若子查询包含子查询，则无法走通。

\2)   基于上一点，使用栈的数据结构，来存储子查询中的数据上下文。每次遇到TOK_QUERY节点，将当前的记录采集信息的map和list都压栈。

\3)   任何一个节点，对其直接及间接子节点都存在影响，提供了一种语境（Context）的限制作用。

\4)   JOIN关键字，意味着多个表作为输入，对于并列平等的数据，可以使用列表，或者说数组来存储。

\5)   TOK_UNION, 下面会有 2 .. n 个 TOK_QUERY, 是并列关系，所以，也需要数组。

\6)   当select * 出现时，需要根据表名，从Hive的metastore 中读取所有的字段。

\7)   选择正确的数据结构，会大大有利编程。结构越抽象，越统一，数据结构越相同，代码越好写。对于相同对象的表示，前后阶段，不同场景，最好保持统一的数据结构表示。比如，字段要用 (db, tbl, field) 的三元组，不要有些场景用二元组( tbl, field)。

 

 

## 五、应用场景

目前，大数据平台基于血缘系统的产品应用有如下几样：

 

1、知识中心的血缘展示

这是最基本的使用，标记数据的来龙去脉，助力数据分析。如下所示。

![img](https://cdn.nlark.com/yuque/0/2020/png/1608572/1592309907004-56f80fe1-c8f7-4edf-b806-81ec97efc5ea.png)

（1）标记的边，意味着一条血缘关系，点击这条边，会显示（2）中的Hive Sql，指示产生血缘关系的Hive Sql。（3）可以指定查看血缘关系的来源和去向，并指定层数。

 

产品进入路径：链接为http://bdp.sf-express.com/datamap，进入大数据平台知识中心，选择数据库和数据表，即可选择查看血缘关系。 如：http://bdp.sf-express.com/datamap/#/info/adl/convenience_store_info

 

2、统计表的引用次数，可以区分数据的冷热。

3、敏感传递，实现字段、 表、库的权限级别控制。表和字段的属主，可以通过标签系统，给字段赋予一个级别，这步操作，可以传递给子字段。从而让权限控制更加自动化。

 

