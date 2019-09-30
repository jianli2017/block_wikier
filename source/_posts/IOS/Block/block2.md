---
title: Block的类型
date: 2019-09-29 12:07:12
tags: Block
categories: Block
toc: true
---

本文理解block的本质

<!--more-->


## block的底层实现

示例代码

```
{
    void (^block)(void);
    {
        Persion *persion = [[Persion alloc] init];
        persion.age = 20;
        block = ^(){
            NSLog(@"%d",persion.age);
        };
    }
    //由于block被强引用对象持有，block是__NSMallocBlock__ ，持有persion对象
    NSLog(@"%@", [block class]);
}
//超出block作用域，block释放，persion也释放
//执行persion的dealloc方法
```


翻译为CPP代码 

```
{
    void (*block)(void);
    {
        Persion *persion = objc_msgSend(objc_msgSend(objc_getClass("Persion"), sel_registerName("alloc")), sel_registerName("init"));
        objc_msgSend(persion, sel_registerName("setAge:"), 20);
        block = &__main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA, persion, 570425344));
    }
    
    NSLog((NSString *)&__NSConstantStringImpl__var_folders_hs_21g8l7ps1p9cmwpn1nbnfn700000gn_T_main_6d037c_mi_2, objc_msgSend(block, sel_registerName("class")));
}
NSLog((NSString *)&__NSConstantStringImpl__var_folders_hs_21g8l7ps1p9cmwpn1nbnfn700000gn_T_main_6d037c_mi_3);
```

下面是`__main_block_impl_0`的定义：

```
struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    //定义了捕获的persion对象
    Persion *persion; 
    //构造函数中多了一个persion对象
    __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, Persion *_persion, int flags=0) : persion(_persion) {
        impl.isa = &_NSConcreteStackBlock;
        impl.Flags = flags;
        impl.FuncPtr = fp;
        Desc = desc;
    }
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
	//使用捕获的persion对象
    Persion *persion = __cself->persion; // bound by copy
    
    NSLog((NSString *)&__NSConstantStringImpl__var_folders_hs_21g8l7ps1p9cmwpn1nbnfn700000gn_T_main_6d037c_mi_1,((int (*)(id, SEL))(void *)objc_msgSend)((id)persion, sel_registerName("age")));
}

//增加捕获对象的引用计数器
static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {_Block_object_assign((void*)&dst->persion, (void*)src->persion, 3/*BLOCK_FIELD_IS_OBJECT*/);}

//减少捕获对象的引用计数器
static void __main_block_dispose_0(struct __main_block_impl_0*src) {_Block_object_dispose((void*)src->persion, 3/*BLOCK_FIELD_IS_OBJECT*/);}

//__main_block_desc_0多了两个函数指针
static struct __main_block_desc_0 {
    size_t reserved;
    size_t Block_size;
    void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
    void (*dispose)(struct __main_block_impl_0*);
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};
```


## 点点滴滴

```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.m
```