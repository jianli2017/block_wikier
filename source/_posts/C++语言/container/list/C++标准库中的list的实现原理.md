---
title: C++标准库中的list的实现原理
date: 2018-06-26 15:07:12
tags: list
categories: C++
toc: true
---

本文抄袭至[C++标准库中的list的实现原理](https://blog.csdn.net/loveyou11111111/article/details/49763197)

<!--more-->

在C++中采用了大量的标志模板库（STL）实现程序的设计，这种设计方式使得不同类型的对象都能通用，而不再是C语言中的通常对于不同的类型需要重新设计或者或者比较采用间接的指针操作。C++中的这种方式简化了写代码的复杂度，但是增加了编译器的复杂度和难度。
 
在数据结构中链表是比较基本的类型，在C++中链表是基于模板的类，因此在实际的使用过程中需要涉及到实际的类型。

```
#include<list>
list<int> lint;
```

在C++中关于list的接口比较丰富，主要是关于大小，数据的插入、删除等。但是在C++中引入了迭代器的概念，这个迭代器是关于关于容器中比较重要的一部分，因为这种迭代器使得算法等问题与容器独立开来，迭代器实质上还是指针，准确的将是一个封装了指针的类。
 
迭代器类的创建应该包含下面的操作，首先应该支持的操作符至少包括如下（operator*()，operator++()，operator++(int)，operator==()， operator!=()）,当然也会存在const_iterator这样的常迭代器，也就是只允许访问，不能修改对象的迭代器，当然存在迭代器的构造函数、复制控制函数，这些函数都是必须要存在的，因为设计到了指针的操作问题，构造函数应该存在参数是链表节点指针的定义，只有存在这个定义才能间接的访问节点对象。
当然在类中至少存在返回迭代器的begin()和end()函数，这两个函数返回的迭代器分别指向链表的开始和链表结束的下一个地址，这是迭代器中经常容易理解错误的地方。
 
在C++中通常创建const_iterator类，然后iterator直接继承const_iterator。
 
下面说说list类设计的基本思路：
首先、创建链表节点对象，实质上是完成对传递进来的类型的封装操作，同时构成一个双向链表的基本要素（prev、next指针）。节点肯定要存在构造函数，而且是直接初始化三个成员变量。
其次、创建迭代器类，实质上就是封装一个节点指针，通过节点指针实现操作，至少要实现的操作符已说明。这两个类都要设置List为友元类，因为这样才能用List直接操作迭代器的相关操作。
最后、依靠上面的迭代器类和节点类，创建一个List类，该类中主要完成一些基本操作。其中需要注意的就是迭代器的操作，比如删除元素和插入元素以后迭代器的变化问题等。
 
需要注意的是在List中采用了哨兵节点，这个哨兵节点并不算实际的操作对象，也就是为了保证肯定有目标所指向，存在一个head对象，这个对象的next就是实际的数据，而tail是迭代器所能到达的最后一个对象，但是这个对象并不是合理的区域，实际上end()实际上就是指向了tail节点，这两个节点head和tail就是哨兵节点。具体的参看代码。

## 源码解读

实现的基本形式如下：


```
#ifndef __MYLIST_H_H_
#define __MYLIST_H_H_

#include<iostream>

namespace myspace
{

        templateObject>
        class List
        {
        private:
                /*封装对象，形成链表节点*/
                struct Node
                {
                        Object data;
                        struct Node *prev;
                        struct Node *next;

                        /*节点构造函数*/
                        Node(const Object &d = Object(), Node *p = NULL, Node *n = NULL)
                        :data(d), prev(p), next(n){}
                };

        public:
                /*创建一个常量迭代器类，这是容器设计的关键*/
                class const_iterator
                {
                public:
                        const_iterator():current(NULL)
                        {}

                        /*重载迭代器的值*/
                        const Object & operator*()const
                        {
                                return retrieve();
                        }
                        /*重载前向++操作符*/
                        const_iterator & operator++ ()
                        {
                                current = current->next;

                                return *this;
                        }
                        /*重载后向++操作符，因为是一个局部对象不能返回引用*/
                        const_iterator operator++(int)
                        {
                                const_iterator old = *this;
                                ++(*this);

                                return old;
                        }
                        /*判断迭代器是否相同，实质上就是判断指向的节点是否相同*/
                        bool operator==(const const_iterator &rhs) const
                        {
                                return current == rhs.current;
                        }
                        /*调用==操作符*/
                        bool operator!=(const const_iterator &rhs)const
                        {
                                return (!(*this == rhs));
                        }
                protected:
                        /*迭代器实质就是一个节点指针*/
                        Node *current;

                        /*获得链表中的内容*/
                        Object & retrieve() const
                        {
                                return current->data;
                        }

                        /*基于指针参数的迭代器构造函数，保证只有List使用*/
                        const_iterator(Node *p):current (p)
                        {}

                        /*友元类，可以调用迭代器的私有成员*/
                        friend class List<Object>;
                };
                /*一般迭代器，直接继承const_iterator*/
                class iterator : public const_iterator
                {
                public:
                        iterator():const_iterator()
                        {}

                        /*得到对象的值*/
                        Object &operator*()
                        {
                                return retrieve();
                        }

                        /*基于const的重载*/
                        const Object& operator*()const
                        {
                                return const_iterator::operator*();
                        }
                        /*前向++操作符*/
                        iterator &operator++()
                       {
                                current = current->next;

                                return *this;
                        }
                        /*后向++操作符*/
                        iterator operator++(int)
                        {
                                iterator *old = *this;
                                ++(*this);
                                return old;
                        }

                protected:
                        /*基于节点的迭代器构造函数*/
                        iterator(Node *p):const_iterator(p)
                        {}

                        friend class List<Object>;
                };
        public:
                List()
                {
                        init();
                }
                ~List()
                {
                        clear();
                        delete head;
                        delete tail;
                }

                List(const List &rhs)
                {
                       /*创建哨兵节点*/
                        init();
                        /*复制数据*/
                        *this = rhs;
                }

                const List & operator=(const List &rhs)
                {
                        if(this == &rhs)
                                return *this;
                        /*清除原有的信息*/
                        clear();
                        /*添加新的对象*/
                        for(const_iterator itr = rhs.begin(); itr != rhs.end(); ++ itr)
                                push_back(*itr);

                        return *this;
                }

                /*得到迭代器，实质上就是得到节点指针*/
                iterator begin()
                {
                        /*iterator()是构造函数*/
                        return iterator(head->next);
                }

                const_iterator begin()const
                {
                        return const_iterator(head->next);
                }

                iterator end()
               {
                        return iterator(tail);
                }

                const_iterator end()const
                {
                        return const_iterator(tail);
                }

                int size()const
                {
                        return theSize;
                }

                bool empty()const
                {
                        return size() == 0;
                }

                void clear()
                {
                        while( !empty())
                                pop_front();
                }

                /*得到第一个元素*/
                Object & front()
                {
                        /*采用了迭代器begin()*/
                        return *begin();
                }
                const Object &front()const
                {
                        return *begin();
               }

                Object &back()
                {
                        /*end()指向最后一个对象的下一个地址，因此需要--*/
                        return *--end();
                }
                const Object &back()const
                {
                        return *--end();
                }
                /***********************************************
                *从头插入新的节点，这时候的begin已经不再是begin
                *因此插入操作会导致迭代器出错
                ***********************************************/
                void push_front(const Object &x)
                {
                        insert(begin(), x);
                }
                /*从后插入新的节点，这时候会将end后移*/
                void push_back(const Object &x)
                {
                        insert(end(), x);
                }

                /*从头弹出一个对象*/
                void pop_front()
                {
                        erase(begin());
                }
                void pop_back()
                {
                       erase(--end());
                }

                /*插入对象，参数是迭代器和数据*/
                iterator insert(iterator itr, const Object &x)
                {
                        /*得到当前迭代器的指针*/
                        Node *p = itr.current;
                        theSize ++;

                        /*
                        *Node *np = Node(x,p->prev,p);
                        this means that np->prev = p->prev,
                        and np->next = p;

                        update the p->prev and p->prev->next;
                        *p->prev->next = np;
                        *p->prev = np;
                        */
                        return iterator(p->prev=p->prev->next= new Node(x,p->prev, p));
                }

                /*删除迭代器处的对象,因此删除也会导致迭代器破坏*/
                iterator erase(iterator itr)
                {
                        /*得到当前迭代器的指针*/
                        Node *p = itr.current;
                        /*得到新的迭代器，并初始化*/
                        iterator retVal(p->next);
                        /*更新链表的链接关系*/
                        p->prev->next = p->next;
                        p->next->prev = p->prev;
                        /*删除对象*/
                        delete p;
                        /*使得对象数减少*/
                        theSize --;
                        /*返回新的迭代器*/
                        return retVal;
                }

                /*删除迭代器指向的对象*/
                iterator erase(iterator start, iterator end)
                {
                        /*for中不使用++itr的原因是erase之后
                         *就是下一个迭代器，因此不需要++操作*/
                        for(iterator itr = start; itr != end; )
                        {
                                /*该操作会导致迭代器更新到下一个*/
                                itr = erase(itr);
                        }
                        return itr;
                }

        private:
                /*链表中的数据成员*/
                int theSize;
                Node *head;
                Node *tail;
                /*初始化函数*/
                void init()
                {
                        theSize = 0;
                        /*create two sentinel node*/
                        /*构建两个哨兵节点，也就是两个并不算在结构体中的对象*/
                        head = new Node;
                        tail = new Node;
                        /*绑定起来*/
                        head->next = tail;
                        tail->prev = head;
                }
        };
}

#endif
```

