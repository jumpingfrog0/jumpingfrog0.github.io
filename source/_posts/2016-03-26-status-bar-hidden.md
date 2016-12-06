----
title: iOS Status Bar 的隐藏
----

自己一个人负责项目的好处是：代码想怎么写就怎么写，然而坏处是：代码要多渣有多渣，踩坑也是必须的

iOS开发也已有一年半，踩过的坑实在是太多了，之前只是简单的记录下笔记，现在开始整理总结成文章，希望形成一套自己的理论体系。

--------------------------------------  我是一条华丽的分隔线 --------------------------------------

之前一直以为，隐藏 *status bar* 只是一句简单的语句调用就搞定了，但在做公司的项目的过程中，发现居然隐藏不了，纳闷了许久后算是明白了点，没想到也是蛮多坑的。

> 以下内容均是基于 **iOS 9** 及其以上的，如有出入，请善用搜索引擎

### Status Bar 的正常隐藏

###### view-controllers 控制 status bar 的隐藏

在iOS 9中，*status bar* 的隐藏默认是通过 *view-controlls* 控制的，即每个控制器决定是否隐藏 *status bar* 。

只需在 *controller* 中重载 `prefersStatusBarHidden` 函数

```swift
override func prefersStatusBarHidden() -> Bool {
    return true
}
```

###### 全局控制 status bar 的隐藏

如果想要全局控制，只需两步：

* 在Info.plist中，添加属性 `View controller-based status bar appearance` 为 `NO`
* 添加如下代码
  
  ``` swift
  UIApplication.sharedApplication().statusBarHidden = true
  ```
  
如果想改成每个控制器自行控制，将 `View controller-based status bar appearance` 为 `YES`即可

### Status Bar 隐藏不了的情况

但是，有时候也会有意外发生，上述方法并不能如愿隐藏 *status bar* ,那就是 **在*ParentViewController* 中添加一个全屏的 *ChildViewController* ，此时想用 *ChildViewController* 来控制状态栏时** ，就会失效，即使 *ChildViewController* 中的`prefersStatusBarHidden`方法返回的是`YES`，也无法隐藏 *status bar* 。

**解决办法是：重载 `childViewControllerForStatusBarHidden`方法**

苹果官方文档 [UIViewController Class Reference](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewController_Class/index.html#//apple_ref/occ/instm/UIViewController/childViewControllerForStatusBarHidden) 是这样解释的：
> Called when the system needs the view controller to use for determining status bar hidden/unhidden state.
> 
> Return Value:
> 
>  The view controller whose status bar hidden/unhidden status should be used. Default return value is nil.

> Discussion:
> 
If your container view controller derives derives the hidden state of the status bar from one of its child view controllers, implement this method to specify which child view controller you want to control the hidden/unhidden state. If you return nil or do not override this method, the status bar hidden/unhidden state for self is used.

> If you change the return value from this method, call the `setNeedsStatusBarAppearanceUpdate` method.

大概意思就是，如果你想要让你的 *container view controller* 的 *child view controller* 控制 *status bar* 的隐藏状态的话，就重载该方法，决定使用哪个 *child view controller* 来控制 *隐藏/非隐藏* 的状态。如果返回 *nil* 或不重载该方法，就用它自己来控制 *status bar* 的状态。可以通过调用 `setNeedsStatusBarAppearanceUpdate` 方法来改变该方法返回的值，即再调用该方法一次。

```swift
class StatusBarHiddenParentController: UIViewController {
    
    var childController: StatusBarHiddenChildController?

    override func viewDidLoad() {
        super.viewDidLoad()

        childController = StatusBarHiddenChildController.fromStoryboard("Main")
        addChildViewController(childController!)
        view.addSubview(childController!.view)
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

上述代码最终能将状态栏隐藏，即使在 *StatusBarHiddenParentController* 中的`prefersStatusBarHidden`返回的是`NO`

注意：因为`childViewControllerForStatusBarHidden`会比`viewDidLoad` 先调用，所以在`viewDidLoad`中要调用`setNeedsStatusBarAppearanceUpdate`

有时候我们需要关闭控制器视图，如下列三种情况：

* 将控制器从navigationController的堆栈中pop出去
* 将present出来的控制器dismiss掉
* 将子控制器从父控制器中移除

*push* 和 *present* 两种方式展示的 *view controller* 在 *pop* 和 *dismiss* 时都能够自动还原 *status bar* 的状态。但是，把子控制器从父控制器中移除时，就会出现奇怪的问题， *status bar* 并不能自动还原，因此还需要特别处理。

我们可以设置一个Bool变量 `statusBarHidden` 用来记录目前 *statusBar* 是否隐藏，在 `prefersStatusBarHidden` 函数中返回 `statusBarHidden`值。在要移除子控制器的函数中做两步操作：

1. 先将 `statusBarHidden` 设置为 `False`, 并调用 `setNeedsStatusBarAppearanceUpdate` 刷新状态栏
2. 再将子控制器移除。

```swift
class StatusBarHiddenRemoveWayChildController: UIViewController {
    
    var statusBarHidden: Bool = true
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let removeButton = UIButton(frame: CGRectMake(30, 80, 100, 30))
        removeButton.setTitle("remove", forState: .Normal)
        removeButton.addTarget(self, action: "remove", forControlEvents: .TouchUpInside)
        view.addSubview(removeButton)
    }
    
    // 1. set `statusBarHidden` into `false`, and then refresh status bar
    // 2. remove from parent controller
    
    func remove() {
        statusBarHidden = false
        setNeedsStatusBarAppearanceUpdate()
        
        view.removeFromSuperview()
        removeFromParentViewController()
    }
    
    override func prefersStatusBarHidden() -> Bool {
        return statusBarHidden
    }
}
```

### 使用 Method Swizzling 隐藏 Status Bar

对于 *status bar* 隐藏不了的情况，除了上面介绍的方法，iOS 中还有一种强大的黑魔法 *[Method Swizzling](http://nshipster.com/method-swizzling/)*，这里设计到了 **Objective-C** 语言的 *[runtime](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/func/method_getImplementation)* 特性，这是一个值得深入学习和研究的知识。

我们只需要在 *ChildeViewController* 中对 *ParentViewController* 的 `prefersStatusBarHidden` 方法进行 *Hook* ，然后偷天换日，换成我们自己实现的方法，使其返回 `true` ，然后刷新状态栏，就可以隐藏 *status bar* 了。

代码如下：

```swift

