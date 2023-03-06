---
date: 2017-08-03 18:23
status: public
title: 【算法】哈希表、Bitmap与布隆过滤器
---

> 问题：资源定位、查询(search hit, search miss)、搜索问题，分类计数问题，过滤问题，去重问题，索引问题。

### A、其他方法

1.有序线性查找，使用**二分查找**，复杂度O(logN)。或者半线性问题，先用二分查找，再在其中进行进行线性搜索。二分查找也可以结合到插入排序。若未有序，考虑直接进行线性搜索或者先做个复杂度为对数的排序。

2.凸函数，可以使用**下降法**。有一阶下降法，或者牛顿下降法来加速。

3.特殊问题，比如开平方。可以用二分法，也可以用面积压平法,更新式 x = (x+S/x)/2

4.平衡树，比如**红黑树**(TreeMap)，非线性多叉层次，插入删除复杂度O(logN)。迭代key的时候可以是有序的。

5.**哈希表**(Java中HashMap有50%的空间浪费，Python中dict), M叉非线性，O(1)的数组(桶)查询+O(K)的单链表遍历。迭代key的时候在人类可读性上是无序的。若要线程安全考虑ConcurrentHashMap，Redis，人为造成的非原子操作还是得自己加锁。

### B、哈希表

#### 一、单机版Hash原理

1.什么是Hash：把键转换为数组索引。若没有内存限制，可以直接把键放到数组(基础类型，栈区)的索引，而**内存被设计为访问任意位置的时间都一样，是O(1)**的。两个问题：hash函数的设计，键碰撞解决。

> 个人理解，若冲突解决采用拉链法其实就是分桶。每个桶放相同索引的数据，利用桶的访问是O(1)的，但是就算对桶的访问不是O(1)的，那么也已经实现了**Modularization**（模块化，这就是深度神经网络变deep比变flat可以使用更少参数的原因），复杂度是要开平方的。
>
> 依然是分治法+冲突解决。当然也要考虑hash函数本身计算快速。
>
> 思考1：不妨把这些内存桶拓展为磁盘文件、数据库、机器节点。
>
> 思考2：那么不妨使用层次化的桶，大桶套小桶，这个可以多叉树来做。或者让一个数据被多个桶引用，或者所实现**OLAP**的多粒度查询。也可以直接使用嵌套的（nested）哈希表先进行粗粒度的查询+细粒度的查询。

2.问题一：设计**hash函数**

hash函数用来将键转换为索引（即找到桶的过程）。要求**计算便捷**，且让键尽量能**均匀散列**到索引上（各个桶上）。比如手机号的后三位比前三位要均匀得多。

(1)取模hash。

index = key % M

a.底层是做除法。(16位机器mov ax,8;mov bl,5;div bl; 那么ah里放余数，al里放商)

b.桶的数量M在数学上建议为素数会比较均匀。但是在Java的HashMap中实现为2的幂次(考虑效率，扩容的效率)，初始值为16(DEFAULT_INITIAL_CAPACITY = 1 << 4,修改时可以为capacity << 1, DEFAULT_LOAD_FACTOR = 0.75f，装填因子影响着rehash，就是扩容时重新hash，可能需要迁移很多数据)。

c.整数可以直接取模，字符串可以对各个位置取模相加，或者乘上31的幂次相加，或者跳跃着取字符。

(2)平方哈希法。

index = (value*value) >> 28; 

a.乘法比除法效率高，取乘法的中间部分可能比较均匀。

b.无须关心溢出问题。

(3)斐波那契法：乘上特殊的数。

index = (value *2654435769) >> 28;

注：数学证明从略。16位乘上40503，64位乘上11400714819323198485.



3.问题二：冲突解决

(1)拉链法：如果在这个索引上冲突了，那么就创建一个节点以单链表的形式压进去，访问的时候第一次访问索引（桶O(1)，粗粒度查询），第二次遍历这个索引上的单链表（O(K)，K为链长，细粒度查询）。

