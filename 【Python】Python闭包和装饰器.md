---
date: 2017-02-19 12:53
status: public
title: 【Python】Python闭包和装饰器
---

## 一、闭包
- 提供一个函数执行的context。
```python
def wrapper(arg1):
    def func(arg2):
        print(arg1, arg2)
    return func
func1 = wrapper("hello")

def wrapper2(arg2, arg1):
    print(arg1, arg2)
func2 = partial(wrapper2, arg1="hello")

func1("dogless")
func2("dogless")
```
## 二、装饰器
- 提供一个context用于重复的功能，比如数据库的连接，关闭，以及错误处理。多个业务逻辑可以复用这个入口,比如func_a和func_b 可以共用一个Decorater_A。
- 把Decorater_A 换成Decorater_B的时候可以保持那些业务逻辑全部不变，从而进行最小更换，实现modularization。方便测试或者更换数据库接口。
- 控制函数的行为。

```python
# database
database_host="localhost"
def db_wrapper(fn):
    # 负责正确打开和关闭数据连接,提供句柄
    def call_func(*args,**kwargs):
        connection =Connection(database_host,autoconnect=True)
        response=fn(connection,*args,**kwargs)
        connection.close()
        return response
    return call_func

@db_wrapper
def create_table(connection,*args):
    connection.create(*args)

@db_wrapper
def drop_table(connection):
    pass
```
```python
# 带参装饰器，三层
def d2(arg1,arg2):
    def call_func(fn):
        def inner_call(*args):
            print 'before test',arg1,arg2
            fn(*args)
        return inner_call
    return call_func
@d2('a', 'b')
def test(arg1, arg2):
     print 'test', arg1, arg2
test('c', 'd')

[output]
before test a b
test c d
[/output]
```

[Github Link](https://github.com/Dogless-plus/Python_Note/blob/master/Python%E9%97%AD%E5%8C%85%E5%92%8C%E8%A3%85%E9%A5%B0%E5%99%A8.md)