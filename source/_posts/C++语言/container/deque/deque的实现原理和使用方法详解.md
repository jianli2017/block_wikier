---
title: deque的实现原理和使用方法详解
date: 2018-06-26 10:07:12
tags: deque
categories: C++
toc: true
---

本文抄袭至[STL源码剖析——deque的实现原理和使用方法详解](https://blog.csdn.net/baidu_28312631/article/details/48000123)。只用于本人学习。

<!--more-->

## Deque 简介

1. deque是“double—ended queue”的缩写，和vector一样都是STL的容器，deque 是双端数组，而 vector 是单端的。
2. deque 在接口上和 vector 非常相似，在许多操作的地方可以直接替换。
3. deque 可以随机存取元素（支持索引值直接存取，用[]操作符或at()方法，这个等下会详讲）。
4. deque 头部和尾部添加或移除元素都非常快速。但是在中间插入元素或移除元素比较费时。
5. 使用时需要包含头文件 #include<deque> 。

## Deque 实现原理

### deque 的中控器

deque是连续空间（至少逻辑上看来如此），连续线性空间总令我们联想到array或vector。**array无法成长，vector虽可成长**，却只能向尾端成长，而且其所谓的成长原是个假象，事实上是<font color = blue>**（1）另觅更大空间；（2）将原数据复制过去；（3）释放原空间三部曲。**</font>如果不是vector每次配置新空间时都有留下一些余裕，其成长假象所带来的代价将是相当高昂。

<font color=blue>**deque系由一段一段的定量连续空间构成。**</font>一旦有必要在deque的前端或尾端增加新空间，便配置一段定量连续空间，串接在整个deque的头端或尾端。deque的最大任务，便是在这些分段的定量连续空间上，维护其整体连续的假象，并提供随机存取的接口。避开了“重新配置、复制、释放”的轮回，代价则是复杂的迭代器架构。

受到分段连续线性空间的字面影响，我们可能以为deque的实现复杂度和vector相比虽不中亦不远矣，其实不然。主要因为，既是分段连续线性空间，就必须有中央控制，而为了维持整体连续的假象，数据结构的设计及迭代器前进后退等操作都颇为繁琐。deque的实现代码分量远比vector或list都多得多。

<font color = blue>**deque采用一块所谓的map作为主控**。这里所谓map是一小块连续空间，其中每个元素（此处称为一个节点，node）都是指针，**指向另一段（较大的）连续线性空间，称为缓冲区**。**缓冲区才是deque的储存空间主体**。SGI STL 允许我们指定缓冲区大小，默认值0表示将使用512 bytes 缓冲区。</font>

deque的整体架构如下图所示：

![deque的整体架构](http://of685p9vy.bkt.clouddn.com/C++/deque/dequeStruct.png)

### deque 的迭代器

让我们思考一下，deque的迭代器应该具备什么结构，首先，它必须能够指出分段连续空间（亦即缓冲区）在哪里，其次它必须能够判断自己是否已经处于其所在缓冲区的边缘，如果是，一旦前进或后退就必须跳跃至下一个或上一个缓冲区。为了能够正确跳跃，deque必须随时掌握管控中心（map）。所以在迭代器中需要定义：当前元素的指针，当前元素所在缓冲区的起始指针，当前元素所在缓冲区的尾指针，指向map中指向所在缓区地址的指针，分别为cur, first, last, node。

指针结构如下图所示：

![指针结构](http://of685p9vy.bkt.clouddn.com/C++/deque/dequePointer.png)
    

在上面介绍中我们大致了解了deque 的基本概念和实现原理，现在我就开始介绍如何使用 deque。

## deque 使用

### deque 对象的默认构造

deque 采用模板类实现，deque 对象的默认构造形式：deque<T> dequeT；   

```
deque<int> deqInt;            //一个存放int的deque容器。  
deque<float> deqFloat;       //一个存放float的deque容器。  
deque<string> deqString;     //一个存放string的deque容器。  
... 
```
尖括号内还可以设置指针类型或自定义类型。

### deque 元素添加移除操作

```
deque.push_back(elem);       //在容器尾部添加一个数据  
deque.push_front(elem);     //在容器头部插入一个数据  
deque.pop_back();           //删除容器最后一个数据  
deque.pop_front();          //删除容器第一个数据  
```

### deque 的数据存取  

```
deque.at(idx);    //返回索引idx所指的数据，如果idx越界，抛出out_of_range。  
deque[idx];      //返回索引idx所指的数据，如果idx越界，不抛出异常，直接出错。  
deque.front();   //返回第一个数据。  
deque.back();    //返回最后一个数据  
```

### deque 与迭代器 

```
deque.begin();  //返回容器中第一个元素的迭代器。  
deque.end();  //返回容器中最后一个元素之后的迭代器。  
deque.rbegin();  //返回容器中倒数第一个元素的迭代器。  
deque.rend();   //返回容器中倒数最后一个元素之后的迭代器。  
```

### deque 对象的带参数构造  

```
deque(beg,end);    //构造函数将[beg, end)区间中的元素拷贝给本身。注意该区间是左闭右开的区间。  
deque(n,elem);   //构造函数将n个elem拷贝给本身。  
deque(const deque &deq);  //拷贝构造函数。  
```
### deque 的赋值
```
deque.assign(beg,end);   //将[beg, end)区间中的数据拷贝赋值给本身。注意该区间是左闭右开的区间。  
deque.assign(n,elem);  //将n个elem拷贝赋值给本身。  
deque& operator=(const deque &deq); //重载等号操作符   
deque.swap(deq);  // 将deq与本身的元素互换  
```
```
deque<int> deqIntA,deqIntB,deqIntC,deqIntD;  
deqIntA.push_back(1);  
deqIntA.push_back(3);  
deqIntA.push_back(5);  
deqIntA.push_back(7);  
deqIntA.push_back(9);  
  
deqIntB.assign(deqIntA.begin(),deqIntA.end());  // 1 3 5 7 9  
      
deqIntC.assign(5,8);        //8 8 8 8 8  
  
deqIntD = deqIntA;      //1 3 5 7 9  
  
deqIntC.swap(deqIntD);      //互换  
```
### deque 的大小

```
deque.size();      //返回容器中元素的个数  
deque.empty();     //判断容器是否为空  
deque.resize(num);   //重新指定容器的长度为num，若容器变长，则以默认值填充新位置。  
                    //如果容器变短，则末尾超出容器长度的元素被删除。     
deque.resize(num, elem);  //重新指定容器的长度为num，若容器变长，则以elem值填充新位置。  
                        //如果容器变短，则末尾超出容器长度的元素被删除。                  
```

### deque 的插入

```
deque.insert(pos,elem);   //在pos位置插入一个elem元素的拷贝，返回新数据的位置。  
deque.insert(pos,n,elem);   //在pos位置插入n个elem数据，无返回值。  
deque.insert(pos,beg,end);   //在pos位置插入[beg,end)区间的数据，无返回值。  
```

### deque 的删除  

```
deque.clear();  //移除容器的所有数据  
deque.erase(beg,end);  //删除[beg,end)区间的数据，返回下一个数据的位置。  
deque.erase(pos);    //删除pos位置的数据，返回下一个数据的位置。  
```

## 源码

 让我们思考一下，deque的迭代器应该具备什么结构，首先，它必须能够指出分段连续空间（亦即缓冲区）在哪里，其次它必须能够判断自己是否已经处于其所在缓冲区的边缘，如果是，一旦前进或后退就必须跳跃至下一个或上一个缓冲区。为了能够正确跳跃，deque必须随时掌握管控中心（map）。所以在迭代器中需要定义：当前元素的指针，当前元素所在缓冲区的起始指针，当前元素所在缓冲区的尾指针，指向map中指向所在缓区地址的指针。
在进行迭代器的移动时，需要考虑跨缓冲区的情况。

重载前加(减)，在实现后加(减)时，调用重载的前加(减)。
重载+=,实现+时，直接调用+=,实现-=时，调用+=负数，实现-时，调用-=.
//当需要实现新的功能时，最好使用已经重载好的操作，即方便有安全。。。。

另外，deque在效率上来说是不够vector好的，因此有时候在对deque进行sort的时候，需要先将元素移到vector再进行sort，然后移回来。

构造函数：根据缓冲区设置大小和元素个数，决定map的大小；给map分配空间，根据缓冲区的个数，分配缓冲区，默认指定一个缓冲区；
 设置start和finish迭代器，满足左闭右开的原则。
 push_back:如果空间满足，直接插入；不满足，调用push_back_aux。
 push_back_aux:先调用reverse_map_at_back,若符合某种条件，重换一个map；分配空间。
 reserve_map_at_back:看看map有没有满，满的话，调用reallocate_map。
 reallocate_map:如果前端或后端pop过多，就会导致大量的空闲空间，如果是这种情况，则不用新分配空间，调整一下start的位置即可；
 如果不够，则需要重新申请空间。
 pop：析构元素，如果是最后一块还需要删除空间。
 erase：需要判断，前面的元素少还是后面的元素少，移动较少的部分。
 insert：判断位置，如果为前端或后端直接调用push操作，否则，移动较少的一端。
 
deque的构造与内存管理：

由于deque的设计思想就是由一块块的缓存区连接起来的，因此它的内存管理会比较复杂。插入的时候要考虑是否要跳转缓存区、是否要新建map节点（和vector一样，其实是重新分配一块空间给map，删除原来空间）、插入后元素是前面元素向前移动还是后面元素向后面移动（谁小移动谁）。而在删除元素的时候，考虑是将前面元素后移覆盖需要移除元素的地方还是后面元素前移覆盖（谁小移动谁）。移动完以后要析构冗余的元素，释放冗余的缓存区。


```
//   deque的特性:  
//   对于任何一个迭代器i  
//     i.node是map array中的某元素的地址. i.node的内容是一个指向某个结点的头的指针  
//     i.first == *(i.node)  
//     i.last  == i.first + node_size  
//     i.cur是一个指向[i.first, i.last)之间的指针  
//       注意: 这意味着i.cur永远是一个可以解引用的指针,  
//            即使其是一个指向结尾后元素的迭代器  
//  
//   起点和终点总是非奇异(nonsingular)的迭代器.  
//     注意: 这意味着空deque一定有一个node, 而一个具有N个元素的deque  
//          (N是Buffer Size)一定有有两个nodes  
//  
//   对于除了start.node和finish.node之外的每一个node, 每一个node中的元素  
//   都是一个初始化过的对象. 如果start.node == finish.node,  
//   那么[start.cur, finish.cur)都是未初始化的空间.  
//   否则, [start.cur, start.last)和[finish.first, finish.cur)都是初始化的对象,  
//   而[start.first, start.cur)和[finish.cur, finish.last)是未初始化的空间  
//  
//   [map, map + map_size)是一个合法的非空区间  
//   [start.node, finish.node]是内含在[map, map + map_size)区间的合法区间  
//   一个在[map, map + map_size)区间内的指针指向一个分配过的node,  
//   当且仅当此指针在[start.node, finish.node]区间内  
  
inline size_t __deque_buf_size(size_t n, size_t sz)    
{    
  return n != 0 ? n : (sz < 512 ? size_t(512 / sz) : size_t(1));    
}  
  
// __deque_iterator的数据结构  
template <class T, class Ref, class Ptr, size_t BufSiz>  
struct __deque_iterator  
{  
    typedef __deque_iterator<T, T&, T*>             iterator;  
    typedef __deque_iterator<T, const T&, const T*> const_iterator;  
    static size_t buffer_size() {return __deque_buf_size(0, sizeof(T)); }  
  
    typedef random_access_iterator_tag iterator_category;  
    typedef T value_type;  
    typedef Ptr pointer;  
    typedef Ref reference;  
    typedef size_t size_type;  
    typedef ptrdiff_t difference_type;  
    typedef T** map_pointer;  
  
    typedef __deque_iterator self;  
  
    // 保持与容器的联结  
    T* cur;       // 此迭代器所指之缓冲区中的现行元素  
    T* first;     // 此迭代器所指之缓冲区的头  
    T* last;      // 此迭代器所指之缓冲区的尾（含备用空间）  
    map_pointer node;    // 指向管控中心  
  
////////////////////////////////////////////////////////////////////////////////  
// 这个是deque内存管理的关键, 其模型如下  
////////////////////////////////////////////////////////////////////////////////  
//  
//       ---------------------------------------------  
// map-->|   |   |   |   |   |   | ..... |   |   |   |<------------------  
//       ---------------------------------------------                  |  
//             |                                                        |  
//             |                                                        |  
//             |   node                                                 |  
//             |   缓冲区buffer, 这里实际存储元素                          |  
//             |   ---------------------------------------------        |  
//             --->|   |   |   |   |   |   | ..... |   |   | X |        |  
//                 ---------------------------------------------        |  
//                   ↑       ↑                             ↑            |  
//             ------        |                             |            |  
//             |             |                             |            |  
//             |   -----------   ---------------------------            |  
//             ----|-----        |                                      |  
//                 |    |        |                                      |  
//                 |    |        |                                      |  
//                 |    |        |                                      |  
//              ---------------------------                             |  
//              | cur | first | end | map |------------------------------  
//              ---------------------------  
//              迭代器, 其内部维护着一个缓冲区状态  
////////////////////////////////////////////////////////////////////////////////  
    __deque_iterator(T* x, map_pointer y)  
        : cur(x), first(*y), last(*y + buffer_size()), node(y) {}  
    __deque_iterator() : cur(0), first(0), last(0), node(0) {}  
    __deque_iterator(const iterator& x)  
        : cur(x.cur), first(x.first), last(x.last), node(x.node) {}  
  
    reference operator*() const { return *cur; }  
  
    // 判断两个迭代器间的距离  
    difference_type operator-(const self& x) const  
    {  
        return difference_type(buffer_size()) * (node - x.node - 1) +  
            (cur - first) + (x.last - x.cur);  
    }  
  
////////////////////////////////////////////////////////////////////////////////  
// 下面重载的这些运算符是让deque从外界看上去维护的是一段连续空间的关键!!!  
  
// 前缀自增  
////////////////////////////////////////////////////////////////////////////////  
// 如果当前迭代器指向元素是当前缓冲区的最后一个元素,  
// 则将迭代器状态调整为下一个缓冲区的第一个元素  
////////////////////////////////////////////////////////////////////////////////  
// 不是当前缓冲区最后一个元素  
//  
// 执行前缀自增前的状态  
// first          cur                     end  
// ↓               ↓                       ↓  
// ---------------------------------------------  
// |   |   |   |   |   |   | ..... |   |   | X | <----- 当前缓冲区  
// ---------------------------------------------  
//  
// 执行完成后的状态  
// first              cur                 end  
// ↓                   ↓                   ↓  
// ---------------------------------------------  
// |   |   |   |   |   |   | ..... |   |   | X | <----- 当前缓冲区  
// ---------------------------------------------  
//  
////////////////////////////////////////////////////////////////////////////////  
// 当前元素为当前缓冲区的最后一个元素  
//  
// 执行前缀自增前的状态  
// first                              cur end  
// ↓                                   ↓   ↓  
// ---------------------------------------------  
// |   |   |   |   |   |   | ..... |   |   | X | <----- 当前缓冲区  
// ---------------------------------------------  
//  
// 执行完成后的状态  
// first                                  end  
// ↓                                       ↓  
// ---------------------------------------------  
// |   |   |   |   |   |   | ..... |   |   | X | <----- 下一缓冲区  
// ---------------------------------------------  
// ↑  
// cur  
//  
////////////////////////////////////////////////////////////////////////////////  
    self& operator++()  
    {  
        ++cur;    // 切换至下一个元素  
        if (cur == last)    // 如果已达到缓冲区的尾端  
        {  
            set_node(node + 1);    // 就切换至下一节点（亦即缓冲区）  
            cur = first;           // 的第一个元素  
        }  
        return *this;  
    }  
  
    // 后缀自增  
    // 返回当前迭代器的一个副本, 并调用前缀自增运算符实现迭代器自身的自增  
    self operator++(int)  
    {  
        self tmp = *this;  
        ++*this;  
        return tmp;  
    }  
  
    // 前缀自减, 处理方式类似于前缀自增  
    // 如果当前迭代器指向元素是当前缓冲区的第一个元素  
    // 则将迭代器状态调整为前一个缓冲区的最后一个元素  
    self& operator--()  
    {  
        if (cur == first)    // 如果已达到缓冲区的头端  
        {  
            set_node(node - 1);    // 就切换至前一节点（亦即缓冲区）  
            cur = last;            // 的最后一个元素  
        }  
        --cur;  
        return *this;  
    }  
  
    self operator--(int)  
    {  
        self tmp = *this;  
        --*this;  
        return tmp;  
    }  
  
////////////////////////////////////////////////////////////////////////////////  
// 将迭代器向前移动n个元素, n可以为负  
////////////////////////////////////////////////////////////////////////////////  
//                     operator+=(difference_type n)  
//                                   ↓  
//                      offset = n + (cur - first)  
//                                   |  
//                                   |---------- offset > 0 ? &&  
//                                   |           移动后是否超出当前缓冲区?  
//               ----------------------------  
//           No  |                          |  Yes  
//               |                          |  
//               ↓                          |---------- offset > 0?  
//           cur += n;                      |  
//                              ----------------------------  
//                          Yes |                          | No  
//                              |                          |  
//                              ↓                          |  
//                   计算要向后移动多少个缓冲区                |  
//                   node_offset =                         |  
//                   offset / difference_type              |  
//                   (buffer_size());                      ↓  
//                              |           计算要向前移动多少个缓冲区  
//                              |           node_offset = -difference_type  
//                              |           ((-offset - 1) / buffer_size()) - 1;  
//                              |                          |  
//                              ----------------------------  
//                                           |  
//                                           |  
//                                           ↓  
//                                       调整缓冲区  
//                              set_node(node + node_offset);  
//                                    计算并调整cur指针  
////////////////////////////////////////////////////////////////////////////////  
    // 以下实现随机存取。迭代器可以直接跳跃n个距离  
    self& operator+=(difference_type n)  
    {  
        difference_type offset = n + (cur - first);  
        if (offset >= 0 && offset < difference_type(buffer_size()))  
            cur += n;        // 目标位置在同一缓冲区内  
        else  
        {           // 目标位置不在同一缓冲区内  
            difference_type node_offset =  
                offset > 0 ? offset / difference_type(buffer_size())  
                : -difference_type((-offset - 1) / buffer_size()) - 1;  
            // 切换至正确的节点（亦即缓冲区）  
            set_node(node + node_offset);  
            // 切换至正确的元素  
            cur = first + (offset - node_offset * difference_type(buffer_size()));  
        }  
        return *this;  
    }  
  
    self operator+(difference_type n) const  
    {  
        self tmp = *this;  
  
        // 这里调用了operator +=()可以自动调整指针状态  
        return tmp += n;  
    }  
  
    // 将n变为-n就可以使用operator +=()了,  
    self& operator-=(difference_type n) { return *this += -n; }  
  
    self operator-(difference_type n) const  
    {  
        self tmp = *this;  
        return tmp -= n;  
    }  
  
    reference operator[](difference_type n) const { return *(*this + n); }  
  
    bool operator==(const self& x) const { return cur == x.cur; }  
    bool operator!=(const self& x) const { return !(*this == x); }  
    bool operator<(const self& x) const  
    {  
        return (node == x.node) ? (cur < x.cur) : (node < x.node);  
    }  
  
    void set_node(map_pointer new_node)  
    {  
        node = new_node;  
        first = *new_node;  
        last = first + difference_type(buffer_size());  
    }  
};  
  
  
// deque的数据结构  
template <class T, class Alloc = alloc, size_t BufSiz = 0>  
class deque  
{  
public:                         // Basic types  
    typedef T value_type;  
    typedef value_type* pointer;  
    typedef value_type& reference;  
    typedef size_t size_type;  
    typedef ptrdiff_t difference_type;  
  
public:                         // Iterators  
    typedef __deque_iterator<T, T&, T*, BufSiz>       iterator;  
  
protected:                      // Internal typedefs  
  
    typedef pointer* map_pointer;  
  
    // 这个提供STL标准的allocator接口, 见<stl_alloc.h>  
    typedef simple_alloc<value_type, Alloc> data_allocator;  
    typedef simple_alloc<pointer, Alloc> map_allocator;  
  
    // 获取缓冲区最大存储元素数量  
    static size_type buffer_size()  
    {  
        return __deque_buf_size(BufSiz, sizeof(value_type));  
    }  
  
    static size_type initial_map_size() { return 8; }  
  
protected:                      // Data members  
    iterator start;               // 起始缓冲区  
    iterator finish;              // 最后一个缓冲区  
  
    // 指向map, map是一个连续的空间, 其每个元素都是一个指针，指向一个节点（缓冲区）  
    map_pointer map;  
    size_type map_size;   // map容量  
  
public:  
    iterator begin() { return start; }  
    iterator end() { return finish; }  
  
    // 提供随机访问能力, 其调用的是迭代器重载的operator []  
    // 其实际地址需要进行一些列的计算, 效率有损失  
    reference operator[](size_type n) { return start[difference_type(n)]; }  
  
    reference front() { return *start; }  
    reference back()  
    {  
        iterator tmp = finish;  
        --tmp;  
        return *tmp;  
    }  
  
    // 当前容器拥有的元素个数, 调用迭代器重载的operator -  
    size_type size() const { return finish - start;; }  
    size_type max_size() const { return size_type(-1); }  
  
    // deque为空的时, 只有一个缓冲区  
    bool empty() const { return finish == start; }  
  
public:                         // Constructor, destructor.  
    deque() : start(), finish(), map(0), map_size(0)  
    {  
        create_map_and_nodes(0);  
    }  
  
    deque(size_type n, const value_type& value)  
        : start(), finish(), map(0), map_size(0)  
    {  
        fill_initialize(n, value);  
    }  
  
    deque(int n, const value_type& value)  
        : start(), finish(), map(0), map_size(0)  
    {  
        fill_initialize(n, value);  
    }  
  
  
    ~deque()  
    {  
        destroy(start, finish);     // <stl_construct.h>  
        destroy_map_and_nodes();  
    }  
  
    deque& operator= (const deque& x)  
    {  
        // 其实我觉得把这个操作放在if内效率更高  
        const size_type len = size();  
        if (&x != this)  
        {  
            // 当前容器比x容器拥有元素多, 析构多余元素  
            if (len >= x.size())  
                erase(copy(x.begin(), x.end(), start), finish);  
            // 将x所有超出部分的元素使用insert()追加进去  
            else {  
                const_iterator mid = x.begin() + difference_type(len);  
                copy(x.begin(), mid, start);  
                insert(finish, mid, x.end());  
            }  
        }  
        return *this;  
    }  
  
public:  
    void push_back(const value_type& t)  
    {  
        // 最后缓冲区尚有两个（含）以上的元素备用空间  
        if (finish.cur != finish.last - 1)  
        {  
            construct(finish.cur, t);     // 直接在备用空间上构造元素  
            ++finish.cur;     // 调整最后缓冲区的使用状态  
        }  
        // 容量已满就要新申请内存了  
        else  
            push_back_aux(t);  
    }  
  
    void push_front(const value_type& t)  
    {  
        if (start.cur != start.first)      // 第一缓冲区尚有备用空间  
        {  
            construct(start.cur - 1, t);   // 直接在备用空间上构造元素  
            --start.cur;     // 调整第一缓冲区的使用状态  
        }  
        else    // 第一缓冲区已无备用空间  
            push_front_aux(t);  
    }  
  
    void pop_back()  
    {  
        if (finish.cur != finish.first)    // 最后缓冲区有一个（或更多）元素  
        {  
            --finish.cur;    // 调整指针，相当于排除了最后元素  
            destroy(finish.cur);    // 将最后元素析构  
        }  
        else  
            // 最后缓冲区没有任何元素  
            pop_back_aux();    // 这里将进行缓冲区的释放工作  
    }  
  
    void pop_front()  
    {  
        if (start.cur != start.last - 1)    // 第一缓冲区有两个（或更多）元素  
        {  
            destroy(start.cur);    // 将第一元素析构  
            ++start.cur;           //调整指针，相当于排除了第一元素  
        }  
        else  
            // 第一缓冲区仅有一个元素  
            pop_front_aux();    // 这里将进行缓冲区的释放工作  
    }  
  
public:                         // Insert  
  
////////////////////////////////////////////////////////////////////////////////  
// 在指定位置前插入元素  
////////////////////////////////////////////////////////////////////////////////  
//             insert(iterator position, const value_type& x)  
//                                   |  
//                                   |---------------- 判断插入位置  
//                                   |  
//               -----------------------------------------------  
// deque.begin() |          deque.emd() |                      |  
//               |                      |                      |  
//               ↓                      ↓                      |  
//         push_front(x);         push_back(x);                |  
//                                                             ↓  
//                                                 insert_aux(position, x);  
//                                                 具体剖析见后面实现  
////////////////////////////////////////////////////////////////////////////////  
  
    iterator insert(iterator position, const value_type& x)  
    {  
        // 如果是在deque的最前端插入, 那么直接push_front()即可  
        if (position.cur == start.cur)  
        {  
            push_front(x);  
            return start;  
        }  
        // 如果是在deque的末尾插入, 直接调用push_back()  
        else if (position.cur == finish.cur)  
        {  
            push_back(x);  
            iterator tmp = finish;  
            --tmp;  
            return tmp;  
        }  
        else  
        {  
            return insert_aux(position, x);  
        }  
    }  
  
    iterator insert(iterator position) { return insert(position, value_type()); }  
  
    // 详解见实现部分  
    void insert(iterator pos, size_type n, const value_type& x);  
  
    void insert(iterator pos, int n, const value_type& x)  
    {  
        insert(pos, (size_type) n, x);  
    }  
    void insert(iterator pos, long n, const value_type& x)  
    {  
        insert(pos, (size_type) n, x);  
    }  
  
    void resize(size_type new_size) { resize(new_size, value_type()); }  
  
public:                         // Erase  
  
    iterator erase(iterator pos)  
    {  
        iterator next = pos;  
        ++next;  
  
        // 清除点之前的元素个数  
        difference_type index = pos - start;  
  
        // 如果清除点之前的元素个数比较少, 哪部分少就移动哪部分  
        if (index < (size() >> 1))  
        {  
            // 就移动清除点之前的元素  
            copy_backward(start, pos, next);  
            pop_front();   // 移动完毕，最前一个元素冗余，去除之  
        }  
        else   // 如果清除点之后的元素个数比较少  
        {  
            copy(next, finish, pos);  // 就移动清除点之后的元素  
            pop_back();   // 移动完毕，最后一个元素冗余，去除之  
        }  
        return start + index;  
    }  
  
    iterator erase(iterator first, iterator last);  
    void clear();  
  
protected:  
  
    // 详解见实现部分  
    void push_back_aux(const value_type& t);  
    void push_front_aux(const value_type& t);  
    void pop_back_aux();  
    void pop_front_aux();  
  
    iterator insert_aux(iterator pos, const value_type& x);  
    void insert_aux(iterator pos, size_type n, const value_type& x);  
  
    // 分配内存, 不进行构造  
    pointer allocate_node() { return data_allocator::allocate(buffer_size()); }  
  
    // 释放内存, 不进行析构  
    void deallocate_node(pointer n)  
    {  
        data_allocator::deallocate(n, buffer_size());  
    }  
  
};  
  
  
////////////////////////////////////////////////////////////////////////////////  
// 清除[first, last)区间的所有元素  
////////////////////////////////////////////////////////////////////////////////  
//                  erase(iterator first, iterator last)  
//                                   |  
//                                   |---------------- 是否要删除整个区间?  
//                                   |  
//               ------------------------------------------  
//           Yes |                                        | No  
//               |                                        |  
//               ↓                                        | --- 判断哪侧元素少  
//            clear();                                    ↓  
//       -----------------------------------------------------------------  
// 左侧少 |                                                         右侧少 |  
//       |                                                               |  
//       ↓                                                               ↓  
//   copy_backward(start, first, last);            copy(last, finish, first);  
//   new_start = start + n;                        new_finish = finish - n;  
//   析构多余的元素                                  析构多余的元素  
//   destroy(start, new_start);                    destroy(new_finish, finish);  
//   释放多余内存空间                                释放多余内存空间  
//   for (...)                                     for (...)  
//      ...                                             ...  
//   更新map状态                                    更新map状态  
////////////////////////////////////////////////////////////////////////////////  
template <class T, class Alloc, size_t BufSize>  
deque<T, Alloc, BufSize>::iterator  
deque<T, Alloc, BufSize>::erase(iterator first, iterator last)  
{  
    if (first == start && last == finish)   // 如果清除区间是整个deque  
    {  
        clear();              // 直接调用clear()即可  
        return finish;  
    }  
    else  
    {  
        difference_type n = last - first;   // 清除区间的长度  
        difference_type elems_before = first - start;   // 清除区间前方的元素个数  
        if (elems_before < (size() - n) / 2)   // 如果前方的元素个数比较少  
        {  
            copy_backward(start, first, last);  // 向后移动前方元素（覆盖清除区间）  
            iterator new_start = start + n;     // 标记deque的新起点  
            destroy(start, new_start);          // 移动完毕，将冗余的元素析构  
            // 以下将冗余的缓冲区释放  
            for (map_pointer cur = start.node; cur < new_start.node; ++cur)  
                data_allocator::deallocate(*cur, buffer_size());  
            start = new_start;   // 设定deque的新起点  
        }  
        else    // 如果清除区间后方的元素个数比较少  
        {  
            copy(last, finish, first);  // 向前移动后方元素（覆盖清除区间）  
            iterator new_finish = finish - n;     // 标记deque的新尾点  
            destroy(new_finish, finish);          // 移动完毕，将冗余的元素析构  
            // 以下将冗余的缓冲区释放  
            for (map_pointer cur = new_finish.node + 1; cur <= finish.node; ++cur)  
                data_allocator::deallocate(*cur, buffer_size());  
            finish = new_finish;   // 设定deque的新尾点  
        }  
        return start + elems_before;  
    }  
}  
  
template <class T, class Alloc, size_t BufSize>  
void deque<T, Alloc, BufSize>::clear()  
{  
    // 以下针对头尾以外的每一个缓冲区  
    for (map_pointer node = start.node + 1; node < finish.node; ++node)  
    {  
        // 将缓冲区内的所有元素析构  
        destroy(*node, *node + buffer_size());  
        // 释放缓冲区内存  
        data_allocator::deallocate(*node, buffer_size());  
    }  
  
    if (start.node != finish.node)   // 至少有头尾两个缓冲区  
    {  
        destroy(start.cur, start.last);  // 将头缓冲区的目前所有元素析构  
        destroy(finish.first, finish.cur);  // 将尾缓冲区的目前所有元素析构  
        // 以下释放尾缓冲区。注意：头缓冲区保留  
        data_allocator::deallocate(finish.first, buffer_size());  
    }  
    else   // 只有一个缓冲区  
        destroy(start.cur, finish.cur);   // 将此唯一缓冲区内的所有元素析构  
        // 注意：并不释放缓冲区空间，这唯一的缓冲区将保留  
  
    finish = start;   // 调整状态  
}  
  
  
// 只有当finish.cur == finish.last - 1 时才会被调用  
// 也就是说，只有当最后一个缓冲区只剩下一个备用元素空间时才会被调用  
template <class T, class Alloc, size_t BufSize>  
void deque<T, Alloc, BufSize>::push_back_aux(const value_type& t)  
{  
    value_type t_copy = t;  
    reserve_map_at_back();  
    *(finish.node + 1) = allocate_node();    // 配置一个新节点（缓冲区）  
    __STL_TRY  
    {  
        construct(finish.cur, t_copy);         // 针对标的元素设值  
        finish.set_node(finish.node + 1);      // 改变finish，令其指向新节点  
        finish.cur = finish.first;             // 设定finish的状态  
    }  
    __STL_UNWIND(deallocate_node(*(finish.node + 1)));  
}  
  
// Called only if start.cur == start.first.  
template <class T, class Alloc, size_t BufSize>  
void deque<T, Alloc, BufSize>::push_front_aux(const value_type& t)  
{  
    value_type t_copy = t;  
    reserve_map_at_front();  
    *(start.node - 1) = allocate_node();  
    __STL_TRY  
    {  
        start.set_node(start.node - 1);        // 改变start，令其指向新节点  
        start.cur = start.last - 1;            // 设定start的状态  
        construct(start.cur, t_copy);          // 针对标的元素设值  
    }  
    catch(...)  
    {  
        start.set_node(start.node + 1);  
        start.cur = start.first;  
        deallocate_node(*(start.node - 1));  
        throw;  
    }  
}  
  
// 只有当 finish.cur == finish.first 时才会被调用  
template <class T, class Alloc, size_t BufSize>  
void deque<T, Alloc, BufSize>:: pop_back_aux()  
{  
    deallocate_node(finish.first);      // 释放最后一个缓冲区  
    finish.set_node(finish.node - 1);   // 调整finish状态，使指向  
    finish.cur = finish.last - 1;       // 上一个缓冲区的最后一个元素  
    destroy(finish.cur);                // 将该元素析构  
}  
  
// 只有当 start.cur == start.last - 1 时才会被调用  
template <class T, class Alloc, size_t BufSize>  
void deque<T, Alloc, BufSize>::pop_front_aux()  
{  
    destroy(start.cur);    // 将第一个缓冲区的第一个（也是最后一个、唯一一个）元素析构  
    deallocate_node(start.first);    // 释放第一缓冲区  
    start.set_node(start.node + 1);  // 调整start状态，使指向  
    start.cur = start.first;         // 下一个缓冲区的第一个元素  
}  
  
  
////////////////////////////////////////////////////////////////////////////////  
// 在指定位置前插入元素  
////////////////////////////////////////////////////////////////////////////////  
//              insert_aux(iterator pos, const value_type& x)  
//                                   |  
//                                   |----------- 判断pos前端元素少还是后端元素少  
//                                   |  
//               -----------------------------------------------  
//         前端少 |                                       后端少 |  
//               |                                             |  
//               ↓                                             |  
//           进行相关操作                                   进行相关操作  
////////////////////////////////////////////////////////////////////////////////  
// 下面以pos前面元素少的情形进行说明, 为了简化, 假设操作不会超过一个缓冲区区间  
//  
// 插入前状态  
//           start            pos                                 end  
//             ↓               ↓                                   ↓  
// ---------------------------------------------------------------------  
// |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   | X |  
// ---------------------------------------------------------------------  
//  
// 需要进行操作的区间  
//                需要拷贝的区间  
//                 -------------  
//       start     |           |                                  end  
//         ↓       ↓           ↓                                   ↓  
// ---------------------------------------------------------------------  
// |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   | X |  
// ---------------------------------------------------------------------  
//             ↑   ↑       ↑   ↑  
//        front1   |       |   |  
//                 |       |   |  
//            front2       |   |  
//                         |   |  
//                       pos   |  
//                             |  
//                          pos1  
// 拷贝操作完成后  
//  
//         这是[front2, pos1)  
//             ------------- --------- 这里是给待插入元素预留的空间  
//       start |           | |                                    end  
//         ↓   ↓           ↓ ↓                                     ↓  
// ---------------------------------------------------------------------  
// |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   |   | X |  
// ---------------------------------------------------------------------  
//         ↑  
//   这里存储的是原来的front()  
//  
////////////////////////////////////////////////////////////////////////////////  
  
template <class T, class Alloc, size_t BufSize>  
typename deque<T, Alloc, BufSize>::iterator  
deque<T, Alloc, BufSize>::insert_aux(iterator pos, const value_type& x)  
{  
    difference_type index = pos - start;   // 插入点之前的元素个数  
    value_type x_copy = x;  
  
    // 前面的时候用的移位操作, 这里怎么不用了呢^_^?  
    if (index < size() / 2)    // 如果插入点之前的元素个数比较少  
    {  
        push_front(front());       // 在最前端加入与第一元素同值的元素  
        iterator front1 = start;   // 以下标示记号，然后进行元素移动  
        ++front1;  
        iterator front2 = front1;  
        ++front2;  
        pos = start + index;  
        iterator pos1 = pos;  
        ++pos1;  
        copy(front2, pos1, front1);    // 元素移动  
    }  
    else    // 插入点之后的元素个数比较少  
    {  
        push_back(back());         // 在最尾端加入与最后元素同值的元素  
        iterator back1 = finish;   // 以下标示记号，然后进行元素移动  
        --back1;  
        iterator back2 = back1;  
        --back2;  
        pos = start + index;  
        copy_backward(pos, back2, back1);    // 元素移动  
    }  
    *pos = x_copy;    // 在插入点上设定新值  
    return pos;  
} 
```
## 源网址
* [STL源码剖析——deque的实现原理和使用方法详解](https://blog.csdn.net/baidu_28312631/article/details/48000123)
* [STL源码剖析---deque](https://blog.csdn.net/Hackbuteer1/article/details/7729451)