a.如下图是对模3哈希，使用3个桶，桶中各有一条链表。数据0先到达，压到了桶0的链尾。新到的数据**插入在链头**，这样**有两个好处，好处一是插入新数据时不必遍历链表去尾部而是直接插入，好处二是认为新来的数据更可能被查询（比如我们更多回忆昨天未完成的工作，而对于十年前的事情可能回忆的次数比较少，这是数据的时效性。这是个人观点，其他博客没有这方面资料。）但是，若我们认为数据是有重复的，而新一次的插入操作需要考虑是否更新旧数据的时候，好处一并不能成立，因为要遍历这个桶才能确定是需要更新还是要插入，当如果数据是带有时间戳，那么一般可以认为待插入的数据本身就是唯一了。**

b.性能分析：效率取决与桶的数量M的选取，hash函数针对数据分布的在各个桶中均匀性，最大链长（木桶短板原理）。若M足够大，那么查询是O(1)的，但是耗费数组的开销。若K太大，那么处于尾部的数据访问需要O(K)。极端情况（**哈希攻击**）下所有的数据都在一个桶中，那么后面的数据的访问需要遍历N个节点，效率低到了O(N),容易导致机器崩溃，解决的办法或许有达到一定冲突次数时使用随机的hash函数。Java中的HashMap设计了装填因子（数据量/桶数量，默认0.75），如果数据量超过了装填因子需要扩容一倍，并且重新散列，那么有大量数据需要迁移了，需要谨慎设计装填因子，关于数据迁移问题有『一致性哈希』算法，见下文。

![](~/1hash_link.jpg)


(2)线性探查法（开放地址法的一种）

思路：冲突时找到下一个空位来存储。比如0位置存了3，1位置存了4，2位置存了5，3位置存了0，那么新来一个6的位置本应该是0，可是0被占用了，那么他只能跑到4位置之后去了。当访问时，从索引位置到下一个空位之间顺序查找，到空位都找不到那么search miss了。

缺点：第一，冲突很多时可能要存储在离索引很远的位置。第二，删除时可能要涉及索引与下一个空位之间一大块数据的迁移。第三，导致了数据堆积现象，几个位置冲突来冲突去了。第四，像我在拉链法中提到的若数据具有**『时效性』**那么是非常糟糕的，后来的数据总是在很远的位置，但是对于『反时效性』数据这可能是比较好的。

(3)其他开放地址法，比如随机冲突，从周围的位置找一个放一下。比如再哈希方法，或许可以设计第二个哈希来辅助存储，或许可以设计备用存储位置。

4.代码实现Python

注：这个代码只关注实现原理，若需要高效哈希表请改用其他编程语言。

```python
# coding = utf-8


class HashTable:
    class KV:
        """
        for storage
        """
        def __init__(self, k, v):
            self.k = k
            self.v = v

        def __repr__(self):
            return "k:%s,v:%s" % (self.k, self.v)

    def __init__(self, init_capacity=4, init_load_factor=0.75):
        self.capacity = init_capacity
        self.load_factor = init_load_factor
        self.size = 0
        self.buckets = None
        self._init()
        self.hash = self._mod_hash

    def _init(self):
        """
        prepare buckets
        :return: bucket tuple
        """
        self.buckets = tuple([] for _ in range(self.capacity))  # list is mutable, whilst tuple is immutable

    def _rehash(self):
        """
        inflate table
        """
        self.capacity = self.capacity << 1
        new_buckets = tuple([] for _ in range(self.capacity))
        for bucket in self.buckets:
            for item in bucket:
                idx = self.hash(item.k)
                kv = HashTable.KV(item.k, item.v)
                new_buckets[idx].append(kv)
        tmp = self.buckets
        self.buckets = new_buckets
        del tmp

    def _mod_hash(self, key):
        """
        alternative hash function
        :param key: int key
        :return: int idx
        """
        return key % self.capacity

    def contains(self, key):
        """
        in or not in the table
        :param key: int key
        :return: True or False
        """
        idx = self.hash(key)
        for item in self.buckets[idx]:
            if key == item.k:
                return True
        return False

    def put(self, k, v):
        """
        put item
        :param k: int item_key 
        :param v: object item_value
        """
        bucket = self.buckets[self.hash(k)]  # shallow copy
        need_insert = True
        for item in bucket:
            if item.k == k:
                item.v = v  # update value
                need_insert = False
        if need_insert:
            kv = HashTable.KV(k, v)
            bucket.insert(0, kv)
            self.size += 1
        if float(self.size) >= self.capacity * self.load_factor:
            self._rehash()
        return self

    def delete(self, key):
        """
        try to remove item
        :param key: int item_key
        """
        deleted = False
        idx = self.hash(key)
        bucket = self.buckets[idx]  # shallow copy
        for i, item in enumerate(bucket):
            if key == item.k:
                bucket.pop(i)
                self.size -= 1
                deleted = True
        return deleted

    def __repr__(self):
        return self.buckets.__str__()
```