class StatusBarHiddenSwizzlingChildController: UIViewController {

	override func viewWillAppear(animated: Bool) {
        super.viewWillAppear(animated)
        
        guard let parentViewController = parentViewController else {
            return
        }
        
        if parentViewController.respondsToSelector("setNeedsStatusBarAppearanceUpdate") {
            
            hookPrefersStatusBarHidden(parentViewController)
        }
    }
    
    func hookPrefersStatusBarHidden(parentViewController: UIViewController) {
        
        let originalSelector = Selector("prefersStatusBarHidden")
        let swizzledSelector = Selector("hook_prefersStatusBarHidden")
        
        let originalMethod = class_getInstanceMethod(parentViewController.dynamicType, originalSelector)
        let swizzledMethod = class_getInstanceMethod(self.dynamicType, swizzledSelector)
        
        let didAddMethod: Bool = class_addMethod(parentViewController.dynamicType,
            originalSelector,
            method_getImplementation(swizzledMethod), method_getTypeEncoding(swizzledMethod))
        
        if didAddMethod {
            class_replaceMethod(self.dynamicType,
                swizzledSelector,
                method_getImplementation(originalMethod),
                method_getTypeEncoding(originalMethod))
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod)
        }
        
        dispatch_async(dispatch_get_main_queue()) { () -> Void in
        
            parentViewController.prefersStatusBarHidden()
            parentViewController.setNeedsStatusBarAppearanceUpdate()
        }
    }
    
    // must recover the hook when view will disappear
    
    override func viewWillDisappear(animated: Bool) {
        super.viewWillDisappear(animated)
        
        guard let parentViewController = parentViewController else {
            return
        }
        
        if parentViewController.respondsToSelector("setNeedsStatusBarAppearanceUpdate") {
            
            hookPrefersStatusBarHidden(parentViewController)
        }
        
    }
    
    func hook_prefersStatusBarHidden() -> Bool {
        return true
    }
}
```

> 注意事项：
> 
> * 必须在 *ChildViewController* 的 `viewWillDisappear` 中将 *Hook* 的方法还原回来，否则可能会出现奇怪的问题。
> * *ParentViewController* 中尽可能重载 `prefersStatusBarHidden` 方法，因为 *Method Swizzling* 有很多细节需要谨慎处理，如果 `hookPrefersStatusBarHidden` 写的有bug，会导致奇怪的问题。


### 滚动ScrollView或TableView时隐藏Status Bar和Navigation Bar

京东和淘宝客户端的商品搜索结果页面，在滚动时可以隐藏 *status bar* 和 *navigation bar* ，我自认为这是一个很好的设计，身为一个有追求的程序员，这个功能的实现那当然也不能放过啦~~

*NavigationController* 有一个属性 `hidesBarsOnSwipe`，可以实现轻扫时隐藏 *navigation bar* ，与之对应的手势是它的另一个属性`barHideOnSwipeGestureRecognizer`，只要给这个手势添加一个方法来控制 *status bar* ，就可以实现同时隐藏 *status bar* 和 *navigation bar* 了。

```swift
class StatusBarAndNavigationBarHiddenOnSwipeController: UITableViewController {
    
    var hideStatusBar = false

    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.backgroundColor = UIColor.grayColor()
        
        navigationController?.barHideOnSwipeGestureRecognizer.addTarget(self, action: "swipe:")
        navigationController?.hidesBarsOnSwipe = true
    }
    
    override func prefersStatusBarHidden() -> Bool {
        return hideStatusBar
    }
    
    func swipe(recognizer: UISwipeGestureRecognizer) {
        
        hideStatusBar = navigationController?.navigationBar.frame.origin.y < 0
        
        UIView.animateWithDuration(0.2) { () -> Void in
            
            self.setNeedsStatusBarAppearanceUpdate()
        }
    }
    
    override func preferredStatusBarUpdateAnimation() -> UIStatusBarAnimation {
        return .Slide
    }
}
```

本文章所有示例代码下载地址：[Demo](/download/2016-03-26-status-bar-hidden-demo.zip)

-------
扩展阅读：[SWIZZLE](http://swifter.tips/swizzle/)