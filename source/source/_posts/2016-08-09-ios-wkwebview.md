---
layout: post
title: 【IOS开发】WKWebView封装APP
date: 2016-08-09 11:11:54
tags:
	- IOS开发
	- Objective-C
	- WKWebView
	- XCode
category: IOS
comments: true
---

一年多没接触xcode了，这一年主要用python做开发，刹一接触xcode代码，还是有点陌生的感觉。在网上闲逛了一通，发现网上的ios教程用swift编写的比oc的多多了。看来苹果的swift推广的比较好。我偶尔写写简单的app，objective-c用过一段时间，这次还是用oc，swift等有时间了好好研究一下。
前段时间有朋友让做一个ipad程序，用webview封装一个网站，实现一个独立的app应用。
功能虽然简单，实现起来发现ios开发的好些功能都有涉及，丢了一年的ios开发中的概念捡起来不易，于是记录下来，以免后面重复造轮子时又忘了。

**主要功能介绍：**
- 自适应iphone和ipad
- 屏幕翻转自适应拉伸
- 自定义导航栏返回按钮
- 网页加载进度条显示
- 主屏幕左滑后退

<!-- more -->

以创建app的流程来编写。
# 一、新建工程
## 1，新建一个Single View Application。
依次点击 File -> New -> Project，选择IOS->Application->Single View Application。
 ![](http://7xo67b.com1.z0.glb.clouddn.com/2016-08-09/ios1.png)

点击Next，设置程序名称和组织名称和标识，选择开发语言为Objective-C，支持设备为Universal（iphone和ipad都支持），其它默认。
 ![](http://7xo67b.com1.z0.glb.clouddn.com/2016-08-09/ios2.png)
点击Next，Create项目。

## 2，程序目录结构
### a, 默认生成的目录结构如下：
 ![](http://7xo67b.com1.z0.glb.clouddn.com/2016-08-09/ios3.png)

### b，storyboard
程序默认创建的Main.storyboard中有一个ViewController，本程序需要导航栏，先拖一个Navigation Controller到storyboard中
 ![](http://7xo67b.com1.z0.glb.clouddn.com/2016-08-09/ios4.png)

删除默认的TableViewController，将Navigation Controller和View Controller关联，拖动首启动箭头从View Controller到Navigation Controller。
 ![](http://7xo67b.com1.z0.glb.clouddn.com/2016-08-09/ios5.png)

# 二，代码实现
浏览器用WKWebView。ViewController头文件如下：
```
//ViewController.h

#import <UIKit/UIKit.h>
@import WebKit;


@interface ViewController : UIViewController<WKUIDelegate, WKNavigationDelegate>

@property (nonatomic,strong)WKWebView *webView;

@end
```
## 1，WKUIDelegate委托
WKWebView默认只能访问https开头的网站，为了能够支持http开头的普通网站，实现下面的委托方法。

```
#pragma mark - WKUIDelegate
//系统阻止了不安全的连接
- (WKWebView *)webView:(WKWebView *)webView createWebViewWithConfiguration:(WKWebViewConfiguration *)configuration forNavigationAction:(WKNavigationAction *)navigationAction windowFeatures:(WKWindowFeatures *)windowFeatures
{
    
    if (!navigationAction.targetFrame.isMainFrame) {
        
        [webView loadRequest:navigationAction.request];
    }
    
    return nil;
}
```

## 2，WKNavigationDelegate委托
didFinishNavigation方法主要作用用来更新导航栏的显示。updateNavigationItems在后续讲解导航栏时介绍。
```
#pragma mark - WKNavigationDelegate
//页面已全部加载
- (void)webView:(WKWebView *)webView didFinishNavigation:(null_unspecified WKNavigation *)navigation {
    [self updateNavigationItems];
}
```

## 3，自定义导航栏
定义变量
```
@property(strong, nonatomic) UIBarButtonItem *navigationBackBarButtonItem;
@property(strong, nonatomic) UIBarButtonItem *navigationRefresheBarButtonItem;
```
创建函数如下：
```
#pragma mark - 导航按钮

//自定义返回按钮
- (UIBarButtonItem *)navigationBackBarButtonItem {
    if (_navigationBackBarButtonItem) return _navigationBackBarButtonItem;
    UIImage* backItemImage = [[[UINavigationBar appearance] backIndicatorImage] imageWithRenderingMode:UIImageRenderingModeAlwaysTemplate]?:[[UIImage imageNamed:@"backItemImage"] imageWithRenderingMode:UIImageRenderingModeAlwaysTemplate];
    UIGraphicsBeginImageContextWithOptions(backItemImage.size, NO, backItemImage.scale);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextTranslateCTM(context, 0, backItemImage.size.height);
    CGContextScaleCTM(context, 1.0, -1.0);
    CGContextSetBlendMode(context, kCGBlendModeNormal);
    CGRect rect = CGRectMake(0, 0, backItemImage.size.width, backItemImage.size.height);
    CGContextClipToMask(context, rect, backItemImage.CGImage);
    [[self.navigationController.navigationBar.tintColor colorWithAlphaComponent:0.5] setFill];
    CGContextFillRect(context, rect);
    UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    UIImage* backItemHlImage = newImage?:[[UIImage imageNamed:@"backItemImage-hl"] imageWithRenderingMode:UIImageRenderingModeAlwaysTemplate];
    UIButton* backButton = [UIButton buttonWithType:UIButtonTypeSystem];
    NSDictionary *attr = [[UIBarButtonItem appearance] titleTextAttributesForState:UIControlStateNormal];

     
   [backButton setAttributedTitle:[[NSAttributedString alloc] initWithString:@"返回" attributes:attr] forState:UIControlStateNormal];
    UIOffset offset = [[UIBarButtonItem appearance] backButtonTitlePositionAdjustmentForBarMetrics:UIBarMetricsDefault];
    backButton.titleEdgeInsets = UIEdgeInsetsMake(offset.vertical, offset.horizontal, 0, 0);
    backButton.imageEdgeInsets = UIEdgeInsetsMake(offset.vertical, offset.horizontal, 0, 0);
   
    [backButton setImage:backItemImage forState:UIControlStateNormal];
    [backButton setImage:backItemHlImage forState:UIControlStateHighlighted];
    [backButton sizeToFit];
    
    [backButton addTarget:self action:@selector(toBack) forControlEvents:UIControlEventTouchUpInside];
    _navigationBackBarButtonItem = [[UIBarButtonItem alloc] initWithCustomView:backButton];
    return _navigationBackBarButtonItem;
}

//刷新按钮
- (UIBarButtonItem *)navigationRefreshBarButtonItem {
    if (_navigationCloseBarButtonItem) return _navigationCloseBarButtonItem;
      
   _navigationCloseBarButtonItem = [[UIBarButtonItem alloc] initWithTitle:@"刷新" style:0 target:self action:@selector(navigationIemHandleRefresh:)];
    return _navigationCloseBarButtonItem;
}
```

按钮点击响应函数
```
- (void)toBack{
    if ([self.webView canGoBack]){
        [self.webView goBack];
    }
}

- (void)navigationIemHandleRefresh:(UIBarButtonItem *)sender {
//    [self.navigationController popViewControllerAnimated:YES];
//    [self dismissViewControllerAnimated:YES completion:NULL];
    [self.webView reload];
}

```

调度函数，由WKNavigationDelegate调起
```
- (void)updateNavigationItems {
    [self.navigationItem setLeftBarButtonItems:nil animated:NO];
    if (self.webView.canGoBack) {// Web view can go back means a lot requests exist.
        UIBarButtonItem *spaceButtonItem = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemFixedSpace target:nil action:nil];
        spaceButtonItem.width = -6.5;
        self.navigationController.interactivePopGestureRecognizer.enabled = NO;
        if (self.navigationController.viewControllers.count == 1) {
            [self.navigationItem setLeftBarButtonItems:@[spaceButtonItem, self.navigationBackBarButtonItem, self. navigationRefreshBarButtonItem] animated:NO];
        } else {
            [self.navigationItem setLeftBarButtonItems:@[self. navigationRefreshBarButtonItem] animated:NO];
        }
    } else {
        self.navigationController.interactivePopGestureRecognizer.enabled = YES;
        [self.navigationItem setLeftBarButtonItems:nil animated:NO];
    }
}
```

## 4，进度条
定义变量
```
@property (nonatomic,strong)UIProgressView *progressView;

- (UIProgressView *)progressView {
    if (_progressView) return _progressView;
    CGFloat progressBarHeight = 2.0f;
    CGRect navigationBarBounds = self.navigationController.navigationBar.bounds;
    CGRect barFrame = CGRectMake(0, navigationBarBounds.size.height - progressBarHeight, navigationBarBounds.size.width, progressBarHeight);
    _progressView = [[UIProgressView alloc] initWithFrame:barFrame];
    _progressView.trackTintColor = [UIColor clearColor];
    _progressView.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleTopMargin;
    return _progressView;
}

```

添加到View
```
- (void)viewWillAppear:(BOOL)animated{
    ...
    [self.navigationController.navigationBar addSubview:self.progressView];
     ...
}

```

## 5，键值观察（Key-value observing,  KVO）
KVO允许一个对象观察另一个对象的属性。该属性值改变时，会通知观察对象。与NSNotificationCenter通知相似，多个KVO观察者可以观察单一属性。此外，KVO更动态，因为它允许对象观察任意属性，而不需任何新的API。

在viewDidLoad函数中添加：
```
    [self.webView addObserver:self forKeyPath:@"estimatedProgress" options:NSKeyValueObservingOptionNew context:NULL];
    [self.webView addObserver:self forKeyPath:@"title" options:NSKeyValueObservingOptionNew context:NULL];
```
定义键值变化的响应代码
```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context{
    if ([keyPath isEqualToString:@"estimatedProgress"]){
        if (object == self.webView){
            [self.progressView setAlpha:1.0f];
            [self.progressView setProgress:self.webView.estimatedProgress];
            if (self.webView.estimatedProgress >= 1.0f){
                [UIView animateWithDuration:0.3 delay:0.3 options:UIViewAnimationOptionCurveEaseIn animations:^{
                    [self.progressView setAlpha:0.0f];
                }completion:^(BOOL finished){
                    [self.progressView setProgress:0.0f];
                }];
            }
        }
        else{
            [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
        }
        
    }
    else if([keyPath isEqualToString:@"title"]){
        if (object == self.webView){
            self.title = self.webView.title;
        }
        else{
            [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
        }
    }
}
```

别忘了在程序退出时注销键值观察
```
- (void)viewWillDisappear:(BOOL)animated{
    [self.webView removeObserver:self forKeyPath:@"estimatedProgress"];
    [self.webView removeObserver:self forKeyPath:@"title"];
}
```

# 三，总结
程序运行效果如下：
 ![](http://7xo67b.com1.z0.glb.clouddn.com/2016-08-09/ios6.png)

IOS开发的流程比较清晰明了，实现界面现在都可以在storyboard中拖拽实现，真正需要代码实现的是一些界面之外的逻辑流程，都可以按模块来各自实现。理解透了，开发起来是比较顺手的。