5.JAVA中的HashMap(摘自其他博客，未验证)

用hashcode()来判断落在哪个桶中，用equals()来继续从单链表中判断key是否相等。使用Entry<K,V>来存储对象。

#### 二、多机版P2P网络

1.网络分类

(1)非结构化网络：缺少结构，动态添加删除节点依然健壮，但是定位资源须遍历(洪泛)网络。

(2)**结构化网络**：结构清晰，不够健壮，任何节点都能高效定位资源。采用一致性哈希或者分布式哈希(DHT)算法。

2.搜索策略

(1)中央索引：比如使用一个NameNode来维护数据空间，搜索高效，但是集群规模变大时变得不可行。

(2)本地索引(Flooding)：本地先搜索，搜索不到会遍历(洪泛)网络，耗费流量、CPU、内存。对于热数据(经常访问的数据)需要可能有多个备份以加速访问，对于冷数据搜索缓慢。

(3)分布式索引:保存特定的邻居表以加速访问。添加删除节点时没有非结构化网络鲁棒。

3.(中央索引)**一致性哈希(Consistent Hashing)**

将节点和数据映射到同一个线性空间，节点负责保存前一个节点到本节点之间的数据。查找O(1)，添加删除时只需要移动少量数据，而**不需要像HashMap那样移动大量数据。**

(1)性质:平衡性、单调性、分散性(去冗余)、负载。 

(2)原理：如下图，考虑hash范围是环形。每个服务器有多个hash点，对每个数据来说，找到顺时针（逆时针也一样的）方向的最近一个服务器入口，将数据挂载到这里。那么寻找数据时，只需要计算hash值执行相同的过程就能快速找到服务器，之后从服务器中找到数据。当某个服务器挂掉了，那么把那些数据重新挂载到下一个服务器入口即可，只是这个服务器上的数据需要迁移，而不需要全体数据重新哈希。如果添加了新的服务器，那么只需要从他的下一个入口里拿出一部分数据挂载到新服务器上。之所以一个服务设置多个入口，是为了使负载更均衡，而各个服务器的性能也可能有差异的。


![](~/2consitent_hash.jpg)


4.分布式哈希（DHT）

CHORD算法：哈希查找不再由中央控制，而在每个节点上存储一个『后继字典（跳跃路由表）』，这个字典记录了一定范围内的后继节点。如果找到了，那么直接跳到相应的后继节点上取数据，如果找不到，跳到最远的那个后继节点上，之后从那个节点上继续查询。比如下图，若在B1节点上寻找5，那么本节点立刻找到了；如果找7，那么立刻找到了C1节点上；如果在B1节点上找23，那么最远处是12，已经无法找到在本机或者直接后继上找到，那么跳到最远的后继A2上找，A2的查找范围为9-24，那么23发现是在A1节点上。这样，查找效率是logN的。[TODO]另外还有KAD算法，以后有时间再学习。

![](~/3DHT.jpg)


### C.Bitmap位图算法
#### 一、原理

