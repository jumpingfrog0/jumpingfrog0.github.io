----
title: 自定义Xcode模板
----

Xcode 拥有一个很好的内置模板，创建文件时会在头部生成一段注释，包含了文件名、创建者、创建时间、版权等信息，但有时候我们需要根据不同的需求自定义一套模板。

Xcode 的 iOS 代码模板

![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/xcode_ios_source_templetes.png)

空的`Swift`文件的默认模板是这样的：

	//
	//  FileName.swift
	//  ProjectName
	//
	//  Created by Your Name on 12/29/15
	//  Copyright (c) 2015 Company. All rights reserved.
	//
	
	import Foundation

在一些开源项目中，我们有时候会看到在源代码头部有一大串长长的`MIT`协议声明，每个文件都有，当然，那些开源大牛们肯定不会傻傻的每个文件复制粘贴，这么麻烦的事情身为一个程序员肯定是忍不了的，一定有一个一劳永逸的办法，那就是为空的`Swift`文件创建自定义模板。

### 创建模板

Xcode 会在 `~/Library/Developer/Xcode/Templates` 寻找自定义模板。

Xcode会把一个目录当作一个组（group），我们可以在上述目录下创建一个名为`Custom`的组，然后把系统自带的`Swift`模板拷贝进去。

在终端中运行如下命令：

	$ mkdir -p ~/Library/Developer/Xcode/Templates/Custom
	$ cp -R /Applications/Xcode.app/Contents/Developer/Library/Xcode/Templates/File\ Templates/Source/Swift\ File.xctemplate ~/Library/Developer/Xcode/Templates/Custom/
	
拷贝完后，我们可以看到有这几个文件：

	$ cd ~/Library/Developer/Xcode/Templates/Custom/Swift\ File.xctemplate
	$ ls
	TemplateIcon.png          TemplateIcon@2x.png
	TemplateInfo.plist        ___FILEBASENAME___.swift
	
打开 `___FILEBASENAME___.swift`, 代码如下：

	//
	//  ___FILENAME___
	//  ___PROJECTNAME___
	//
	//  Created by ___FULLUSERNAME___ on ___DATE___.
	//___COPYRIGHT___
	//
	
	import Foundation
	
这就是我们正在找的模板，把它改成你想要的格式就可以了。

我们加入`MIT`协议声明，如下：

	//  ___FILENAME___
	//  ___PROJECTNAME___
	//
	//  Created by ___FULLUSERNAME___ on ___DATE___.
	//
	//
	//  Copyright (c) ___YEAR___ ___FULLUSERNAME___
	//
	//  Permission is hereby granted, free of charge, to any person obtaining a copy
	//  of this software and associated documentation files (the "Software"), to deal
	//  in the Software without restriction, including without limitation the rights
	//  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
	//  copies of the Software, and to permit persons to whom the Software is
	//  furnished to do so, subject to the following conditions:
	//
	//  The above copyright notice and this permission notice shall be included in
	//  all copies or substantial portions of the Software.
	//
	//  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
	//  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
	//  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
	//  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
	//  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
	//  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
	//  THE SOFTWARE.
	//
	
	import Foundation
	
重启Xcode，新建文件 *File* → *New* ，单击 *Custom* ，你可以看到新的模板

![](http://jumpingfrog0-images.oss-cn-shenzhen.aliyuncs.com/xcode_custom_templetes.png)

Enjoy coding!!!