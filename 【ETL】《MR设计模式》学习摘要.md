---
date: 2017-07-15 21:53
status: public
title: 【ETL】《MR设计模式》学习摘要
---

# MR设计模式
> **个人感悟1**：花一天时间看《MapReduce设计模式》，有些写得不错。然而不得不再花半天时间来写一个读书笔记，一方面自己也许回头要看，另一方面n年后在某个面试官来问我MapReduce会不会的时候，而这些概念很大可能已经**out of cache**，可以记录一下过去了解过思考过，只是我的cache又被另外一些事物覆盖了。下文拟采用**python函数式编程**来重新描述一下这些模式，不保证**开箱即用**。
> **个人感悟2**：你背熟100本cookbook，你就能做出很好吃的菜了吗？那是不是cookbook就没用了呢，不是的，当你做菜难吃的时候回来看看。
> **个人感悟3**：这些模式是解决常见问题的**可行方法**，不等于特定问题的**最佳实践**。对于一个敏捷开发，可用则用，对于一个反复迭代的项目，请克服先入为主，他山之石很有用。Python Zen里提到“there should be one， and preferably only one ，obvious way to do it”。比如Pandas的设计初初看来有悖这个原则，比如选择一列的方式有ix，iloc，loc等等，但是其实用多了你会采用多次尝试之后一种特定的方案。**没有银弹**，只有熟能生巧。
> map-reduce的是利用key-value 分组，是groupby-apply的分布式实现。其艺术在于reducer尽量均匀，目的在解决IO与CPU的矛盾。网络IO是重要资源。

###代码约定
```
ss() 即 .strip().split()
map(mapper,combiner=None) 不区分flatmap和map
iterreducebykey  迭代处理
groupreducebykey 分组处理，即 groupbykey().mapvalues()
foldreducebykey 带初始值的迭代处理
```
###函数式编程的几点好处
1.**无副作用**，不会引入额外的改变，当你传入x1就是对x1处理，传入x2就是对x2处理，相同的x1传入保证了**一致性**结果。天然的『**面向数据编程**』。为什么说是面向数据，因为spark中的操作都是针对rdd对象开展。
2.正因为无副作用，很适合作为数据处理的**管道（Pipeline）**。
3.管道是不变的，以不变的管道应万变的数据，**天然支持了多线程**。而pyspark正是这么做的。至于Tensorflow之类是**符号编程**，那就是符号（比如placeholder，layer）加管道。
4.**函数**本身是**变量（Object）**，自己本身容易被传递、修改和柯里化（携带一定context），容易做成闭包或者装饰器。
5.对于java来说，**函数式接口**简化了接口的使用，接口中只有一个函数时只需要实现这个函数即可。
6.python很适合作为**函数式数据操纵语言**，因为语言标准里就支持了这些特性，而且同时可以作为多个系统的**胶水**。