1.原理仍然是『桶』，只是用位(bit)来表示桶，不支持一个桶中数据冲突(就是说一个桶只能存一个值，因为一个桶只能在0和1两种状态切换，其信息量上限是1bit)。

2.在JAVA中(为了可移植性)规定一个int占4字节(32位)，那么存一个int数据(-2^31 ~ 2^31-1) 需要占用32位(bit)。而事实上我们可以用32位分别表示32个数据(类似onehot编码)，比如xxxxx110表示不存在数字0，存在数字1，存在数字2(习惯上用1表示存在，也方便初始化；若存在一个映射关系，那么也可以用任意位置表示任意数值或者其他类型的元素)。

实践上使用字节数组(bytearray)来存储，那么访问的时候需要知道两个位置(想象为矩阵中找元素)，第一个是寻找数组中的位置(O(1)的行查找，axis=0，内存寻址设计为计算内存地址偏移可以立刻到达)，第二个是寻找在这个字节中这个位的位置(O(1)的列查找，axis=1，8个bit一批的计算)

若是int型，存储量被压缩了32倍；若是char型（Java中规定2个字节，C中默认1个字节），比如可以用第0位表示有无「a」，第1位表示有无「z」，那么被压缩了16倍；若是字符串，若是文件，那么压缩了更多。

思考：JAVA中的hashcode（）找桶使用的返回值是int型，如果使用Bitmap来做索引而不是内存地址用做索引(桶入口)，那么可以节约32倍的索引存储。但是又要引入额外的问题，比如批量操作问题(机器默认的行为是按字节（byte默认8bit））处理，那么需要额外的编解码才能找到某个bit)，比如哈希冲突方案的重新设计。

3.**利用Bitmap进行排序仍然是桶排(基于bit的桶排)。前提元素无重复，否则存在覆盖问题，那么元素数量变少了（自动去重）。**

#### 二、应用

1.大数据计算：四十亿个**无重复**int数排序。(经典面试题)

(1)bitmap方法：考虑4字节存储，那么是16GB，那么通过小内存机器通过内部排序是不行了。但是若采用bitmap来存储排序那么需要0.5GB，也许在小内存机器上就跑得动内部排序了。

(2)使用『胜者树』多路归并：比如分成100路，划分100个文件各自排序。再对100个句柄每次从中选择最小的top来pop出来到一个缓冲，缓冲达到一定数量数量时写一次数据文件。

(3)考虑Map-Reduce方法：预扫描数据，获得100万的随机样本。对100万个数据排序，顺序划分为100块(每块约为1万个数据)都获得上下界，共200个值,注意最小处和最大处需要重置。那么通过这些『取值范围字典』可以在map的过程中构造100个不同的key，通过shuffle将数据流入100个reduce中，各个reduce内进行排序且各自输出到100个文件中，将这些文件简单拼接起来就是全局排序结果。

2.磁盘空闲块的管理，服务器路由，黑白名单，数据压缩。

#### 三、程序实现

1.实现说明

(1)定位两个索引(相当于找矩阵的两个下标)：

第一，bytearray中的索引：

```python
idx_array = num >> 3  # 相当于除以8。
#原因，右移一位就是除以2。
```

第二，byte中的索引：

```python
idx_byte = num & 0x07  # 相当于模8。
# 原因，xxxxxabc  % 8 = (xxxxx000+00000abc) % 8 = abc =  xxxxxabc & 0x07
```

(2)add

```python
self.bm[idx_array] |= 1 << idx_byte
# 或上那个位置，如果本身存在就是覆盖(去重)。
```

(3)remove

```python
self.bm[idx_array] &= ~(1 << idx_byte)
# 这个字节上将该bit置零，其他位置1的mask，按位与上去。
```

(4)contains

```python
self.bm[idx_array] & (1 << idx_byte) != 0
# 这个这节上与上这一位置1，与0向量比较。
```

(5)存储范围为 0 ~ capacity * 8 - 1

