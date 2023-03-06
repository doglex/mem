---
date: 2017-08-09 21:29
status: public
title: 《算法4th》杂记
---

### A.分治法：

把问题转化为更小规模的问题。

### B.两个栈实现四则运算

一个栈用来压操作数，一个用来压操作符。该pop就pop，操作后压回结果到操作数的栈。

### C.Java简介

本身都是为了提高代码的可重用性，可读性。

软件工程容易陷入《人月神话》中提到的窘境，就是debug的速度跟不上产生bug的速度，而向项目加派人手反而使项目更加落后，因为有许多沟通成本、同步成本。学习一些技巧可以减少一些坑。

但是事实上『**没有银弹**』，没有一个通用的实践可以解决所有问题，而目前认为优秀的算法也许要随着硬件的升级、平台的变化变得不再适用。所以永远需要与时俱进的程序员。在《异形：契约》电影中提到大卫说『人类创造了他，人类会死，而他永生』，大卫追求的是一种『科学』，一种『创造』，这和人类其实是一样的。那么有一天人类可以借助『机器学习』让机器自动写程序，让机器可以『创造』，那么又是谁创造了人类？那么『机器人创造』是否是『人类创造』的子集呢？如果机器人可以比人类走得更远，那么人类可以比上帝走得更远或许是真的。我们所谓的『创造』或许静态的，人类的能动性是或许静态的，就像动画里的主人公永远不知道下一刻会发生什么，而我们却可以随意的暂停，快进。

- **面向对象OOP：继承、封装、多态**

  理解：

  1.继承是说别的类里好的东西可以拿过来，我实现得好的东西也可以给别人用，同时别人要是和我用同一个概念的东西那么我就已经会使用了『动态绑定』就可以了，那么拿过来的时候别人的实现要和我做适配适配不上那么我可以**重写(override)**，java**不允许多重继承**，若有很多公用的东西而这些东西来自不同的地方那么我们就可以用接口来表示他们。接口是可以被大家都使用而且可以使用多个接口，接口中可能是需要把方法设为虚方法（好像jdk1.7还是1.8后可以允许默认的接口实现，也允许接口有一些简单的数据，本来这些是接口和抽象类的重要区别现在看起来倒不是这样），那么一个类要实现一个接口就必须实现这个**接口**的所有功能，不然这个接口不完整那么类的实例就残废了（该调用方法的时候发现偷工减料了，方法失败了），所以当所有类都保证可以实现接口的功能那么通过接口就可以随意的调方法而不必担心某个方法未实现了（generics）。那么如果当接口中只有一个函数的时候我们就确定知道我们要实现的就是这个这个函数而不是别的什么函数，那么这个函数就不需要单独去命名他了把，我们call这个接口就可以直接（匿名地，不会引起歧义）call到他，那么这就是**函数式接口**（jdk1.8引入）。

  2.封装：我不想给别人用的东西，别人通过常规的途径是获取不到的。而且最好是一开始就不给别人用(private)，那么别人要用这个东西的唯一方式就是告诉我他要用(通过**setter，getter**之类的方法)，而且是要找得到我的时候（public）。java默认有包(package)保护不让包外未知的人过来访问，那么使用protected权限可能会被子孙所泄露，所以不建议使用protected（除非非常清楚他的代价）。一般建议是用private，实在要公开再用public，从原则上说数据应该是private的而操作或者访问数据需要通过一些开放的public方法。

  3.多态：我只需要说『开饭』，那么每个人都开始吃（不管他们是什么种族），以他们自己的方式吃饭（**『动态绑定』**，C++里好像是默认为静态绑定）。只要他们都是『人』这个类的子孙类或者都是他们都能识别『吃饭』这个接口，那么该吃饭的时候他们自己就知道怎么吃了。那么哪怕java是一种强类型语言，也可以同时地轻松操作各种不同类型的对象了。实现的时候Java会去搜索继承链上的相关的方法列表，找到最合适，匹配上当前对象的。


- **ADT**（Abstract Data Type）：operations revolve around adding，removing。

  理解：其实分布式计算不就这样吗，大多是计算围绕数据，如果数据在一个集群而计算资源在另一个『传输距离』遥远的位置那么代价很高。数据拥有他的操作。或者也可以固定一些管道，让数据流过管道（函数式编程or符号编程，未来的数据可以用具有确定形状shape的占位符placehold表示）。


- generics 泛型编程、模板、泛型类

  理解1：我们不想写重复的东西，写了东西之后对各种类型不必重写一份（在Python中是非常简单的，但是Python反而需要反过来考虑强类型问题），那么可以用一个模板类<T>表示，一个不够用多个，甚至可以有一些高级的使用，在真正使用的时候再指定<T>，当然如果行为是确定的也可以交给编译器自己去推导，但是翻译成二进制后代码运行前需要保证类型是确定的。

  理解2：一些通用的好用的方法大家都喜欢去实现他，比如toString(python中有魔法方法)，hashCode，equals。

  理解3：一些好用的接口大家能达成一致，比如Iterable，Comparable。

