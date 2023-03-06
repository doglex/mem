---
date: 2017-08-01 23:35
status: public
title: 【图形】NP-POLY算法解读
---

>  问题：2D平面上如何判断一个点是否落在某个多边形内？

## 一、判断点是否在凸多边形内

下列方法的缺点：计算复杂，且只适用于凸多边形

#### 1.面积法

点与各个边分别组成三角形，面积求和是否等于凸多边形面积。

若等于凸多边形面积，在其中。

在计算时，一旦S1+S2+。。。大于凸多边形面积，则在其外。

### 2.角度判别法

点若在其中，与各个边的夹角之和为360度。

在计算时，一旦大于360，则在其外。

### 3.叉乘法

如图，若采用笛卡尔方向（右手定则），按顺时针。若叉乘反向，在凸多边形外，若全部同向，则在凸多边形内。
![](~/1_cross.jpeg)

### 二、判断点是否在一般的多边形内：射线法

### 1.原理（根据多篇博客归纳）
任作一条过这个点且与多边形相交的**射线**，若点在多边形内，则两侧的交点（由该射线的法线划分）各为奇数，否则不能满足。反过来，**一侧的交点为奇数可以判断点在多边形内**。

**不妨取该射线为y=y0**。（原因：若点在多边形内，根据这条**地平线**必能找到相交，即y_min<=y<=y_max。当比较一条边与该射线是否有交点时，**仅需比较这条边两个端点的y值与这条地平线的y的关系**。）

### 2.例子

1.允许空洞。从图中可以看出这个原理对复杂多边形是适用的。

(case1) 在与不在多边形内。

(case2)多边形存在多边空洞。



![](~/2_case12.jpg)
2.去重。

(case3)左侧交点有两个（重叠），一个在AB上，一个在AC上，需要消去一个。

(case4)左侧与三条边有交点，只接受其中一个交点。


![](~/3_case34.jpg)


3.小结

对于多边形的边而言，用两个端点的y值与水平射线的y值比较，**接受两种交点：(1)一等一小(2)一大一小。**

注：一等一小的意思是指一个端点低于水平射线，一个端点在水平射线上。一大一小同理。

4.其他细节

若这个点恰好落在多边形的边上，结果是不确定的，可能判断为多边形内也可能判断为多边形外。

5.留给读者的开放问题：

(1)如果取竖直射线或者其他方向会怎么样？是否能更方便计算？是否对于一些特殊的多边形需要特殊考虑射线的情况？

### 3.程序实现

1.第一种实现Python

```python
def is_in_poly(x, y, poly_xs, poly_ys):
    """
    whether one point is inside a 2-D poly
    :param x: point_x
    :param y: point_y
    :param poly_xs: xs of poly
    :param poly_ys: ys of poly
    :return: true or false
    """
    odd_cross = False
    for i in range(len(poly_xs)):
        j = i - 1 if i > 0 else len(poly_xs) - 1
        if poly_ys[i] < y <= poly_ys[j] or poly_ys[j] < y <= poly_ys[i]:
            if poly_xs[i] + (y - poly_ys[i]) * (poly_xs[j] - poly_xs[i]) / (poly_ys[j] - poly_ys[i]) < x:  # wont divide 0
                odd_cross = False if odd_cross else True
    return odd_cross


def demo():
    """
    test functional
    """
    assert is_in_poly(1, 0.5, (0, 2, 4, 2), (0, 2, 0, -2)) is True
    assert is_in_poly(1, 0.9, (0, 2, 4, 2), (0, 2, 0, -2)) is True
    assert is_in_poly(1, 2, (0, 2, 4, 2), (0, 2, 0, -2)) is False
    assert is_in_poly(1, -2, (0, 2, 4, 2), (0, 2, 0, -2)) is False
    print("it works")


demo()
```

代码提示：

（1）只需要更改交点的奇偶性，二状态切换，最后输出奇状态。注意要遍历所有边一次。

（2）只需要考虑左侧的交点(<x)

(3)  仅在满足一等一小或者一大一小的条件下求解交点。而求解交点的目的只是为了知道是在左侧还是右侧。

（4）求解边与地平线的交点是简单的，考虑两瓶不同浓度的液体混合，那么浓度介于其中，如下图给出解释


![](~/4_hw.jpg)

（5）除零问题不存在是由外层IF语句保证了的，或者所，『两等』本事不符合交点条件。

2.第二种实现Python