2.Python实现Bitmap与位图排序算法

```python
# coding=utf-8

class BitMap:
    """
    class for bitmap_sort
    """

    def __init__(self, capacity=10):
        self.capacity = capacity
        self.bm = bytearray(b'\0' * self.capacity)
        self.limit_max = capacity * 8 - 1
        self.limit_min = 0
        self.size = 0

    @staticmethod
    def _get_position(x):
        """
        convert key to position in the bitmap
        :param x: int key
        :return: int row, int column
        """
        # imagine one matrix, row at idx_array, column at idx_byte
        idx_array = x >> 3  # x // 8
        idx_byte = x & 0x07  # x % 8
        return idx_array, idx_byte

    def _check_range(self, x):
        """
        check if the key is acceptable
        :param x: int key
        """
        if x > self.limit_max or x < self.limit_min:
            raise ValueError("value:%s is too large or too small" % x)

    def put(self, x):
        """
        add key into the bitmap
        :param x: int key
        """
        self._check_range(x)
        idx_array, idx_byte = self._get_position(x)
        self.bm[idx_array] |= 1 << idx_byte
        self.size += 1

    def remove(self, x):
        """
        remove key from the bitmap
        :param x: int key
        """
        self._check_range(x)
        idx_array, idx_byte = self._get_position(x)
        self.bm[idx_array] &= ~(1 << idx_byte)
        self.size -= 1

    def contains(self, x):
        """
        check if a key is in the bitmap
        :param x: int key
        :return: True or False
        """
        self._check_range(x)
        idx_array, idx_byte = self._get_position(x)
        return self.bm[idx_array] & (1 << idx_byte) != 0

    def reset(self, capacity=None):
        """
        reset the bitmap to a zero-array with new capacity
        :param capacity: int
        """
        self.capacity = capacity if capacity else self.capacity
        self.__init__(self.capacity)

    def put_batch(self, xs):
        """
        put keys into the bitmap
        :param xs: int [] keys
        """
        [self.put(x) for x in xs]

    def decode_bm(self):
        """
        decode the bitmap into a human-readable int array
        :return: int []
        """
        ret = []
        i = self.limit_min
        while i <= self.limit_max:
            if self.contains(i):
                ret.append(i)
            i += 1
        return ret

    def bitmap_sort(self, xs):
        """
        sort numbers using a bitmap
        :param xs: int []
        :return: int []
        """
        capacity = max(xs) // 8 + 1
        self.reset(capacity=capacity)
        self.put_batch(xs)
        return self.decode_bm()

    @classmethod
    def sort(cls, xs):
        """
        just class method for sorting with bitmap
        :param xs: int []
        :return: int []
        """
        capacity = max(xs) // 8 + 1
        bm = BitMap(capacity)
        bm.put_batch(xs)
        return bm.decode_bm()

    def show(self):
        """
        show the bitmap
        :return: int []
        """
        print([mi.__str__() for mi in self.bm])

    def test(self):
        """
        check the functional of the class
        :return: raise or return True
        """
        from random import randint
        x = [randint(0, 1000) for _ in range(20)]
        for _ in range(100):
            try:
                assert tuple(sorted(set(x))) == tuple(BitMap().bitmap_sort(x))
            except AssertionError:
                print sorted(x)
                print BitMap().bitmap_sort(x)
        for _ in range(100):
            try:
                assert tuple(sorted(set(x))) == tuple(BitMap.sort(x))
            except AssertionError:
                print sorted(x)
                print BitMap.sort(x)
        print("it works")
        return True


if __name__ == '__main__':
    BitMap().test()
```



### D.布隆过滤器 Bloom Filter

#### 一、布隆过滤原理

1.和前两类算法相同，仍然是『**分桶**』的原理。

