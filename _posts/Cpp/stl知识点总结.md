---
title: STL知识点总结
date: 2020-12-05 19:37:37
categories: Cpp
tag: c++
---

# STL知识点总结

## STL六大组件：
* 容器
* 分配器
* 算法
* 迭代器
* 适配器
* 仿函数

## 容器

### 容器的分类
1. Sequence Containers(序列式容器)
    1. Array(c++11)
    2. Vector
    3. Deque（分段连续）
    4. List（双向链表）
    5. Forward-List(c++11)（单向链表）
    6. slist（非标准）
2. Associative Containers(关联式容器)
    1. Set/Multiset（红黑树）
    2. Map/Multimap（红黑树）(Multimap不能使用[]操作)
3. Unordered Containers(无序容器)(c++11)
    1. Unordered Set/Multiset（HashTable Separate Chaining）
    2. Unordered Map/Multimap（HashTable Separate Chaining）
    
#### list容器
实际上是一个**环状双向链表**：刻意在环状list尾部加一个空白节点，用以符合STL[前闭后开)区间

## 分配器
额外的分配器有：
```c++
#include <memeory> // 包含 std::allocator
// 非标准分配器
#include <ext\array_allocator.h>
#include <ext\mt_allocator.h>
#include <ext\debug_allocator.h>
#include <ext\pool_allocator.h>
#include <ext\bitmap_allocator.h>
#include <ext\malloc_allocator.h>
#include <ext\new_allocator.h>
```

## 算法

分为质变算法和非质变算法

## 迭代器

* 输入迭代器
* 输出迭代器
* 前向迭代器
* 双向迭代器
* 随机迭代器


## c++重要的资源
www.cplusplus.com
cppreference.com
gcc.gnu.org
