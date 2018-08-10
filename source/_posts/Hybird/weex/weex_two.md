---
title: weex系列抄之二---weex原理
date: 2018-05-04 12:07:12
tags: weex
categories: weex
toc: true
---

## 关于Weex

Weex是一套跨平台的动态页面解决方案，让开发者可以用前端的语法写出Native级别的体验，这一点核心功能与React Native是相同的，但RN并不是今天的主角， 这里也不多花笔墨介绍。WEEX宣称「Write Once, Run Everywhere」，同一份代码可以在不同端上运行。Weex是如何做到的呢？

话不多说，先上一张上镜率特别高的流程图，了解一下Weex的工作流程：

![](http://of685p9vy.bkt.clouddn.com/hybird/weex/weex1.jpg)

在服务端，开发者将写好的Weex文件转换成JS bundle并部署到服务器上供终端下载；终端会在合适的时机拉取JS Bundle，同时利用WeexSDK 中预先准备好的 JavaScript 引擎解析执行JS bundle，在执行过程中通过JS-Native Bridge产生各种终端能够识别的命令进行界面渲染或数据存储、网络通信、调用设备功能、用户交互响应等移动应用的场景实践。

## Weex框架

Weex源码可以在Github(https://github.com/apache/incubator-weex)上下载到，先看下0.16.1版本下的文件目录结构：

![](http://of685p9vy.bkt.clouddn.com/hybird/weex/weex2.jpg)

目录的划分比较清楚，一个目录基本就是对应一个功能模块，我们可以对其做一个归类，把它分为三端：JS端、桥接端和纯Native端。见下图：（灰色的方块代表一个文件目录）

![](http://of685p9vy.bkt.clouddn.com/hybird/weex/weex3.jpg)

## JS端

JS端主要内容是Weex源码中的native-bundle-main.js文件，它提供了一系列Weex的基础JS方法，作用相当于一个库，因此我们又称之为JS Framework。Weex把JS Bundle拆分为基础JS库和业务JS代码， 并把JS库带到安装包中，这样一来，页面请求的JS Bundle就只需要包含业务代码，体积会变得很小，对于加载速度提升大有裨益。JS Framework会在WeexSDK初始化时被加载到内存中。

## 桥接端（Bridge）

桥接层负责JS和Native的通信，主要依靠一个全局的JSContext作为媒介。WeexSDK初始化时会往这个全局的JSContext中注入一些方法，举个栗子：

```
- (void)registerCallNative:(WXJSCallNative)callNative
{
    JSValue* (^callNativeBlock)(JSValue *, JSValue *, JSValue *) = ^JSValue*(JSValue *instance, JSValue *tasks, JSValue *callback){
        NSString *instanceId = [instance toString];
        NSArray *tasksArray = [tasks toArray];
        NSString *callbackId = [callback toString];  
      return [JSValue valueWithInt32:(int32_t)callNative(instanceId, tasksArray, callbackId) inContext:[JSContext currentContext]];
    };

    _jsContext[@"callNative"] = callNativeBlock
}
```

而终端调用JS也是通过取JSContext对象，调用invokeMethod:withArguments:方法实现。

```
- (JSValue *)callJSMethod:(NSString *)method args:(NSArray *)args
{    
    return [[_jsContext globalObject] invokeMethod:method withArguments:args];
}
```

## 纯Native端

<font color=red>主要的业务都代码都是这一端，包括JS Bundle的请求、UI渲染、性能统计等等。功能层面对其自上而下划分，又可拆分为接口层（Interface）、功能层（Function）、基础层（Basic）。</font>

接口层顾名思义，就是对外暴露API的模块，是最贴近开发者的一层。通过Engine可以对SDK进行初始化，同时注册一些通用的Component和Module，JS Framework会在此时被加载。Controller不必多说，可以使用这个现成的Controller创建一个weex页面，我们需要做的仅仅是传递一个url给它。但如果想要自已实现一个Weex Controller而不是继承它，这个时候就需要用到Model中的WXSDKInstance，Weex渲染过程的各个阶段都会在WXSDKInstance中有回调，JS Bundle的请求也是在这个类中发出。

先绕过功能层，看基础层。基础层提供了一些基础的、与业务无强相关的功能。如Network就是对NSURLSession进行一次再封装，提供基础的下载功能；Event用来定义一些标准手势事件；Layout则是页面布局相关实现，布局引擎采用C语言，可以跨平台使用。Event和Layout最终都服务于Component。

最后是代码最重的功能层，其中Monitor和DevTool是相对比较独立的，Monitor是测速模块，DevTool用来支持远程调试的，可以不集成到代码中，不影响编译，此处不谈。

Module、Component和Handler是Weex三贱客，它们都是采用插件的形式集合到SDK中，很方便扩展。注册的时机也相同，SDK会在初始化时调用registerDefaultModules/Compoents/Handlers加载一些标准的插件。

```
+ (void)registerDefaults
{
    [self _registerDefaultComponents];
    [self _registerDefaultModules];
    [self _registerDefaultHandlers];
}
```

```
// register some default components when the engine initializes.+ (void)_registerDefaultComponents
{
    [self registerComponent:@"container" withClass:NSClassFromString(@"WXDivComponent") withProperties:nil];
    [self registerComponent:@"div" withClass:NSClassFromString(@"WXComponent") withProperties:nil];
    [self registerComponent:@"text" withClass:NSClassFromString(@"WXTextComponent") withProperties:nil];
    [self registerComponent:@"image" withClass:NSClassFromString(@"WXImageComponent") withProperties:nil];
    ……
}
```

从注册时的命名上不难看出，<font color=red>Component实现的是UIKit的功能。</font>那还有两贱客是干吗的？<font color=red>我们可以认为Module是终端提供给JS的功能模块，为了让JS获得终端的能力，例如网络请求的能力（WXStreamModule）、定时器的能力（WXTimerModule）等。三贱客里，Component和Module都是可以直接和JS通信的，而Handler不行，Handler仅仅作为面向协议编程的一种手段，在纯Native端使用</font>。

Loader就是采用了Handler的形式对Network进行了进一步的再封装，用这种方式会让Loader模块更灵活一些，开发者完全可以通过重新注册Handler来挂载新的网络请求方法，实现一些自定义的功能。

<font color=red>除了Component和Module，还有一个模块可以直接和JS通信，就是结构图中与桥接端相邻的Manager（与桥接端相邻代表二者可以直接通信），Manager模块主要包括WXComponentManager和Factory，WXComponentManager用来做Component的调度，而Factory用来保存Component和Module的配置。</font>

讲完了WeexSDK源码框架，或许读者还是会有不少疑问：说了那么一堆高大上的名词，然并卵，我还是不知道Weex是怎么跑起来的。那我们就更具体一点，去看看Module和Component的世界吧。

## Module

前面有一个高频词汇：注册。<font color=red>所谓注册，其实在实现上就是往全局字典里面以Key-Value的形式保存一些模块的信息。</font>Weex三贱客都需要注册，Handler的注册上文提到过，其实就是以Protocol Name为Key，往全局字典里面写入一个实现了该Protocol的对象。Module和Component的注册则更接近一些，都是以注册时的标签为Key，而保存的value是一个WXInvocationConfig派生类对象，可以瞄一眼WXInvocationConfig携带的信息，包括：标签名、类名、同步方法和异步方法。

```
@interface WXInvocationConfig : NSObject

@property (nonatomic, strong) NSString *name;
@property (nonatomic, strong) NSString *clazz;

@property (nonatomic, strong) NSMutableDictionary *asyncMethods;
@property (nonatomic, strong) NSMutableDictionary *syncMethods;

- (instancetype)initWithName:(NSString *)name class:(NSString *)clazz;
- (void)registerMethods;

@end
```

以WXDomModule为例，在WXDomModule的类实现文件中有一坨被WX_EXPORT_METHOD宏定义包裹的selector：

```
WX_EXPORT_METHOD(@selector(createBody:))
WX_EXPORT_METHOD(@selector(addElement:element:atIndex:))
WX_EXPORT_METHOD(@selector(removeElement:))
WX_EXPORT_METHOD(@selector(moveElement:parentRef:index:))
WX_EXPORT_METHOD(@selector(addEvent:event:))
WX_EXPORT_METHOD(@selector(removeEvent:event:))
……
```

```
#define WX_EXPORT_METHOD(method) WX_EXPORT_METHOD_INTERNAL(method,wx_export_method_)

#define WX_EXPORT_METHOD_INTERNAL(method, token) \
+ (NSString *)WX_CONCAT_WRAPPER(token, __LINE__) {
 \    return NSStringFromSelector(method); \
}

#define WX_CONCAT_WRAPPER(a, b)    WX_CONCAT(a, b)
```

```
+ (NSString *)wx_export_method_40 {   
 return NSStringFromSelector(@selector(createBody:));
}
```

```
- (void)registerMethods
{
    Class currentClass = NSClassFromString(_clazz);

    while (currentClass != [NSObject class]) {
        unsigned int methodCount = 0;
        Method *methodList = class_copyMethodList(object_getClass(currentClass), &methodCount);
        for (unsigned int i = 0; i < methodCount; i++) {
            NSString *selStr = [NSString stringWithCString:sel_getName(method_getName(methodList[i])) encoding:NSUTF8StringEncoding];
            BOOL isSyncMethod = NO;
            if ([selStr hasPrefix:@"wx_export_method_sync_"]) {
                isSyncMethod = YES;
            } else if ([selStr hasPrefix:@"wx_export_method_"]) {
                isSyncMethod = NO;
            } else {
                continue;
            }

            NSString *name = nil, *method = nil;
            SEL selector = NSSelectorFromString(selStr);
            if ([currentClass respondsToSelector:selector]) {
                method = ((NSString* (*)(id, SEL))[currentClass methodForSelector:selector])(currentClass, selector);
            }

            NSRange range = [method rangeOfString:@":"];
            if (range.location != NSNotFound) {
                name = [method substringToIndex:range.location];
            } else {
                name = method;
            }

            NSMutableDictionary *methods = isSyncMethod ? _syncMethods : _asyncMethods;
            [methods setObject:method forKey:name];
        }

        free(methodList);
        currentClass = class_getSuperclass(currentClass);
    }

}
```

调用完registerMethods方法后，WXDomModule的Config包含的信息如下：

![](http://of685p9vy.bkt.clouddn.com/hybird/weex/weex4.jpg)

<font color=red>同步方法和异步方法存储的Key列表就是JS端可调用的函数名列表。</font>趁热打铁，继续看下JS端是具体是怎么调用这些暴露出去的方法。

在前文第二节Weex框架中有提到，WeexSDK会在桥接层往JSContext注入一些方法作为JS调用Native的通道，其中callNativeModule方法就是用来调用Native Module的（JS调用Module不仅限于callNativeModule方法）。

```
[_jsBridge registerCallNativeModule:^NSInvocation*(NSString *instanceId, NSString *moduleName, NSString *methodName, NSArray *arguments, NSDictionary *options) {
    WXSDKInstance *instance = [WXSDKManager instanceForID:instanceId];
    WXModuleMethod *method = [[WXModuleMethod alloc] initWithModuleName:moduleName methodName:methodName arguments:arguments options:options instance:instance]; 
   return [method invoke];
}];
```

| Moudule | 能力
| --- | --- |
| WXDomModule | 提供Demo解析能力 |
| WXNavigatorModule | 提供控制UI能力 |
| WXStreamModule | 提供网络请求能力 |
| WXAnimationModule | 提供动画能力
 |
| WXModalUIModule | 提供alert、toast等模态UI展示能力 |
| WXWebViewModule | 提供webview基础能力
 |
| WXInstanceWrap | 提供访问终端instance实例能力 |
| WXTimerModule | 提供定时器能力 |
| WXStorageModule | 提供持久化能力 |
| WXClipboardModule | 提供剪切板能力 |
| WXGlobalEventModule | 提供全局事件(监听通知)能力 |
| WXCanvasModule | 提供绘图能力 |
| WXPickerModule | 提供DatePicker和TimePicker能力 |
| WXMetaModule | 提供设置视口(viewport)能力 |
| WXWebSocketModule | 提供WebSocket能力 |
| WXVoiceOverModule | 提供VoiceOver能力 |

## Component

理解了Module后再来看一下Component。前面提到过Component的主要作用对应UIKit,每一个Component类就与一种UI类型强相关，如tableView、imageView。Component维护了一个生命周期，这一点跟UIViewController有点相似：

![](http://of685p9vy.bkt.clouddn.com/hybird/weex/weex5.jpg)

Component的init方法有许多参数要传，包括样式、属性、事件等都可以在初始化时传入；loadView时，WXComponent的派生类需要返回一个UI类型实例，它会被赋值给Component的view属性，跟Component关联起来；loadView之后会走到addEvent，这里允许我们添加一些自定义的事件（常用的单击、长按等事件已经实现，在初始化时传入即可，不需要操作addEvent方法）；在viewDidLoad中可以对view做个性化的配置，然后启动布局。

Weex允许在view加载出来了以后再去updateStyles/Attributes，JS可以直接访问到这个Component对象。

JS调用Component的原理和Module基本一样，通过注入的callNativeComponent、callUpdateAttrs等一系列方法，在调用过程中生成一个WXComponentMethod对象，然后再利用NSInvocation invoke触达Native。除此之外，JS还可以通过WXComponentManager间接调用Component。

<font color=red>WXComponentManager是Component的调度器，可以直接和JS通信。注入JSContext的方法中与其相关的有callAddElement、callRemoveElement、callAddEvent等,通过这些方法直接调用WXComponentManager即图示中的链路①。而在首屏渲染时通常走的是链路②，即JS Framework在解析JS Bundle时会先访问WXDomModule，然后再由WXDomModule间接地调用WXComponentManager，两种方式其实没有太大差别，在首屏全部使用WXDomModule会更容易监控Dom解析过程而已。</font>

![](http://of685p9vy.bkt.clouddn.com/hybird/weex/weex6.jpg)

addComponent的作用类似于addSubview，WXComponentManager会先用JS传递过来的componentData创建Component对象，然后再把生成的Component添加到它supercomponent的树结构中，同时把Component关联的view加到视图层级上去，之后再对它的children结点递归调用addComponent。

```
- (void)_recursivelyAddComponent:(NSDictionary *)componentData toSupercomponent:(WXComponent *)supercomponent atIndex:(NSInteger)index appendingInTree:(BOOL)appendingInTree
{
    WXComponent *component = [self _buildComponentForData:componentData supercomponent:supercomponent];
    [supercomponent _insertSubcomponent:component atIndex:index];
    if (!component->_isTemplate) {
        [supercomponent insertSubview:component atIndex:index];
    }

    NSArray *subcomponentsData = [componentData valueForKey:@"children"];
    BOOL appendTree = !appendingInTree && [component.attributes[@"append"] isEqualToString:@"tree"];   
     // if ancestor is appending tree, child should not be laid out again even it is appending tree.
    for(NSDictionary *subcomponentData in subcomponentsData){
        [self _recursivelyAddComponent:subcomponentData toSupercomponent:component atIndex:-1 appendingInTree:appendTree || appendingInTree];
    }
    [component _didInserted];  
      if (appendTree) {   
           // If appending tree，force layout in case of too much tasks piling up in syncQueue
        [self _layoutAndSyncUI];
    }
}
```

![](http://of685p9vy.bkt.clouddn.com/hybird/weex/weex7.jpg)

WXComponentManager中会起一个DisplayLink，在每个定时周期循环遍历各个元素，检查是否需要更新布局，需要布局的cssNode会在is_dirty字段标识。<font color=red>如果超过1s没有布局任务，DisplayLink会进入休眠状态直至下一次唤醒。</font>

## WXSDKInstance

<font color=red>在了解过Module和Component的大致原理后，对Weex已经有一个基本认知，但距离整个流程跑通还欠缺一点。分散的Module和Component本身是不会工作的，还需要一个动力，这时我们的WXSDKInstance就要粉墨登场了。</font>

还记得最初的那张Weex框架图吗？ WXSDKInstance（在Model模块里）是整个Weex页面加载的起点，它会去服务端请求JS Bundle，没有JS Bundle我们什么事都干不了！

WXSDKInstance下载JS Bundle后，会把它传给JS Framework，JS Framework解析JS Bundle并通过WXDomModule往RootView上渲染视图。每个WXSDKInstance都会绑定一个独立的WXComponentManager。

用图表示的话，流程大概是下面这样子，WXSDKInstace负责串联各部分模块并带动整个流程：

![](http://of685p9vy.bkt.clouddn.com/hybird/weex/weex8.jpg)

在一些关键的流程上，WXSDKInstance都会有回调，如图上标注出的onCreate()是在下载完JS Bundle，RootView被创建出时回调；renderFinish()是在首屏Dom解析完成后回调；除此之外，还有onFailed()会在加载失败时回调，onJSRuntimeException()在JS执行异常时回调。这些回调都是对外暴露的，我们可以这些回调上做一些定制化的内容。




## 参考

1. [官方教程](http://weex-project.io/cn/guide/)
2. [官方手册](http://weex-project.io/cn/references/)
3. [企鹅电竞weex实践—— iOS SDK的小九九](https://mp.weixin.qq.com/s/Hh3vXjQ8nZDJny1bd0xYOw)
