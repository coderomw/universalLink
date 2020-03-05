# 项目添加universal-link跳转，升级微信SDK
苹果对还未从UIWebView更新到WKWebView的应用做出了明确规定：新应用最晚于2020年4月份，更新的应用最晚于2020年12月前，都要更新到WKWebView，未更新的应用将会被拒审。<br/>

<img src="https://upload-images.jianshu.io/upload_images/2360306-00c815c04dcb37dc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200" width="30%" /><br/>
而微信从1.8.6版本开始才将UIWebView更换为WKWebView，所以最好将微信SDK升级到1.8.6以上。<br/>
<img src="https://upload-images.jianshu.io/upload_images/2360306-af26e5b63183a69e.png?imageMogr2/auto-orient/strip|imageView2/2/w/658" width="30%" />

<br/>升级微信SDK后会发现注册方法变了，新增了一个参数universalLink。

```
/*! @brief WXApi的成员函数，向微信终端程序注册第三方应用。
 *
 * 需要在每次启动第三方应用程序时调用。
 * @attention 请保证在主线程中调用此函数
 * @param appid 微信开发者ID
 * @param universalLink 微信开发者Universal Link
 * @return 成功返回YES，失败返回NO。
 */
+ (BOOL)registerApp:(NSString *)appid universalLink:(NSString *)universalLink;
```


## universal link是个什么东西呢？

>Seamlessly link to content inside your app, or on your website in iOS 9 or later. With universal links, you can always give users the most integrated mobile experience, even when your app isn’t installed on their device.

universal link是苹果在iOS9上推出的一种能通过https链接跳转APP的功能，可以使用相同的网址打开网址和APP。当你的应用支持universal link，用户点击链接时，会直接跳转到你的APP，而不需要经过Safari。当你的应用不支持时，会打开这个链接显示。比如在Safari和别的应用中点击某个链接，可以直接跳转到你的应用来，可以根据链接携带的信息进行解析，做出相应处理。



## 怎么支持universal link呢？

1.首先要有一个支持https协议的域名，在该域名的根目录上传文件，文件名为apple-app-site-association，没有后缀名，格式为

```
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "477QH88KPY.com.company.aaa",
                "paths": ["/aaa/*"]
            },
            {
                "appID": "477QH88KPY.com.company.bbb",
                "paths": ["/bbb/*"]
            }
        ]
    }
}
```
这里为两个应用，多少个应用可自行添加，appID为teamId.bundleId，paths路径的意思是，在存放apple-app-site-association文件的域名后拼上什么是支持跳转到你APP的，星号代表通配符，比如腾讯的配置，[https://help.wechat.com/apple-app-site-association](https://help.wechat.com/apple-app-site-association)
```
{
    "applinks": {
        "apps": [],
        "details": [{
            "appID": "532LCLCWL8.com.tencent.xin",
            "paths": ["/cgi-bin/newreadtemplate", "/app/*"]
        }, {
            "appID": "88L2Q4487U.com.tencent.xin",
            "paths": ["/cgi-bin/newreadtemplate", "/app/*"]
        }, {
            "appID": "8P7343TG54.com.tencent.wc.xin",
            "paths": ["/cgi-bin/newreadtemplate", "/app/*"]
        }, {
            "appID": "SG5PVJM4JW.com.tencent.wx",
            "paths": ["/cgi-bin/newreadtemplate", "/app/*"]
        }, {
            "appID": "2CJ5YJ6LY5.com.tencent.wechat.xin",
            "paths": ["/cgi-bin/newreadtemplate", "/app/*"]
        }, {
            "appID": "8H3NQF6GGQ.com.tencent.xin",
            "paths": ["/cgi-bin/newreadtemplate", "/app/*"]
        }, {
            "appID": "8P7343TG54.com.tencent.wc.xin.SDKSample",
            "paths": ["/sdksample/*"]
        }]
    }
}
```
<br/><br/>

2.到xcode中配置文件存放的域名，以applinks:开头，拼上域名。当手机下载完应用或者apple-app-site-association文件发生更改时，会去请求这个根目录文件，从而响应支持的跳转格式。<br/>
<img src="https://upload-images.jianshu.io/upload_images/2360306-b21470df3026c010.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200" width="30%" /><br/>
LSApplicationQueriesSchemes中新增weixinULAPI<br/>
<img src="https://upload-images.jianshu.io/upload_images/2360306-95de9e8a313d8180.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200" width="30%" />
<br/>具体细节可以参考微信SDK文档[https://developers.weixin.qq.com/doc/oplatform/Mobile_App/Access_Guide/iOS.html](https://developers.weixin.qq.com/doc/oplatform/Mobile_App/Access_Guide/iOS.html)

<br/>

<br/>

3.这时候跑一下应用，我们可以到Safari测试一下，以微信为例，输入链接[https://help.wechat.com/app/]()，下拉页面，会看到在“微信”中打开（系统iOS9.0以上，微信版本7.0.7及以上）。如果这时候出现了你的应用，说明文件配置成功了。

<img src="https://upload-images.jianshu.io/upload_images/2360306-b7fccd5cf3cac9a8.jpeg?imageMogr2/auto-orient/strip|imageView2/2/w/1200" width="30%"/>



<br/><br/>

4.更新微信SDK至1.8.6及以上版本，到[微信开放平台](https://open.weixin.qq.com)添加好universal link，registerApp的时候赋值上你在微信开放平台填写的universal link。

![](https://upload-images.jianshu.io/upload_images/2360306-1105a227ea7191f9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200)

这里的发送和接收请求的方法新增了回调，表示跳转到微信或者跳转回来是否成功，成功会返回success为YES。

```
/*! @brief 发送请求到微信，等待微信返回onResp
 *
 * 函数调用后，会切换到微信的界面。第三方应用程序等待微信返回onResp。微信在异步处理完成后一定会调用onResp。支持以下类型
 * SendAuthReq、SendMessageToWXReq、PayReq等。
 * @param req 具体的发送请求。
 * @param completion 调用结果回调block
 */
+ (void)sendReq:(BaseReq *)req completion:(void (^ __nullable)(BOOL success))completion;

/*! @brief 收到微信onReq的请求，发送对应的应答给微信，并切换到微信界面
 *
 * 函数调用后，会切换到微信的界面。第三方应用程序收到微信onReq的请求，异步处理该请求，完成后必须调用该函数。可能发送的相应有
 * GetMessageFromWXResp、ShowMessageFromWXResp等。
 * @param resp 具体的应答内容
 * @param completion 调用结果回调block
 */
+ (void)sendResp:(BaseResp*)resp completion:(void (^ __nullable)(BOOL success))completion;
```

<br/><br/>

5.通过universal link从微信跳转回应用时，不再走通过scheme跳转的openUrl:方法了，要实现continueUserActivity方法，判断一下类型。这里的url格式是你在微信后台填写的universal link拼上你的AppID。我这里因为分享和支付用的两个AppID，所以分开处理了一下，交给两个不同的单例，各自实现onResp:的回调。

<img src="https://upload-images.jianshu.io/upload_images/2360306-4e15b4b4e8f10090.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200" width="30%" />

以上。

欢迎共同交流学习。