- 链式结构：有数据和next方法，方便访问。比如单链表，二叉树，图。

- 高级类型：

  1.背包（Bag）：只添加，不能删除的集合。（布隆过滤器也是认为只能添加的）

  2.**队列**  (FIFO):先到先服务。（Queue，ArrayQueue，）for-statement中顺序。AutoResizing，0.75.

  3.**栈**（FILO）：后到先服务。（Stack）for-statement中逆序。

  4.**数组**Array：数组也是对象，索引立刻访问，预设大小。

  5.**链表LinkedList**：线性搜索，动态增长。链表可以用于实现队列、栈。

- Autoboxing：自动装箱和自动拆箱，基本类型（int）和基本包装类型（Integer）自动转换，方便对象操作。

### D、算法分析：

分析原则：reproducible，falsifiable，qualitative observation。取对数log-log容易分析，一般都转线性（power-law）了。

|  描述  |     英文      |  Big-O   |
| :--: | :---------: | :------: |
|  常数  |  constant   |   O(1)   |
|  对数  | logaritmic  | O(logN)  |
|  线性  |   linear    |   O(N)   |
| 线性对数 | linearihmic | O(NlogN) |
| 平方级  |  quadratic  |  O(N2)   |
| 立方级  |    cubic    |  O(N3)   |
| 指数级  | exponential |  O(2N)   |

注意事项：

- 很大的常数项
- 隐式内部循环
- 指令时间未必一致
- 系统需要与其他程序竞争
- 对输入数据强依赖
- 多参数
- 考虑最差时间worst-case

### E、排序Sorting

#### 一、为何排序？

方便搜索,比如二叉搜索。

certification，running time，extra memory，types of data

java.util.Arrays.sort,java.util.Collections.sort（Arraylist应该用后者）

Reduction:duplicates rankings,priority queues,meidan,order

#### 二、冒泡排序BubbleSort

冒泡时两两交换，进行n趟，复杂度平方级。可以是稳定的（原来相等的key保持原来顺序不变）。

1.Python实现1：

```python
def bubble_sort(alist):
    for _ in range(len(alist)):
        for i in range(len(alist) -1):
            if alist[i] > alist[i+1]:
                alist[i],alist[i+1] = alist[i+1],alist[i]
    return alist
```

分析：

- 没有做类型检查（对Java等强类型语或许传入的是强类型array，但也要保证有compareTo，或者重载了大于号）。
- 没有确定类型，未必有\__len__()属性。有这个属性，需要满足长度2以上。
- alist使用浅拷贝可能是危险的，除非非常明确他的全局意义，否则无意中可能覆盖全局数据。或者说本身排序是原地的。
- 另外一种风险就是，可能传入的是一个迭代器，数据是流式数据，甚至无法在原地操作，需要设置接收器。

2.python实现2：

```python
def bubble_sort(blist):
    try:
        blist = list(blist)
    except:
        raise ValueError("unacceptable type")
    for _ in range(len(blist)):
        for i in range(len(blist) -1):
            if blist[i] > blist[i+1]:
                blist[i], blist[i + 1] = blist[i + 1], blist[i]
    return blist
```

- try-except 语句本身开销巨大。虽然避免了迭代器或者浅拷贝的风险。
- 另外，blist的返回类型是列表，仍然是不够安全的。改成元组会安全一些，但是后续使用依然是无法保证的。

#### 三、桶排BucketSort  

一个一个放入对应的桶。那么需要桶的空间足够大，依次放好，去掉空桶。时间复杂度O(N)。空间换时间。

具体实现请参考我的上一篇博客《哈希表、Bitmap与布隆过滤器》。

#### 四、选择排序SelectionSort

每次从待排列表中选出一个最小的元素加入已排序列表。

复杂度O(N2),对输入有序还是无序不敏感。

方式一：使用外部空间依次添加。

方式二：原地，未排序中的与已排序的交换。

1.Python实现1（外部空间）：

```python
def selection_sort(alist):
    alist = list(alist)
    blist = []
    while len(alist):
        min_idx , min_value = 0, alist[0]
        for i, v in enumerate(alist):
            if v < min_value:
                min_idx = i
                min_value = v
        blist.append(alist.pop(min_idx))
    return blist
```

分析：

- 未做类型检查，不能保证list()能正确获得数据列表，不能保证数据类型重载了小于号。
- append和pop语句有可能是代价比较高，将效率依赖于列表的实现了。

