---
date: 2017-07-10 21:52
status: public
title: 【ETL】《Hive编程指南》摘要
---

> 思考1：一个矛盾，分区字段的数量太少，那么不方便查询，而且如果两个作业共用一个分区，那么insert_overwrite就容易把另一个作业的结果覆盖。分区数量太多，那么容易造成大量小文件，哪怕是只有一条记录（而这条记录就是一个作业的结果），hdfs就得分给他一个数据块，造成namenode压力。是否考虑非hdfs存储？

Hive表的管理借助元数据库（derby或者mysql），存储借助HDFS，查询借助HQL转成MapReduce，锁借助Zookeeper
一.基本类型：
```
TINYINT,SMALLINT,BIGINT,BOOLEAN,FLOAT,DOUBLE,TIMESTAMP,BINARY
INT,STRING —string一般是比较好用的，而Hive基本可以无缝转换
```
二.集合类型
STRUCT,MAP,ARRAY
三.分隔符
```
记录分隔 \n
字段分隔 \001 ctrl+A）cat someone.txt | awk -F "\001" 'BEGIN{OFS="\t";}{print $1,$2,$3}’ > b.txt
ARRAY和STRUCT 分隔 (ctrl+B)
MAP键值分隔(ctrl+C)
```
四.读时模式
与传统数据库不同，写入时不进行类型检查。读出时类型检查，不符合设为null。
五.不支持行级插入，不支持事务(其实是可以的)。增加了外部程序扩展。
六.Hive数据库本质是目录或者命名空间。分区是符合条件的文件夹，子文件夹，比如.../event_day=20170606/event_name=‘on’/…,其他字段在这种文件夹内，可以节约这个分区字段的存储开销，而且查找分区时速度快，也可以整块删除分区。
七.表
```
CREATE DATABASE x； — - 为每个数据库创建一个目录，在hive.metastore.warehouse.dir的顶层目录下的x.db
CREATE DATABASE  IF NOT EXISTS x； — - 对于连续程序可以忽略错误
CERATE DATABASE LOCATION ‘/my/location’ COMMENT ‘replace default storage’; — 修改dfs中的目录
SCHEMA 可以替代TABLE关键字
SHOW DATABASES LIKE ‘h.*’;
USE namespace xs; —使用为当前空间
use x； —使用为当前数据库
show tables；
drop database if exists x cascade； — cascade 表示迭代删除其中的表，restrict 表示要检查其中是否还有表。
alter database x set DBPROPERTIES (‘editor’=‘xs’) — 其他元信息是不能修改的，仅仅这个描述信息可以修改
```
八.默认创建的是内部表，hive拥有完全管理权，但是hadoop dfs工具也是可以删除表的。也可以创建外部表，使用EXTERNAL关键字，指定LOCATION。当删除外部表时，不会删除数据，只是删除derby（单线程）或者mysql（多线程）里的元信息。
对于hive管理的表，可以复制表结构
```
DESCRIBE EXTENED tablename； — 查看是否管理表
create external table if not exists mydb.e3 like mydb.e1 location ‘/my/path'
```
九.分区
where语句对于分区字段来说就是文件目录过滤器，依赖是根据这个目录是否存在
当where里没有指定分区字段的值时，mapreduce会扫描整个hdfs的这个表，非常消耗性能
```
set hive.mapred.mode = strict ; —-严格要求指定分区字段，而nonstrict则不需要
show partitions table name；
alter table table0 add partition (year=2012,month=1,day=2)  — 添加分区
```
容错：hive不关心分区是否存在，查询不到就不过是fetch到0条而已。
十.表
```
drop table if exists atable； — 可以从.Trash中恢复
alter table a rename to b; — 重命名表
```
十一.装载数据
hive不支持行级insert，需要进行批量载入。
— 从文件载入
```
LOAD DATA LOCAL INPATH ‘${ENV:HOME}/calivfonia-employee’
OVERWRITE INTO TABLE employee
PARTITION (country = ‘US’, state = ‘CA’); —若分区不存在会自动创建
```
— 从查询载入
```
insert overwrite table employees
partition (country = ‘US’, state = ‘OR’)
select * from staged_emplyees se
where se.onty = ‘US’; —若overwrite则会先删除原来相应的分区，如果是into则是追加到该分区
```
— 动态分区插入
```
insert overwrite table employee 
partition (country = ‘CN’, state)
select  xxx, se.st 
from staged_emplyee se;
需要开启动态插入功能
set hive.exec.dynamic.partition =true;
其他相关参数 
hive.exec.dynamic.partition.mode
hive.exec.max.dynamic.partitions.pernode
hive.exec.max.dynamic.partitions
hive.exec.max.created.files
```
十二.导出数据
文件方式
```
hadoop fs -cp source_path des_path
```
查询方式
```
insert overwrite directory ‘/user/xxx’
select xxx;  — dos
insert overwrite local  directory ‘/home/xxx’
select xxx;  — local path
```
注意导出为000000_0 文件，如果有多个reducer，则有000000_1等，导出的文件中默认分隔符\001和\002，但是不可见
十三.查询
```
select name, salary from employee;
select e.name from employee; 
注意：返回的集合类型数据是带双引号的，如果获取不到NULL则不加引号。
select name, properties[“key”], dogs[0] from employee;  —map类型，数组类型,也可以用点操作选择列
```
1.正则表达式指定列
select price.* from atable;
2.使用列值进行计算
select round(salary*(1-deductions[“Tax"])) from employee;
cast(salary AS FLOAT) < 100000.0; --类型转换
3.算术运算
A+B,A-B,A*B,A/B,A%B,A&B,A|B,A^B,~A
4.常用函数
常用函数
round，floor，ceil，rand,exp,ln,log10,log2,pow,sort,bin,hex,unhex,
conv,abs,prod,sim,asin,cos,tan,degrees,radians,positive,negative,sign,e,pi
5.聚合（分组减少）函数
通常与 groupy by 结合来做，select name from xxx group by name; 本身name就是做了distinct，比直接用distinct要好，因为distinct只有一个reducer。
group by 使用时要注意是正确的聚合逻辑。
set hive.map.aggr = true； 会消耗更多内存在map阶段进行『顶级』聚合过程，加速聚合过程
count(1) —比count(*)效率高，数行数，不含有NULL值的行。
count(expr) —符合条件的行数
count(distinct expr)
count (distinct col) 如果是分区列可能有得到0的bug
sum(col),sum(distinct col)
avg (col),avg(distinct col)
min(col),max(col)
variance(col),var_pop(col) 求方差， var_samp(col) 取一部分的方差
covar_pop(col1,col2),covar_samp(col1,col2) 协方差
corr(col1,col2) 相关系数
percentile() 分位点处的值
histogram() 直方图数据
collect_set() 去重之后的集合
6.表生成函数
将单列扩展为多列或者多行
explode  将集合类型生成到多行
inline   将Struct类型提取出来插入表中
json_tuple,parse_url_tuple 获得tuple
stack(n,col1,,,,colM) 将M列转换为N行，每行有M/N个字段，n要求常数
7.其他函数
ascii，base64，binary，cast类型转换，concat拼接字符串，concat_ws带分隔符的拼接,context_ngrams,decode,encode
find_in_set,format_number,get_json_object,in,in_file,insert,length,locate,lower,lcase,lpad,ltrim,printf,regexp_extract,regexp_replace,repeat,reverse,rtrim,sentenses,size,space,slit,str_to_map,substr,translae,unbase65,from_unixtime,unix_timestamp,to_date,year,month,day,minute,second,datediff,date_add,date_sub,from_utc_timestamp,to_utc_timestamp
8.LIMIT 语句
限制返回的行数
9.列别名
用空格或者as
select salary as s from atable；
10.嵌套查询
```
from ( select xxx from xxx) t select t.xxx where t.xxx = ‘xxxx';
```
11.选择查询case...when...then用于单列
select name, case when salary <1000 then ‘low’ when salary >1000 and salary <5000 then ‘middel’
else ‘high’ as brack from employee;
12.什么情况下Hive可以避免进行mapreduce
第一，当过滤条件只有分区字段时自动应用了本地模式。
第二，设置hive.exec.mode.local.auto=true 则会在其他情况下也尝试本地模式。
十四.where语句
1.不能在where中使用列别名。不过可以用嵌套查询的方式
2.谓词操作
返回true或者false
可以用于where，JOIN...ON,Having
```
=  相当于其他语言的 ==
<=> 如果都为NULL则True，其中任一为NULL则NULL。注意是0.9.0以后的hive支持，而厂版的QE是hive0.8.0.
<> 或者 !=
<,>,<=,>=
A [NOT] BETWEEN B AND C 三个值任意为NULL则NULL，否则相当于  B<=A<=C
A IS NOT NULL
A [NOT] LIKE B   —  %号匹配任意字符，_匹配单个字符
A RLIKE B, A REGEXP B —正则匹配，与JDK一致，返回TRUE或者False，比如’.*(China|Ontario).*’
AND
OR
NOT
```
3.浮点数的陷阱
float的0.2与double的0.2比较结果更大，而不是相等。
第一种方法是定义字段类型为DOUBLE。
第二种方法是显式指定cast as float。
十五、GroupBY语句
1.通常与聚合函数一起使用。
类似于Pandas中的Groupby-apply框架，类似于MR中的Map-ReduceByKey
一个Key就是一个逻辑上的group。但是也可以直接apply，apply不限于聚合操作。
select year(ymd),avg(price_close) 
from stocks
where exchange = ’NASDAQ’ and symbol = ‘AAPL’
group by year(ymd);
— 可以发现，year(ymd)在一组中是一致的，如果查询的字段不一致，将报错。
— 但是groupby-apply，或者map-groupbykey- mapValues之类的操作将自由多了
2.HAVING子句
对Grouby的查询后的结果做限制，相当于嵌套了一层查询。
select year(ymd),avg(price_close) 
from stocks
where exchange = ’NASDAQ’ and symbol = ‘AAPL’
group by year(ymd)
having avg(price_close) > 50.0;
十六、JOIN语句
只支持SQL中的等值连接。
1.INNER JOIN 当两个表中都存在的数据保留
```
select a.ymd,a.price_close,b.price_close
from stocks a JOIN stocks b ON a.ymd = b.ymd
where a.symbol = ‘AAPL’ and b.symbol =‘IBM’
```
— ON是连接条件，where可以限制各个表中的数据，同时可以使用表别名。先JOIN，后Where过滤。如果不清楚顺序，应该用嵌套查询。
— 这里使用同一个表，是自连接。
— ON的条件是等值连接，mapreduce中难以实现非等值，标准SQL是支持非等值的，Pig工具支持交叉生产
— ON中不支持OR语句
Left semi JOIN，优化了的内连接，查询结果中只需要左表的值。
Hive不支持右半开连接。
2.多表连接
```
select a.ymd,a.price_close,b.price_close，c.price_close
from stocks a JOIN stocks b ON a.ymd = b.ymd
              JOIN stocks c ON a.ymd = c.ymd
where a.symbol = ‘AAPL’ and b.symbol =‘IBM’
```
— MR的顺序先连接a，b；再将结果与c连接。从左至右连接。
优化1：ON键连续连接，HIVE尝试将前面的表缓存起来，后扫描后面的表。
因此，用户需要让表的大小从小到大从左至右排列。
优化2：『标记』告诉查询优化器哪张是大表，作为驱动表。
select /*+STREAMTABLE (s) */ s.ymd,s.symbol,sprice_close,d.divided
from stocks s JOIN dividends d ON s.ymd = d.ymd AND symbol = d.symbol 
where s.symbol = ‘AAPL’;
优化3：map-side JOIN
3.Left Outer Join 左外连接，左边有的key将保留，在右表匹配不上的列的值填充NULL。比如算增量时。
Right Outer Join 右外连接。
FULL Outer JOIN 两边任意ON键都放进来。
4.笛卡尔积。N行xM行
5.map-side JOIN
可以在map的时候将小表放入内存，像查字典一样，从而减少reduce。
set hive.auto.convert.join = true;
set hive.mapjoin.smalltable.fileze = 25000000;
SELECT /* MAPJOIN (d) */ s.ymd,s.symbol,s.price_close,d.dividend
from stocks s join dividends on s.ymd = d.ymd and s.symbol = d.symbol
where s.symbol = ‘AAPL’;
— Hive0.7以后可以不指定mapjoin的表。
— 如果数据是分桶的，则大表也可以优化。简单的说，必须按on键分桶，且一张表的桶数是另一张表的若干倍。
set hive.optimize.bucketmapJOIN =true;
— hive对于右外连接和全外连接不支持这个优化。

十七、排序
1.Order By和 Sort BY
尽量不用order by。
ORDER BY 是全局排序，所有数据需要通过一个reducer，当hive.mapred.mode 设置为strict的时候严格要求加LIMIT
SORT BY 是局部排序，各个reducer各自排序，不保证全局有序
都可以使用升序asc，降序desc，缺省是asc
2.含有SORT BY的DISTRUBUTE BY
DISTRIBUTE BY 可以控制map的输出在reducer中是如何划分的，如果不指定，默认是按哈希值进行均匀划分。可以手工指定键，以保证在这个键上的sort by是有序的。
```
select s.ymd,s.symbol,s.price_close 
from stocks s 
distribute by s.symbol
sort by s.symbol ASC, s.ymd asc
```
— 可以保证在各个s.symbol上是有序的
— distribute by 一定要在 sort by 之前
3.CLUSTER BY 是 SORT BY+DISTRUBUTE BY的简写，当两者的键一致时
```
select s.ymd,s.symbol,s.price_close
from stocks s 
cluster by s.symbol;
```

十八、类型转换
1.cast (value AS TYPE)
2.浮点型转整数建议用 round 和 floor 函数，不要用cast
3.转换为binary值
select (2.0*cast(cast (b as string) as double)) from src;

十九、抽样查询
1.分桶查询
select * from numbers TABELSAMPLE( BUCKET 3 OUT OF 10 ON rand()) s;  —随机抽取
select * from numbers TABELSAMPLE( BUCKET 3 OUT OF 10 ON number) s;  —依赖于某列，是确定的，但是要注意哈希的均匀性问题
select * from numbersflat where number % 2 = 0;
2.数据块抽样
select * from numbers TABELSAMPLE(0.1 Percent) s;
— 是基于行数的，不是所有文件格式适用
— 最小抽样单元是数据块（与基于行数不矛盾），当数据块小于128M时，返回了所有行。
— 种子信息可以在hive.sample.seednumber配置

二十、合并查询
1.UNION ALL
类似pandas中的concat，可以对多个查询结果合并。
```
select log.ymd,log.level,log.message,
from (
    select l1.ymd, l1.level, l1.message, ‘LOG1’ as source 
    from log1 l1
UNION ALL
    select l2.ymd, l2.level, l2.message, ‘LOG2’ as source 
    from log1 l2
) log 
sort by log.ymd asc;
```
2.UNION
会去重，需要扫描整个表。效率低，建议用UNION ALL。

二十一、视图
1.视图是一个逻辑上的表，并不存储数据，只是一个表的象征。
可以先执行视图，来简化子查询。
实际上是制定了一个逻辑流程。
对于查询效率没有实质上的提升。
hive没有物化视图。
视图是只读的，只允许修改TABLEPROPETIES的描述信息。
原语句
```
from (
select * from people join cart 
on (cart.people_id = people.id)
where firstname = ‘john'
) a
select a.lastname where a.id = 3;
```
以视图修改
```
create view shorter_join as 
select * from people join cart
on (cart.people_id = people.id)
where firstname = ‘john’;
select lastname from shorter_join where id = 3;
```
2.通过视图限制访问
create view safe_table as 
select firstname, lastname from userinfo;
3.通过视图从map、struct、array中取出一部分数据，从而把数据压平。
4.删除视图
drop view if exists shipments；

二十二、HIVE索引
1.hive支持的索引有限。
可以帮助裁剪数据减少mapreduce压力。但对有些查询是没用的。
需要消耗额外的存储，创建索引本事也消耗计算资源。
可以用EXPLAIN 语句来查看某个语句是否用了索引。
2.创建索引
除了S3外，对外部表和视图都可以建立索引。
create index employees_index 
on table employees (country)
as ‘org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler’  — 也可以是BITMAP类型
with deferred rebuild 
IDXPROPERTIES (‘creator’=‘me’.’at’=’some time’)
in table employee_index_table
partition by (country, name);
3.重建索引
数据改变时索引无法自动触发重新索引。
alter index employees_index
on table employees
partition (country = ‘cn’)
rebuild;
4.显示索引
show formatted index on employees；
5.删除索引
表或者分区删除时，相应的索引也会被删除。hive用户不允许在删除表之前删除索引。
drop index if exists employees_index on table employee；

二十三、模式设计
应尽量避免反模式
1.按天划分的表
supply_2017_01_06
这样表的数量增长是很快的，是一种反模式。
需要改成一张表的各个分区，用event_day作为分区字段。
2.分区
分区很有用，减小了全盘扫描。
分区应该是一些文件的合并，每个分区下的文件应该足够大。如果分区数过多，就是文件夹数量过多。而NameNode需要保存元信息，保存不下了。
3.hive不保证唯一键和标准化，数据可能是重复的，是为了保证TB级和PB级的计算。
4.同一个数据的多次处理，只需要一轮扫描
低效率
insert overwrite table sales
select * from history where action = ‘purchase’;
insert overwrite table credits 
select * from history where action = ‘returned’; 
高效率
from history 
insert overwrite sales select * where action = ‘purchase’;
insert overwrite credits select * wherer action =‘returned’;
5.设计中间表，按天划分分区，减少很多重复工作。
6.数据分桶
设计分桶字段，比如对user_id字段分区就不好了。
桶的数量是固定的，根据hash将数据分到不同的桶中。
一个优点是mapsideJOIN，另一个优点是分桶采样。
7.为表增加列
有serDe功能，Hive对列是宽松的，可以增加列，没有值的地方就是NULL。
8.使用列式存储
hive默认行式存储（Hbase是列式存储），但是serDe也支持列式存储。
列式存储在下列情况是比较好的：字段中大量重复数据，列的数量很多。
9.总是使用压缩。
MapReduce任务是偏于IO密集型的，有多余的CPU用来压缩解压缩。但是对于CPU开销大的机器学习就不要用压缩了。

二十四、调优
hive会自动转换为map-reduce
1.使用EXPLAIN解释
抽象语法树ABSTRACT SYNTAX TREE
执行阶段STAGE DEPENDENCIES，一个stage可能是map，reduce，limit，抽样，合并等等
STAGE PLANS
2.使用EXPLAIN EXTENDED 可以有更多的信息
3.LIMIT调整
很多情况LIMIT语句会执行整个查询
关闭整个查询
set hive.limit.optimize.enable = true
还有两个相关的参数
hive.limit.optimize.row.max.size和hive.limit.optimize.limit.file
缺点就是JOIN和GROUP BY的操作可能看到结果是很不相同的。
3.JOIN 优化
mapsideJOIN
小表放在前面
4.对于小任务临时使用本地模式
set oldjobtracker = ${hiveconf:mapred.job.tracker};
set mapred.job.tracker = local;
set mapred.tmp.dir = /home/edward/tmp;
select xxx;
set mapred.job.tracker = ${oldjobtracker};
set hive.exec.mode.local.auto =true; --自动启用
5.并行执行
有些stage间的依赖并不大。可以允许并行。
set hive.exec.parallel = true;
6.严格模式
第一，限制分区表必须要有分区过滤才能查询。
第二，ORDER BY 必须有LIMIT。
第三，限制笛卡尔积查询。
set hive.mapred.mode = true;
7.调整mapper和reducer的个数
考虑数据量大小和数据的操作类型
dfs -count 可以查看输入量大小
设置 hive.exec.reducers.max
mapred.reduce.tasks
需要尝试几个固定值试试
一般默认值是可以的。
8.JVM重用
不同的task不需要重新启动JVM，可以共用一个JVM。
坏处就是会被task插槽会被占用直到job结束。
set mapred.job.reuse.jvm.num.tasks = 10;
9.索引
10.配置动态分区
控制输出流
hive.exec.dynamic.partition.mode
hive.exec.max.dynamic.partitions
hive.exec.max.dynamic.partitions.pernode
dfs.datanode.max.xcievers
11.推测执行
侦测将执行慢的TaskTracker加入黑名单
进行一些重复的计算
mapred.map.tasks.speculative.executor = true ;
mapred.reduce.tasks.speculative.executor = true ;
12.单个mapreduce中多个group by
hive.multigroupby.singlemr = false ;
13.虚拟列
通过查询这些字段，用户可以知道在哪个文件哪行数据出现了出错。
set hive.exec.rowoffset = true;

二十五、文件格式和压缩
1.安装编解码器
hive -e “set io.compression.codecs”;
2.压缩选择
BZip2压缩大，但是CPU开销大
Gzip次之
之后是LZO和Snappy
3.开启中间压缩
对map和reduce的中间数据进行压缩减少传输
set hive.exec.compress.intermediate = true;
mapred.map.output.compression.codec;
4.输出结果压缩
hive.exec.compress.output
mapred.output.compression.codec
5.sequence file
可分割的方式压缩
6.存档分区 HAR
归档减小NameNode的压力，但是查询效率不高，也不是压缩的

二十六、插件
1.Log4J
hive-log4j.properties用于控制CLI和其他本地执行组件的日志
hive-exec-log4j.properties用于空转mapreduce task内的日志
2.可以连接java调试器到hive进行单步调试
3.使用hook，在执行前，执行后。
4.hive_test可以单元测式

二十七、函数UDF
hive通过管道将数据传递到另一个处理系统中，处理后传回来。
1.自带函数
show functions；
describe function concat；
describe function extended  concat；
2.UDAF 聚合函数：多变少
3.UDTF 表生成函数：少变多，甚至零输入
但是，hive限制了从表中生成其他列。
4.JAVA
需要继承UDF类并实现evalute()函数
打包成JAR文件
create temporary function zodiac as ‘org.apache.hadoop.hive.contrib.udf.example.UDFZodiacSIgn’;
或者是GenericUDF类，支持更好的null处理。
5.UDF中访问分布式缓存
UDF可以访问分布式缓存，本地文件系统，分布式文件。但是性能差了。

二十八、Streaming
MAP(),REDUCE(),TRANSFORM()
建议使用通用的TRANSFORM
Streaming为外部进程开启了一个IO管道，数据传入标准输入，之后从标准输出返回到Streaming API job
TRANSFORM用的是UTF-8
1.简单变换
select transform(a,b) using ‘/bin/cat’ as newA, newB from default.a; —恒等变换
select transform(a,b) using ‘/bin/cat’ as (newA INT, newB DOUBLE) from default.a; —改变类型
select transform(a,b) using ‘/bin/cut -f1’ as newA from a; —投影
select transform(a,b) using ‘/bin/sed s/4/10/‘ as newA,newB from a; —一部分不输出到标准输出的话，行数是可以更改的
2.分布式缓存
将文件或者依赖的数据分发到每个节点，让每个节点都能执行这个文件
add file ${env:HOME}/prog_hive/ctof.sh
select transform (col1) using ‘ctof.sh’ as convert from a;
3.使用其他程序
只需要控制好输入输出即可
每行使用tab进行分隔
一行产生多行或者减少行只需要控制输出行即可，python的print是自带换行符的
select transform (col1) using ‘python ctof.py’ as convert from a;
select transform (col1) using ‘perl ctof.pl’ as convert from a;
自定义python解释器
使用add cachearchive的方法把tar.gz添加到分布式缓存，Hive会自动解压压缩包
解压后的目录名是和压缩包名称一样的：
./<压缩包名字>/<压缩的相对路径>/bin/python
add cachearchive ${env:my_workbench}/share/python2.7.tar.gz;
transform中使用：
using 'python2.7.tar.gz/bin/python my.py' 
4.将Streaming用于聚合
比如求和
```
#!/usr/bin/perl
my $sum=0;
while (<STDIN>){
my $line = $_;
chomp($line);
$sum=${sum}+${line};
}
print $sum;
```
```
add file ${env:HOME}/aggregate.pl;
select transform (number)
using ‘perl aggregate.pl’ as total from sum;
```
5.利用CLUSTER BY进行wordcount
CLUSTER BY 可以保证同一个key在同一个reducer中，且数据有序，但是一个cluster可能有多个key
相当于集成了distribute by 和 sort by
注意UDF分发到各个机器上，不好利用全局字典
```
import sys
for line in sys.stdin:
    words = line.strip().split()
    for word in words:
        print “%s\t1”%(word.lower())

import sys
last_key, last_count = None,0 
for line in sys.stdin:
    key, count = line.strip().split(“\t”)
    if last_key and last_key ！= key:
        print “%s\t%t%d” (last_key,last_count)
    else:
        last_key = key
        last_count += int(count)
if last_key:
    print “%s\t%t%d” (last_key,last_count)
```
```

CREATE TABLE docs (line STRING);
CREATE TABLE word_count (word STRING, count INT)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ‘\t’;

FROM (
FROM docs
SELECT TRANSFORM (line) USING ‘${env:HOME}/mapper.py’
AS word, count 
CLUSTER BY word ) wc
INSERT OVERWRITE TABLE word_count
SELECT TRANSFORM (wc.word, wc.count) USING ‘${env:HOME}/reducer.py’
AS word,count; )
```
6.Streaming 也可以用JAVA

二十九、自定义Hive文件和记录格式
STORED AS TEXTFILE; —方便awk，sed，grep等工具,也可以压缩。不过对于二进制文件用文本存储浪费空间和IO
STORED AS SEQUECEFILE; —是含有key-value的二进制文件，适合Hadoop,Spark中转
STORED AS RCfile; —是列式存储的，当列中的排重量很小或者列特别多或者列中很多null值时，相对高效。
SerDe 记录格式，有好几种，比如RegexSerDe正则序列化和反序列化，csvSerDe，tsvSerDe
自定义格式。
XML UDF
XPath
JSON SerDe
Avro Hive SerDe 主要特点是一个进化的模式驱动的二进制数据存储模式

三十、Hive的Thift服务
bin/hive - - service hiveserver &
netstat -nl | grep 10000 

三十一、存储处理程序、NoSQL
1.结合了InputFormat,OutputFormat
实现HiveStorageHandler接口连接即可
2.HBase
3.Cassandra
4.DynamoDB

三十二、安全
hive.files.umask.value
hive.metastore.autorization.storage.checks
hive.metastore.execute.setugi
hive.security.authorization.enabled
hive.security.authorization.createtable.owner.grants
hive.security.authorization.enabled

三十三、锁
Hive通常是一次性写入的，细粒度的锁不需要。
但是缺少在update和insert时的列、行、查询级别的锁，因为Hive是多用户的。
所有的锁必须由单独的系统来协调。
1.使用Zookeeper支持锁
Zookeeper对Hive用户是透明的
2.显式锁和独占锁
LOCK TABLE people EXCLUSIVE;
UNLOCK TABLE people;

三十四、Oozie支持工作流程
类似QE引擎，也可以网页控制
1.MapReduce
2.Shell
3.Java动作
4.Pig
5.Hive
6.DistCp：集群数据拷贝

三十五、留给读者讨论
> 我目前还不清楚下面问题的答案，也还没有空余的时间仔细研究，欢迎读者留言探讨。

1.where x in ('a','b')和 where x = 'a' or x = 'b'在解释为mapreduce的时候会不会不同？抛开具体的问题，in语句和or语句在哪些情况下的mapreduce解释不同？如何在不使用explain的情况下确知这件事情？
2.Hive对lazy evaluation的支持到底如何？当多个判断条件用and级联在where中的时候，这些判断条件中用分区字段，有普通字段，如何恰到好处的安排多个判断条件的顺序以达到一个平均查询时间上的最优？Hive是如何“智能”的做这件事情的？