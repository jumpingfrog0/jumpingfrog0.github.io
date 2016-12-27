---
layout: post
title:  "iOS 填坑系列 - 状态栏变化"
date:   2016-12-23
author: Jumpingfrog0 (黄东鸿）
comments: true
tags: iOS
---

## 概述

相信很多iOS开发者都做过改变状态栏样式和隐藏状态栏，这个功能也挺简单的，但应该也有不少人踩过其中的坑。苹果特别喜欢动不动就改，每个版本都不一样，这个方法这个版本好好的，下个版本就非得用另外一种方式实现才行。而苹果对于状态栏修改的方式一直折腾不休，作为一个经历了从iOS 7 到 iOS 10开发者，在这方面也一直被困扰着，一直不知道到底哪种方式才是正确的，每次都要试一试才敢确定。踩了太多坑，于是在这里总结一下吧。

> 本文是基于 iOS 10的，有些方面对比了 iOS 9 和 iOS 10 的异同，其他不同版本请Google。

## 隐藏和修改样式

在 **iOS 9 和 10** 中，默认情况下，状态栏的改变是通过 `view-controlls` 控制的，由每个控制器决定状态栏的样式以及是否隐藏状态栏。

只需在控制器中重载 `prefersStatusBarHidden` 和 `preferredStatusBarStyle `就可以控制是否隐藏状态栏以及修改状态栏的样式。

```swift
// 隐藏状态栏
override var prefersStatusBarHidden: Bool {
    return true
}

// 修改状态栏的样式为白色
override var preferredStatusBarStyle: UIStatusBarStyle {
    return .lightContent
}
```

### 全局控制状态栏

在 **iOS 9** 中，全局控制状态栏的改变，只需两步：

1. 在`Info.plist`中，添加属性 `View controller-based status bar appearance`，设置为 `NO`
2. 添加如下代码:

```
// 修改状态栏的样式为白色
UIApplication.shared.setStatusBarStyle(.lightContent, animated: true)

// 隐藏状态栏
UIApplication.shared.setStatusBarHidden(true, with: .fade)
```

注意：如果不设置`View controller-based status bar appearance`或设置为 `YES`，则上面两句代码不生效。

在 **iOS 10** 中，上述两个方法已经被弃用了，苹果推荐使用在控制器中重载`prefersStatusBarHidden` 和 `preferredStatusBarStyle `的方法来控制状态栏的改变。但是如果你硬是要使用的话，还是会有效果的，只不过会看到两个警告罢了。

	'setStatusBarStyle(_:animated:)' was deprecated in iOS 9.0: Use -[UIViewController preferredStatusBarStyle]
	'setStatusBarHidden(_:with:)' was deprecated in iOS 9.0: Use -[UIViewController prefersStatusBarHidden]

如果不想看到这两个警告，在 **iOS 10** 中，还有另外一种方式可以全局控制，除了设置`View controller-based status bar appearance`为 `NO`外，需要修改【General】-->【Deployment Info】-->【Status Bar Style】。
![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/status_bar_style.png)

### 改变状态栏的方向

```swift
UIApplication.shared.statusBarOrientation = .portrait
```

## 填坑

### 状态栏改变不了的情况

在一个控制器中添加一个 *ChildViewController* ，如果想在 *ChildViewController* 来控制状态栏，就会失效。

解决办法是：**重载 childViewControllerForStatusBarHidden方法，并在`viewDidLoad`中刷新一次状态栏**

```
class StatusBarHiddenParentController: UIViewController {
    
    var childController: StatusBarHiddenChildController?
    override func viewDidLoad() {
        super.viewDidLoad()
        childController = StatusBarHiddenChildController.fromStoryboard("Main")
        addChildViewController(childController!)
        view.addSubview(childController!.view)
        
        // 刷新状态栏
        setNeedsStatusBarAppearanceUpdate()
    }
    override func childViewControllerForStatusBarHidden() -> UIViewController? {
        return childController
    }
    
    override func prefersStatusBarHidden() -> Bool {
        return false
    }
}

class StatusBarHiddenChildController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }
    override func prefersStatusBarHidden() -> Bool {
        return true
    }
}
```

还有一个需要注意的问题是，当 *ChildViewController* 从父控制器中移除时，需要再次刷新状态栏，具体做法请参考我以前的文章 [iOS Status Bar 的隐藏](http://jumpingfrog0.github.io/2016/2016-03-26-status-bar-hidden/) 。

### 在View中控制状态栏的改变

我在做 [JFPlayer](https://github.com/jumpingfrog0/JFPlayer) 的时候，封装了一个播放视频的View，在播放器从窗口切换为全屏时，需要改变状态栏的方向以及隐藏状态栏。因为是要做成一个简单易用的库，所以要封装的更彻底，不能把细节暴露出去，这时就要在View（而不是控制器）中改变状态栏了，有三个问题需要解决：

* 项目中的`View controller-based status bar appearance`的值是`YES`还是`NO`，基于这个判断是由控制器决定状态栏还是全局控制。
* 如果是控制器决定状态栏，那如何获取该View的父控制器
* 如果是控制器决定状态栏，那控制器如何知道此时是否应该隐藏呢

##### 控制状态栏

```swift
extension UIApplication {
    
    func jf_usesViewControllerBasedStatusBarAppearance() -> Bool {
        let key = "UIViewControllerBasedStatusBarAppearance"
        guard  let object = Bundle.main.object(forInfoDictionaryKey: key) else {
            return true
        }
        
        return (object as! Bool)
    }
    
    func jf_updateStatusBarAppearanceHidden(_ hidden: Bool, animation: UIStatusBarAnimation, fromViewController sender: UIViewController) {
        
        if jf_usesViewControllerBasedStatusBarAppearance() {
            sender.setNeedsStatusBarAppearanceUpdate()
        } else {
            UIApplication.shared.setStatusBarHidden(hidden, with: animation)
        }
    }
}

```

##### 获取View的父控制器

```swift
extension UIView {
    var parentViewController: UIViewController? {
        var parentResponder: UIResponder? = self
        while parentResponder != nil {
            parentResponder = parentResponder!.next
            if let viewController = parentResponder as? UIViewController {
                return viewController
            }
        }
        return nil
    }
}
```

第三个问题，需要借助一个变量 `statusBarIsHidden` 标记下此时状态栏是否该隐藏，并在控制器中重载`prefersStatusBarHidden`方法进行判断。 

实现代码如下。

1) 在点击全屏按钮时，改变状态栏，调用 `updateStatusBarAppearanceHidden`方法更新状态栏

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

fileprivate func updateStatusBarAppearanceHidden(_ hidden: Bool) {
    statusBarIsHidden = hidden
    if let parentViewController = self.parentViewController {
        UIApplication.shared.jf_updateStatusBarAppearanceHidden(self.statusBarIsHidden, animation: .none, fromViewController: parentViewController)
    }
}

```

2) 在控制器中重载 `prefersStatusBarHidden`

```swift
 override var prefersStatusBarHidden: Bool {
    guard let hidden = player?.statusBarIsHidden else {
        return false
    }
    return hidden
}
```