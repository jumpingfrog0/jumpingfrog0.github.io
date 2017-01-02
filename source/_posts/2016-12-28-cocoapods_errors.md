---
layout: post
title:  "Cocoapods常见问题总结及解决方法"
date:   2016-12-28
author: Jumpingfrog0 (黄东鸿）
comments: true
tags: iOS
---

`Cocoapods`对 `Swift` 的支持不是很好，每次 `Xcode` 或 `Cocoapods` 版本更新，总有一些意外发生，这里总结下这一年半以来遇到的各种错误及其解决办法，省的下次遇到又要Google一遍。

#### 安装淘宝 `Ruby`源的问题 Error fetching http://ruby.taobao.org/

由于墙的原因，安装 `Cocoapods` 时有可能执行完install命令半天都没反应，这时就需要将 `Ruby` 的默认源 `https://rubygems.org/` 替换成淘宝的源，但会遇到以下错误：

```shell
$ gem sources -a http://ruby.taobao.org/
$ Error fetching http://ruby.taobao.org/:
  bad response Not Found 404 (http://ruby.taobao.org/specs.4.8.gz)
```
这是因为，淘宝把源从 `http` 换成 `https` 了，而网上大多数教程是旧的，没有更新，正确的是：

```shell
$ gem sources -a https://ruby.taobao.org/
```

#### Pod Error in Xcode “Id: framework not found Pods”

执行完 `pod install` 后，打开 *workspace* 文件，编译，遇到以下错误：

```shell
Id: framework not found Pods clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

stackoverflow解决方法的：[http://stackoverflow.com/questions/31139534/pod-error-in-xcode-id-framework-not-found-pods](http://stackoverflow.com/questions/31139534/pod-error-in-xcode-id-framework-not-found-pods)

步骤如下：

* 打开 workspace
* 点击 Xcode 左边的蓝色项目图标
* 在右边选择对应的 **Target**
* 单击 **Genral** 的 tab
* 找到 **Linked Frameworks and Libraries** 的分组
* 删除 Pods frameworks（或 删除**Embedded Binaries**下的**Pod.framework**）
* 关闭 Xcode，运行 **pod update**，
* **Clean**，然后重新 Run/Build

#### 提交App到Itunes Connect时报错 ERROR ITMS-90635

提交 App 到 ITunes Connect 审核时报以下错误：

	ERROR ITMS-90635: "Invalid Mach-O Format. The Mach-O in bundle "XXXX!.app/Frameworks/BRYXBanner.framework" isn’t consistent with the Mach-O in the main bundle. The main bundle Mach-O contains armv7(machine code) and arm64(machine code), while the nested bundle Mach-O contains armv7(bitcode) and arm64(bitcode). Verify that all of the targets for a platform have a consistent value for the ENABLE_BITCODE build setting." 
	WARNING ITMS-90080: "The executable 'Payload/XXXX!.app/Frameworks/Bolts.framework' is not a Position Independent Executable. Please ensure that your build settings are configured to create PIE executables. For more information refer to Technical Q&A QA1788 - Building a Position Independent Executable in the iOS Developer Library."
	
stackoverflow 解决方法：[http://stackoverflow.com/questions/37634627/upload-to-itunesconnect-failing](http://stackoverflow.com/questions/37634627/upload-to-itunesconnect-failing)
	
这是因为 Xcode 7 (或以上）默认支持 `bitcode`，而你的项目关闭了`bitcode`（即在 【Build Settings】 中设置了 【Enable Bitcode】为 **NO**），需要在 `Podfile`中进行额外配置：

	post_install do |installer| 
	  installer.pods_project.targets.each do |target| 
	    target.build_configurations.each do |config| 
	      config.build_settings['ENABLE_BITCODE'] = 'NO' 
	    end 
	  end 
	end
	
#### 升级Xcode8后报错“target overrides the `EMBEDDED_CONTENT_CONTAINS_SWIFT`build setting”

升级 Xcode 8后，运行 `Pod install`， 报以下错误：

```
[!] The X target overrides the `EMBEDDED_CONTENT_CONTAINS_SWIFT` build setting defined in `X’. This can lead to problems with the CocoaPods installation
- Use the `$(inherited)` flag, or
- Remove the build settings from the target.

[!] The `X` target overrides the `ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES` build setting defined in `X'. This can lead to problems with the CocoaPods installation
- Use the `$(inherited)` flag, or
- Remove the build settings from the target.
```

stackoverflow 解决方法：[http://stackoverflow.com/questions/39569743/errors-after-updating-to-xcode-8-no-such-module-and-target-overrides-the-em](http://stackoverflow.com/questions/39569743/errors-after-updating-to-xcode-8-no-such-module-and-target-overrides-the-em)

步骤如下：

* 找到 【Project/Targets】-->【Project Name】-->【Build Settings】
* 搜索 "ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES"
* 右键 "debug"，选择 "other"，输入 **$(inherited)**
* "release" 也做同样的操作，然后重新 "pod install"

如下图：
![](https://i.stack.imgur.com/DF001.png)

#### 指定编译版本

指定Target的 Swift 编译版本为 3.0

```
post_install do |installer|
    installer.pods_project.targets.each do |target|
        target.build_configurations.each do |configuration|
            configuration.build_settings['SWIFT_VERSION'] = "3.0"
        end
    end
end
```

但如果是 Objective-C 跟 Swift 混编的项目, 想要引入 OC 的第三方库的话, 还需要添加另一项参数

	configuration.build_settings['ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES'] = 'NO'
	