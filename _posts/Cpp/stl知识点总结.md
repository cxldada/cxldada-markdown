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

## 迭代器

### 迭代器类型

* 输入迭代器: input_iterator_tag
* 输出迭代器: output_iterator_tag
* 前向迭代器: forward_iterator_tag : public input_iterator_tag
* 双向迭代器: bidirectional_iterator_tag : public forward_iterator_tag
* 随机迭代器: random_access_iterator_tag : public bidirectional_iterator_tag

### 迭代器的特性
```c++
template<typename _Tp>
struct __List_iterator {
    typedef bidirectional_iterator iterator_category; // 迭代器分类
    typedef _Tp value_type;// 迭代器所指内容的类型
    typedef _Tp* pointer;// 迭代器分类
    typedef _Tp& reference;// 迭代器分类
    typedef ptrdiff_t difference_type;// 能够表示两个迭代器之间的距离，所使用的类型
};
```

### iterator_traits的作用
如果使用指针作为迭代器时，`algorithms`中的算法就无法使用`iterator`类来获取迭代器的相关属性。所以就有了`iterator_traits`这个中间件

`iterator_traits`用来分离`class iterator`和`non-class iterator`

```c++
template<class I>
struct iterator_traits {
    typedef typename I::value_type value_type;
    ...
};

// 两个偏特化来处理non-class iterator
template<class T>
struct iterator_traits<T*> {
    typedef random_access_iterator_tag iterator_category;
    typedef T value_type;
    ...
};
template<class I>
struct iterator_traits<const T*> {
    typedef random_access_iterator_tag iterator_category;
    typedef T value_type; // 注意不是const T
    ...
};
```

### 标准库中是的其他traits
* type traits: `.../C++/type_traits`
* iterator traits: `../C++/stl_traits`
* char traits: `../C++/char_traits`
* allocator traits: `../C++/alloc_traits`
* pointer traits: `../C++/ptr_traits`
* array traits: `../C++/array`

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
    
### list和forward_list容器
1. 实际上是一个**环状双向链表**：刻意在环状list尾部加一个空白节点，用以符合STL[前闭后开)区间
2. 迭代器是类

### vector

1. 内存空间是两倍增长
2. 新创建的vector可用空间为0
3. 每次空间成长都会大量调用元素的拷贝构造函数和析构函数，非常影响程序的效率。
4. G2.9版vector的迭代器使指针，而新版的迭代器是类。

```c++
// 大概的类声明
// G2.9版
template <class T, class Alloc = alooc>
class vector {
public:
    typedef T value_type;
    typedef value_type* iterator;
    typedef value_type& reference;
    typedef size_t size_type;
protected:
    iterator start;
    iterator finish;
    iterator end_of_storage;
public:
};
```

### array容器
1. c++11新加入的容器
2. 为了能够让标准库的算法和功能能够像使用其他容器一样使用数组而引入的
3. 不可以扩充容量
4. 迭代器是指针

```c++
// array大概类型声明
// TR1版本
template <typename _Tp, std::size_t _Nm>
struct array {
    typedef _Tp value_type;
    typedef _Tp* pointer;
    typedef value_type* iterator;

    value_type _M_instance[_Nm ? _Nm : 1];
    ...
};
```

### deque容器

1. 读作：'戴克'
2. 容器本身并不是连续空间，而是使用分段连续的方式来模拟连续空间
3. `insert`操作会判断离那一端比较近，进而向那一端插入
4. deque模拟连续空间，主要是通过迭代器的重载操作符完成的。见下面的代码
5. 要扩充缓冲区节点大小的时候，原来缓冲区节点中的内容会被拷贝到新缓冲区节点的中段，为了方便向两边扩展

```c++
// dequp大概声明
// G2.9版本
template <class T, class Alloc = alloc, size_t BufSiz = 0>
class deque {
public:
    typedef T value_type;
    typedef __deque_iterator<T,T&,T*,BufSiz> iterator;
protected:
    typedef pointer* map_pointer; // T**
protected:
    iterator start;
    iterator start;
    map_pointer map;
    size_type map_size;
};

// deque迭代器的大概声明
// G2.9版本
template <class T, class Ref, class Ptr, size_t BufSiz>
struct __deque_iterator {
    typedef random_access_iterator_tag iterator_category;
    typedef T value_type;
    typedef Ptr pointer;
    typedef Ref reference;
    typedef size_t size_type;
    typedef ptrdiff_t difference_type;
    typedef T** map_pointer;
    typedef __deque_iterator self;

    T* cur;
    T* first;
    T* last;
    map_pointer node;
    ...
};
inline size_t __duque_buf_size(size_t n, size_t sz) {
    return n != 0 ? n : (sz < 512 ? size_t(512 / sz) : size_t(1))
}

// 迭代器相关的重载
reference operator*()const {
    return *cur;
}

reference operator->()const {
    return &(operator*());
}

// 两个迭代器之间的buff总长度 + 起始缓冲区的长度 + 结束缓冲区的长度
difference_type operator-(const self& x) const {
    return difference_type(buffer_size()) * (node - x.node - 1) + (cur - first) + (x.last - x.cur);
}

void set_node(map_pointer new_node) {
    node = new_node;
    first = *new_node;
    last = first + difference_type(buffer_size());
}

self& operator++() {
    ++cur;
    if(cur == last) {
        set_node(node + 1);
        cur = first;
    }
    return *this;
}

// 好的写法是后++调用前++
self operator++(int) {
    self tmp = *this;
    ++*this;
    return tmp;
}

self& operator--() {
    if(cur == first) {
        set_node(node - 1);
        cur = last;
    }
    --cur;
    return *this;
}

// 好的写法是后--调用前--
self operator--(int) {
    self tmp = *this; --this;
    return tmp;
}

self& operator+=(difference_type n) {
    difference_type offset = n + (cur - first);
    if(offset >= 0 && offset < difference_type(buffer_size()))
        cur += n;
    else {
        // 计算要跳几个缓冲区节点
        difference_type node_offset = offset > 0
                                        ? offset / difference_type(buffer_size())
                                        : -difference_type((-offset - 1) / buffer_size()) -1;

        set_node(node + node_offset);
        cur = first + (offset - node_offset * difference_type(buffer_size()));
    }
}

self operator+(difference_type n) const {
    self tmp = *this;
    return tmp += n;
}

self& operator-=(difference_type n) {
    return *this += -n;
}

self operator-(difference_type n) const {
    self tmp = *this;
    return tmp -= n;
}

reference operator[](difference_type n) {
    return *(*this + n);
}
```

