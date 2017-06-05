---
title: WKWebView+webkit
date: 2017-03-28 9:45:04
tags:
    - WKWebView
    - WebKit
    - JavaScript
categories: 技术分享
---
近年来由于混合编程的发展趋势，越来越多的企业选择通过Native + Hybrid方式来减轻开发压力和减少开发成本 。 iOS8以后，Apple新推出了WKWebview来替换UIWebView。

WKWebview通过WKWebViewConfiguration注册可以被Js调用的方法。
{% codeblock lang:objc %}
WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc] init];
configuration.userContentController = [WKUserContentController new];
[configuration.userContentController addScriptMessageHandler:self name:@"setUserInfo"];
[configuration.userContentController addScriptMessageHandler:self name:@"copyToClipboard"];
[configuration.userContentController addScriptMessageHandler:self name:@"alertScanView"];
{% endcodeblock %}

同时在代理WKScriptMessageHandler中获取Js调用的方法名字
{% codeblock lang:objc %}
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message{
}
{% endcodeblock %}

Native调用Js  （使用字符串拼接Js代码串）
{% codeblock lang:objc %}
NSString *action = [NSString stringWithFormat:@"%@('%@','')",actionName,result];
[self.webView evaluateJavaScript:action completionHandler:^(id _Nullable obj, NSError * _Nullable error) {      
}];
{% endcodeblock %}