##I、简介
1.**MR框架**：给出了什么可以做，什么不可以做的界限。相对过去的其他模型的可选项少。复杂的一面是，在框架的约束下，找到解决问题的方案非常有难度。
2.**模式**：解决问题的通用方法。
3.**作业**与**集群**上的其他任务存在**竞争关系**。
4.作业被分成一系列运行在分布式集群中的**map**和**reduce**任务。map任务负责数据载入、解析、转换、过滤，一个reduce负责处理map任务的输出结果的一个子集。
### 一、框架
1.两个过程，八个阶段：map任务(record reader,mapper,combiner,partitioner),reduce任务(shuffle混排、sort、reducer、output format)。
2.record reader 将输入的分块(split)解析成记录，即键值对(key-value),通常键是文件中的位置，比如行号（注意split间是无序的，本身存储就是无序的）。
3.mapper将kv对重组为新的kv对，为reducer准备。mapper一般是本地的。
4.**combiner**（optional）是本地的聚合，相当于局部的reducer。对于满足交换律(AB == BA)和结合律(A,BC == AB,C)的计算可以在局部计算再接着全局计算，发送(emit)的数据将少了，大大降低了网络IO。对于提升性能很重要，无副作用。
5.**partitioner** 的作用是将mapper或者combiner输出kv对拆分为分片(shard)，每个reducer对应一个分片。默认是计算key的取模哈希，然后按照reducer的个数分发到reducer，可以确保不同mapper产生的相同的key的数据分发到同一个reducer，不能保证reducer中只有一种key。一般不需要改写partitioner，等待reducer拉取即可。
6.shuffle和sort。Map发送到Reducer的数据是不能保证发送顺序和抵达顺序的，靠拉取，使用**context.write**传送数据。排序的目的是把相同key的记录聚集在一起。shuffle过程不可定制。开发人员仅可定制排序分组的Comparator对象。
7.reducer将分组好的数据作为输入，执行reduce操作，聚合、过滤、合并等操作。发送处理后的kv对到输出。其中存在(热点)数据倾斜问题，可以控制combiner、partitioner或者修改处理流程解决。
8.output format控制record writer将结果写出，HDFS或者其他存储（需要注意线程安全）。
### 二、WordCount示例
包括Mapper重写map，Reducer重写reduce，驱动程序(设置job的上下文)
reduce的签名不同于map，有一个迭代器，支持迭代处理
mapper不关心键，reducer关心
一般一个reducer一个输出文件
1.kv版本
```python
job: (Object key, Text line) -> split.map(k,v) -> context.write -> reduce(k,v) -> context.write -> output
rdd.flatmap(lambda x:(w,1) for w in x.ss()).shuffle().reducebykey(sum)
rdd.flatmap(lambda x:(w,1) for w in x.ss()).groupbykey().mapvalues(sum)
job.setCombinerClass(Reducer.class);
```
2.wordcount的单线程版本,即collection.Counter
```python
d = {}
for w in ws:
    if w in d:
        d[w] = d[w]+1
    else:
        d[w] = 1
```
另一种看起来简洁但是低效的写法
```python
d = {}
for w in ws:
	d[w] = d.get(w,default=0) + 1
```

### 三、Pig和Hive
都能翻译成MapReduce任务
优点在于可读性、可维护性、开发周期、自动调优
缺点是还有10%的问题不能在高级抽象层次解决，还要回来写mapreduce

##II、摘要模式，Describe()
1.目的：提取最小值，最大值，平均值，中位数，标准差。即特征抽象。
2.过滤：只选取所需的字段输入，减少IO，同时异常数据处理。
3.Combiner可以极大的减少网络IO传送reduce端。只需要这个计算满足结合律和交换率，即任意改变值的顺序并且随意分组不会改变结果。那么，你将可以利用combiner带来的便利。
4.一般情况不需要定制partitioner，但是当任务的数据量特别大，或者**数据倾斜**严重（reducer某个key分到的数据特别多，以至于其他并行的reducer都要等他很久，或者单个节点上处理不出来）时候，你需要定制partitioner，使得reducer的task个数合适。注：reducer或者mapper依然是竞争的，reducer槽(资源线程数)有限，得等这个task做完然后把另一个reducer task加载进来，当然可以一定程度共用jvm，而不被重启槽。
5.已知应用：单词计数，记录计数，最大最小值，平均值、中位数、标准差(不能直接用combiner)。
6.SQL语句
```sql
select MIN(n),MAX(n),COUNT(*) from table group by col2;
```
###一、最大、最小值、计数

```python
calc = lambda x,y : y if x is None else (
min(x[0],y[0]),
max(x[1],y[1]),
sum(x[2],y[2]))
rdd.map(lambda x: (id,(d,d,1)) for id,d in x.ss(),fold_combiner=(None,calc))
   .foldbykey(init=None,iter_reducer=calc)
```
** reducer和combiner是可以使用同一个reduce操作的。因为局部和全局的操作是一致的。**
若使用java代码来定制元组需要实现**Writeable接口**。
###二、平均值
本身不能直接支持结合律和交换律。
为了让combiner复用reducer代码，即局部和全局保持一致操作，那么需要增加一列用来计数。
当然，java语言仍然要注意值过大时的溢出问题。
```python
calc = lambda a,b: (
a[0]+b[0],
(a[0]*a[1]+b[0]*b[1])/(a[0]+b[0]))
rdd.map(mapper  = lambda x:(id,(1.0,len(d)) for id,d in x.ss(),
   combiner = calc)
   .iterreducebykey(calc)
#注意很多编程语言里的地板除问题，而python的类型是动态的。python2和python3的除法特性还不同，如何要确保这件事，请进行类型转换cast。
```
###三、中位数与标准差
不满足交换律和结合律，但是可以压缩传输。
1.直接实现
```python
def get(x):
    x = sorted(x)
    m, s = x.mid(), x.std()
    return tuple(m,s)
rdd.map(lambda x: (id,len(id)) for id,d in x.ss())
   .groupreducebykey(get)
```
2.内存优化，利用Treemap计数压缩
前提是假设每个数会出现多次，相当于有序的压缩存储，当然也可以不有序，仅仅是压缩目的。
注意**treemap**（红黑树）是有序的，而**hashmap**（拉链法取模哈希桶存储）在人类可读性上是无序的。
原书中实现了Itrable_SortedMap_Writable
```python
def sort_counter(kvs):
	d = collections.defaultdict()
	for k, v in kvs:
    	if k in d:
           d[k] += v
        else:
           d[k] = v
rdd.map(lambda x: (id,(k,1)) for id, k in x.ss(),
        combiner = sort_counter)
   .groupreducebykey(reducers=(mergetrees,
                    getmid,
                    getstd))
```