2.**思想1(空间效率)：桶可以复用，一方面是数据可以映射到多个桶中，另一方面是一个桶可以映射出多个数据。实践上采用k个(独立的)哈希函数来做映射。这一点上看，若要保证准确率那么空间开销应该比哈希表大，但是其实比哈希表开销小，因为采用了BitMap而不是HashCode(4字节)来做映射，因为不需要复原数据，只需要知道数据的存在性，存在性只需要一个bit就可以表示了，冲突解决的方式也采用了『直接覆盖』而不需要拉链或者开发地址空间。那么Bitmap的开销不是更小吗，是的，Bitmap适合于数据比较集中的情况，若数据比较分散，那么布隆过滤器是一种中庸的做法，只有当k个hash值在bitmap上的位置都为1时才能认为数据很大可能是历史数据，否则一定不是历史数据。**

**3.思想2（时间效率）：不要求精确查询，可以允许错误(False Positive，不存在历史中的数据误判为历史数据)，那么可以使用『敏捷失败』的思想(敏捷失败认为如果可以预知或者已知查询失败那就立刻失败，比如机器学习中资源有限，那么如果一个参数已知不可行了立刻就换下一个参数，比如一些关键bug发生时应该让程序立刻跳出而不是继续计算其他数据或者进入无尽的冒泡。有些程序的进度条是永远没有结果的，判断程序已经失败那就是节约时间。)。布隆过滤的复杂度O(K)是常数级的，如果不在其中那么确定不再其中。如果还需要确知在其中，还需要设计一层精确查询，这样是两层查询(深度学习中的Modularization，认为deep is better than fat)，粗粒度查询+细粒度查询，但是已经比直接的单层精确查询性能好很多了。相当于做了一层预判，如果从概率上大量是search miss，那么由原先的对数或者线性复杂度将被平均到O(1)上，那么设计一层布隆过滤层是值得的。**

#### 二、布隆过滤的缺点

1.需要预先设计好空间开销。随着历史数据量的增大，对于一些依赖于小误判率的应用可能无法适应误判率的增大，这是bitmap带来的缺点。不过有一些布隆过滤器是支持扩展的。

2.只能添加元素，不能删除元素。因为一个bit被多个数据共享。

3.无法返回元素本身。因为冲突的解决是直接覆盖。

4.还是有一定的误判率。

#### 三、布隆过滤器的应用

- 黑白名单，email过滤
- 网络爬虫已爬列表
- 设计为粗粒度查询加速查询

#### 四、误判率(False Positive)的推导

1.前提

假设数据能够随机均匀地插入到bitmap中

又假设历史数据和待过滤数据的分布一致

令m为桶数，k为哈希函数数量，n为历史数据量

2.推导如下图

结论是使用9.6bit表示一个数据时，对于本来应该search miss的数据只有1%的误判。


![](~/4_bf.jpg)


#### 五、代码实现Python

