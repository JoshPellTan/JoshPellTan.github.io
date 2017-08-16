---
layout:     post
title:      iOS 9 Universal links 实践
date:       2017-08-14 08:08:13
summary:    Wellcome To My Blog.
categories: study
thumbnail: cogs
tag:
 - Universal links
 - APP间跳转

 - Universal links 从微信QQ等其他App跳转到你自己APP的实践
---

最近一个多月公司项目UI大改，做的都是大量重复的简单工作，一是没时间二是没有什么新的技术点，唯一遇到的新的值得写个小文章记录一下的就是这个了。

### Universal links简介
Universal links是苹果在WWDC 2015上为iOS 9宣布加入的一个所谓通用链接的深层链接特性，[视频介绍](https://developer.apple.com/videos/play/wwdc2015/509/)和[官方说明文档](https://developer.apple.com/library/content/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW1),官方描述是Deep link，是系统级的跳转功能，通过传统 HTTP 链接来启动 App, 使用相同的网址打开网站和 App。重要的是不需要特别的schema，其他App阻止不了跳转！很多App垂涎已久的流量转化可以实现了！

### 流程简述
现在有一个你公司自己的网页（网址随意，与跳转无关），网页上有一个“打开App”按钮，点击效果是打开一个链接（需要配置，后面详述），若已安装App则直接会打开（苹果系统操作，不用管），若没有则H5延时判断跳转苹果商店，结束。这样一个流程对比使用schema来打开App，一是简化了用户的操作，二是不用获取初始App的授权，简直是广大中小App运营者的福音啊

### Universal links使用和配置
    
想使用Universal links说难也不难，但说简单也不简单，它的不简单是由于需要配置的东西比较多，而且一步都不能少，接下来就给大家说一下需要配置哪些东西。

`1.（服务器端）一个通过https配置的域名，证书最好不要自制，尽量使用第三方证书，测试方法：用Safari打开该域名，如果中间没有警告提示能直接打开网址，说明证书配置成功`

`2.（服务器端）编辑一个JSON内容格式的文件，放到上面配置好的域名子目录下，苹果规定是域名的根目录或者是well-known目录下，测试方法：浏览器打开https://<domain>/apple-app-site-association 或者 https://<domain>/.well-known/apple-app-site-association能看到你的JSON文件的内容，苹果会在适当的时候去访问这个文件，以此来决定你的跳转能否支持，文件内容如下：`

```
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "9JA89QQLNQ.com.apple.wwdc",
                "paths": [ "/wwdc/news/", "/videos/wwdc/2015/*"]
            },
            {
                "appID": "ABCD1234.com.apple.wwdc",
                "paths": [ "*" ]
            }
        ]
    }
}
```
参数说明：`"apps": [],`格式固定，不用修改；`details`：本域名支持跳转的app信息（如公司旗下App不止一个，则需挨个配置）；`appID`:格式为“`teamID`. `bundle id`”（打开苹果开发者中心，登陆自己账号，identifiers->App IDs,点击该App使用的ID即可看到）；`paths`：支持跳转的网址目录，“*”表示配置网址的所有子目录都支持跳转。

这一步有个[官方测试方法](https://search.developer.apple.com/appsearch-validation-tool/)，但仅做参考，因为我配置好App端和服务器端，都能打开App了，这个测试网址还检测我的配置网址有问题。

`3.（App端）打开你项目targets->Capabilities下的Associated Domains开关，并将你前面配置的域名添加到这里，支持添加多个域名，如图：`
<img src="http://cc.cocimg.com/api/uploads/20150902/1441166240882211.png" width="400" height="" alt="photos"/>

`4.（App端）以上东西配置好后一定不要忘了这一步，更新Certificates证书到项目中！删除原有钥匙串管理中的证书，下载安装新的证书`

`5.（App端）这个应该写在第一点，就是本功能仅支持iOS 9 beta 2和Xcode 7 beta 2及以上版本`

`6.(App端)如果前面几步都做正确的话，这时候你已经能从H5打开App了，这时候就需要来实现App端的功能了，如果需求是打开App后会有相关操作，则需要实现**- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler**方法了，通过代码**NSURL *url = userActivity.webpageURL**可以获取本次跳转的链接信息，App端可以和其约定参数进行后续操作`

Universal links工作流程：
<img src="http://cc.cocimg.com/api/uploads/20150902/1441174097574453.png" width="400" height="" alt="photos"/>

以上几步就能确保你的App能在iOS9以后的系统中打开了，enjoy your self!

### Universal links缺点

`1.只适用于iOS 9+，大型App有许多低版本用户，不适用`

`2.总会进入到先前App所在的页面，如果需求是在特定界面，会需要一些额外操作`

`3.必须拥有并维护一个网站，许多小型App或个人开发者无法负担`

### 第三方解决方案

1.**[HOKO](https://hokolinks.com)**（境外服务上商）

2.**[魔窗](http://www.magicwindow.cn)**（境内服务商）

## PS
当你跳转到App后，手机右上角会出现你配置的支持跳转的域名，你如果点击了它，后面的点击跳转都会走Safari，不会再跳App。这是苹果的机制，恢复方法是：在Safari中打开你配置的域名，在页面中下拉一下，页面上方会出现在App打开的提示，点击恢复跳转！