###四、索引模式
索引值为key
partitioner可以定制来有效负载均衡
最终结果可以是一堆文件
combiner可以与reducer用相同代码
```python
index = lambda xs: " ".join(xs)
rdd.map(lambda x: (link,data) for link, data in x.ss()
	combiner = index)
   .groupreducebykey(index)
```
###五、外部计数器计数（map-only）
适用于大数据集上的数据汇总，要求计数器数目很小
利用了context.getCounter()
当然要求外部结构线程安全，这要求框架级别的保证
是**map-only**的任务，没有传到reduce的网络IO，效率较高
```python
safe_d = {'a':0,'b':0}
rdd.map(lambda x: safe_d[k].increment(1) for k in x.ss() if k in d)
```
注：**这种用法在Pandas的groupby-apply或者apply中太经典了，和外部数据交互而已，本身的返回是None，修改了外部数据。**



##III、过滤模式，Filter（）
1.spark中是有filter函数的，根据true和false来保留true
###一、过滤 Filter（map-only）
1.过滤掉不感兴趣的大部分
2.适用场景：
- 近距离观察数据
- 跟踪某个事件的线索
- 分布式过滤
- 数据清理，移除低分数据
- 简单随机抽样
- 黑名单blacklist

3.SQL中相当于where语句
4.只有map阶段，没有排序阶段和reduce阶段，输出也是无序的，本身加载进来的时候就是无序的。当然也可以用identity reducer来收集结果。
5.分布式grep
**利用context.write不写出即可，在hive的streaming中也是不写出即可
写出的部分用(NullWritable.get(),value)来构造kv对**
```python
# 1
rdd.map(lambda x: x if x.match(re) else None)
   .filter(lambda x: x is not None)
# 2
rdd.map(lambda x: x if x.match(re) else None)
   .filter(lambda x: x)  #当然''，[]之类会被误伤，但是这是kv对，不要紧的
# 3
rdd.filter(particial ( match,pattern = 're')) # 柯里化传函数过去
```
6.随机抽样例子
spark中是有sample相关函数的
hive中有桶采样,或者where语句里填入随机数
```python
# 1
rdd.map(lambda x: x if rand() < 0.2 else None).filter(lambda x:x)
# 2
rdd.map(lambda x: x if random(0,10) == 9 else None).filter(lambda x:x)
```

###二、布隆过滤，粗粒度查询
1.**布隆过滤**：复用**少量的固定大小存储**（比如多个hash计算到bit，或者是一些hotkey，比如句子级判断可以用单词为**hotkey**），训练历史数据。新数据来的时候，当某个hash的位得到0时就可以排除在外。理论上有1%左右的**误判（false positive）**，绝对没有漏判（false negative）。
2.布隆过滤本质上是一种『**敏捷失败**』（不精确的**lazy evaluation**），即快速排除查询范围，从而降低平均的查询时间。若不能接受1%的误差，可以重新对范围内的数据用其他方法验证。前提是概率上大量查询为范围外的，这样可以**加速平均查询**，而且查询字典的存储很小。
注：『敏捷失败』的意思是如果程序注定失败，就让他立刻失败，不要再冒泡来冒泡去了。相应的有「敏捷成功」，意思是如果要成功将立刻让他成功。
3.场景：
- 移除大多数不关心的值
- **爬虫的已爬列表**
- 检查函数成本高时设计一个粗粒度的**预检查函数**，利用『敏捷失败』，当然还可以设置多级粒度查询