2.Python实现2（原地修改）：

```python
def selection_sort(alist):
    for i in range(len(alist) - 1):
        for j in range(i+1, len(alist)):
            if alist[j] < alist[i]:
                alist[j], alist[i] = alist[i], alist[j]
```

分析：

- 将实现依赖于浅拷贝，假想了alist是个list。那么如果alist是个元组或者迭代器呢？
- 可以直观看出使用的技巧是双指针交换法，一个前向探索指针。很容易判断复杂度是平方级，而且是输入有序无序不敏感的。
- 可以看出，选择排序可以是稳定的。

#### 五、插入排序InsertionSort

1.**插入排序**：假设前面都已经排好序，新元素向前扫描，该交换就交换，不能交换了说明已经好了（类似抓扑克牌）。

假设数据本身大部分有序，那么可以达到O(N)。若大量无序，仍然需要平方级O(N2)。

2.python实现1（原地交换）：

```python
def insertion_sort(alist):
    for j in range(1, len(alist)):
        for i in range(j,0,-1):
            if alist[i] < alist[i-1]:
                alist[i-1], alist[i] = alist[i],alist[i-1]
            else:
                break
```

- 假设了alist是mutable的array。原地双指针，一个记录当前位置，一个逐步逆向探索并交换。
- 这样做的一个坏处就是有可能需要进行多个交换，下一个实现将对这个缺陷进行改进。
- 插入每次只能将数据移动一位，改进有希尔排序跨数据插入，见下文。

3.python实现2（非原地）：

```python
def insertion_sort(alist):
    alist = list(alist)
    for j in range(1, len(alist)):
        i = j-1
        while i >= 0 and alist[i] > alist[j]:
            i -= 1
        alist.insert(i+1,alist.pop(j)) # 就算到达左侧端点，也可以以0索引插入
    return tuple(alist)
```

- 只定位到特殊位置进行插入。
- 不会对全局（外部数据）产生影响，为什么这点是重要的，因为这个数据未必一个线程独占，可能有其他写线程在同时操作这个数据。
- 注意比最左端还小时该如何插入。

4.**希尔排序**（shellsort，也叫**缩小增量排序**）：是改进的**插入排序**，实质是分组插入，复杂度介于O(nlogn)与O(N2)，每次比较相隔较远（gap）位置的数据，不断缩小gap至1（至少要保证一个1的gap的存在，最后一步调用几乎线性的全局插入排序），使得数据逐步有序。（同时消除多个元素的交换）

5.python实现3（原地）：

```python
def shell_sort(alist):
    length = len(alist)
    gap = 1
    while gap < length // 3:
        gap = 3 * gap + 1  # 1,4,13,40,...
    while gap >= 1:
        for i in range(gap,length):
            j = i
            while j >= gap and alist[j] < alist[j-gap]:  # 注意这里需要大于等于，需要达到边缘
                alist[j],alist[j-gap] = alist[j-gap],alist[j]
                j -= gap
        gap = gap // 3  # 注意python2和python3中除法的区别
```

- 注意除法，注意取数据到达边缘，注意一定要取到gap=1的时候（出口条件）。
- 这个实现有个缺陷，就是gap只能以3的倍数的模式来取，在下一个实现中把gap的设置单独解耦出来，更自由一些。

6.python实现4（非原地）：

```python
def shell_sort(arr):
    def gap_gen(size):
        # gap生成器，生成一堆gap
        while size > 1:
            size //= 2
            yield size

    def insertion_sort_reverse(nums):
        nums = nums[:]  # 深拷贝
        N = len(nums)
        for i in range(1, N):
            for j in range(N-1, 0, -1):
                if nums[j] > nums[j - 1]:  # 反向的插入排序
                    nums[j], nums[j - 1] = nums[j - 1], nums[j]
        return nums

    for gap in gap_gen(len(arr)):
        arr[::-gap] = insertion_sort_reverse(arr[::-gap])
    return arr
```

- 不管是gap生成器还是排序器都是可插拔的。
- 需要保证一个gap是1，最后一遍全局的排序。

#### 六、归并排序MergeSort

1.归并排序：两两归并，merge的时候从头比较比较只需O(n)。整体复杂度为O(NlogN)。

注：也可以多路归并，可以可以从尾部开始比较，比如剑指offer上一道题求数组的逆序对。

可以top-down递归，也可以bottom-up循环。

流程：sort(left)、sort(right)、merge(left、right) 

改进：

- 当片段足够小时可以使用其他的排序，比如插入排序。
- 先线性测试是否有序，若有序就不用排了。
- merge时与辅助空间切换着用，少一半拷贝。

2.python实现1（递归top-down）：

