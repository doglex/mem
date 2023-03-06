---
date: 2017-04-21 14:25
status: public
title: Python杂记(逻辑误区/性能优化/Pandas+Jupyter)
---


## 一、逻辑误区
1.break不出两层循环怎么办
- 修改成一层循环
- 设置一个status变量来维护跳出标志
- 放进函数里，return或者raise可以忽略所有循环（raise开销可能大）

2.如何调试
- 使用print(或sys.stdout.write)输出，使用raise
- 使用测试用例（doctest、单元测试）等
- 使用pycharm断点调试，多用todo
- 使用jupyter(ipython)边写边调试，再用pycharm进行整理。文件量一大，用jupyter交互很方便，不用重新加载。

3.字典要注意是hashable的key
- 列表是mutable的，元组是hashable

4.内存引用问题
- L=[0]*10 不会有问题，LL= L*3 就要出问题，引用到一起去了
- 函数中参数传入一个列表，或者字典，之后在函数中的修改是全局的（堆区）。合适的时候可以利用这个特性而省去global。

5.装饰器（@另一篇博客）
6.Hook技术
如stdout = sys.stdout
7.元编程。不要用元编程，除非你非常清楚他的行为，可能产生的后果。
```python
def tryMe(self):
    print('I am inside the class. Cool')
def x(self):
    self.me = "dogless"
MyClass = type('A_Class', (), {'tryMe': tryMe,'__init__':x})
m = MyClass()
m.tryMe()
print(m.me)
print(MyClass)
```
8.编码问题
用python3可以少很多问题。实在不行再考虑encode、decode，字符串前加u。codecs库或许可以处理文件编码。
9.multiprocessing闪退
- 使用多进程时，要注意arg后面的参数要加逗号
- 闪退原因是类里定义了函数，命名空间变成main了，需要直接挂到类上去
- target函数有装饰器可能导致闪退
- 不正当的 monkey patch

10.class中的return self，让对象支持串联的点操作
11.class中的方法要变成静态(比如spark的sc的要求)，那样用@staticmethod装饰。在一开始就初始化了，类的所有对象都可以调用，不需要self参数。
12.当内存需求大于2G时，不能用32位的python，需要改用64位的Python
13.尽量用with来管理上下文
14.不用virtualenv，改用虚拟机。重新解压一个虚拟机只需要两分钟。
15.try里是raise不出来的，要注意这个问题
```python
try:
    raise SystemExit("raised")
except:
    print("not raised")
```
16.虽然生成器好用，比如生产者-消费者模型，但是不要滥用
17.更很多语言不一样，if里不能赋值，调用后再判断。比如很多语言用while来读取标准输入的时候。
18.后缀不需要是py的，可以是任意文本
19.除法//是地板除，/在python2里是地板除，python3里是真实除。如果无法转成int可能要手工round。
20.rjust，ljust可以给字符串填充宽度
21.文件处理常用 line.strip().split()

## 二、性能调优
0.一般是分治，分支界限，贪心，动态规划，减少冗余数据。
1.如果内存可以，一次读文件（或者1000行），一次写文件（使用换行符拼装）。不要一行一行的弄，太费IO。print也是不要做多行print，而使用换行符拼接。使用join效率高。
2.reduce、map、filter、lambda 很慢的，虽然编程思维高级，千万别用。lambda在pySpark中用一用也不要紧。
3.zip好像还是挺快的，做大量迭代的时候。zip(*)是解包。利用*和两个*传参或解包。一个*是array类型，两个是key-value形式的。
4.tuple比list快几百倍，若后续不修改，优先使用tuple，这就是tuple存在的理由。另外，元组解析式是一次性的，但是比迭代器解析比列表解析快。
5.字典dict的效率可以认为O(1)。如果dict很大，也会慢，可以嵌套dict来加快。多用dict、set类型的O(1)性能。
6.多进程加协程效率高
- 计算密集型，使用multiprocessing。core的数量一般设为CPU线程数，如果都是小任务，可以设大一些，跑满。
- IO密集型，考虑gevent(requests库有grequests版本)。猴子补丁中 thread和socket不能覆盖掉标准库的，monkey是给python3用的。GIL下协程仍然是一个线程，协程和子进程不同在于函数中可以中断。
- 另外通过shell写批处理进行多进程也可以，减少需要Manager来锁的部分，尽量各种运行

