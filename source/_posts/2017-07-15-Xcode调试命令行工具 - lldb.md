
---
layout: post
title:  "Xcode调试命令行工具 - lldb"
date:   2017-7-15
author: Jumpingfrog0 (黄东鸿）
comments: true
tags: iOS
---

## Xcode调试命令行工具 - lldb

`LLDB`是XCode内置的为我们开发者提供的调试工具，可以在设置断点的时候在控制台中输入相关的lldb命令进行调试。

### help

* help ：列出所有的命令
* help \<command> : 列出某个命令更多的细节，例如 help print

### print

* print \<expr> : 打印需要查看的变量，简写 p
* call 与 print 命令功能一样
* po \<expr> : 打印对象的 description 方法的结果
* 打印不同格式可以用 p/x number 打印十六进制，p/t number 打印二进制，p/c char 打印字符。这里是完整清单 [https://sourceware.org/gdb/onlinedocs/gdb/Output-Formats.html](https://sourceware.org/gdb/onlinedocs/gdb/Output-Formats.html)

```
(lldb) p (int)[[[self view] subviews] count]
```

最后你看到的输出会是：

	(int) $2 = 2

你会发现输出的信息中带有$1、$2的字样。实际上，lldb 的每次查询结果会保存在一些持续变量中，($[0-9]+)，这样你可以在后面的查询中直接使用这些值。比如现在我接下来要重新取回$1的值：

```
(lldb) po $1
(UIView *) $1 = 0x0824c800 <UITableView: 0x824c800; frame = (0 20; 768 1004); clipsToBounds = YES; autoresize = W+H; gestureRecognizers = <NSArray: 0x74c3010>; layer = <CALayer: 0x74c2710>; contentOffset: {0, 0}>
```

### expression

* expression \<cmd-options> -- \<expr> : 声明一个临时变量或改变一个变量的值，简写 expr 或 e

```
// 把self.str 的值改为 "111"
(lldb) e self.str = @"111";
```

```	
// 声明一个临时变量str2
(lldb) e NSString *$str2 = @"222"`
```

> **注意：要使用 `$` 符号**

### image

image 命令可用于寻址，有多个组合命令。比较实用的用法是用于寻找栈地址对应的代码位置。比如下面一段代码：

	NSArray *array = @[@"1", @"2"];
    NSLog(@"%@", array[2]);
    
这段代码有明显的错误，程序运行这段代码后会抛出下面的异常：

```
2017-07-14 22:26:40.836 octest[34972:1928364] *** Terminating app due to uncaught exception 'NSRangeException', reason: '*** -[__NSArrayI objectAtIndex:]: index 2 beyond bounds [0 .. 1]'
*** First throw call stack:
(
	0   CoreFoundation                      0x000000010d0c6b0b __exceptionPreprocess + 171
	1   libobjc.A.dylib                     0x000000010cb2b141 objc_exception_throw + 48
	2   CoreFoundation                      0x000000010d00438b -[__NSArrayI objectAtIndex:] + 155
	3   octest                              0x000000010c55ac29 -[ViewController buttonDidClick:] + 153
	4   UIKit                               0x000000010d4ebd82 -[UIApplication sendAction:to:from:forEvent:] + 83
	5   UIKit                               0x000000010d6705ac -[UIControl sendAction:to:forEvent:] + 67
	6   UIKit                               0x000000010d6708c7 -[UIControl _sendActionsForEvents:withEvent:] + 450
	7   UIKit                               0x000000010d66f802 -[UIControl touchesEnded:withEvent:] + 618
	8   UIKit                               0x000000010d5597ea -[UIWindow _sendTouchesForEvent:] + 2707
	9   UIKit                               0x000000010d55af00 -[UIWindow sendEvent:] + 4114
	10  UIKit                               0x000000010d507a84 -[UIApplication sendEvent:] + 352
	11  UIKit                               0x000000010dceb5d4 __dispatchPreprocessedEventFromEventQueue + 2926
	12  UIKit                               0x000000010dce3532 __handleEventQueue + 1122
	13  CoreFoundation                      0x000000010d06cc01 __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 17
	14  CoreFoundation                      0x000000010d0520cf __CFRunLoopDoSources0 + 527
	15  CoreFoundation                      0x000000010d0515ff __CFRunLoopRun + 911
	16  CoreFoundation                      0x000000010d051016 CFRunLoopRunSpecific + 406
	17  GraphicsServices                    0x0000000110e9ba24 GSEventRunModal + 62
	18  UIKit                               0x000000010d4ea134 UIApplicationMain + 159
	19  octest                              0x000000010c55b6df main + 111
	20  libdyld.dylib                       0x000000010ff3065d start + 1
)
libc++abi.dylib: terminating with uncaught exception of type NSException
```

我们可以看到出错是在堆栈信息的第3行提示的 `buttonDidClick` 方法中，我们怀疑出错的地址是 0x000000010c55ac29，可以根据执行文件名判断，或者最小的栈地址）。为了进一步精确定位，我们可以输入以下的命令：

	(lldb)image lookup --address 0x000000010c55ac29
	
命令执行后返回：

	Address: octest[0x0000000100001c29] (octest.__TEXT.__text + 473)
    Summary: octest`-[ViewController buttonDidClick:] + 153 at ViewController.m:61
    
我们可以看到，出错的位置是 ViewController.m 的第61行。

### 无法确定类型

```
(lldb) e NSArray *$array = @[ @"Saturday", @"Sunday", @"Monday" ]
(lldb) p [$array count]
2
(lldb) po [[$array objectAtIndex:0] uppercaseString]
SATURDAY
(lldb) p [[$array objectAtIndex:$a] characterAtIndex:0]
error: no known method '-characterAtIndex:'; cast the message send to the method's return type
error: 1 errors parsing expression
```

LLDB 有时无法确定返回的类型，这种事情常常发生，手动指定类型就好了：

	(lldb) p (char)[[$array objectAtIndex:$a] characterAtIndex:0]
	'M'

### 断点管理

* breakpoint list ： 可以查看所有断点， 简写 br li

#### 设置断点触发条件

断点可以设置条件，只有当条件满足时，才会进入断点，并且可以设置进入断点时执行某些操作，比如打印log，执行lldb命令等。

这种应用场景主要是在循环遍历时，想要断点跟踪就只能通过这种方式了，除非添加NSLog打印，但是这种需要手动添加代码，在调试时才想到要添加一些打印语句，这时候又得重新运行，这太慢了，所以懂得设置断点触发条件将会大大提高效率。

操作如下：

断点触发条件（打印log）
![](http://os3yasu4i.bkt.clouddn.com/QQ20170714-202920.png)

断点触发条件（执行lldb命令）
![](http://os3yasu4i.bkt.clouddn.com/QQ20170714-203055.png)

### 打印视图层次结构

	(lldbd) po [self.view recursiveDescription]
	
### 更新UI

	(lldb) e ((UIButton *)sender).backgroundColor = [UIColor yellowColor]
	(UICachedDeviceRGBColor *) $0 = 0x000061800007e280
	(lldb) e (void)[CATransaction flush]
	
### 流程控制

* continue : 继续执行下去到达下一个断点(process continue)，或者使用缩写 c
* next : 单步执行到下一个语句(process step-over)，缩写 n
* step : 跳进一个函数调试(process step-into)，缩写 s
* finish : 继续执行到下一个断点或返回语句，然后再次停止(process step-out)
* return \<RETURN EXPRESSION> : 会在当前断点处直接返回出函数，函数剩余部分不会被执行(process \<RETURN EXPRESSION>)

### 查找Button的target

查看按钮按下后谁会接收到按钮发出的action
	
	(lldb) po [sender allTargets]
	{(
	    <ViewController: 0x7fa6415083f0>
	)}
	
	(lldb) po [sender actionsForTarget:(id)0x7fa6415083f0 forControlEvent:UIControlEventTouchUpInside]
	<__NSArrayM 0x600000056e60>(
	buttonDidClick:
	)

更多命令参考：[The LLDB Debugger](http://lldb.llvm.org/lldb-gdb.html)

另外，facebook 开源了一个扩展的 [LLDB命令库](https://github.com/facebook/chisel)，有兴趣可以看看。

--------
参考文档：

[LLDB调试命令初探](http://www.starfelix.com/blog/2014/03/17/lldbdiao-shi-ming-ling-chu-tan/)

[与调试器共舞 - LLDB 的华尔兹](https://objccn.io/issue-19-2/)
