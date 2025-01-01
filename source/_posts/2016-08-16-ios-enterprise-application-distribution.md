---
layout: post
title:  "iOS 企业内部(In-House)应用分发"
date:   2016-08-16
author: Jumpingfrog0 (黄东鸿）
comments: true
tags: iOS
---

去年8月份的时候由于公司的需求，开发了一个只在司机间使用的企业内部的iOS应用，当时就打算写一篇文章来总结一下了，然而我是重度拖延症患者，一直拖着没完成。一年过去了，今天才想起来这回事，才开始动手写这篇文章。

## 企业内部应用

企业内部应用，即只在企业部门和员工内部使用、不对外公开的应用。苹果提供了专门的`In-House`证书用来发布这种应用，可以分发给任意的手机，只要通过一个URL即可下载安装，不用上传到`App Stroe`审核。我把企业内部应用也叫做`In-House`应用。

`In-House`应用，有时需要根据部门需求进行版本的快速迭代，因为不需要`App Store`审核，所以可以做到随时修改，随时发布，节省了大量的时间。`In-House`证书还可以用于应用的内测分发，现在大部分的内测分发平台如蒲公英，Fir.im等的公测功能就是使用`In-House`证书实现的。

#### 必须具备的两个条件：

* 企业开发者账号。99$的普通开发者账号不行，必须以企业的名义申请一个299$的企业开发者账号`Apple Developer Enterprise Program`
* 带SSL证书的域名。企业内部应用需要把ipa文件上传到服务器，然后通过一个链接来下载安装，而苹果很重视安全性，要求这个链接的域名必须具有SSL证书，支持 https ，否则无法安装。

SSL证书其实并不是必需的，可以使用一些知名的云存储服务，比如亚马逊的AWS，阿里云等，这些大公司的云存储都支持Https，我用的就是AWS的S3云存储，但299$的企业开发者账号就避免不了。

#### 准备的文件

* `ipa`文件。
* `plist`文件。名称必须与`ipa`文件一致，用于配置bundle id、版本号、`ipa`文件的URL、应用图标等。
* @1x 和 @2x 的Icon。下载安装时显示应用图标。

## 打包

1. 创建发布证书(**Production Certificates**)，选择`In-House`类型的，过程我就不赘述了，和其他证书一样。
![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/BF3C42FA-2DBD-4CD2-9B0F-F1A7FB8EBBF9.png)
2. 创建配置文件(**Distribution Provisioning Profiles**) 
![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/61A9C0ED-8A61-455F-803D-27F2F80BD358.png)
![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/F5FE4D91-A39D-43BE-9173-9583DC027F89.png)
3. 在Xcode选择对应的Code Signing 和 Provisioning Profile, Archive
![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/54CDD771-A9ED-4A79-AA73-C5DA1DE9B6A2.png)
4. 导出 ipa 文件

![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/C2A751F9-A8B6-4888-877A-62DEB4F96A3D.png)

## plist 文件

Xcode 5 及其以前打包`In-House`应用会一起生成`ipa`和`plist`文件，但Xcode 6 以后就只有`ipa`文件了，所以要手动生成 `plist`文件。
文件格式如下：

```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
	<dict>
		<key>items</key>
		<array>
			<dict>
				<key>assets</key>
				<array>
					<dict>
						<key>kind</key>
						<string>software-package</string>
						<key>url</key>
						<string>[INSERT THE URL FOR YOUR IPA HERE. e.g : https://s3-us-west-2.amazonaws.com/folder/appName-version.ipa]</string>
					</dict>
					<dict>
						<key>kind</key>
						<string>full-size-image</string>
						<key>needs-shine</key>
						<true/>
						<key>url</key>
						<string>[INSERT THE URL FOR INSTALLATION @2x ICON HERE. e.g : https://s3-us-west-2.amazonaws.com/folder/images/Icon@2x.png]</string>
					</dict>
					<dict>
						<key>kind</key>
						<string>display-image</string>
						<key>needs-shine</key>
						<true/>
						<key>url</key>
						<string>[INSERT THE URL FOR INSTALLATION ICON HERE. e.g : https://s3-us-west-2.amazonaws.com/folder/images/Icon.png]</string>
					</dict>
				</array>
				<key>metadata</key>
				<dict>
					<key>bundle-identifier</key>
					<string>[INSERT BUNDLE ID HERE]</string>
					<key>bundle-version</key>
					<string>[INSERT VERSION HERE]</string>
					<key>kind</key>
					<string>software</string>
					<key>title</key>
					<string>[INSERT APP TITLE HERE. The Title will present to the user installing the app]</string>
				</dict>
			</dict>
		</array>
	</dict>
	</plist>
```

在`plist`文件中输入`ipa`的URL、安装时显示的 icon 的url、bundle id、版本号、安装前的提示信息。

## 发布与安装

#### 发布

把`ipa`、配置好的`plist` 文件和图标一起上传AWS的S3云存储上即可。

#### 安装

iOS的企业内部应用是通过访问`plist`文件来安装的，因为`plist`文件中包含了对应的`ipa`文件和图标的URL，iPhone会根据URL自动下载并安装应用程序。

在iPhone的`Safari`浏览器中输入：

	itms-services://?action=download-manifest&url=https://s3-us-west-2.amazonaws.com/folder/appName-version.plist
	
首先会询问是否打开要打开链接，点击“打开”

![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/515BAA10430C80AC9421639148B387E3.jpg)

然后询问是否要安装App，点击“安装”

![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/ACEEDF6B084EAC46F70F6D1494CB5EDF.jpg)

## 自动更新

为了避免每次发布后都需要通知别人更新App的麻烦事，自动更新是必备的。与后台沟通，设计一个更新接口`GET /updates`

###### Query Parameters

| name | required | type | description |
| ----- | -------- | ---- | ----------- |
| platform | YES | String | 平台，ios / android| 
| version | YES | String | 当前版本号，比如：1.0.0 |

###### Response

Status-Code: 200 OK

	{
  		"data": {
  			"update" : true,
  			"lastest" : "1.0.1",
  			"url" : "itms-services://?action=download-manifest&url=https://s3-us-west-2.amazonaws.com/folder/appName-1.0.1.plist"
    	},
  		"error": {}
	}
	
App一启动时，调用`GET /updates`接口传递平台参数和当前版本号给后台进行检查，后台判断当前版本是否为最新版，如果不是最新版，则返回最新版本号和对应的下载链接，然后用浏览器打开返回的URL进行安装即可。

代码如下：

```swift
func upgrade()
    {
        let params = [
        	"platform" : "ios", 
        	"version" : CKAppInfo.releaseVersion()
        ]
        ObjectRequest(URLRequest: UpdateRouter.CheckUpdate(params)).load(
            successHandler: { (promotion: Promotion?) -> Void in
                SWLog(promotion)
                
                if promotion?.update == true { // different version
                    
                    let alert = DVAlertController(title: "Upgrade", message: "update to latest version", preferredStyle: UIAlertControllerStyle.Alert)
                    alert.addAction(UIAlertAction(title: "Upgrade", style: UIAlertActionStyle.Default, handler: { (_) -> Void in
                        let url = NSURL(string:promotion!.url!)
                        UIApplication.sharedApplication().openURL(url!)
                    }))
                    
                    alert.show(animated: true, completion: nil)
                    SWLog("different version")
                }
            }, failHandler: { (error) -> Void in
                self.creatAlert("checkout upgrade failed")
        })
    }
```

## 不信任的企业开发者

如果你的手机是 iOS 9 系统，那么第一次打开企业级应用时，你并不能打开App，而是看到下面的信息：

![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/iphone6-ios9-enterprise-untrusted_enterprise_app.png)

苹果是一个极其重视安全性的公司，iOS 9 以后，安装的企业级应用在第一次打开之前必须要用户手动去信任这些App，参考[Guidelines for installing custom enterprise apps on iOS](https://support.apple.com/en-us/HT204460)。具体步骤如下：

* 打开 **设置** --> **通用** --> **描述文件与设备管理**
* 在 **企业级应用** 分组下，点击 **信任** 开发者的证书