```python
def is_in_poly(x, y, poly_xs, poly_ys):
    """
    whether one point is inside a 2-D poly
    :param x: point_x
    :param y: point_y
    :param poly_xs: xs of poly
    :param poly_ys: ys of poly
    :return: true or false
    """
    odd_cross = False
    for i in range(len(poly_xs)):
        j = i - 1 if i > 0 else len(poly_xs) - 1
        odd_cross ^= (poly_ys[i] < y <= poly_ys[j] or poly_ys[j] < y <= poly_ys[i]) \
                and (poly_xs[i] + (y - poly_ys[i]) * (poly_xs[j] - poly_xs[i]) / (poly_ys[j] - poly_ys[i]) < x)
    return odd_cross
```

代码提示：思路和第一种一模一样，只是利用异或(XOR)的状态交换性质。（用-1乘以-1也可以做到，当然也可以使用累加器来做到）

3.考虑半边加速

```python
def is_in_poly(x, y, poly_xs, poly_ys):
    """
    whether one point is inside a 2-D poly
    :param x: point_x
    :param y: point_y
    :param poly_xs: xs of poly
    :param poly_ys: ys of poly
    :return: true or false
    """
    odd_cross = False
    for i in range(len(poly_xs)):
        j = i - 1 if i > 0 else len(poly_xs) - 1
        if (poly_ys[i] < y <= poly_ys[j] or poly_ys[j] < y <= poly_ys[i])\
                and (poly_xs[i] < x or poly_xs[j] < x):  
            if poly_xs[i] + (y - poly_ys[i]) * (poly_xs[j] - poly_xs[i]) / (poly_ys[j] - poly_ys[i]) < x:
                odd_cross = False if odd_cross else True
    return odd_cross
```

代码提示：如果确定是落在右半边了的那也不必去求求解交点了。
**如果确定交点是在左侧了，也不必去求解了。实现是简单的，留给读者吧。但是这个条件的引入是否能配合上lazy evaluation，考虑引入新语句减小的开销和带来的新开销请读者自己根据特定的场景权衡。**

4.其他版本

考虑x,y的组织形式，将同一个点的x，y放在同一个变量中或者比x存一个向量y存一个向量的依赖索引定位的形式更鲁棒。python中可以用元组或者namedtuple，java中也很方便构造Pair类型。

### 三、射线法的优化

### 1.性能分析

1.若有N个顶点，那么首尾相接可以得到N条边。需要遍历一遍N条边并且修改奇状态（或计数器）。

2.若将『交点判断』和『求解交点』的过程视为常数（实际上不是），那么复杂度是线性的O(N),即随着N的增多需要的时间呈线性增长。事实上不是，有一些边可以通过不同程度的lazy evalution跳过，而有一些边则必须计算交点位置，这个取决与多形性的形状以及这个点所处的位置。

### 2.分布式实现

这个算法是可以并行的。把这个点分发到各个task中，同时每个task分发一些边，每个task计算奇状态或者累加器，最后将所有task的结果进行异或或者求和。最后要验证一下确实每条边有且仅被计算了一次，可能需要额外的计数器。思路是简单的，值不值得调度上的开销取决于问题的规模。

### 3.外部优化

1.使用一个外接矩形。利用『**敏捷失败**』的思想。

外接矩形的计算是简单的，遍历这些多边形顶点取得x_min,x_max,y_min,y_max。这个外接矩形恰好平行于横纵坐标。

若落在矩形外，那么一定不在多边形内（执行了敏捷失败）。若落在矩形内，再次运行NP-Poly算法。那么相当于一个粗粒度查询+细粒度查询。（**布隆过滤器也是这么做的**）。

而这个矩形的访问往往能达到O(1)而且可以利用上Lazy Evaluation的判断条件。

这样，在大面积（概率）的点不落在矩形内时。在平均意义上变将O(N)平均到了O(1)。

注：敏捷失败的意思是如果程序注定失败，就让他立刻失败。同理，有敏捷成功。这样做的好处是，省下了大量额外的处理。在机器学习中，如果一个模型的参数注定失败，就立刻失败，去调其他参数，那么就是在有限的时间和空间资源下增加了尝试机会。当然，敏捷失败可能是需要一个阈值的。就像等车，有的人在等2分钟不来就打出租了，有的人要20分钟。

2.使用一个内接矩形或者多个内接矩形的组合。利用『**敏捷成功**』的思想。

这个取决于多边形的形状，同时假设落在多边形中的点的概率比较大。

也可以和外接矩形结合使用。

3.考虑概率问题，有一些hotspot，在这些hotspot的附近出现得概率非常大。那么考虑单独为这些hotspot建立矩形，敏捷成功。

### 4.洞的问题

落在区域中 = 不落在洞中 and 落在外框中。 

注：洞和外框都是多边形，只是调用两次算法。



> 参考：

> http://blog.csdn.net/hgl868/article/details/7947272

> http://blog.csdn.net/luyuncsd123/article/details/27528519

> http://blog.csdn.net/hjh2005/article/details/9246967