```python
def merge_sort(alist):
    alist = list(alist)
    if len(alist) <= 1:
        return alist
    elif len(alist) == 2:
        if alist[0] > alist[1]:
            alist[1], alist[0] = alist[0], alist[1]
        return alist
    mid = len(alist) // 2
    left = merge_sort(alist[:mid + 1])
    right = merge_sort(alist[mid + 1:])
    ret = []
    while len(left) and len(right):
        if left[0] <= right[0]:
            ret.append(left.pop(0))
        else:
            ret.append(right.pop(0))
    ret.extend(left)
    ret.extend(right)
    return ret
```

- 递归的效率比循环低，很多编程语言对递归层次有限制。
- pop方式或许效率有点低，改用索引可能好一点，但是注意尾部。append的效率可能也是低的。
- list(alist)和alist[:]的效率情况未知，但是前者至少支持迭代器过来的流式数据。

3.python实现2（循环bottom-up）

```python
def merge_sort(alist):
    delta = 1
    indexs = list(range(len(alist)))
    while delta < len(alist):
        for start in indexs[::2*delta]:
            mid = start + delta
            end = mid + delta
            left = alist[start:mid]
            right = alist[mid:end]
            copy = []
            while left and right:
                if left[0] <= right[0]:
                    copy.append(left.pop(0))
                else:
                    copy.append(right.pop(0))
            copy.extend(left)
            copy.extend(right)
            alist[start:end] = copy
        delta *= 2
    return alist
```

- 使用循环比递归效率高。
- 在其他语言中不能直接使用end，end可能数组越界，而python中在编程语言级别自动解决了end问题。
- pop和append效率是低的，仍然建议使用指针，同时还是手动解决一下越界问题吧。
- copy数组建议是与原数组来回切换着用，减少一半拷贝量。
- 同时注意到alist实际上是原地的，只是借助了辅助空间copy数组，这样可能仍然是危险的。
- 基于这个代码改多路归并是简单的，只需要改delta的跳跃范围与merge逻辑。


#### 七、快速排序QuickSort

1.快速排序：通过一趟排序将要排序的数据分割为独立的两个部分，其中一部分比哨兵大，一部分比哨兵小，每部分递归的进行。

2.缺点：

- **输入敏感**：若数据**无序，能达到O(NlogN)**的复杂度（归并和堆排都能达到这个）。若数据基本有序或者大量重复值，性能降低为平方级O(N2)，原因是partition偏向一边去了。所以输入请一般先随机shuffle一下。
- **快排不稳定。（至少书上是这么说的，具体还得看实现）**

3.快排优点：

- 使用广泛，容易实现，时间复杂度小
- 可以不需要额外空间（而堆排和归并需要）in-place
- 可提升：片段小时可以用插排改进，可以设置双哨兵，三路快排（3-way quicksort）。而堆排没有这些改进。
- 对分块（partition）**可并行**。

4.python实现1（单边扫描）:

```python
def quick_sort(alist):
    if len(alist) <= 1:
        return alist
    dog = alist[0]
    left, right = [], []
    [left.append(v) if v <= dog else right.append(v) for v in alist[1:]]
    return quick_sort(left) + [dog, ] + quick_sort(right)
```

- 需要额外临时空间
- 递归比循环效率低
- 扫描不如双边快
- 对于重复值不够友好

5.python实现2（双边扫描递归版）：

```python
# 哨兵位置用于交换，右扫置左，左扫置右
def quick_sort(alist,left,right):  # 传入数据列表，左右范围
    if left >= right: return alist  # 跳出条件
    dog = alist[left]  # 既是哨兵。也通过记录历史，空出了dog的位置用于交换。
    low,high = left, right  # 对入口范围进行记录
    while left < right:
        while left < right and alist[right] >= dog:
            right -= 1  
        alist[left] = alist[right]  # 确保右边大，第一次dog位置可用于存储
        while left < right and alist[left] <= dog:
            left += 1  # 接下来右边多余的位置可用于存储 
        alist[right] = alist[left]
    alist[left] = dog   # 此时left == right，放回哨兵
    quick_sort(alist,low,left-1)  # 递归
    quick_sort(alist,left+1,high)
    return alist
```

- 技巧在于哨兵的位置可以用于交换，而之前交换完的空位又可以用于交换。最后扫描到不能扫描。
- 注意仍然是递归的，是原地的，可能是有害的。

6.python实现3（双边扫描非递归版）：

