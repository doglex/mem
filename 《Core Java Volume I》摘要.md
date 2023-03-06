---
date: 2017-05-10 16:30
status: public
title: '《Core Java Volume I》摘要'
---

> 原书与c++比较
> 增加与python的比较
```
# 权限
- public 公共可见
- private 自己可见
- protected 自己、子孙（有些子孙不是本包的）、本包可见
- 默认 本包可见

# 栈区和堆区
- 栈区(stack)：一般速度快，单线程使用。当前方法，局部变量，基本类型。
- 堆区(heap): 比较慢（要安全），多线程共用。有常量池，方法代码，静态方法，对象。

# C1:概述
1.简单性：是C++语法的"纯净版"，没有头文件、指针、结构、联合、操作符重载、虚基类，文件小（小型机）
2.面向对象：重点在对象（数据），用接口取代多重继承，运行时自省
3.分布式 TCP/IP
4.健壮性：消除重写内存和损坏数据可能性
5.安全性
6.体系中立：字节码
7.可移植性(int 固定为4字节)，所有数值类型占有字节与平台无关
8.即时编译(早期java是解释型)
9.高性能：编译器消除函数调用（即“内联”）
10.动态性（其实静态语言）

#C2:Java程序设计环境
1.JDK，开发工具；JRE，运行环境 
> 建议：.tar.gz解压安装而不是RPM

2.环境变量：JAVA_HOME=.../bin;   PATH = $JAVA_HOME:$PATH
3.源文件 src.zip
4.编译运行
```
javac Welcome.java   //生成.class文件（字节码）
java Welcome         //不要.class后缀
```
# C3:基本程序设计
##3.1、程序设计
1.源代码的文件名必须与共on类的名字相同，并用.java后缀
2.一个文件只能一个公共类 //若有多个公共类，编译成class时就混乱了
3.每个Java程序必须有一个main方法，而且是public static的
4.代码块{}
```java
public class ClassName{
	public static void main(String [] args){
    	program statements;}
}

