---
layout: post
title:  "iOS 填坑系列 - 横竖屏切换"
date:   2016-12-22
author: Jumpingfrog0 (黄东鸿）
comments: true
tags: iOS
---
	
## 概述

**写代码就是在不断填坑的过程中慢慢成长，程序员哪有不遇坑的呢？**

这篇文章来谈谈iOS中横竖屏切换的一些坑，横竖屏切换在App中很常见，本来我也以为做这个功能是很简单的一件事，但半年前我在做公司项目的过程中就遇到了不少麻烦，使用了一种比较tricky的方法，在屏幕方向切换时程序偶尔会崩掉，虽然后来经过修改解决了，但导致控制屏幕方向的代码散落在不同角落，不易阅读，维护起来更不方便。前阵子在写一个播放器[JFPlayer](https://github.com/jumpingfrog0/JFPlayer)时，采用了另一种比较好的方法，便在此总结一下各种坑吧。

这里分几种情况：

* 所有界面都支持横竖屏切换
* 只有一个（或几个）界面固定方向，其他界面支持横竖屏切换
* 只有一个（或几个）界面支持横竖屏切换，其他界面固定方向

## 一般情形

### 所有界面都支持横竖屏切换

这是最简单的，只需要在【General】-->【Deployment Info】-->【Device Orientation】勾选上相应地方向就行了。 
![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/Device_Orientation.png)

这样设备是竖屏时所有界面都是竖屏的，设备是横屏时所以界面都是横屏的。

> 注意：
> 
> 这里有一个坑，在 iOS 9 以后，横屏时状态栏会隐藏，如果想要显示状态栏，需要手动控制。
> 
> 在 `Info.plist` 中 设置 `View controller-based status bar appearance` 值为 `YES`，在 view controller 中重写 `prefersStatusBarHidden` 返回 `false`
> 
> ```swift
> override var prefersStatusBarHidden: Bool {
    return false
}
> ```

## 特殊情形

### 只有一个（或几个）界面固定方向，其他界面支持横竖屏切换

其实这种情况一般比较少见，在【General】-->【Deployment Info】-->【Device Orientation】勾选希望支持的方向，然后在需要固定方向的视图控制器中实现如下两个方法即可。

```swift
override var shouldAutorotate: Bool {
    return true
}
    
override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
    return .landscape
}
```
### 只有一个（或几个）界面支持横竖屏切换，其他界面固定方向

这是最常见的情况，大多数App都会是这种需求，比如视频直播类App，只有一个界面需要支持横竖屏，其他界面都是竖屏。

这种情况有两种方法来实现。

#### 方法一

**设置设备仅支持竖屏，监听屏幕旋转的通知，在收到通知后手动旋转视图。**

**不推荐！！**

在【General】-->【Deployment Info】-->【Device Orientation】勾选方向：
![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/Device_Orientation_only_portrait.png)

取消屏幕自动旋转：

```swift
override var shouldAutorotate: Bool {
    return false
}
```

在 `viewDidLoad` 中监听通知：

```swift
NotificationCenter.default.addObserver(self, selector: #selector(didChangeOrientation), name: NSNotification.Name.UIDeviceOrientationDidChange, object: nil)
```

控制视图旋转：

```swift
func didChangeOrientation() {
    if (UIDevice.current.orientation == .portrait) {
        UIView.animate(withDuration: 0.2, animations: {
            self.view.transform = .identity
            self.view.bounds = CGRect(x: 0, y: 0, width: UIScreen.main.bounds.width, height: UIScreen.main.bounds.height)
        })
    } else {
        
        UIView.animate(withDuration: 0.2, animations: { 
            self.view.transform = CGAffineTransform(rotationAngle: CGFloat(M_PI_2));
            self.view.bounds = CGRect(x: 0, y: 0, width: UIScreen.main.bounds.height, height: UIScreen.main.bounds.width)
        })
    }
}
```

##### 填坑一

这种方法需要精确地控制该界面的所有View（包括子View）的旋转以及旋转的方向，设备竖屏时和横屏时旋转的角度不一样，Home键在左和在右旋转的角度也不一样，在界面复杂时，特别麻烦，所以不推荐。

##### 填坑二

因为只设置了竖屏，所以当横屏时，如果有键盘弹出，键盘是竖屏时的样式。

解决办法：在【General】-->【Deployment Info】-->【Device Orientation】中加上横屏时的方向。

> 注意：
> 
> 在仅支持竖屏模式下，不能直接重载`shouldAutorotate`并返回`true`。程序会崩掉，并抛出这个错误
> 
> ```
> Terminating app due to uncaught exception 'UIApplicationInvalidInterfaceOrientation', reason: 'Supported orientations has no common orientation with the application, and [Demo.ViewController shouldAutorotate] is returning YES'
> ```
> 原因是，你如果设备仅支持竖屏，不能在view controller中控制界面自动旋转。



#### 方法二

**设置设备支持横竖屏，让其自动旋转，实现一个基类控制器只支持一个方向，其他固定方向的界面继承基类，同时监听屏幕旋转通知，处理一些特殊需求。**

**强烈推荐！！**

在【General】-->【Deployment Info】-->【Device Orientation】勾选方向：
![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/Device_Orientation.png)

在基类控制器中重写两个控制横竖屏的方法：

```swift
class BaseViewController: UIViewController {
    override var shouldAutorotate: Bool {
        return true
    }
    
    override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
        return .portrait
    }

}
```

##### 填坑一

如果VieController 是放在UINavigationController或者UITabBarController中，需要重写它们的方向控制方法。

```swift
class RootNavigationController: UINavigationController {
    override var shouldAutorotate: Bool {
        guard let topViewController = topViewController else {
            return true
        }
        return topViewController.shouldAutorotate
    }
    
    override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
        guard let topViewController = topViewController else {
            return .all
        }
        return topViewController.supportedInterfaceOrientations
    }
    
    override var preferredStatusBarStyle: UIStatusBarStyle {
        guard let topViewController = topViewController else {
            return .default
        }
        return topViewController.preferredStatusBarStyle
    }
}
```

##### 填坑二

播放器中都会有这样一个功能，点击按钮将界面变成全屏，该怎样做呢？

答案是：**将设备强制横屏，改变状态栏方向**

```swift
func fullScreenButtonPressed(_ button: UIButton?) {
        
    // force change device and status bar orientation, that toggle the UIApplicationDidChangeStatusBarOrientation notification
    if isFullScreen {
        
        UIDevice.current.setValue(UIInterfaceOrientation.portrait.rawValue, forKey: "orientation")
        updateStatusBarAppearanceHidden(false)
        UIApplication.shared.statusBarOrientation = .portrait
    } else {
        
        UIDevice.current.setValue(UIInterfaceOrientation.landscapeRight.rawValue, forKey: "orientation")
        updateStatusBarAppearanceHidden(false)
        UIApplication.shared.statusBarOrientation = .landscapeRight
    }
}
```

其实在 `stackoverflow` 上你能搜到另外一个答案

```objective-c
- (void)interfaceOrientation:(UIInterfaceOrientation)orientation
{
    if ([[UIDevice currentDevice] respondsToSelector:@selector(setOrientation:)]) {
        SEL selector             = NSSelectorFromString(@"setOrientation:");
        NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:[UIDevice instanceMethodSignatureForSelector:selector]];
        [invocation setSelector:selector];
        [invocation setTarget:[UIDevice currentDevice]];
        int val                  = orientation;
        [invocation setArgument:&val atIndex:2];
        [invocation invoke];
    }
}

// 横屏
- (IBAction)landscapAction:(id)sender {
    [self interfaceOrientation:UIInterfaceOrientationLandscapeRight];
}

// 竖屏
- (IBAction)portraitAction:(id)sender {
    [self interfaceOrientation:UIInterfaceOrientationPortrait];
}
```

但我不知道如何将这段代码改为Swift，也不知道这段代码在 iOS 10 下是否仍然奏效。

##### 填坑三

在大多数视频类App中，播放视频的窗口一开始是在界面上方的一小块区域，点击全屏按钮或设备横屏时才全屏的，该如何实现这个功能呢？

可能你们会想，我们可以监听设备旋转的通知，在设备旋转时，判断现在是横屏还是竖屏，然后改变view的约束条件（constraint)来改变大小。

这里有一个小技巧，我们只需要设置好view的长宽比与屏幕的长宽比一致就可以了，一切交给 `auto layout`，不用我们去操心。

```swift
// push入这个控制器的上一个控制器必须只支持竖屏，不然在手机横着时，push入这个控制器时视频的尺寸有问题。
player.snp.makeConstraints { (make) in
    make.top.equalTo(view.snp.top)
    make.left.equalTo(view.snp.left)
    make.right.equalTo(view.snp.right)
    make.height.equalTo(view.snp.width).multipliedBy(UIScreen.main.bounds.width/UIScreen.main.bounds.height)
    // 宽高比也可以为 16:9
}
```

> 注意：
> 
> 使用上述技巧时，push入这个控制器的上一个控制器必须只支持竖屏，不然在手机横着时，push入这个控制器时视频的尺寸有问题。


#### 其他方法

本来还有另外一种方法的，但只在 iOS 9下有效，半年前我用的就是这个方法。我在写这篇文章时重新试了下，发现在iOS 10下已经行不通了，也在这里记录下吧。

在 `AppDelegate` 中声明一个属性 `allowRotation` 来标记是否允许旋转，实现如下方法控制横竖屏：

```swift
func application(_ application: UIApplication, supportedInterfaceOrientationsFor window: UIWindow?) -> UIInterfaceOrientationMask {
    if allowRotation {
        return .landscape
    }
    return .portrait
}
```

然后在控制器的`viewWillAppear`方法中，把 `allowRotation` 设为 `true`，这样界面就会横屏显示了。

```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    
    let appDelegate = UIApplication.shared.delegate as? AppDelegate
    appDelegate?.allowRotation = true
}
```
这里也有一个坑，在view退出前，要把 `allowRotation` 设为 `false`, 不然退出后，之前竖屏显示的界面会变成横屏。

```swift
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    
    let appDelegate = UIApplication.shared.delegate as? AppDelegate
    appDelegate?.allowRotation = false
}
```

其实在 iOS 9下还有其他坑的，需要在这个控制器的界面显示或消失是精确地修改 `allowRotation` 的值，比如在该界面横屏时弹出竖屏的登录窗口、在该界面横屏时退回来竖屏的界面等，不然随时会闪退，总之就是会有一些乱七八糟意料之外的错误出现，但现在我的系统已经升级为 iOS 10 了，已经无法重现了。

**这不是一个好方法，也不推荐使用！！**

## 总结

* 控制横竖屏切换还是使用系统默认的方式最好，不用去操心子view是否旋转，旋转的方向等，也可以避免一些奇奇怪怪的问题发生，少踩一些坑！！故**推荐方法二**。
* iOS 每升级一个版本都会有一些方法不能用，导致频频闪退，巨坑！！
* 使用的方法不对必然会踩一些莫名其妙的坑，在此谨记。
