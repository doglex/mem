---
date: 2017-02-20 16:26
status: public
title: 【小工具】用户级别Unix内存回收
---

# 需求描述

* 内存占用是**如何产生**的：通常情况下，系统在一个程序运行结束的时候会自动回收这个程序使用过的内存。然而，有时候并不能回收，比如流过的数据文件超级大的时候，unix会缓存这个数据。比如256G空闲内存，处理50G的数据，读入内存差不多占用了150G，程序结束的时候内存还是剩下106G，无法立即回收，而当第二次运行这个程序时，内存就不够用了，但是Unix很傻，并不知道去触发回收，程序就卡住了。而这150G的缓存，可能得几个小时，或者几天，才能触发回收。

* 通常情况下有如下的系统级回收，可以立即回收内存：
```
sudo su;
sync ;  echo 1 > /proc/sys/vm/drop_caches;   echo 2 > /proc/sys/vm/drop_caches;   echo 3 > /proc/sys/vm/drop_caches;
```
* **然而，上面这些命令需要root权限**，很多用户都没有管理员权限，或者是一些远程机器，更加没有root权限。 
* **需要一个用户级别的内存回收**

# 解决方案
- 原理：使用多个进程去抢占内存，被占用的内存就会慢慢的释放出来，然后结束这些进程，这部分抢过来的内存就可以别其他需要大内存的程序使用了。
- Python程序示例：
```python
#coding=utf-8

from multiprocessing import Pool
from os import getpid

def worker():
    a=list(range(200000000))
    b=list(range(200000000))
    for i in range(10):
        a.extend(b)

def memory_dog(loops=1000,cores=200):
    main_pid=getpid()
    print("Eating...wating for 1 min .step 1: press Ctrl +Z")
    print("step 2: enter\t kill -9 %s"%main_pid)

    pool=Pool(processes=cores)
    for i in range(loops):
            pool.apply_async(worker)
    pool.close()
    pool.join()
    print("all done!")

if __name__ == '__main__':
    memory_dog()
```
[Github Link](https://github.com/Dogless-plus/Light_Weapon/blob/master/release_memory.py)