4.代码示例
使用自带的BloomFiler,加载到分布式高速缓存中，每个node上都能访问到
思考：高速缓存也是内存，如果实在太大，或许可以考虑加载到分布式磁盘上？
```python
bf = BloomFilter(history)
sc.broadcast(bf)  # 在java中有setup过程在mapper之前执行，但是实际上setup可以插入到任意执行位置？
rdd.filter(lambda x: bf.value.contains(x)) # 每个executor可以从高速缓存中反序列化出来
```

###三、TOP K 模式
1.相对其他过滤器，输出大小是固定的
1.若是普通的，可以用固定大小的堆进行**堆排**。或者初始化固定数量的桶，用溢出法。
2.**全局有序是非常消耗资源的，然而top10却不需要。每个处理器不断缩小数据到top10即可，最后一个reducer输出。这里mapper和reducer代码是可以复用的。**
3.**适用于K相对数据小**的情况，如果K很大，那么还是全局排序吧，两者界限很难说：
- 异常点(outlier)分析
- 选取感兴趣的指标

4.M个map和则在reducer端需要K*M个数据，那么M太小则前面处理不过来，M太大则后面IO太大且不好处理。
5.思考：可不可以使用多级reduce来解决这个问题呢，那么就需要设计一下key或者partitioner了,或者随机分配到reducer上。
6.代码示例：
```python
def topten(xs):
    heap = Heap()
    for x in flat(xs[-1]):
    	heap.push(x)
        if heap.size > 10: heap.pop()
    return (1,heap.values())
rdd.map(topten)
   .groupreducebykey(topten)
#注意rdd未必是行式输入的
```
###四、去重 Distinct()，drop_duplicated()
1.动机：重复数据占空间，引起顶层分析的倾斜。也是相当昂贵的。
2.输出：输入是无序的，输出也是无序的。
3.场景：
- 数据去重
- 抽取重复
- 规避内连接的数据膨胀：比如连接的时候有一堆nan，那么100该nan就可以连接出10000条。

4.SQL语句：SELECT * FROM sometable;
5.性能分析：如果输出太多小文件，那么给namenode造成压力，因此要求每个reducer尽量不小于block的大小输出。reduce的数目一般设为槽的数量，但是如果数据太大，可以考虑两倍槽的数量。
```python
def distinct (iternums):
    for v in iternums:
        return v
rdd.map(lambda x:(x,None), combiner=distinct)
   .groupreducebykey(distinct) 
   # combiner可以减少大量网络IO
```

##IV、数据组织
1.个别的价值被放大，节约存储
###一、分层结构
1.目的：创造出分层结构的数据，比如JSON和XML。要求插入方便，查询方便。
2.场景：
- 数据源被外键连接
- 数据是基于行的结构化的
- 预连接数据
- 为HBase或者MongoDB准备数据

3.SQL一般在关系型数据库不便于SQL分析。
4.注意点：mapper发送的多少数据，构造reducer处是否会数据倾斜。
5.帖子和评论的关联
hadoop中有MultipleInputs对象
```python
def reducer(xs):
	title, data = None,[]
    for flag, value in xs:
    	if flag == "title": title = value
        else: data.add(value)
    return None if not title else mk_xml(title, data)
rdd1.map(lambda x:(id,("title",value)) for id, value in x.ss())
.union(
rdd2.map(lambda x:(id,("data",value)) for id, value in x.ss()))
.groupreducebykey(reducer)
.filter(lambda x:x)
```
###二、分区 Partition 与分箱 
提高选取、删除、存储效率。
在Hive中的文件夹层次。使用where语句过滤。
按日期分区是常见做法。
1.必须提前知道多个分区
2.自定义分区器partitioner
3.应用场景：
- 按连续值分区
- 按类别裁剪到不同分区
- 分片