```python
def quick_sort(alist):
    alist = list(alist)
    to_scan_pairs = [(0,len(alist)-1),]
    while to_scan_pairs:
        left, right = to_scan_pairs.pop(0)
        if left >= right: continue
        dog = alist[left]  # 既是哨兵。也通过记录历史，空出了dog的位置用于交换。
        low,high = left, right  # 对入口范围进行记录
        while left < right:
            while left < right and alist[right] >= dog:
                right -= 1
            alist[left] = alist[right]  # 确保右边大，第一次dog位置可用于存储
            while left < right and alist[left] <= dog:
                left += 1  # 接下来右边多余的位置可用于存储
            alist[right] = alist[left]
        alist[left] = dog   # 此时left == right，放回哨兵
        to_scan_pairs.append((low,left-1))
        to_scan_pairs.append((left+1,high))
    return alist
```

- 只需要将待扫描情况压入队列即可实现循环版的双边扫描快排。
- 支持了迭代器输入，非原地修改。

7.**三路快排**3-way QuickSort（jdk采用过三分区的三路快排，也采用过五分区的双基准快排）

针对的问题：**重复数据多**，若采用双边扫描很容易O(N2),何不增加一个分区用来解决这个问题，当然也可以考虑归并排序。

```python
def quick_sort(alist, left, right):
    if left >= right: return alist
    dog = alist[left]  # 取出哨兵
    lt, gt, i = left, right, left
    while i <= gt:  # 用一个指针向右扫描,直到到达右侧边界
        if alist[i] > dog:  # 若应该处于右侧，那么换到右边，并且将右边边界左移
            alist[i], alist[gt] = alist[gt], alist[i]
            gt -= 1
        elif alist[i] < dog:  # 若应该处于左侧，那么换到左边，并且将左边边界前推，且这个指针前推
            alist[i], alist[lt] = alist[lt], alist[i]
            lt += 1
            i += 1
        else:
            i += 1  # 若相等，那么前推
    quick_sort(alist, left, lt)  # 左半边都是比哨兵小的
    quick_sort(alist, gt+1, right) # 右半边都是比哨兵大的，注意这里是gt+1
    return alist
```

- 注意是原地的，递归的，改进可以参考上一种实现。

8.**双基准排序**，使用两个哨兵，理论上比单个哨兵要快一些（为什么这么说，实现更复杂了，那么效率还更低那是说不过去的）。实现可参考网上或者jdk源码，我暂时不做研究。

9.快排注意事项：

- 内存占用问题
- 考虑边界
- 混乱度输入敏感
- 注意循环结束条件
- 注意稳定性问题

#### 八、优先队列PriorityQueue与堆排HeapSort

1.在优先队列中，元素被赋予优先级，最高优先级的先pop。

2.常被用于调度系统。也被用于数组取top-K。

3.几种优先队列的实现

| 数据结构 | 插入效率    | 移除效率    |
| ---- | ------- | ------- |
| 有序数组 | O(N)    | O(1)    |
| 无序数组 | O(1)    | O(N)    |
| 堆    | O(logN) | O(logN) |

可以认为堆是有序数组和无序数组的折衷。

4.一个堆相关的问题取top-K。

