---
title: 通用webview开发记录
date: 2018-02-04 12:07:12
tags: webview
categories: webview
toc: true
---


1. [为什么做通用webView](#webview1)
2. [通用webView与混合式app的联系](#webview2)
3. [WKWebview介绍](#webview3)
4. [通用webView与JS交互形式](#webview4)
5. [现有webView支持那些JS交互](#webview5)
6. [插件的扩展方法](#webview6)
7. [webview中遇到的问题](#webview7)
8. [调试Webview的JS代码方法](#webview8)
9. [参考](#webview9)

<!--more-->

# <div id="webview1"> 1、为什么做通用webView</div> 

1. 两套webview代码：维护复杂、两者交互难。比如：使用在线模板加载美信页面就会遇到问题。
2. JS交互不统一：美信模板普遍使用URL方式于Native交互，在线模板主要使用JSBridge交互，并且不同模块的Bridge也有一定的区别，不是统一的。

为了解决上面问题，就需要融合两套webview，并使用统一的JS交互机制，JS交互机制最后使用的是现有混合app的js交互机制--cordova，同时为了提高性能，统一将UIwebview替换为WKWebview。

所以主要工作有两部分：

1. 使用wkwebview构建新的综合模板
2. 重新规划、开发混合app和Webview的插件

## <div id="webview2"> 2、通用webView与混合式app的联系</div> 

混合app和webview有非常相似的功能，但也有一定的区别，下图的目的是说明它俩的关系：

![webView与混合式app](http://of685p9vy.bkt.clouddn.com/hybird/webview/webview/webview_cordova.png)

图1

混合app和通用webview都是使用WKWebview容器渲染html页面，并使用统一的JS交互机制于Native交互。唯一不同点是：混合app启动的时候将远端的html文件以插件或组件的形式下载到本地，当渲染混合app的时候，根据插件信息，查找本地的html文件，并加载。而通用webview直接使用URL加载部署在远端服务器上的html文件。所以混合app的体验、性能都优于通用webview。
    

## <div id="webview3">3、WKWebview 介绍</div>

iOS8之后苹果推荐使用WKWebView替代UIWebView，其主要的有点有：

1. WKWebView更多的支持HTML5的特性
2. WKWebView更快，占用内存可能只有UIWebView的1/3 ~ 1/4
3. WKWebView高达60fps的滚动刷新率和丰富的内置手势
4. WKWebView具有Safari相同的JavaScript引擎
5. WKWebView增加了加载进度属性
6. 将UIWebViewDelegate和UIWebView重构成了14个类与3个协议

### 3.1 创建WKWebview

WKWebview既可以使用alloc、init方法创建，也可以使用自己的方法创建，下面是创建WKWebview的实例代码：
		
	WKUserContentController* userContentController = [[WKUserContentController alloc] init];
	[userContentController addScriptMessageHandler:self name:CDV_BRIDGE_NAME];
	    
	WKWebViewConfiguration* configuration = [[WKWebViewConfiguration alloc] init];
	configuration.userContentController = userContentController;
	
	WKWebView* wkWebView = [[WKWebView alloc] initWithFrame:frame configuration:configuration];
	
### 3.2 WKWebview加载页面

	WKWebView *webView =self.webViewEngine.engineWebView;
	strUrl = [strUrl stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
	NSURL *url = [NSURL URLWithString:GMK_Str_Protect(strUrl)];
	if (nil == url)
	{
	    [self notFoundHtml];
	    return;
	}
	NSMutableURLRequest* appReq = [NSMutableURLRequest requestWithURL:url
                								cachePolicy:NSURLRequestUseProtocolCachePolicy
	                                                  timeoutInterval:20.0];
	if (nil == appReq)
	{
	    [self notFoundHtml];
	    return;
	}
	    
	[self.webViewEngine loadRequest:appReq];
	
### 3.3 WKWebview代理

WKWebView有两个代理,WKUIDelegate 和 WKNavigationDelegate。WKNavigationDelegate主要处理一些跳转、加载处理操作；WKUIDelegate主要处理UI相关的操作：确认框，警告框，提示框。因此WKNavigationDelegate更加常用。下面是WKNavigationDelegate的代理列表：

	///二、页面是否加载 ，相当于shouldStartLoadWithRequest
	- (void) webView: (WKWebView *) webView decidePolicyForNavigationAction: (WKNavigationAction*) navigationAction decisionHandler: (void (^)(WKNavigationActionPolicy)) decisionHandler
	{
	    //允许跳转
	    decisionHandler(WKNavigationResponsePolicyAllow);
	    //不允许跳转
	    //decisionHandler(WKNavigationResponsePolicyCancel);
	}
	
	///三、页面开始加载
	- (void)webView:(WKWebView*)webView didStartProvisionalNavigation:(WKNavigation*)navigation
	{
	
	}
	///四、收到响应
	- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler 
	{
	    //允许跳转
	    decisionHandler(WKNavigationResponsePolicyAllow);
	    //不允许跳转
	    //decisionHandler(WKNavigationResponsePolicyCancel);
	}
	///五、收到数据
	- (void)webView:(WKWebView *)webView didCommitNavigation:(WKNavigation *)navigation 
	{ 
	}
	///六、页面加载完成
	- (void)webView:(WKWebView*)webView didFinishNavigation:(WKNavigation*)navigation
	{
	}
	///七、页面加载失败
	- (void)webView:(WKWebView*)theWebView didFailNavigation:(WKNavigation*)navigation withError:(NSError*)error
	{
	    
	}
	/// 八、跳转失败
	- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(null_unspecified WKNavigation *)navigation withError:(NSError *)error
	{
	    
	}
	/// 九、重定向
	- (void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(null_unspecified WKNavigation *)navigation
	{
	}
	
	///十、这个目的解决白屏问题  （ios9以上）
	- (void)webViewWebContentProcessDidTerminate:(WKWebView *)webView API_AVAILABLE(macosx(10.11), ios(9.0))
	{
	    [self loadRequestWithUrl:self.strCurrentURL];
	}
	
WKUIDelegate有下面三个代理：

	- (void)                        webView:(WKWebView*)webView
	     runJavaScriptAlertPanelWithMessage:(NSString*)message
	                       initiatedByFrame:(WKFrameInfo*)frame
	                      completionHandler:(void (^)(void))completionHandler
	
	- (void)                        webView:(WKWebView*)webView
	   runJavaScriptConfirmPanelWithMessage:(NSString*)message
	                       initiatedByFrame:(WKFrameInfo*)frame
	                      completionHandler:(void (^)(BOOL result))completionHandler
	
	- (void)                        webView:(WKWebView*)webView
	  runJavaScriptTextInputPanelWithPrompt:(NSString*)prompt
	                            defaultText:(NSString*)defaultText
	                       initiatedByFrame:(WKFrameInfo*)frame
	                      completionHandler:(void (^)(NSString* result))completionHandler
	                      
### 3.3 WKUserContentController


A WKUserContentController object provides a way for JavaScript to post
 messages to a web view.
 
 WKUserContentController 对象为JavaSript提供了向webview发送消息的方式。
 
 
 The user content controller associated with a web view is specified by its
 web view configuration.
 
 WKUserContentController 通过WKWebViewConfiguration 关联到webview。
 
 下面的代码是创建WKUserContentController，并关联到WKWebview的代码：
 
	///创建WKUserContentController
	WKUserContentController* userContentController = [[WKUserContentController alloc] init];
	[userContentController addScriptMessageHandler:self name:CDV_BRIDGE_NAME];
	    
	///关联到webview
	WKWebViewConfiguration* configuration = [[WKWebViewConfiguration alloc] init];
	configuration.userContentController = userContentController
	
关联以后，JS就可以通过下面的代码向WKWebview发送消息。

	window.webkit.messageHandlers.<name>.postMessage(<messageBody>)
	
WKWebview接收到JS的消息后，会触发WKScriptMessageHandler协议的userContentController：didReceiveScriptMessage:方法：

	- (void)userContentController:(WKUserContentController*)userContentController
	  didReceiveScriptMessage:(WKScriptMessage*)message
	{
		if (![message.name isEqualToString:CDV_BRIDGE_NAME]) {
		    return;
		}
		
		CDVViewController* vc = (CDVViewController*)self.viewController;
		NSArray* jsonEntry = message.body; // NSString:callbackId, NSString:service, NSString:action, NSArray:args
	}

在这个方法中就可以根据消息的内容，调用Native的功能。

### 3.4 动态注入JS

WKWebview支持将JS预先注入到Webview中，可以定制带document加载前或者后执行注入的JS。注入JS是将WKUserScript关联的webview的configuration中，代码示例如下。

	/// 4. cookie信息脚本()
	WKUserScript * cookieScript = [[WKUserScript alloc]
                                       initWithSource:mstrCookieInfo
                                       injectionTime:WKUserScriptInjectionTimeAtDocumentStart forMainFrameOnly:NO];
	if (GMK_Ary_Is_Valid(webView.configuration.userContentController.userScripts))
	{
	    [webView.configuration.userContentController removeAllUserScripts];
	}
	if (script)
	{
	    [webView.configuration.userContentController addUserScript:script];
	}

# <div id="webview4"> 4、通用webView与JS交互</div> 
下图是cordova中JS和Native的交互机制
![cordova中JS和Native的交互机制](http://of685p9vy.bkt.clouddn.com/hybird/webview/webview/Cordova_JS.png)

下面是 cordova源码的结构
![cordovad的组织结构](http://of685p9vy.bkt.clouddn.com/hybird/webview/webview/Cordova_class_ship.png)

下面是 cordova初始化webview的流程
![cordovad的组织结构](http://of685p9vy.bkt.clouddn.com/hybird/webview/webview/Cordova_init.png)


## <div id="webview5">5、现有webView支持那些JS交互</div> 

在开发通用webview的时候，统一规划了插件。规划后插件列表如下：

1. cordova-plugin-gome-AddressBook 通信录插件，包含选择联系人接口
2. cordova-plugin-gome-business 业务类型插件，包括上传金融加密 face++，我的国美关闭浮层等接口
3. cordova-plugin-gome-cashier收银台插件
4. cordova-plugin-gome-CustomerService在线客服插件
5. cordova-plugin-gome-device  设备信息插件，包括拨打电话、获取设备信息接口
6. cordova-plugin-gome-location 定位插件，包括获取四级区域信息接口
7. cordova-plugin-gome-login登录插件，包括调起登录，用户信息，是否登录接口
8. cordova-plugin-gome-network 网络请求插件，包括发送网络请求、获取网络连接状态接口
9. cordova-plugin-gome-picture 图片插件，包括调起原生插件、保存图片
10. cordova-plugin-gome-route 路由插件，包括打开vc，关闭当前页面接口
11. cordova-plugin-gome-share 分享插件，包括调起分享组件接口
12. cordova-plugin-gome-shopChat购物车插件，包括刷新购物车数量接口
13. cordova-plugin-gome-statistics统计插件 ，webview埋点时候
14. cordova-plugin-gome-view视图插件，包括toast、 初始化导航栏、错误页面、显示隐藏导航栏接口

所有插件的详细信息请参考：[plugin插件列表](http://m-wiki.ds.gome.com.cn/android/hybrid/Plugins.html)

# <div id="webview6">6、插件的扩展方法</div> 

如果现有的插件不能满足新的需求，需要扩展新的插件，扩展方法如下：

* 首先确定插件的别名，这个名字需要三端一样，然后确定插件的类名（这个按照代码规范命名就可以）。

* 将插件别名和插件类名的对应关系添加到cordova的配置文件中。配置文件查找方法、书写方法如图：
 
![插件扩展方法1](http://of685p9vy.bkt.clouddn.com/hybird/webview/webview/cordova_config1.png)
图1
![插件扩展方法2](http://of685p9vy.bkt.clouddn.com/hybird/webview/webview/cordova_config2.png)
图2
![插件扩展方法3](http://of685p9vy.bkt.clouddn.com/hybird/webview/webview/cordova_config3.png)
图3
		
* 创建插件类，这个类必须继承于CDVPlugin类；

* 定义插件方法，插件方法需要三端统一。方法的格式就是：-(void)<methodName>:(CDVInvokedUrlCommand *)command；

*  实现方法，实现插件方法分3部分：解析参数判断、完成Native的功能、将结果回调给JS；

参数解析：
	
CDVInvokedUrlCommand 类是对JS传递过来参数的封装，我们从arguments属性中获取参数，CDVInvokedUrlCommand的属性定义如下：

	@property (nonatomic, readonly) NSArray* arguments;
	@property (nonatomic, readonly) NSString* callbackId;
	@property (nonatomic, readonly) NSString* className;
	@property (nonatomic, readonly) NSString* methodName;

回调结果：

回调的数据格式需要预先定义好，然后使用下面的代码将数据回调给JS：

	NSDictionary *dic = @{GMM_ViewPlugin_Successed:@"Y"};
	CDVPluginResult * result=[CDVPluginResult resultWithStatus:CDVCommandStatus_OK 
	                                       messageAsDictionary:dic];
	[self.commandDelegate sendPluginResult:result callbackId:command.callbackId];
	
# <div id="webview7">7、webview中遇到的问题</div> 

* _blank 问题：

   a便签中如果有_blank属性，表示在新的窗口打开页面，但是我们打不开，解决办法是添加如下代理的实现，直接在当前窗口加载url

	-(WKWebView *)webView:(WKWebView *)webView createWebViewWithConfiguration:(WKWebViewConfiguration *)configuration forNavigationAction:(WKNavigationAction *)navigationAction windowFeatures:(WKWindowFeatures *)windowFeatures
	{
	    if (!navigationAction.targetFrame.isMainFrame) {
	        [webView loadRequest:navigationAction.request];
	    }
	    return nil;
	}
	
* cookie问题、白屏问题

  这个问题都是参考 [WKWebView 那些坑](http://www.cnblogs.com/NSong/p/6489802.html)解决的，都有不错的解决方法

* NSURLErrorCancelled 问题。

 webview如果上一个请求没有完成，立即操作界面，触发新的网络请求，wk代理会收到错误，错误码是NSURLErrorCancelled，这个时候不应该展示错误页面，需要直接return。
 
*  webview出现错误页面诊断方法 
![插件扩展方法3](http://of685p9vy.bkt.clouddn.com/hybird/webview/webview/cordova_error.png)
遇到这个界面，请先自行定位，定位方法：
  * 首先找到GMBCordovaVC的WK失败代理：
  
			webView: didFailNavigation: withError:
			webView:didFailProvisionalNavigation: withError:
  * 然后打印出错误原因
  * 根据NSError中的错误码分析错误的原因，解决问题。

    下面是我遇到的几个记录下来的错误码，通过分析，大部分能明白错误的原因：
  
	Error Domain=WebKitErrorDomain Code=102 "Frame load interrupted" UserInfo={_WKRecoveryAttempterErrorKey=
	<WKReloadFrameErrorRecoveryAttempter: 0x60000062fa80>, NSErrorFailingURLStringKey=https://login.m.gomeplus.com/login.html?return_url=Ly9jYXNoaWVyLm0uZ29tZXBsdXMuY29tL3ZvdWNoZXJfbGlzdC0wLmh0bWw=, NSErrorFailingURLKey=https://login.m.gomeplus.com/login.html?return_url=Ly9jYXNoaWVyLm0uZ29tZXBsdXMuY29tL3ZvdWNoZXJfbGlzdC0wLmh0bWw=, NSLocalizedDescription=Frame load interrupted}
	
	这个是加载中访问了https://login.m.gomeplus.com/login.html url，打断了加载。

	Error Domain=NSURLErrorDomain Code=-1002 "unsupported URL" UserInfo={_WKRecoveryAttempterErrorKey=<WKReloadFrameErrorRecoveryAttempter: 0x608000233ae0>, NSErrorFailingURLStringKey=itms-appss://itunes.apple.com/cn/app/guo-mei-zai-xian/id486744917?mt=8, NSErrorFailingURLKey=itms-appss://itunes.apple.com/cn/app/guo-mei-zai-xian/id486744917?mt=8, NSLocalizedDescription=unsupported URL, NSUnderlyingError=0x604000653440 {Error Domain=kCFErrorDomainCFNetwork Code=-1002 "(null)”}}
	
	这个是JS中访问了不支持的scheme,itms-appss
	
	Error Domain=NSURLErrorDomain Code=-1202 "The certificate for this server is invalid. You might be connecting to a server that is pretending to be “m.gomeholdings.com” which could put your confidential information at risk." UserInfo={NSURLErrorFailingURLPeerTrustErrorKey=<SecTrustRef: 0x600000301200>, NSLocalizedRecoverySuggestion=Would you like to connect to the server anyway?, NSErrorFailingURLStringKey=https://m.gomeholdings.com/content/2017042516183080569_.html, NSErrorFailingURLKey=https://m.gomeholdings.com/content/2017042516183080569_.html, NSErrorPeerCertificateChainKey=(
	    "<cert(0x7fef30f1fc40) s: *.gomeholdings.com i: Charles Proxy CA (3 \U516b\U6708 2016, bogon)>",
	    "<cert(0x7fef30f12390) s: Charles Proxy CA (3 \U516b\U6708 2016, bogon) i: Charles Proxy CA (3 \U516b\U6708 2016, bogon)>"
	), NSErrorClientCertificateStateKey=0, NSLocalizedDescription=The certificate for this server is invalid. You might be connecting to a server that is pretending to be “m.gomeholdings.com” which could put your confidential information at risk., _WKRecoveryAttempterErrorKey=<WKReloadFrameErrorRecoveryAttempter: 0x600000829e60>, _kCFStreamErrorDomainKey=3, NSUnderlyingError=0x600000a52f90 {Error Domain=kCFErrorDomainCFNetwork Code=-1202 "(null)" UserInfo={_kCFStreamPropertySSLClientCertificateState=0, _kCFNetworkCFStreamSSLErrorOriginalValue=-9813, _kCFStreamErrorDomainKey=3, _kCFStreamErrorCodeKey=-9813}}, _kCFStreamErrorCodeKey=-9813}
	
	这个是认证服务器证书失败。

# <div id="webview8">8、调试Webview的JS代码方法</div>


在项目的过程中，有时会遇到webview刷新不出界面的现象，这时，就需要调试html页面，查找问题，进而判断是否是咱们的问题。调试使用safari自带的功能就可以完成。具体步骤：

* 打开safari开发模式。在safari菜单栏，选择safari、偏好设置、高级，在最下面将 “在菜单中显示开发菜单” 选中。
  ![插件扩展方法3](http://of685p9vy.bkt.clouddn.com/hybird/webview/webview/safari_config_1.png)
* 链接到webview界面。选择开发、Simulator、 对应的页面，打开调试器。
 ![插件扩展方法3](http://of685p9vy.bkt.clouddn.com/hybird/webview/webview/safari_config_2.png)
* 调试代码，查找错误，判断是否是JS的错误
  ![插件扩展方法3](http://of685p9vy.bkt.clouddn.com/hybird/webview/webview/safari_config_3.png)



# <div id="webview9">9、参考</div>

1. [IOS进阶之WKWebView](https://www.jianshu.com/p/4fa8c4eb1316)
2. [iOS 给webView加进度条(WKWebView)](http://blog.csdn.net/pangshishan1/article/details/52212994)
3. [WKWebView 那些坑](http://www.cnblogs.com/NSong/p/6489802.html)
4. [WKWebView强大的新特性](http://www.cocoachina.com/cms/wap.php?action=article&id=21818)