4.分区代码
```python
rdd.map(lambda x: (year,value) for year, value in x.ss())
   .setpartition(lambda key, base=2008: key- base)
   .iterreducebykey(lambda x:x)
```
5.分区是一个区一个文件，而分箱则是一个箱（文件夹）多个文件。可以是map-only的，借助于MultipleOutputs类的功能写出到不同文件夹。
6.分箱代码
```python
bins = {"hive":[],"pig":[]}
rdd.map(lambda x:bin[key].put(value) for key, value in x.ss() 
									if key in bins)
```
###三、全排序 Order by
1.是全局的，非常昂贵。
2.排序好处：方便二叉查找
3.方法：两个阶段，第一个阶段是分析阶段：随机抽样给出各个reducer的上下界。第二个阶段是排序阶段，将各个数据分发到各个上下界上，各自排序，这样拿出来就是有序的。
4.SQL语句：select * from data order by col1;
5.代码示例
对reducer排序可以借助TotalOrderPartitioner类
```python
N = 10000
small = sorted(rdd.takesample(N).collect())
n = N // 10
window = [(i,(small[i*step],small[i*step+step])) for i in range(10)]
sc.broadcast(table)
def search(x):
   for i, v in window:
       if x.isinside_window(i,x)
rdd.map(search)
   .reducebykey(sort)
# 还有两边的边界问题，须修改
```
###四、混排 shuffle()
无序化，匿名化。
可以之间使用随机键，但是要注意reducer展开的文件数，namenode压力。
```python
rdd.map(lambda x: (random(0,100),x))
   .iterreducebykey(lambda x: x)
```

##V、连接  Join
###一、连接简介
1.外键（foreign key）：用于多表连接的一个或多个字段，两个表中都要有这个字段。
注：在Pandas中做merge时有on，或者两个表中名称不同时有left_on,right_on。how有left，right，inner，outer等。
2.内连接：两边都有的key。
注：左半连接是特殊的内连接，两边key都结果中只需要左表值，相当于一个查询key是否在右表中。
3.左(外)连接：左边有的key就出现在结果里了，如果key在右表不存在则其他值用null填充。
右(外)连接:同理。
4.(全)外连接：左右任意出现过key就有，没有值用null填充。
5.反连接：全外连接减去内连接，即左有右没或者左没右有。
6.笛卡尔积(Cartesian product)或者叉乘(cross product)，是生成n x m条记录。
7.**注意事项**：如果有大量重复值，比如大量nan，会大大拖慢性能。
###二、Reduce端连接
1.直接实现的连接方法，但是速度很慢，因为大块数据要发送到reduce阶段，消耗大量网络IO。不幸的是，对于很多大数据集的**唯一方法**。
2.可以修改paritioner或者使用散列的方式，使得中间结果的键值对相对均匀一些。
3.代码
```python
rdd1.map(lambda x:(x.key,("B",x)))
.union(
rdd2.map(lambda x:(x.key,("C",x))))
.groupreducebykey(joiner)
.filter(lambda x:x)
def joiner(xs):
	bs,cs=[],[]
    for flag, v in xs:
    	if flag == "B": bs.add(v)
        else: cs.add(v)
    return logic(bs,cs)
def logic_inner(bs,cs):
	ret = [(b,c) for b in bs for c in cs]
    return ret if ret else None
def logic_left(bs,cs):
	ret =[]
    for b in bs:
    	if cs: [ret.add((b,c)) for c in cs]
        else: ret.add((b,default_null))
    return ret if ret else None
def logic_anti(bs,cs):
	if not bs or not cs:
    	[ret.add((b,default_null)) for b in bs]
        [ret.add((c,default_null)) for c in cs]
    return ret if ret else None
```
4.可以先在map中使用布隆过滤器来大大减小reduce端连接的规模。

##三、Map端连接(也叫map-side join,也叫复制连接replicated join)
1.除了一个作为驱动表的大表外，其他都是小表，可以将小表做成HashMap在分布式缓存中，在map阶段进行O（1）查询，从而没有了reduce阶段，也没有了combiner、partitioner。
2.**速度很快**，但是要求小表，受限于你愿意为map和reduce的每个task配置的内存大小。
3.代码示例
```python
b = {x.key:x for x in btable}
sc.broadcast(b)
rdd.map(joiner)
   .filter(lambda x:x)
def joiner(x):
	b=b.value
    return (x,b[x]) if x in b else None
def left_outer_joiner(x):
	b = b.value
    return (x,b[x]) if x in b else (x,default_null)
# 还有个小问题，若btable中有重复键时还需要展开一下
```
##四、组合连接（桶连接）
1.非常少见，因为要预先格式化好数据。仅仅是业务上确实经常需要才这么做。
2.对key进行相同的hash，两张表做成相同的桶数。每个对应的桶里两两做join。
3.使用CompositeInputFormat
##五、笛卡尔积
1.是平方级的，效率低，使用CartesianInputFormat读取。
2.应用场景有媒体文件的相似度分析。
3.代码示例
```python
iter((b,c) for b in bs for c in cs)
```