```
5.所有函数都属于某个类，main必须有一个外壳类
6.println换行，print不换行(sout)
7.注释： //行注释，/*...*/块注释，/\*\*.../文档字符串注释
## 3.2、数据类型（强类型语言，8种基本类型(primitive type)
1.四种整型 int，short，long，byte
>长整型后缀L或l
>十六进制前缀0x或0X
>八进制前缀0
>二进制前缀0B或0b
>允许下划线 (_000_000) 区分
原码(0正1负，符号位;)
反码(正不变；负符号位不变，其余反)，存储在计算机内部
补码(正不变；负的反码加一)

2.两种浮点型
float，后缀f或F
double，后缀d或D //默认是double型
三个特殊值Double.POSITIVE_INFINITY,Double.NEGATIVE_INFINITY,Double.NaN (0/0,负数平方根，判断用Double.isNaN)

3.char类型用单引号'A',unicode编码，占两个字节（C++中一个字节）
从 \u0000到\uffff，要当心不要在注释中出现\u
**不要在程序中使用char类型，除非确实需要处理UTF-16单元，最好用String抽象数据类型**

4.boolean类型：false和true。不能与整型转换(C++中可以)

##3.3变量
1.逐一声明变量可以提高可读性
2.初始化（使用前必须初始化）
```
int a = 12; //栈区不须new
int a; a = 12;
Map <String, Integer> map = new HashMap<>(); //堆区，非基本类型，对象。须new来实例化，模板里需要是非基本类型
```
3.常量：用final表示只能被赋值一次
一般设大写，一般是static final，让类内共用，外部再加public
>static静态（共用）：一开始就初始化，类拥有，不是对象拥有。static方法只能调static属性或者方法，因为非static是实例化后的对象所拥有的。（有特例，比如String，可以调）

4.运算符
(1)地板除（整数除）
(2)除零异常，浮点数除0得无穷大或者NaN
(3)异或A^B^A = B, A^A =0

5.数学函数与异常
在Math类中。要严格的话用StrictMath。

6.类型转换(cast)
(1)小到大自动转换（仅在运算到时）
(2)强制类型转换（int）(Math.round(x))
(3)布尔类型转换请用条件式  ?:

7.结合运算符 +=,*=
8.自增，自减 ++i，i++,--i,i--(先后使用i)
> 请确保i是可修改的
> 建议不要使用，以免产生疑惑

9.逻辑
==, equals（字符串，浮点，isNaN）,> ,>= ,&& ,|| ,?:
== 是内存地址判断，equals由自定义类的equals方法，同时修改hashcode方法

10.位运算(加法)
无进位用异或a^b,判断进位用与&，进位用<<1
```java
int Add(int a, int b){
int jw = a & b;
int jg = a ^ b;
while (jw){
	int t_a = jp;
    int t_b = jw<<1;
    jw = t_a && t_b;
    jg = t_a ^ t_b;} 
return jg;}
```
11.运算符优先级：建议始终使用括号
12.枚举类型  enum Size {SMALL, MEDIUM, LARGE} ;Size s = Size.MEDIUM;
>可以获得枚举值或者null

13.字符串String：是final的，不可被继承
(1)不是Primitive Type，内容在常量区(属于堆区),引用在栈区。
(2)用双引号可以不用new
(3)子串substring(start,end+1)创建了一个新串
(4)拼接
- 支持+号。将其他类型与字符串拼接，自动变字符串(或者Autoboxing后有toString方法)。常用于输出语句。
System.out.println(1+2+"hello"+3+4); 输出3hello34（加号的结合性）
- 多个拼接用String.join
String.join(",","S","M","L")  //用逗号拼接
//python中用",".join(...)其中是可迭代内容就行，但是要求各元素是str
- 频繁拼接用StringBuilder
- 需要多线程的拼接用StringBuffer，是线程安全的

(5)不可变字符串String(比如+号是新创建了对象，常量区)
优点：编译器可以让字符串共享。(回收是自动的，有强引用，弱引用，软引用，虚引用)
//设计者认为共享带来的高效率远胜于提取、拼接字符串带来的低效率
(6)检测字符串相等用equals
//自定义类要设计equals方法，用==的话两个对象引用有可能不一样的。
(7)空串与null串
空串检测：  str.length() == 0 或 str.equals("")
null串：   str ！= null && str.length() !=0   //在null上调用方法会出错
(8)码点CodePoint，对一些特殊字符方便处理
(9)其他API：charAt,compareTo,equals,startswith,indexOf,lastindexof,replace,substring,tolowerCase,trim(删除头尾空格)(python中有strip)
split(re)方法传入“”将不能去掉连续空格，而python中的split()不传参可去掉连续空格
(10)构建字符串
StringBuilder -> append -> toString
StringBuffer 线程安全，但是慢

14.输入输出
(1)读取输入
Scanner in = new Scanner(System.in)  //（可以获取标准输入或者shell重定向）import java.util.Scanner
//nextline,next,nextInt,nextDouble,hasNext
//密码输入用Console类
(2)格式化输出
System.out.printf("%8.2f",x); //8宽（用空格填充），2个小数点
System.out.printf(",.2f",x); //千位用逗号
//s总是可以格式化，优先用Formattable接口的formatTo方法。不行，再用toString方法。
(3)文件输入
Scanner in = new Scanner(Paths.get("myfile.txt"),"UTF-8");
//不要直接传"myfile.txt"，会当作字符串输入处理
//要记得用close关闭流
(4)文件输出
PrintWriter out = new PrintWriter("myfile.txt","UTF-8") ;
//out,print,printf,println
//要记得catch 上 IOException

15.控制流程
//没有goto，但是可以用break或者continue可以跳到指定label:{}
(1)块作用域
不能嵌套在两个块中声明同名变量，但可修改
变量声明后不可修改类型(python是可以的)
(2)条件语句
if(){;}else if {;} else{;}
//一定带括号，不能像python一样无括号
(3)循环
while(true){;}
do{;}while{;} //保证至少执行一次
(4)确定循环
for (int i =0 ; i 小于 x.length(); i++){;} 
//初始化，计数器执行前检查，更新计算器
//在外部无法使用
//若需要外部继续使用，应先在外部定义
//for循环只是while循环简化形式
(5)多重选择Switch（相当于小型map）
Switch(choice){
case 1:...;break;
case 2:...;break;
default:...;break;}
//要注意break，不然会穿透
//标签可以char，byte，short，int，enum，String

16.中断控制
break 加标签可跳出多层循环/或任何块
continue 加标签，跳到块首
> 建议完全不用break，continue进行编程

17.大数值
BigInteger,BigDecimal //import java.math，有add、multiply函数
18.数组 （[]与Array）
(1)int [] a = new int [100] 
//索引0-index，用[]访问 (堆上创建、栈上引用)
//固定大小，扩容须拷贝到另一个数组（0.75倍增长）
// 必须是同一类型元素，而python中则可以多种元素
(2)foreach 迭代（循环）
for(int e:a){;}  //必须是数组或实现了Iterable 接口的对象
(3)初始化
int [] small = {1,2,3,4}; //隐藏了new
重新初始化 small = new int [] {1,4}  //因为还是数组指针
//允许长度为0，与null不同
(4)数组拷贝
允许重复引用  int [] a = b;
深拷贝  int copied = Arrays.copyOf(lucky,lucky,length)
(5)命令行参数
psvm(String [] args)
(6)数组排序
Arrays.sort() 
集合类型有 Collections.sort()
//Arrays具体，Array抽象
//基本类型，三路快排/双基快排，小的部分用插入；对象类型，用归并排序。
//python中有sorted，带key的sortedd
(7)Arrays API：toString,copyOf,copyOfRange,binarySearch,fill(填充某个值),equals(比较数组各元素)
(8)多维数组
int  [][] square = new int [][] {{1,2,3,4},{1,2,3,4},{1,2,3,4},{1,2,3,4},{1,2,3,4}} ;
或者 int  [][] square =  {{1,2,3,4},{1,2,3,4},{1,2,3,4},{1,2,3,4},{1,2,3,4}} ;//用[][]访问
//axis=0，先行后列
for (double [] row = a ) for (double value:row){;}
//实际上是一维数组的
//不等长数组
int [][] odds = new int [NMAX][];
for (int n=0; n小于 NMAX;n++) odds [n] = new int[n+1];

## C4:对象和类
### 一、面向对象OOP（封装/继承/多态）
1.OOP:只要对象能满足要求，就不关心其功能的具体实现。//将数据放在第一位，操作第二位
2.类
(1)实例化(instance)：类构造(construct)对象的过程
(2)封装/数据隐藏(encapsulation)：将数据（对象的实例域 instance field）与方法（method）组合在某个包中，关键是不让类中方法直接访问其他类的数据。
(3)继承（inheritance）:通过扩展一个类来建立另一个类。原始类是Obeject
3.对象：
(1)对象的行为（behavior）：可以对对象施加哪些操作
(2)对象的状态（state）:操作时，对象如何响应
(3)对象的标识(identity):区分相同行为与状态的对象
> 状态改变必须通过调用方法（setter/getter）
> 对象状态影响它的行为(是否已婚影响是否可调用结婚方法)

4.设计类：先设计类，再添加方法
5.类间关系：依赖(uses-a),聚合(has-a),继承(is-a)
6.UML图（Unified Modeling Language，统一建模语言）

###二、预定义的类
1.使用构造器(constructor)来构造新的实例，new Date().
//在Java中，任何对象变量的值是对存储在另外一个地方的一个对象的引用（堆上创建，栈上引用）
//所有对象在堆中
//将一个方法用于null会运行时错误
2.Local Date类
UTC纪元：1970年1月1号 00:00:00
Date：表时间点
LocalDate：日历表示
用静态方法构造(factory method)实例 
LocalDate.of(1999,12,31) //有getYear,getDay等方法
3.更改器(setter,mutator method)方法修改实例域状态，访问器(getter, access method)获取值

###三、用户自定义类
//完整的程序若干类，一个类中有main
1.多个源文件使用
javac Employee*.java  //编译多个
或 javac EmployeeTest.java //只须主入口，相当于自动make
2.private使实例域/方法仅被自身使用
3.构造器
(1)与类同名
(2)可以多个构造器，参数不同
(3)没返回值
(4)总是伴随new操作调用
(5)不要在构造器中定义与实例域重名的局部变量
(6)一般是public，也有不是的
4.隐藏参数（对象指针this）
(1)总是有隐式参数this指向对象本身//相当于python中的self
(2)static方法没有this，因为对象还不存在(pyspark中的sc要求@staticmethod)
(3)所有方法必须在类内部定义，但是未必内联，因JVM决定
5.封装的优点
(1)改变类的内部实现，除了该类的方法外，不影响其他代码
（比如返回值类型可以固定，不像python随意）
(2)必须通过更改器修改，可执行类型检查
//若需要返回一个可变数据域的拷贝，应该进行clone
6.访问权限
私有方法（一般是辅助方法helper）不对外暴露，不应成为公共接口，也减少IDE的提示呀
7.Final 实例域
一次初始化。防止被人修改。//然而其他语言编写的Native Method是可以突破final的

###四、静态static(共用)
1.静态域(也类域)
类拥有，而每个对象有一个引用拷贝(共享)，不属于任何对象。比如Dog类，白狗、黑狗，用一个static int n_dog来维护狗群大小。
2.静态常量（不用private，本身大家共用）
public static final double PI = 3.14;
3.静态方法
(1)第一次加载(ClassLoader)时初始化
(2)所有参数显式(没有this)，因为还没有实例化，不能向对象操作
(3)不能访问非静态方法，因为非静态方法有this参数，属于对象拥有
4.工厂方法(factory method)来生成新对象
LocalDate.now,LocalDate.of
原因：无法命名构造器；或者当使用构造器时，无法改变所构造的对象类型
5.main方法，也是静态方法。但是可以操作String

###五、方法参数
1.值调用：方法接收的是调用者提供的值
2.引用调用：方法接收的是调查者提供的变量地址
3.Java是值调用(对象引用的时候是拷贝了一个引用，操作后删除拷贝)
//C++两种调用都有

###六、对象构造
1.重载（overload）：同名方法，返回值类型相同，参数不同
//返回类型不是方法签名(signature)一部分，必须相同
//像python不能保证防返回值类型，没有重载，但是有默认参数（靠后）
//重载不止适用于构造器(Constructor)
2.默认参数：Java不支持默认参数，但是可以通过重载来实现该功能
3.无参构造器：使用默认提供的构造器是危险的，其行为不确定
4.用this()可以调用另一个构造器，节省书写
5.初始化块：
类中有一块{},每一次被初始化会执行，不常见不必需
6.对象析构：Java不支持析构器，有finalize方法，但不知何时被调用。有关闭钩Runtime.addShutdownHook.

###七、包
相当于using namespace，一般域名逆序，文件夹组织
1.类的导入
(1)一个类可以使用所属包中的所有类，及其他包中的共有类
(2)直接包全名访问，或import语句
(3)import java.util.* 没负面影响（和python不一样），但是不大明确，而且可以命名冲突
2.静态导入
import static java.lang.System，可以直接使用静态方法和静态域了。
3.将类放入包中
(1)将包名放在源文件开头
package com.hostmann.corejava ; //禁止用户用java.开头的包名
//没有这行的话是一个default package

###八、类路径
1.JAR包用ZIP格式组织
2.classpath不允许通配符*以防止shell命令进一步扩展
3.采用参数 -classpath,-cp来运行
> 不建议用CLASSPATH固定的环境变量或防止jre/lib/ext下。

###九、文档注释
1.插入/\*\*....\*/，可嵌入html代码
2.类注释
3.方法注释  @param　@return @throws
4.静态常量须注释
5.通用注释 @version @author @deprecated  @see
6.包注释： 提供package.html 或 package-info.java
7.注释抽取，用javadoc工具

###十、类设计技巧
1.保证数据私有
2.一定要对数据初始化
3.不要在类中使用过多基本类型。数量过多，使用另一个类。
4.不是所有域都需要访问器和更改器。
5.责过多的类进行分解（解耦）
6.类名和方法名要体现他们的职责
7.优先使用不变的类

##C5:继承
1.继承（Inheritance）于已存在的类创建一个新类。
2.反射（Reflection序运行期间发现更多的类及其属性的能力。
//更python的元编程(type)有类似之处?
###一、类，超类(superclass)，子类(subclass)
1.extends 定义子类
(1)扩展超类的时候，仅需要指出子类和超类的不同之处。
(2)通用的方法在超类中，特殊用途的方法在子类中。
2.覆盖(Override)
超类的方法在子类不再使用。比如，子类不能访问超类的私有域，需要用super.get().
3.子类构造器
public B(a,b) {super(a);this.b=b;}
> this两个用途：隐式参数，调用类的其他构造器
> super两个用途：调用超类方法，调用超类构造器

4.多态：一个变量(使用一个超类或者接口的应用)可以指示多种实际类型，不同的行为。
比如 Map map = new HashMap(); Map map = new TreeMap(); 
这样调用map.get()时遇到不同类型行为不同。
>　调用哪个方法是动态绑定的（dynamic binding）。
>　置换法则：任何地方可以用子类的置换。除非声明final、static、private进行静态绑定。
>　C++中要做动态绑定就需要声明为虚方法

5.继承层次：(1)多个继承链 (2)Java不支持多重继承，但有接口，来代替
6.调用过程
(1)JVM为继承链创建方法表
(2)JVM搜索方法相应的类，知道调用哪个(动态绑定或者静态绑定)
(3)调用方法
7.阻止继承：final 类和final方法。确保子类不会改变语义。比如String是final的。
> 没有被覆盖的代码且简短，可以被JVM用内联(inling)处理

8.类型转换
进行转换的唯一原因：在暂时忽略对象的实际类型后，使用对象的全部功能
只能在继承层次内进行类型转换
超类转子类前进行instaceof检查

9.抽象类
(1)abstract
为了提高程序的清晰度，包含一个或多个抽象方法的类，本身必须声明为抽象。
(2)除抽象方法外，抽象类还可以包含具体的数据和方法。
(3)通用的域和方法放在超类中
(4)抽象方法充当着占位的角色，具体实现在子类中
(5)抽象类不能被实例化。因为没有实现抽象方法，实例化了再调用这个方法会出问题。
//为什么要声明抽象方法，因为这样可以从父类引用来调用。

10.受保护访问
- private 仅本类可见
- public 所有类可见
- 默认无修饰符 对本包可见
- protected 对本包和所有子类可见
> Java中protected安全性比C++差
> 慎用protected属性。因为对这个类的实现修改，就必须通知所有使用这个类的程序员。违背了OOP数据封装性。

###二、Obect：所有类的超类
1.在Java中，只有基本类型不是对象，其他都是。
2.equals方法
(1)getClass()返回一个对象所属的类，不相同的不能进行比较。
(2)equals方法应该具有的特性
- 自反性：x.equals(x)返回true
- 对称性: x.equals(y) 则 y.equals(x)
- 传递性：x.equals(y)，y.equals(x) 则x.equals(z)
- 一致性：反复调用结果应该相同
- 对于任意非空引用x，x.equals(null)应该是false

(3).备注
- 若子类能够拥有自己的相等概念，则对称性需求将强制采用getClass检测
- 若由超类决定相等的概念，那么就可以用instanceof进行检测，这样可以在不同子类对象之间进行相等的比较
- 对于数组类型的域，可以使用静态的Arrays.equals方法检测数组元素是否相等

3.hashCode方法
(1)散列码是由对象导出的一个整型值，默认的散列码是对象的存储地址
(2)Equal是与hashCode定义一致
4.toString方法
(1)getClass().getName()获得类名的字符串
(2)只要用+号与String相连，就会调用toString方法
//python中有__str__()可读性好，__repr__()执行好
(3)print会自动调用toSring
(4)数组打印会得到一堆字符乱码，应该改用Arrays.toString(luckynums)
###三、泛型数组列表 ArrayList
1.添加删除元素时，自动调节数组容量。不够存了，拷到一个更大的数组。
2.一个采用类型参数(type parameter)的泛型类（generics）
```
ArrayList <String> arr = new ArrayList <>(); //还可以传初始大小
```
3.Vector类（子类有Stack）线程安全，慢；LinkedList使用双向链表（增删快）。
4.确认不修改了，可以用trimToSize减少内存占用
5.访问有get，set，foreach，for(),add
6.比较好的做法是 new ArrayList ; add; toArray();
7.未确定类型的ArrayList，虽然有警告，但是只要确定安全，用@Suppress Warning("unchecked")即可

###四、对象包装器
1.每个基本类型都有包装器
int 对应 Integer， double 对应 Double
2.这些包装器是不可变的，一旦构造，不允许修改其中的值
3.final的，不能定义子类
4.使用equals判断相等
5.用到的时候自动装箱拆箱
```
ArrayList <Integer> list = new ArrayList <>();
list.add(3); //在编译时自动变成  list.add(Integer.valueOf(3));
```
6.API: toString, ParseInt, valueOf

###五、参数数量可变的方法
printf(String fmt, Object ... args),用省略号表示任意数量
可以用foreach来解析
public static void main(String ... args)

###六、枚举类
1.所有枚举类都是Enum的子类
2.toString的逆方法是静态方法valueOf

###七、反射 Reflection (java.lang.reflect)
> ?是不是类似python的元编程

1.编写能够动态操纵Java代码的程序
- 在运行时分析类的能力
- 在运行时查看对象，比如编写一个toString方法供所有类使用
- 实现通用的数组操作代码
- 利用Method对象，这个对象很像C++中的函数指针。

2.Class类
(1)所有对象有一个运行时的类型标识，Class类保存了这些信息
(2)获得Class对象的方法
方法一：getClass()可以返回对象的Class对象。
Class c1 = e.getClass()
方法二：Class.forName()可以传入字符串格式的包名来构造Class对象
//可以用于手工加载类，给用户启动快的幻觉。
//只能传入类名或者接口名，否则Checked Exception,故应该进行异常处理
方法三：用.class，比如Random.class 
//不是对象也可以，比如int.class
//JVM为每个类型管理一个对象，可以用==判断。比如 e.getClass() == Employee.class
(3)使用
- 获得类名：e.getClass().getName()
- 动态生成实例： Object m = Class.forName("java.util.Random").newInstance() //若需要带参实例化，则要用Constructor类中的newInstance方法

3.异常处理
- 未检查异常：比如null引用
- 已检查异常：编译器检查是否提供了处理器（handler）

应该精心编写代码来避免错误的发生，而不要花精力在异常处理器上
并不是所有错误都是可以避免的，编译器要求提供处理器
try{;}catch(Exception e){e.printStackTrace();} //Throwable是Exception的超类

4.利用反射分析类的能力
(1)检查类的结构
java.lang.reflect 中有Field（域），Method（方法），Constructor（构造器）三个类
这些类有getName(),Field有getType。Method有报告返回类型的方法，getModefiers返回一个整数值，用不同的位开关描述public和static。Class类有getFields（返回一个数组），getMethods,getConstrators(包括超类公有成员)
还有getDeclareFields...,不包括超类成员
使用Modifier.toString(c1.getModifiers());

5.在运行时使用反射分析对象
Class c1 = harry.getClass();
Field f = c1.getDeclaredField("name");
f.setAccessible(true); //获取权限
Object v = f.get(harry); //也有getDouble，也有f.set(obj,value)
也可以用来编写通用的打印方法 sout.println(new ObjectAnalyzer().toString(squares));

6.反射编写泛型数组
Object newArray = Array.newInstance(componentType, newLength);

7.调用任意方法（函数指针）
存在多个同名方法，需要指定参数
```
Method m1 = Employee.class.getMethod("getName");
Method m2 = Employee.class.getMethod("raiseSalary",double.class)
String n = (String) m1.invoke(harry);
double s = (Double) m2.invoke(harry);

