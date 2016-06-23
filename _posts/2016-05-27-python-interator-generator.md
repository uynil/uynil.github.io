---
layout: post
title: Python Iterator Generator 迭代器 生成器
tags: [Python]
---
1. 迭代器 iterator
2. 生成器 generator

#### 1. iterator
定义
``` python
class ITE(object):
    def __init__(self):
        self.next = 0
        self.val = 1
    #必须
    def __iter__(self):
        return self.val

    #必须
    def next(self):
        return self.next

    #不是必须
    def hasNext():
        return self.next != 0
``` python

优点
    对于无法随机访问的数据结构，迭代器是唯一的访问元素的方式。
    不必事前准备好所有元素的内存。每次迭代到当前元素，占用内存，而在这之前，可以不存在或者销毁比较适合遍历一些比较大的集合
    提供了一个统一访问的接口。定义__iter__(), 以及next()

#### 2. 生成器
定义
    包含yield 语句的函数
    函数执行到yield语句，并停留在yield状态。下次执行接着在yield语句继续执行。
优点
    可以带有函数的简洁，并实现迭代器的功能

### 参考文章
1. [Yield](http://www.ibm.com/developerworks/cn/opensource/os-cn-python-yield/)