##VI、元模式与作业链
1.元模式metapattern:处理模式的模式。
###一、作业链
1.作业链：有些作业可以并行执行，有序则需要将他们的输出发送到其他作业。
2.需要考虑第一个作业的输出：输出临时文件多而小很耗费namenode而且耗费mapper去加载。因此，mapper处理前应该先将小文件合并一下。
3.job.waitFoCompletion可以用于提交一个作业，而job.submit可以启动多个作业，job.isComplete可以非阻塞的检查作业是否完成。如果一个作业失败了，或许应该立刻退出作业链。
4.不管作业是否成功或失败，都应该把临时目录清空。
5.并行作业链：当作业独立时，充分利用集群资源。
6.shell脚本：优点是不必重新编译代码将可以改变作业流程，也可以与其他系统工具交互。缺点是对于并行执行同时检查作业是否完成有点难度。
7.JobControl去控制ControlledJob对象。
###二、链折叠(chain folding)
1.链折叠是一种优化作业链的方法，得到性能提升。
经验：每条记录可以提交至多个mapper或者reducer，然后再提交给一个mapper。
减少在管道中移动的数据量，减少网络IO和临时磁盘IO。
2.将多个连续的map合并到一个：
rdd.map(a).map(b) 变成 rdd.map(ab)
3.最后一个map可以合并到上一个reduce
rdd.xxx().reduce(a).map(b).end() 变成 rdd.xxx().reduce(ab).end()
4.第一个map无法从下一个优化中收益。尽可能在减少数据和增大数据的操作之间拆分每个map阶段。可以将减少数据量的过程放在前一个reducer中。
**注：这一点不是很理解，先存着，慢慢消化。**
5.合并的阶段如果需要的内存太大，那么只能想办法拆开走一步是一步了。
6.尽早过滤更多的数据，也许IO开销就能满足了。
8.ChainMapper.addMapper,ChainMapper.addMapper,ChainReducer.addReducer,ChainReducer.addMapper自动进行链折叠。
###三、作业归并
1.数据加载、解析是可以合并的，比如hive对数据schema-on-load(存储时对数据类型不严格，加载时确定类型)，那么加载是很耗时的。可以共用加载。
2.可以将两个mapper代码放一起，生成kv时用标签标记区分map源，reducer中用if来切换到实际的处理逻辑，用MultipleOutputs把作业的输出分开。
rdd.map(ma).reduce(ra)
rdd.map(mb).reduce(rb)
合并为
rdd.map(ma,mb).reduce(ra,rb)
注：弱弱地问一句，这样真的好吗?if切换真的不会把逻辑打乱吗？

##VII、输入和输出格式
###一、自定义输入输错
RecordReader、InputFormat
OutputFormat、RecordWriter
1.InputFormat（getSplits，createRecordReader）
检查输入配置,数据是否存在(依赖检查)
按照InputSplit类型拆分成逻辑分块，一个分块分配一个map task
创建RecordReader的实现，根据原始的InputSplit生成键值对，逐个发送给对应的mapper
2.RecordReader(initialize,getCurrentKey,getCurrentValue,nextKeyValue,getProcess,close)
创建kv对
3.OutputFormat(checkOutputSpecs,getRecordWriter,getOutputCommiter)
验证作业的输出配置
创建RecordWriter的实现
4.RecordWriter(write,close)
负责将kv对写入文件系统或者其他输出中。
###二、生成数据
可以凭空的生成数据，比如随机数据
###三、外部输出
可以写入非本地的位置，比如jdbc，但是要注意写出时的线程安全，线程池容量
###四、外部源输入
比如Redis输入
###五、分区裁剪
分区裁剪应该是『透明的』，可以在不同数据集上反复执行
比如hive文件分区字段过滤加载，裁剪后map task显然大大减小了

##VIII、未来发展
1.多维数据
map-reduce擅长于一维记录的场景，而忽略了数据先后顺序。
2.流式数据
同时处理1小时的数据对资源有压力
且map-reduce在hdfs的分块存储对流式有要求
有Storm
3.YARN的影响
每个作业一个ApplicationMaster，每个节点负责一个NodeManage，ResourceManger来调度
4.作为库或者组件的模式。