### queue和stack容器

1. 两者都不算是全新设计的容器，底层容器是deque容器，只是在此基础上添加了一些限制
2. 两者都可以指定底层容器，默认为deque，也可以指定为list。（deque会快一些）
   1. stack还可以选择vector做为底层容器，queue不可以。因为vector没有在头部加入元素的方法

### rb_tree容器

1. map和set底层使用的是rb_tree容器

```c++
// G2.9版
template<class Key, class Value, class KeyOfValue, class Compare, class Alloc = alloc>
class rb_tree {
protected:
    typedef __rb_tree_node<Value> rb_tree_node;
    ...
public:
    typedef rb_tree_node* link_type;
    ...
protected:
    size_type node_count; // rb_tree的大小(节点的数量)
    link_type header;
    Compare key_compare; // key的大小比较函数; 应该是一个function object
};
```
### set和multiset容器

1. 底层是rb_tree容器
2. set的key不能重复，multiset可以重复
3. 不能通过迭代器修改data

```c++
// G2.9
template<class Key, class Compare = less<key>, class Alloc = alloc>
class set {
public:
    typedef Key key_type;
    typedef Key value_type;
    typedef Compare key_compare;
    typedef Compare value_compare;

private:
    typedef rb_tree<key_type, value_type, identity<value_type>, key_compare, Alloc> rep_type;
    rep_type t;

public:
    // 通过const iterator来限制修改数据
    typedef typename rep_type::const_iterator iterator;
    ...
};
```

### map和multimap容器

1. 底层也是rb_tree容器
2. 不能修改key，但可以修改value
3. map有独特的`operator[]`重载

```c++
//G2.9
template<class Key, class T, class Compare = less<Key>, class Alloc = alloc>
class map {
public:
    typedef Key key_type;
    typedef T value_type;
    typedef T mapped_type;
    typedef pair<const Key, T> value_type; // 把Key改成const用来限制不能修改Key;
    typedef Compare key_compare;
private:
    typedef rb_tree<key_type, value_type,select1st<value_type>, key_compare, Alloc> rep_type;
    rep_type t;
public:
    typedef typename rep_type::iterator iterator;
    ...
};
```

### hashtable容器

1. 如果链表的元素个数超过了hash表的大小，则hash表的大小扩大两倍并rehash
2. GNU版本提供了hash函数，可以根据需求使用
3. Separate Chaining

```c++
//G2.9
template<class Value, class Key, class HashFcn, class ExtractKey, class EqualKey, class Alloc = alloc>
class hashtable {
public:
    typedef HashFcn hasher;
    typedef EqualKey key_equal;
    typedef size_t size_type;
private:
    hasher hash;
    key_equal equals;
    ExtractKey get_key;

    typedef __hashtable_node<Value> node;

    vector<node*,Alloc> buckets;
    size_type num_elements;
public:
    size_type bucket_count() const { return buckets.size(); }
};

template <class Value>
struct __hashtable_node {
    __hashtable_node* next;
    Value val;
};
```

### unordered_set和unordered_map容器

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

1. 分为质变算法和非质变算法
2. 算法需要知道迭代器的类型
3. 如果容器提供了同名的算法函数，应该使用容器本身的算法

## 仿函数

1. 要重载括号运算符
2. 要继承`unary_function`或`bianry_function`才能融入STL体系中
3. 如果自定义的仿函数没有融入STL体系中，适配器在使用时会报错，因为适配器取不到相应的参数

## 适配器

1. 容器适配器：stack、queue
2. 函数适配器: binder2nd

### bind适配器

1. 可以绑定：
   1. functions
   2. function objects
   3. member functions: _1必须是某个object地址
   4. data members: _1必须是某个object地址

### 迭代器适配器

1. reverse_iterator
2. inserter

## 除STL外的其他标准库方法

### 万能的hash function
```c++
// TR1版本
template<typename... Types>
inline size_t hash_val(const Type&... args) {
    size_t seed = 0;
    hash_val(seed, args...);
    return seed;
}
```

### tuple(元祖): 利用c++11的多模板参数特性，实现递归继承

### type traits: 利用模板的泛化和偏特化实现的

### mobeable: 只复制指针，不复制指向的内容，并把源对象的指针赋空。这就避免了分配空间

## c++重要的资源
www.cplusplus.com
cppreference.com
gcc.gnu.org