```python
# coding=utf-8
import string
import random
import matplotlib

matplotlib.use('TkAgg')
import matplotlib.pyplot as plt
from functools import partial
from bitmap import BitMap


class BloomFilter:
    """
    基于BitMap的布隆过滤器
    """
    N_INSTANCE = 0  # 实例数量

    def __init__(self, size_byte=1 << 6, n_hash=1 << 2):
        self.size_byte = size_byte  # 字节开销
        self.size_bit = 8 * size_byte  # bit开销
        self.n_hash = n_hash  # hash函数数量
        self.bf = BitMap(self.size_byte)  # 存储
        # 通过科里化方式生成n_hash个hash函数列表
        self.hash_functions = [partial(self._hash,
                                       capacity=self.size_bit,
                                       seed=seed)
                               for seed in self._mk_primes(self.n_hash)]

        BloomFilter.N_INSTANCE += 1

    @staticmethod
    def _mk_primes(n):
        """
        制作n个素数，用于hash函数种子
        :param n: int
        :return: int []
        """
        num = 5
        ret = []
        i = 2
        while 1:
            if num % i == 0:
                num += 1
                i = 1
            if i * i > num:
                ret.append(num)
                n -= 1
                i = 1
                num += 1
                if n == 0: break
            i += 1
        return tuple(ret)

    @staticmethod
    def _hash(string, capacity, seed):
        """
        哈希函数.注意这个hash函数效果是很差的，实际使用应该考虑其他hash函数。
        :param string: str,待输入的数据
        :param capacity: int, Bitmap的bit容量（待绑定）
        :param seed: int, hash函数种子，建议为素数(待绑定)
        :return: int, hashcode
        """
        ret = 0
        for si in string:
            ret += seed * ret + ord(si)
        return (capacity - 1) & ret

    def _get_codes(self, a_string):
        """
        取得k个hashcode
        :param a_string: str, 待哈希的数据
        :return: k-dim int tuple
        """
        return tuple(f(a_string) for f in self.hash_functions)

    def put(self, a_string):
        """
        添加一个历史数据
        :param a_string: str,单个数据
        :return: raise or success
        """
        self.bf.put_batch(self._get_codes(a_string))
        return "success"

    def put_batch(self, strings):
        """
        添加多个历史数据
        :param strings: <iterable<str>> 数据集合
        """
        [self.put(a_string) for a_string in strings]

    def contains(self, a_string):
        """
        查询数据是否在历史数据中
        :param a_string: str, 待查询数据
        :return: boolean, True or False
        """
        for code in self._get_codes(a_string):
            if not self.bf.contains(code):
                return False
        return True

    def __contains__(self, a_string):
        """
        为contains函数支持in语法
        """
        return self.contains(a_string)

    def get_load(self):
        """
        查看bitmap的负载（覆盖率）
        :return: float, 负载
        """
        return sum(1 if self.bf.contains(i) else 0
                   for i in range(self.bf.limit_min, self.bf.limit_max + 1)) / float(self.size_bit)

    def show_bf(self):
        """
        查看实际存储情况
        :return: void -> stdout
        """
        self.bf.show()

    @classmethod
    def test(cls):
        """
        可用性自我测试
        :return: void -> raise or success
        """
        m = BloomFilter()
        m.put("xyzd")
        m.put("")
        m.put("uuu")
        m.put("uu1")
        assert m.contains("xy") is False
        assert m.contains("tz") is False
        assert "tz" not in m
        assert "uuu" in m
        print("it works")


class CrossValidation:
    """
    布隆过滤器的分析
    """

    def __init__(self):
        pass

    @staticmethod
    def mk_sample(n=1000):
        """
        制作数据样本
        :param n: int， 数据个数
        :return: 随机数据元组
        """
        return tuple("".join(random.choice(string.ascii_letters) for _ in range(random.randint(1, 100)))
                     for _ in range(n))

    def load_with_put(self, n_bf_byte=128, n_hash=3, n_sample=2000):
        """
        获取不同添加个数下的负载
        :param n_bf_byte: int, 字节开销
        :param n_hash:
        :param n_sample: int, 添加数量上限
        :return: <list<2-dim tuple>>
        """
        bf = BloomFilter(n_bf_byte, n_hash)
        sample = self.mk_sample(n_sample)
        record = []
        for i, sa in enumerate(sample):
            bf.put(sa)
            record.append((i, bf.get_load()))
        return record

    def plt_load_with_put(self):
        """
        绘制负载-数量的分析结果
        :return: void -> file
        """
        params = [(1 << nn_byte, n_hash, 2000) for n_hash in range(1, 8, 2) for nn_byte in range(6, 11, 2)]
        plt.figure()
        for param in params:
            print(param)
            plt.plot(*zip(*self.load_with_put(*param)), label=param.__str__())
        plt.legend()
        plt.grid()
        plt.xlabel("n_add")
        plt.ylabel("load")
        plt.savefig("load_with_put.png", dpi=300)
        plt.show(block=True)

    @staticmethod
    def false_positive(n, data, rates=(9.6, 14.4)):
        """
        输错False Positive误判率
        :param n: int, 训练数据量
        :param data: <list<int>>, 所有数据
        :param rates: m/n, 容量与数据比
        :return: <dict<float:float>>, 不同容量比下的误判率
        """
        if 2 * n > len(data):
            raise ValueError("data is not enough")
        data_train = data[:n]
        data_test = data[n:]
        ret = {}
        for rate in rates:
            m = int(round((n * rate)))
            k = int(round(0.707 * rate) + 1)
            n_byte = m // 8 + 1
            bf = BloomFilter(n_byte, k)
            bf.put_batch(data_train)
            fn = sum(1 if bf.contains(data_i) else 0 for data_i in data_test)
            ret[rate] = float(fn) / len(data_test)
        return ret

    def plt_fasle_positive(self):
        """
        [TODO]BoolmFilter里的hash函数效果是很差的，需要更换
        :return:
        """
        sample = self.mk_sample(312)
        print(self.false_positive(20, sample))


if __name__ == '__main__':
    CrossValidation().plt_load_with_put()
    # CrossValidation().plt_fasle_positive()
```