7.cpickle比pickle快多了，python3中已经没有了。cStringIO同理。
8.列表的拷贝可以用 a[:]就是拷贝了。
9.循环外可做的放循环外。循环是成倍数的。多次迭代的地方一定要优化，IO问题更是如此
10.一般要把递归改成循环快很多。另外，python中最深是50层递归。
11.用%做字符串格式化比format方式快很多，但效率允许的情况下应该用format。
12.不要借助中间变量来做swap，使用元组解析的方式。  a,b,c = c,a,b
13.if i is True 比 if i == True 快，但是if i 更快
14.使用级联比较  a< b < c
15.while 1 比 while True快，True最终转成1。
- for 循环比while 效率高
- 生成器比for循环效率高
- numpy更快。C底层实现。

16.使用两个*比pow函数快，很多内置函数都比较慢
17.使用C扩展，使用PyPy，并行，分布式，性能分析工具
18.正则匹配re效率比Beautifulsoup效率高很多，但是bs4可以自动处理很多问题（比如转义，编码）
19.列表解包
- for u,v in somedict.items()
- for i,v in enumerate(somearr)

20.高效的排序 sorted，是泛型的，可以指定key。而min，max也可以指定key。
21.try-except耗时比if-else高多了，不得已才使用。else能不用就不用可以提高效率，但是可能会导致可读性上的损失。


## 三、Pandas + jupyter
1.如何导入（中文处理、显示格式）
```python
#coding=utf-8
#python2
import warnings
warnings.filterwarnings('ignore')  # 关闭警告
import pandas as pd
% matplotlib inline
import matplotlib.pyplot as plt
from matplotlib.pylab import *
mpl.rcParams['font.sans-serif']=['FangSong']
pd.set_option('display.width', 320)
pd.set_option('display.max_colwidth', -1)
pd.set_option('float_format','{:20,.2f}'.format)
from os import listdir
for d in listdir("."):
    print d.decode("gbk")
top5=pd.read_csv(u"top5_new.csv",encoding="utf-8",dtype={'myimsi':np.str,'imsi':"str"})   #指定dtype
```
2.seaborn中文显示
```python
import matplotlib.pyplot as plt
from matplotlib.pylab import *
import seaborn as sns
sns.set(style="darkgrid")
mpl.rcParams['font.sans-serif']=['FangSong']
```
3.常用功能
- astype类型装换
- axis=0 是列，1 是行操作
- ffill前向填充，fillna,dropna
- rank排名,sort_values排序
- to_csv,to_excel,to_dict,to_list
- df.columns
- cut线性划分，qcut分位数划分
- 装换成numpy时as_matix
- 特别注意和sklearn一起使用时要显式np.array,不然是DataFrame类型
- %%timeit计时

4.数据过滤
- xi7[(xi7['cross_value']>1) & (xi7.name.str.endswith("shuai"))]
- 支持点访问（填变量名，但是要注意与预定义的冲突）和中括号（填字符串）访问

5.实用的groupby-apply处理框架（类似map-reduce）,基本是O(N)的效率
```python
bigdata=[]
def somefunc(sub_df):
  classname= list(sub_df.classname)[0]
  tmp_dict={}
  for item in sub_df.itertuples():
      tmpdata=item["data"]
      if tmpdata in tmp_dict:
          tmp_dict[tmpdata] += 1
      else:
          tmp_dict[tmpdata] = 1
  bigdata.append(tmp_dict)
df.head(1000).grouby(["classname","typename"]).apply(somefunc)
```
- pandas 报一些NaN之类的其实是，index没有正确的reindex，因为sub_df仍然采用了外部df的索引。 **对索引操作一定要十分小心**。索引错误也往往导致了pandas画图错误，或者可以取出数据后画图。
- iloc的效率很低的，因为每次都要取索引（ix比iloc、loc效率低但通用，但是**三者都效率很低,往往导致O(N2)**，这个索引不是O(1)的,千万不要用这三个），最好是用迭代遍历的方式(itertuples)，或者列操作，按行直接用for

6.pandas处理大型excel或者有问题的excel，当做xml进行解析。因为用read_excel已经不行了
```python
df=pd.read_html("reporter_status.xls",encoding="utf-8",)
df[0].to_csv("reporter.csv",encoding="utf-8",header=None,index=None)
```
7.多表操作 merge(how=,on=,left_on=,right_on=),concat([],axis=)
- leftjoin保证了左边的key有
- rightjoin保证了右边的key有
- inner保证左右同时有
- outer保证一边以上有

8.高效构建Dataframe
- 动态增加df是低效的，不要做这种事。宁愿用shell里的cat来拼接，如果需要去掉第一行，那么有sed工具。修改某些列，有awk工具。
- from_dict,list,csv是高效的，用DataFrame()构造是高效的
- 最好一次构建，或者concat
- 一个表太大超出内存，可以读一部分列usecols，可以使用iter参数迭代加载。据说1T以内的数据用Pandas都要比Spark快。