```
- invoke的参数和返回类型都是Object
- 方法指针比接口和lambda都慢，建议仅在必要的时候使用Method对象

###八、继承的设计技巧
1.将公共操作和域放在超类
2.不要使用受保护的域
3.使用继承实现"is-a"关系
4.除非所有的继承方法都有意义，否则不要使用继承
5.在覆盖方法时，不要改变预期的行为
6.使用多态，而非类型信息
7.不要过多的使用反射（反射是脆弱的）

#C6.接口、lambda表达式、内部类、代理
###一、接口
(1)
1.接口不是类，而是对类的一组需求描述。这些类要遵从接口描述的统一格式进行定义。
2.如果类遵从某个接口，那么就履行这项服务。
Arrays.sort排序前提是所属的类必须实现Comparable接口
public interface Comparable {int compareTo(Object other);}
//任何实现Comparable接口的类都需要保护compareTo方法
3.接口中所有方法自动是public，实现的时候必须为public
4.接口中可以定义常量(public static final)。不能有实例域或具体方法。
5.将类实现一个接口:
声明class E implements Comparable
实现接口的所有方法。（不然实例化后不好调用）。实现时必须为public，不然默认是包可见，可能出错。
6.接口不能实例化，但可以指向实现了该接口的类对象。
```
Map <String,Integer> map = new HashMap <>();
```
7.instanceof 可以检查对象算法是否实现了该接口
8.接口可以扩展 public interface Powered extends Moveable{;}
9.每个类只能一个超类，但可以多个接口。使行为灵活，逗号隔开。
(2)接口与抽象类
1.抽象类有个问题，每个类只能扩展于一个类。
2.接口可以提供多重继承的大多数好处，同时避免多重继承的复杂性和低效性。
(3)静态方法
Java9以后可以让接口有静态方法。但一般的做法是放在伴随类中。比如Path/Paths,Collection/Collections
(4)默认方法，default修饰
1.可以设空，节省一部分实现
2.利于"接口演化"，兼容老版本
(5)解决默认方法冲突
1.超类优先
2.若接口冲突，必须覆盖这个方法
class Student implements Person,Named {public String getName(){return Person.super.getName();}}
> 千万不要让一个默认方法重新定义Object类中的方法。

###二、接口示例
1.接口与回调(Call Back)
//回调模式：在某个特定事件发生时应采取的动作
```
public interface ActionListener ;
class TimePrinter implements ActionListener;
ActionListener listener = new TimePrinter();
Timer t = new Timer(1000,listener);
```
2.Comparator接口
```
public interface Comparator <T> {int compare (T first, T second);}
class LengthComparator implements Comparator <String> {;}
Comparator <String> comp = new LengthComparator();
if (comp.compare(words[i],words[j])>0)...
```
3.对象克隆：Cloneable接口
若原对象和浅克隆对象共享的子对象是不可变的，那么共享是安全的。不过通常是可变的，因此需要clone。

###三、Lambda表达式：使用简洁的语法定义代码块。
1.lambda表达式是一个可传递的代码块，可以在以后执行一次或多次。
2.示例
```java
(String a, String b) -> a.length()-b.length();
(String a, String b) -> {if (a.length() 小于 b.length()) return -1; else return 0;} //java支持多行，python只能一行
() -> {sout(0);} //无参要写括号
```
- 类型推导时可不写类型，仅一个参数时可以不加括号
- 返回类型自动推断，但是要确保所有分支返回类型一致

3.函数式接口：lambda需要函数式接口来接收它
**任何一个仅含有一个抽象方法的接口都是函数式接口。**
Arrays.sort(words, (a,b) -> a.length()-b.length());
//一个Comparator接口，lambda可以装换为接口
//不能把lambda表达式赋给Object变量，Object不是一个函数式接口
//java.util.function 中有一些，比较有用的有Predicate。list.removeIf(e->e==null);

4.方法引用:使用现有的方法传入
Timer t = new Timer(1000,System.out::println) //相当于x->System.out.println(x);

5.构造器引用
```
ArrayList <String> name=...;
Stream <Person> stream = names.stream().map(Person:new) ;
List <Person> people = stream.collect(Collectors.toList()) ;
```

6.变量作用域
lambda 表达式中捕获的变量必须实际上是最终变量。
在外部或内部修改变量是危险的。

7.处理lambda表达式
常用函数式接口：Runable,Supplier,Consumer,BiConsumer,Function,BiFunction,Unary Operator, Bianry Operator.
public static void repeat(int n, IntConsumerFunction){
for (int i=0; i小于n; i++)action.accept(i) repeat(10,i-> System.out.println("Count down:"+(9-i)));}
Arrays.sort(people,Comparator.comparing(Person::getLastName) thenComparing(Person::getFirstName));

###四、内部类
public class TalkingClock {public class Timeprinter implements ActionListener{};}
1.使用原因：
(1)内部类方法可以访问该类定义所在的作用域的数据，包括私有数据
(2)内部类对包内其他类隐藏
(3)想要定义一个回调函数且不想编写大量代码时，使用匿名内部类比较便捷
2.使用内部类访问对象状态
内部类总有一个引用，指向创建它的外部类对象(outer或者OuterClass.this)
内部类的静态域必须是final的，我们希望一个静态域只有一个实例。内部类不能static。

3.内部类是否有用、必要和安全
(1)没必要
(2)内部类是一种编译器现象，与虚拟机无关，编译器会把内部类翻译成$
(3)有风险：若内部类访问了私有域，就有可能通过附加在外部类所在包中的其他类访问他们。

4.局部内部类
封在函数里，不能为public或private，对外部世界完全隐藏。//只能用final变量

5.匿名内部类
只创一个对象 new InterfaceType() {data and methods}
双括号初始化：invite(new ArrayList <String> () {{add("1");add("2");}});

6.静态内部类
有时候，使用内部类只是为了把一个类隐藏在另一个类内部，并不需要内部类引用外部类对象。可以声明为static。

###五、代理（Proxy）
利用代理可以在运行时实现了一组给定接口的新类。
这种功能只有在编译时无法确定需要实现哪个接口时才有必要使用。
1.何时使用代理:
有一个表示接口的Class对象，确切类型在编译时无法知道。newInstance无法实例化一个接口，需要在程序运行时定义一个新类。代理类可以在运行时创建新类。
(1)路由对远程服务器方法调用
(2)在程序运行期间，将用户接口事件与动作关联起来
(3)为调试，跟踪方法调用。
2.创建代理对象
需要使用Proxy类的newProxyInstance方法，三个参数：
(1)一个类加载器，用null是默认的
(2)一个Class对象数组，每个元素都是需要实现的接口
(3)一个调用处理器
3.代理类的特性
(1)程序运行中创建，一旦创建，变常规类
(2)所有代理类扩展于Proxy类，一个代理类只有一个实例域，调用处理器(附加数据都在这里)。
(3)所有代理类都覆盖了Object类中的方法toString，equals，hashCode
(4)对于特定的类加载器的预设的接口来说，只能一个代理类
(5)代理类一定是final额public的

#C7.异常、断言和日志（三种错误处理）
###一、处理错误
1.出错时：
(1)向用户通告错误
(2)返回到一种安全状态，并能够让用户执行一些其他命令，或者
(3)允许用户保存所有操作的结果，并以妥善的方式终止程序
2.异常处理的任务就是将控制权从错误产生的地方转移到能够处理这种情况的错误处理器
- 用户输入错误
- 设备错误
- 物理限制
- 代码错误

3.异常分类(派生于Throwable)
Throwable:Error（系统内部错误资源耗尽，无能为力）,Exception（RuntimeException，其他异常）
Error(错误的类型访问，数组访问越界，访问null指针)
Exception(试图在文件尾部后面读取数据，试图打开一个不存在的文件，试图根据给定的字符串查找Class对象)
> 如果是RuntimeException，一定是你的问题。

4.声明受查异常
public Image loadImage (String s) throws FileNotFoundException, EOFException
(1)调用一个抛出异常的方法
(2)程序运行过程中发现错误，并且利用throw语句抛出一个受查异常
(3)程序出现错误
(4)Java虚拟机和运行时库出现的错误
> 若在子类中覆盖了超类的一个方法，子类方法中声明的受查异常不能比超类方法更通用。

5.如何抛出异常
(1)找到一个合适的异常类
(2)创建这个类的一个对象
(3)将对象抛出 
String readData (Scanner in) throws EOFException{;throw new EOFException();}

6.创建异常类
从Exception类中派生或者从其子类派生

###二、捕获异常
(1)若不捕获，程序终止，在控制台打印异常类型和堆栈内容
(2)try跳过其他代码，执行catch中的处理器代码
(3)捕获多个异常，用多个catch
(4)finally{in.close();} //确保文件关闭
(5)try-catch要多做解耦，不要全部扔一堆，宁愿多写几个try-catch
(6)finally有return时，try中的return会被finally的return覆盖，因finally一定执行
(7)finally对in.close仍然可能IOException，重新报错
(8)带资源的try解决上述问题，自动close
try (Resource res = ...) {;} //python中有with环境
(9)堆栈迹元素
t.printStackTrace();
t.getStackTrace();
###三、使用异常的技巧
1.异常处理不能代替简单的测试（也很费性能）：只在异常情况下使用异常机制。（应尽量写出无异常的代码）
2.不要过分细化异常
3.利用异常的层次结构
4.不要压制异常
5.在检测错误时，"苛刻"要比放任好。（早抛出）
6.不要羞于传递异常。（晚捕获）

###四、使用断言
1.assert条件
2.assert条件：表达式
3.启用和关闭断言：对类加载器而言：默认为禁用
java -enableassertions MyApp
java -da MyApp
java -ea:MyClass -ea:com.my...MyApp
4.什么时候使用断言（完成参数检查，为文档假设使用断言）
(1)致命的，不可恢复错误
(2)只用于开发测试阶段

###五、记录日志
1.日志方式
(1)System.out.println()
(2)Logger.getGlobal.info()
(3)全局日志
private static final Logger ; mylogger = Logger.getLogger("com.me");
2.七个级别
SEVERE、WARNING、INFO //常用的三个级别
3.修改日志，管理配置
jre/lib/logging/.properties
4.日志本地化（全球读得懂）
5.处理器
6.过滤器（按日志级别过滤）
7.格式化器（日志格式化）
8.日志记录说明

###六、调试技巧
1.打印
(1)sout.println()
(2)Logger.getGlobal.info()
2.每个类中放单独的main
3.JUnit单元测试
4.日志代理：截获日志，记录、调用超类方法
5.利用Throwable的printStackTrace
6.System.error重定向
7.-verbose参数观察类加载情况
8.-xlint告诉编译器检查普遍错误
9.图形工具监控
10.jmap工具
11.-Xprof标志剖析

#C8.泛型程序设计
###一、为什么要使用泛型程序设计（Generic Programming）
1.类型参数的好处
泛型之前，是用继承Object实现的。有两个问题，第一，获取一个值时，必须进行强制类型装换；第二，添加时没有类型检查。
解决：类型参数(type parameters):
ArrayList <String> files = new ArrayList <String> ();
可读性和安全性都更好了。
###二、定义简单泛型类
public class Pair <T,U> {...}
泛型类可看作普通类的工厂。 //和C++模板类有本质区别
###三、泛型方法
public static <T> T getMiddle(T...a);
###四、类型变量的确定
public static <T extends Comparable> T min (T [] a)多个限定
T extends Comparable & Serializable
###五、泛型代码与虚拟机
虚拟机没有泛型类型对象，所有对象都是普通类
(1)类型擦除
变成限定的类型，不限定是Object
Pair <String> 和 Pair <LocalDate>是一样的
(2)翻译泛型表达式
编译器插入类型转换
(3)桥方法被合成保持多态（bridge method）
(4)为保持安全性，必要时插入强制类型转换
//注解：@Suppress Warnings("unchecked")来关闭警告

###六、约束与局限性（因为类型擦除）
1.不能用基本类型实例化类型参数 
ArrayList <int> 要改成 ArrayList<Integer>,因为int无法擦除成Obeject
2.运行时类型查询是原始类型（擦除后的类型）
3.不能new参数化类型的数组，除非强制转换，不安全
```
Pair <String> [] table = (Pair <String> []) new Pair < ? > [10];
```
4.Varargs警告
向参数可变的方法传递一个泛型类型的实例。消除警告用@SafeVarargs
5.不能实例化类型变量
比如 new T(); //因为会被擦除
比如 T.class
6.不能构造泛型数组
7.泛型类的静态上下文中类型变量无效
8.不能抛出或捕获泛型类的实例
甚至泛型类扩展Throwable都是不合法的
9.可以消除对受查异常的检查
10.注意擦除后的冲突
比如 boolean equals (String) 和 boolean equals(Object)

###七、泛型类型的继承规则
ArrayList <T> 类实现了 List <T> 接口
一个ArrayList <Manager> 可以转换为List <Manager>
但是无法转换为ArrayList <Employee> 或 List<Employee>

###八、通配符类型
1.通配符概念
通配符类型中，允许类型参数变化 Pair < ? extends Employee >
2.超类类型限定 
Pair < ? super Manager >
带有超类型限定的通配符可以向泛型对象写入，带有子类型限定的通配符可以从泛型对象读取。
3.无限定通配符 
Pair < ? > //getFirst只能给Object，setFirst则不能调用。是很脆弱，但是可以实现简单的操作。
4.通配符捕获
只有在有许多限制的情况下是合法的，编译器必须能够确定通配符表达的是单个、确定的类型。

###九、泛型和反射
1.泛型Class类 
Class<T>
2.使用Class<T>进行类型匹配
3.虚拟机中的泛型类型信息

#C9.集合(容器)
1.接口与实现分离
（LILO）Queue (add,remove,size)，（容量有限）可用循环数组或者链表实现。
2.Collection接口(add, Iterator(next, hasNext, remove, forEachRemaning))
如何访问
(1)while (iter.hasNext()) {iter.next();}
(2)for (String e: c) {;}
(3)iter.forEachRemaning (lambda)
//Iterator.next 与 InputStream.read 类似，不断消耗
//python中用yield做生成器
3.泛型实用方法
contains(python中有in)，size，isEmpty，containsAll，equals，addAll，remove，removeAll，clear，retainAll，toArray，
toArray(T []), removeIf()
4.集合中有两个基本接口 Collection(add (List,Set,Queue)) 和 Map(put,get)
5.具体的集合
- ArrayList （变长，索引，数组）
- LinkedList (删除插入快)
- ArrayQueue （循环数组双端队列）
- HashSet     无序集（无重复元素）
- TreeSet     有序集
- EnumSet     枚举集
- LinkedHashSet 有序的
- Priority Queue  优先队列（最小堆）
- HashMap 无序Key-Value,hash,拉链法
- TreeMap 有序Key-Value，红黑树
- EnumMap 枚举映射
- LinkedHashMap  有序的
- WeakHashMap 有利用垃圾回收的
- IdentityHashMap  用 == 而不是equals比较键值（即使内容相同也被视为不同）

6.链表（LinkedList）
(1)数组删除插入元素要挪动后半块
(2)Java中都是双向链表，删除插入快
(3)链表是有序的
(4)add方法取决迭代器的位置，remove取决于迭代器状态。
> 边遍历边修改是危险的。一般从后向前按索引删除是安全的。
> 很多只读，一个可读可写迭代器可以安全。
(5)链表的唯一优点是插入删除效率，若只有少数几个元素，用ArrayList。

7.散列集（HashSet）（哈希，拉链法）
桶数设为预计元素的75%-150%。最好设为素数，以防键的聚集（不平衡）。装填因子默认为0.75，超过时扩大一倍桶数，再重新散列。
8.树集TreeSet。 有序的集合。
9.队列与双端队列。（高效头尾操作，不支持中间插入）
10.优先队列（用最小堆） //任务调度，优先级
11.映射：无序迭代HashMap快，有序选TreeMap
Map <String,Integer> = new HashMap <> ();
(1)更新映射项，可以用put、merge
(2)映射试图，keySet，values、entrySet
//类似python字典的keys，values，items方法
12.视图与包装器
(1)List <String> names = Arrays.asList("Amy","Bob","Cart");
List <String> settings = Collections.nCopies(100,"Default");
(2)子范围 staff.subList(10,20)
(3)不可修改视图
(4)同步视图（线程安全）
Map <String,Employee> map = Collections.sychronized Map (new HashMap <> ());
13.受查视图 Collections.checkedList

###五、算法
Collections.sort(staff)
staff.sort(Comparator.reverseOrder())
使用归并（稳定）可高效排序，Java不这么做，它先转数组，排好后再转回来。
Collections.shuffle(cards);
Collections.binarySearch(),replaceAll,reverse
staff.toArray()
staff.toArray(new String []);
> 编写自己的算法要尽可能用接口。

###六、遗留的集合
1.Vector（子类有Stack(pop,push)）和HashTable是同步的（线程安全），但是慢。
2.若需要并发访问，用ConcurrentHashMap；
3.BitSet是位序列，可以方便操作各个位，比如Eratosthenes筛子法找素数。

#C13.部署JAVA程序
###一、JAR文件
1.一个jar文件既可以包含.class文件，也可以包含其他如图像、音频文件。是ZIP(pack200算法，90%压缩率)压缩的。
(1)创建jar文件
jar cvf outputfilename File1 File2
// -m 参数设置清单文件
(2)清单文件（可用默认的）
/META-INF/MANIFEST.MF
//最简单的内容   Manifest-Version:1.0
(3)可执行文件
- jar cvfe x.jar com.y.MyAppClass files to add
- 或在清单中指定主类 Main-Class：Myappclass //不要带.class
- 运行 java -jar Myjar.jar

(4)资源
AboutPanel.class
getResource/getImage/getAudioClip/getResourceAsStream
重点在于类加载器可以记住如何定位类，然后在同一个位置查找关联的资源
(5)密封
为了保证不会有其他的类加入其中
清单文件添加 Sealed:true

###二、应用首选项存储(Config)
1.有Properties类，getProperty和setProperty方法，以文件a.conf 存储
> python中有ConfigParser
2.中心存储库(类似注册表)
Preference类，可以导出为XML
> 考虑JSON ？

###三、服务加载器
利用ServiceLoader类可以加载一个公共接口的插件

#C14.并发
###一、线程（thread）和进程（process）的本质区别
1.每个进程拥有自己的一整套变量（PCB程序，数据），而线程共享数据
2.线程间通信更有效、容易。
3.有些操作系统中线程更"轻量级"，创建、撤销一个线程比启动新进程的开销小。
###二、使用线程给其他任务提供机会
1.方案一
(1)实现Runable接口 
public interface Runable {void run();}
// 因为是函数式接口（仅一个函数），可以用lambda
Runable r = ()->{task;}
(2)新建一个线程
Tread t = new Thread(r);
(3)启动线程
t.start(); //不要直接调用run，而用start
2.方案二（不推荐）
(1)派生Thread
class MyThread extends Thread {public void run() {task;}}
(2)实例化，start
3.方案三：线程池（若有很多任务）
###三、中断线程
1.return或者没有捕获异常时，线程终止。（以前有stop，现在没了）
2.没有可以强制终止线程的方法。而interrupt方法可以用来请求终止线程（对中断状态boolean置位）
while (!Thread.currentThread).isInterrupted() && more) {do more work;}
- 但是，若线程被阻塞，就无法检测中断状态，产生Interrupted Exception
- 被中断的线程可决定如何响应中断：某些线程会先处理完异常，先不理会中断 try-catch
- 不要直接忽视 Interrupted Exception 至少要设置一下中断状态或者抛出

###四、六种线程状态
1.NEW（新创建）
2.Runable（可运行，start）（可能正在运行也可能没运行，取决于OS）
> PC、服务器一般是按优先级抢占式调度；手机，协作式调度

3.Blocked（被阻塞）（为了内部对象锁被其他线程拥有）
4.Waiting（等待）（等待调度器的条件）
5.TimedWaiting（计时等待）（超时参数）
6.Terminated（被终止，正常return或者异常时被终止）

###五、线程属性
1.线程优先级
(1)setPriority方法//有MIN_PRIORITY值为0，MAX_PRIORITY值为10，默认是NORM_PRIORITY值为5
当线程调度器有机会选择新进程时，首先选高优先级（低优先级的线程可能永远得不到执行）
(2)高度依赖操作系统。不要将程序构建的功能的正确性依赖于优先级。
- 映射的Windows的7个优先级
- 而Oracle为Linux提供的JVM没有优先级
2.守护进程（deamon thread）
t.setDaemon(true);
- 守护线程的唯一用途是为其他线程提供服务（持续运行，只剩下守护线程时就没必要继续运行程序了）
- 守护线程应该永远不去访问固有资源，如文件、数据库，因为它会在任何时候中断

3.未捕获的异常处理器
run方法不能抛出异常，直接线程死了。可以用setUncaughtExceptionHandler为任何线程安装一个处理器，默认处理器为空。
ThreadGroup（线程组）可以实现ThreadUncaughtException接口，优先用父线程组的UncaughtException方法。

###六、同步（线程安全）
1.竞争条件（race condition）
不是原子操作，这个线程做了一半操作（加载到寄存器，增加，未写出），另一个线程抢了控制权（加载到寄存器...），导致线程不安全。
2.锁对象
(1)synchronized 关键词表同步
(2)使用ReetrantLock
```
Lock mylock = new ReetrantLock();
mylock.lock()
try{critical section;} finally {mylock.unlock();}
```
- 保证finally时释放
- try不能是带资源的try（因为解锁方法不是close）
- 当锁住时需要这个锁的线程就被阻塞了
- 可重入锁：有一个持有计数（hold count），被一个锁保护的代码可以调用另一个相同锁的方法，计数为0时释放
- ReetrantLock（boolean fair），公平锁偏爱等待时间最长的线程，也无法确保公平

3.条件对象
账户中没有余额了，等待另一个线程向账户中注入资金。但是，刚刚获得了对bankLock的排他性访问，因此别的线程没有进行存款的机会，因此需要条件对象。（可以有多个条件对象）
sufficientFunds = bankLock.newCondition();
sufficientFunds.await(); //当前线程被阻塞，并放弃锁，直到收到signalAll()来重新激活继续竞争
还有个signal()是释放给随机线程，也容易死锁（deadlock）
当一个线程调用了await，它没有办法激活自身，寄希望于其他线程的signal或者signalAll，如果其他线程不发送signalAll或者signal，便无法再运行了，这便是死锁（deadlock）。故一定要记得发送激活信号。

4.synchronized 关键字
(1)Lock和Condition适合高度定制的场景，但是大多数时候是不需要的。
- 锁用来保护代码片段，任何时候只能有一个线程执行被保护的代码
- 锁可以管理试图进入被保护代码段的线程
- 锁可以拥有一个或多个相关的条件对象
- 每个条件对象管理那些已经被保护代码段还不能运行的线程

(2)内部对象锁 synchronized
```
public synchronized void method() {body;}
等价于
public void method(){
this.intrisicLock.lock();
try{body;}
finally{this.intrisicLock.unlock();}}
```
- 内部对象锁只有一个相关条件（intrisic condition）
- 内部对象锁的wait，nofifyAll，notify方法相当于await，signalAll，signal
- 静态方法同步也可以用它

//intrisic condition
public synchronized vodi transfer(){
while (accounts[from] 小于 amount) wait();
notifyAll();
}

(3)内部锁的局限
- 不能中断一个正在获得锁的线程
- 试图获得锁时不能设置超时
- 每个锁仅有单一条件，可能是不够的

(4)如何使用：
优先使用java.util.concurrent,再synchronized，再Lock/Condition

5.同步阻塞
(1)每个java对象有一个锁。线程可以通过调用同步方法获得锁，还有另一种机制可以获得锁，通过进入同步阻塞。
synchronized (obj) {critical section;}  //锁住了obj
(2)使用一个对象的锁来实现额外的原子操作，实际是客户端锁定（clientside locking）
```
public void transfer (Vector <Double> accounts) 
{synchronized(accounts) {accounts.set(...);accounts.get(....);}}
```
//完全依赖于Vector类对自己的可修改方法使用了内部锁，这样做非常脆弱，不推荐使用。

6.监视器
(1)监视器
- 监视器是只包含私有域的类
- 每个监视器类的对象有一个相关的锁
- 使用该锁对所有的方法进行加锁
- 该锁可以有任意多个相关条件

(2)Java中的每一个对象有一个内部锁和内部条件，若一个方法用synchronized声明，表现像监视器方法，通过调用wait/notifyAll/nofify 来访问条件变量
Java对象在3个方面与监视器不同，使安全性下降
- 域不要求必须private
- 方法不要求必须synchronized
- 内部锁对客户是可用的

7.volatile域
用volatile标记的变量修改时，会将其他缓存中存储变量清除，然后重新读取(刷新)。是一种免锁机制。
- volatile本质是告诉JVM当前变量不确定（重新读取不锁定），synchronized则是锁定变量，其他线程被阻塞
- volatile仅保证变量修改可见性，synchronized保证可见性原子性

8.原子性：一个事务的各个操作要么一起成功，要么一起不执行。
java.util.atomic 包里有AtomicInteger(incrementAndGet,decrementAndGet) 和 LongAdder等类可以保证多线程原子操作

9.死锁
> A:200,B:300;A-B,300,B->A,400.

- 账户A和账户B的余额都不足以转账，两个线程都无法执行下去。（导致所有进程进入阻塞）
- 所有线程集中在一个账户上，都试图从账户上取钱。
- 把signalAll变成signal，容易在一个 线程上死掉了

>Java中没有任何东西可以打破死锁现象，必须仔细设计程序。

10.线程局部变量
ThreadLocal类为各个线程提供单独的实例

11.锁测试与超时
tryLock去试，失败了转去做别的事
```
if (myLock.tryLock()){try{...}finally{mylock.unlock;}} else {;} //也可以加超时参数tryLock（100，TimeUnit.MILLISECONDS）
```

12.读写锁
private ReetrantReadWriteLock rwl = new ReetrantReadWriteLock();
private Lock readLock = rwl.readLock();
private Lock writeLock = rwl.writeLock();
//对所有的读取方法加读锁，对所有的修改方法加写锁

13.为何弃用stop和suspend方法
stop; 对象会被破坏，不安全
suspend; 导致死锁，被挂起的线程等着被恢复，而将其挂起的线程等待获得锁

14.阻塞队列
- ArrayBlockingQueue
- PriorityBlockingQueue
- TranserQueue

15.高效的映射、集和队列
ConcurrentHashMap (不允许有null值)
ConcurrentSkipListMap
ConcurrentSkipListSet
ConcurrentLinkedQueues
//批操作 map.forEach()

16.并行数组算法
Arrays.parrallelSort(words)
Arrays.parrallelSetAll()

17.同步包装器
Collections.synchronized()

18.Callable 和 Future

19.执行器 Executor
创建线程池  Future <T> submit (Callable <T> task)
- 预定执行
- 控制任务组
- For-Join框架
- 可完成Future

20.同步器
- 信号量
- 倒计时门栓
- 障栅
- 变换器
- 同步队列

21.线程与SWING
SWING不是线程安全的
```