#### 五、布隆过滤的其他思考（留给读者）

1.能否设计k不固定的hash函数？（比如根据输入的字符串的长度，长度超过10使用5个哈希函数，长度小于10使用2个哈希函数）

2.对于训练数据确定的问题，如何去寻找（训练）出最优的k个hash函数？如果当做一个机器学习任务，那么如何去做？如何根据数据的分布特征自动学习出布隆过滤器？

3.是否能够训练出某种范围（而对这个范围的判断是O（1）而不是O(K)的）？即通过多个hash函数（或者是多层hash函数，非线性）模拟出二分类问题的超平面边界（目标是False Negative为0且False Positive尽量小）？



### E.哈希表、BitMap与布隆过滤器

1.时间复杂度：查询时哈希表为O(1)，取决于链长。Bitmap为O(2)。布隆过滤器为O(K)，遇到失败时可提前结束。

2.空间复杂度：哈希表是字节以上的存储，需要存储碰撞，HashMap的利用率不到50% 。Bitmap是bit存储，利用率取决于容量的合理设置，三者中利用率可以最高。布隆过滤器基于bitmap使用bit存储，需要预先设置容量，利用率和误判率矛盾。

3.优缺点：哈希表查询快，但是不大适合大数据存储。大量数据存储考虑bitmap或者布隆过滤器，bitmap适合集中数据的集中存储，而布隆过滤器适合分散数据的存储但是有一定的False Positive。

### F.其他思考

1.本质上查询的search miss和search hit是一个**二分类问题**，那么可以用一个机器学习中的分类模型来做。而这样的模型需要具有一个特性就是**一次训练，多次预测**。或者要支持后续训练，无论是单次online还是定期批量online。
2.对于特定分布应该如何考虑，对于未知分布应该将分布假设成什么分布比较合理？



###参考：

> [哈希]
> [浅谈哈希表]http://blog.jobbole.com/79261/
> [散列表原理]http://blog.csdn.net/duan19920101/article/details/51579136
> [Hash算法原理]https://zhuanlan.zhihu.com/p/24271658
> [Java中的HashMap]http://blog.csdn.net/caihaijiang/article/details/6280251
> [Java中的HashMap]http://www.cnblogs.com/yuanblog/p/4441017.html
> [P2P网络]http://blog.csdn.net/dc_726/article/details/42612349
> [一致性Hash]http://blog.csdn.net/cywosp/article/details/23397179
> [Chord]http://blog.csdn.net/wangxiaoqin00007/article/details/7374833
> [Bitmap]
> http://www.cnblogs.com/dyllove98/archive/2013/07/26/3217741.html
> http://www.jianshu.com/p/6082a2f7df8e
> http://blog.csdn.net/liufei_learning/article/details/19303179
> [Bloom Filter]
> http://www.cnblogs.com/allensun/archive/2011/02/16/1956532.html
> https://segmentfault.com/a/1190000002729689
> http://www.dataguru.cn/thread-484931-1-1.html
> http://geek.csdn.net/news/detail/92486
> http://blog.csdn.net/golden1314521/article/details/33268719