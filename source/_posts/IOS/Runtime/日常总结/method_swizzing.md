---
title: 方法交换理解
date: 2019-08-06 11:38:11
tags: runtime
categories: Runtime
toc: true
---

本文理解方法交换写法的含义

<!--more-->

##   方法交换源码解读


下面是方法交换的常用源码：

```
- (void)swizzleMethod:(SEL)origSelector withMethod:(SEL)newSelector
{
    Class class = [self class];
    
    Method originalMethod = class_getInstanceMethod(class, origSelector);
    Method swizzledMethod = class_getInstanceMethod(class, newSelector);
    
    BOOL didAddMethod = class_addMethod(class,
                                        origSelector,
                                        method_getImplementation(swizzledMethod),
                                        method_getTypeEncoding(swizzledMethod));
    if (didAddMethod) {
        class_replaceMethod(class,
                            newSelector,
                            method_getImplementation(originalMethod),
                            method_getTypeEncoding(originalMethod));
    } else {
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
}
```


要先尝试添加原 selector 是为了做一层保护，因为如果这个类没有实现 originalSelector ，但其父类实现了，那 class_getInstanceMethod 会返回父类的方法。这样 method_exchangeImplementations 替换的是父类的那个方法，这当然不是你想要的。所以我们先尝试添加 orginalSelector ，如果已经存在，再用 method_exchangeImplementations 把原方法的实现跟新的方法实现给交换掉。

下面分析 class_addMethod的源码：

```
BOOL 
class_addMethod(Class cls, SEL name, IMP imp, const char *types)
{
    if (!cls) return NO;

    rwlock_writer_t lock(runtimeLock);
    return ! addMethod(cls, name, imp, types ?: "", NO);
}
```

class_addMethod 内部调用了addMethod，并将addMethod的结果取反，作为返回值。其中replace参数（最后一个）传递NO，表示如果类中存在该方法（originalMethod），不替换。

```
static IMP 
addMethod(Class cls, SEL name, IMP imp, const char *types, bool replace)
{
    IMP result = nil;

    runtimeLock.assertWriting();

    assert(types);
    assert(cls->isRealized());

    method_t *m;
    if ((m = getMethodNoSuper_nolock(cls, name))) {
        // already exists
        if (!replace) {
            result = m->imp;
        } else {
            result = _method_setImplementation(cls, m, imp);
        }
    } else {
        // fixme optimize
        method_list_t *newlist;
        newlist = (method_list_t *)calloc(sizeof(*newlist), 1);
        newlist->entsizeAndFlags = 
            (uint32_t)sizeof(method_t) | fixed_up_method_list;
        newlist->count = 1;
        newlist->first.name = name;
        newlist->first.types = strdupIfMutable(types);
        newlist->first.imp = imp;

        prepareMethodLists(cls, &newlist, 1, NO, NO);
        cls->data()->methods.attachLists(&newlist, 1);
        flushCaches(cls);

        result = nil;
    }

    return result;
}
```

该方法首先查找类是否有方法，如果有，返回方法，如果没有添加并返回空，所以addMethod的返回值可以理解为**返回旧的方法**。

所以进一步理解class_addMethod返回值的含义： 它取反了addMethod的结果，所以，添加成功，返回yes，没有添加成功（也就是存在方法），返回no。

方法添加成功，也就是类中没有originalMethod方法，调用了class_replaceMethod，接着看class_replaceMethod方法：

```
IMP 
class_replaceMethod(Class cls, SEL name, IMP imp, const char *types)
{
    if (!cls) return nil;

    rwlock_writer_t lock(runtimeLock);
    return addMethod(cls, name, imp, types ?: "", YES);
}
```

因为上面class_addMethod添加成功，说明原始类是没有旧方法，也就不用管我们添加的旧方法，所以直接调用class_replaceMethod将**添加的方法originalMethod**换为新方法即可，不需要交换，所以，传递给addMethod方法的最后一个参数是yes，**直接替换**。

如果没有添加成功，说明原类中有originalMethod，所以不能直接替换，如果直接替换了，调用原方法的函数就会出问题（死循环），需要交换，条用method_exchangeImplementations交换方法。

```
void method_exchangeImplementations(Method m1_gen, Method m2_gen)
{
    IMP m1_imp;
    old_method *m1 = oldmethod(m1_gen);
    old_method *m2 = oldmethod(m2_gen);
    if (!m1  ||  !m2) return;

    impLock.lock();
    m1_imp = m1->method_imp;
    m1->method_imp = m2->method_imp;
    m2->method_imp = m1_imp;
    impLock.unlock();
}
```

## 参考资料

1. [Draveness git地址](https://github.com/Draveness/analyze)
2. [Classes and Metaclasses](http://www.sealiesoftware.com/blog/archive/2009/04/14/objc_explain_Classes_and_metaclasses.html)
3. [类型编码](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html)
4. [Type Encodings](http://nshipster.cn/type-encodings/)
5. [Tagged Pointer](https://en.wikipedia.org/wiki/Tagged_pointer)