- 堆排的复杂度是对数乘以线性的O(NlogN)，插入和删除堆的复杂度都是这个。
- 复杂度为对数乘以线程的常见排序还有快排（无序情况，若有序则是平方级，有大量重复值考虑三路快排），还有归并排序。快排和堆排都不稳定，归并和插入排序（基于有序可以线性O（N），考虑增强的缩小增量希尔排序）是稳定的。
- 堆是近似的完全二叉树，若不满，则最后一层的后半部分。
- 最小堆认为父节点比左右子节点的值小（或相等），但左右间没有要求。
- 插入插在最后面，进行swimup操作。就是关注最后节点，不断与父节点((i-1)//2处)比较的，不满足条件就交换。否则认为已经满足堆条件，敏捷退出。
- 取出从根节点取，用最后节点补上来，进行sindown操作。就是关注根节点，不断与子节点(2xi+1,2xi+2)处较小的比较，若比较小的大，需要与较小的交换，并且去那个位置继续sinkdown。否则敏捷退出。
- top-k若k较小：(1)可以考虑用一个固定容量的堆，数据流过堆(2)用固定桶是一样的(3)MR方法，在map中各取top-k，m路合并到reduce中取top-k。(4)利用快排的partition方法，partition够用了就在其中取得即可。(5)两路合并取top-k问题：两路各在k//2处，若相等，取出两路。若不等，取出一路，问题规模缩小了一半，继续重来。
- top-k若k较大，考虑全局排序(order by)：(1)胜者树：使用分块，块内排序，k路归并。（2）MR方法，预先扫描10000个数，根据这些随机抽样建立顺序100个分区，有上下界，设计key进入100个reducer中，reducer中在上下界内排序，有序取出这些reducer的结果。
- 堆排看起来像是快排和插排的结合体，根据历史信息进行插入，而且插的时候是二分的插。

5.Python实现堆和堆排

```python
class Heap:
    N_INSTANCE = 0  # num of Heap instances in the memory

    def __init__(self, data = None, heap_type="min"):
        """
        initialize a heap and store it in a list type
        :param data: initial value array, can be None
        :param heap_type: min Heap or max Heap, case insensitive
        """
        self._heap_type = heap_type.lower()
        if self._heap_type not in ("min", "max"): raise ValueError("error heap type")
        self._compare = Heap.compare_smaller if heap_type == "min" else Heap.compare_bigger  # function is object
        self.empty = self.is_empty  # alias entry
        self._storage = []  # 0-based database
        self.size = 0
        if data:
            [Heap.check_float(d) for d in data]  # data type check
            self.push_array(data)
        Heap.N_INSTANCE += 1  # initialized

    def swap(self, idx_i, idx_j):
        """
        exchange locations
        :param idx_i: index_1
        :param idx_j: index_2
        :return: void
        """
        self._storage[idx_i], self._storage[idx_j] = self._storage[idx_j], self._storage[idx_i]

    @staticmethod
    def check_float(x):
        """
        :param x: any type
        :return: legal digit or not
        """
        try:
            float(x)
        except:
            raise ValueError("incompatible data type")

    @staticmethod
    def compare_bigger(left, right):
        """
        operator for maxHeap
        :param left: left digital value
        :param right: right digital value
        :return: left value is bigger than the right value or not
        """
        return left > right

    @staticmethod
    def compare_smaller(left, right):
        """
        operator for minHeap
        :param left: left digital value
        :param right: right digital value
        :return: left value is smaller than the right value or not
        """
        return left < right

    def _swim_up(self):
        """
        balance the last node
        """
        node = self.size - 1
        while node > 0:
            parent = (node - 1) // 2  # notice that floor divide and the difference between Python2 and Python3
            if self._compare(self._storage[node], self._storage[parent]):
                self.swap(node, parent)
                node = parent
            else:
                break  # fail agilely

    def _sink_down(self):
        """
        balance the top node
        """
        node = 0
        while 1:  # 1 is better than True in Python
            child = node * 2 + 1
            if child >= self.size: break  # bound
            values = [(self._storage[child], child), ]
            child += 1
            if child < self.size:
                values.append((self._storage[child], child))
            child = values[0][1]
            if len(values) == 2 and self._compare(values[1][0], values[0][0]):
                # right child is smaller (minHeap) or bigger (maxHeap) than the left child
                child += 1
            if self._compare(self._storage[child], self._storage[node]):
                self.swap(node, child)
                node = child
            else:
                break  # fail agilely

    def push(self, x):
        """
        check one value, then push it into the heap
        :param x: digital(int,float,double,...)
        :return: 'success' or raise error
        """
        Heap.check_float(x)
        self._storage.append(x)
        self.size += 1
        if self.size > 1:
            self._swim_up()
        return "success"

    def push_array(self, xs):
        """
        push multiple values into the heap
        :param xs: digital array
        :return: 'success' or  raise error
        """
        [self.push(x) for x in xs]
        return "success"

    def top(self):
        """
        :return: get the top value of the heap instance
        """
        return self._storage[0] if self.size else "empty"

    def pop(self):
        """
        :return: get the top value, then remove it from heap
        """
        if self.size > 0:
            ret = self._storage[0]
            self._storage[0] = self._storage[-1]  # note the head
            self._storage.pop()
            self.size -= 1
            if self.size > 1: self._sink_down()
            return ret
        else:
            return "empty"

    def pop_all(self):
        """
        get all values out one time
        :return: one digital array
        """
        ret = []
        for i in range(self.size):
            ret.append(self.pop())
        return ret

    def reset_empty(self):
        """
        set heap to empty
        :return: 'success' or raise some error
        """
        self._storage = []
        self.size = 0
        return "success"

    def is_empty(self):
        """
        :return: heap is empty or not
        """
        return self.size == 0

    def print_layers(self):
        """
        only for debugging,show tree view
        """
        layers = []
        layer_size = 1
        start = 0
        end = start + layer_size
        while start < self.size:
            layers.append("+".join([str(si) for si in self._storage[start:end]]))
            start = end
            layer_size *= 2
            end = start + layer_size
        max_width = len(layers[-1])
        for i, layer in enumerate(layers):
            white = " " * ((max_width - len(layer)) // 2)
            layers[i] = white + layer + white
        layers.insert(0, "-" * 20 + "start" + "-" * 20)
        layers.append("-" * 20 + "end" + "-" * 20)
        print("\n".join(layers))

    @classmethod
    def heapsort(cls, data, reverse=False):
        """
        :param data: iterable digital array
        :param reverse: desc sort if set to True
        :return: sorted array
        """
        return Heap(data).pop_all() if not reverse \
            else Heap(data, "max").pop_all()

    def heapsort_with_key(self):
        """
        sort multi-columns
        """
        pass  # todo: require new data structure
```

分析：

- JAVA中有PriorityQueue，但是优先队列的实现不一定要用堆。也可以用无序数组，插入O(1)，取出O(N)。也可以用有序数值，插入O(N)，取出O(1)。堆是介于其中的。JAVA中的优先队列默认是最小堆，如果需要改成最大堆只需要实现Compartor的函数式接口。
- Python的函数可以有处于末尾的默认参数。Java的方法没有默认参数，可以通过重载实现。Python不支持重载。
- Python的Class中的普通方法是实例方法，可以被调用self，类方法是类方法，可以被调用cls，静态方法大家都可以调用。静态方法相当于JAVA中的static，ClassLoad载入，大家共用，静态方法不能调用非静态方法（psvm中调用String除外，因为String是JAVA设计者故意留的特例，String太有用了），原因是非静态方法还没有实例化还不存在无法调用，不然都不知道去哪个实例上去调。
- 构造时O(NlogN),查询顶端是O(1)的,移除顶端是O(logN)的。

#### 九、各种排序比较

|    算法    | 稳定性  | in-place |      时间复杂度       |   空间    |     备注      |
| :------: | :--: | :------: | :--------------: | :-----: | :---------: |
|  **桶排**  |      |          |       O(N)       |  O(N)   |             |
|   冒泡排序   |      |          |      O(N2)       |  O(1)   |             |
|   选择排序   |      |          |      O(N2)       |  O(1)   |             |
|   插入排序   |  稳定  |          |    O(N)~O(N2)    |  O(1)   |   输入有序线性    |
|   希尔排序   |      |          | O(NlogN)~O(N1.2) |  O(1)   |             |
|  **快排**  | 不稳定  |          |  O(NlogN)~O(N2)  | O(logN) | 输入有序、重复度平方级 |
|   三路快排   |      |          |  O(N)~O(NlogN)   | O(logN) |   无序度，重复度   |
| **归并排序** |  稳定  |  额外辅助空间  |     O(NlogN)     |  O(N)   |             |
|  **堆排**  |      |  额外辅助空间  |     O(Nlog)      |  O(1)   |             |

- 使用辅助空间和『外部排序』没有必然联系，外部排序是指支持使用外存的排序。
- 书上是说『插入排序』和『归并排序』两种排序是稳定（相等元素相对前后不变）的。但是其实不稳定是可以通过额外的开销来解决的，比如快排，认为双边扫描（右边扫过来替换时顺序可能发生变化，加入哨兵时可能顺序发生变化），但是可以引入辅助的索引之类的东西或者在单边扫描上使用额外空间来避免。尽信书不如无书。
- 这张表里的指标只是一个参考，具体的指标应该看具体的实现。
- 选择哪种排序方法应该考虑具体的数据分布，考虑接下来的使用情况。

### F、搜索Search

#### 一、概念

1.search hit， search miss

2.中序、前序、后序指的是根的情况。

3.实现要求：

- 泛型Generics
- 无序唯一 duplicate keys
- 覆盖行为  frozen
- key的类型（python中的dict要求hashable，列表是mutable的，可以用元组；java的有些实现不允许key为null，因为null有可能成为垃圾回收的标志；java的hashmap还假设了同一个字典里的key的类型是一致的，也方便迭代器）
- key的存在判断借助于equals方法

4.**符号表**SymbolTable

- Dictionary，Indices，Associative Array
- key-value对：associate a value with a key
- methods:insert,search,delete
- 查询效率O(1)

5.符号表实现

- sequential search，线性搜索，每次遍历，O(N)
- binary searh，先排序，O(logN)
- 链表（linked list），使用线性搜索，对于小型搜索是合适的，且插入删除方便
- 有序数组，可以使用二分查找，插入和删除效率低
- 二叉搜索树BST，有序且易实现，对空间没保证
- 平衡二叉树(Balanced BST)，红黑树（RedBlackTree，**TreeMap**），比BST平衡一些，且有序
- 哈希表（**HashMap**），查找O(1),空间开销大，且无序

#### 二、爆发式搜索

像钓鱼，钓不到的地方就换地方，能钓上鱼的地方就多钓(爆发，burst)。

#### 三、二叉搜索、二分查找、BinarySearch

效率：O(logN)

前提：数组有序。或者断点有序，比如旋转数组，可以先二分查找，再线性搜索。

思路：前后与折半处比较，调整前后范围，大往大找，小往小找。

```python
def binary_search(key,alist):
    alist = list(alist)  # 使用额外空间有几个好处，第一是不影响这个数据继续被其他线程使用，第二是不容易修改全局数据，第三是支持迭代器过来的流式数据
    left, right = 0, len(alist)-1 
    # 入口判断，若有大量范围外导致的 search miss 应该考虑加上这句『敏捷失败』
    if key < alist[left] or key > alist[right]: return -1
    mid = None  # 注意mid在其他语言中可能需要保持全局，但是在python中其实是可以在while之后将mid穿透到下面去的，并没有销毁
    while left <= right:  # 注意保证相遇处。依赖于mit+1和mid-1的更新式
        mid = left + (right-left) // 2  # 注意两个问题：Python2和Python3的除法略有区别，请使用这种无歧义的写法；
                                        # 第二个问题是溢出问题，其他语言可能考虑直接相加的溢出，Python基本没有这个问题，可以直接相加
        if alist[mid] == key: return mid
        elif alist[mid] > key: right = mid - 1  # 注意这种更新方式仅仅针对整数，若将二分查找用于凸函数的0点求值，可能要考虑无最小增量的更新式，但是带来了新的问题是while条件如果继续使用相等会不会陷入死循环，或许要额外增加判断条件或者使用一个最小增量
        else: left = mid + 1  # 同上
    if abs(alist[mid] - key) < 2: return mid  # 也接受估计结果，这样对于抽样之后进行搜索也是有好处的
    return -1  # 注意可以返回search miss 或者可以返回一个最接近的值left
```

- 一定要保证查询范围是有序的，否则输错了错误的结果。但是有一点可以确定是不会陷入死循环。
- 改成递归是简单的，或者可以在全局悬挂一个search hit 状态进行『敏捷成功』。但是不要去写递归，递归是低效的。
- 当然也还可能需要判断搜索范围的长度


#### 四、BST(Binary Search Tree) 二叉搜索树/二叉排序树

- using two links per node， 有left和right两个后继
- 每个节点实现了comparable接口，**所有节点值大于左子树，小于右子树**（类二叉搜索哨兵）
- size(x) = size(x.left) + size(x.right) + 1
- 插入查找复杂度O(logN)~O(N),当非常不平衡时，退化为顺序搜索。
- 删除有点困难，若没有子节点可以直接删除。若有子节点，需要在右子树中找到一个最小的值，替换掉这个根。
- BST树的**平衡性依赖于输入顺序的无序。**


注：平衡不是指所有叶节点的深度差在1以内，而是指两路的差在1以内，递归的子树的深度记为两路的较大值。

#### 五、平衡二叉树（Balanced Binary Tree）

1.典型的有AVL、2-3树、红黑树、B树。

特点：平衡，保证好的访问效率logN。若不平衡，有些地方过深导致访问效率不行。但需要注意的是插入删除时要保持平衡的代价也是比较高的。

2.2-3树

2-node onekey或者 3-node two keys

效率为O(logN/log2) ~O(logN/log3),有较好的插入删除效率。

3. **红黑树** Red-Black BSTs

- 定义1：基于2-3树转化为二叉树。
- 定义2：红链在左；一个节点最多连接一个红链；黑平衡，底部null-link回来的黑链数一致。
- 若把红链隐藏了，就成了2-3树。红黑树本质上是一棵2-3树。相当于结合了二叉搜索树的搜索效率与2-3树的插入效率。
- node中的一个color属性可以用于记录与parent直接是red还是black。
- **左旋右旋**，使其平衡：保持黑平衡，没有连续的红链，红链在左边（红黑性质），保持节点的大小顺序（BST新增）。通过left rotate，right rotate，color filp（若两个都是红色需要进行color flip）
- **性能：除了range search 外其他操作都是logN的，最差情况是2logN，红黑树的高度不会超过2logN。**

4.B数，B+树，B\*树  —> 数据库索引。

#### 六、哈希表（散列表）

查询是O(1)的，牺牲空间50%。无序的。

两个问题：

- 设计哈希函数。一是计算要快速，二是分桶要均匀。
- 冲突解决：拉链法或者线性探查法（开放地址法）。

具体的实现可以参考我的上一篇博客《哈希表、Bitmap与布隆过滤器》。

#### 七、应用符号表SymbolTable

- web中csv索引
- 文件逆序
- **稀疏矩阵存储**，sparse vector
- java.util.TreeMap 是红黑树，不支持rank，select
- java.util.HashMap是拉链法取模哈希，负载因子0.75.
- set类型：无序无重复，过滤filter，dedup，黑白名单。union，intersection，join。
- 字典：phonebook，account information
- indexing：transactions，web search， inverted index

### G、图算法

暂不做了解。