----
layout: post
title:  "使用Hexo搭建静态博客"
date:   2016-03-26
author: Jumpingfrog0 (黄东鸿）
comments: true
tags: blog
----

## 未完待续.....

### 前言

作为一个有技术追求的程序员，没有一个独立域名的技术博客，还怎么在业内混（zhuang)呢(bi)？

最近刚搭建完自己的静态技术博客，索性就写一篇教程来作为自己的第一篇文章了。以后会不定期更新，写些技术总结和经验总结，权当自己的学习笔记了，用来记录自己的成长之路。

读大学的时候，在博客园和CSDN上发表过一些文章，始终觉得编辑的时候特别蛋疼，体验极差，完全不符合程序员的风格，一点都不 *geek* 。听说*[GitHub Pages](https://pages.github.com/)* 可以搭建静态博客，搭建好环境后， 只需要用 *markdown* 来编辑，然后运行命令行自动部署就可以了，这完全就是为程序员打造的嘛，于是我就开始折腾了。 

*Github Pages* 建立的博客可以直接使用 `http://username.github.io `进行访问了，比如我的博客: [http://jumpingfrog0.github.io](http:jumpingfrog0.github.io)，当然也可以配置一级域名，这个后面会讲到。

*Github Pages* 官方的搭建静态博客的方式是 *[Jekyll](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/)*，但是过于繁琐复杂，而程序员永远是懒惰的，总是喜欢用更简单的方式去完成一件事情，于是 *[Hexo](https://hexo.io/)* 诞生了。

*Hexo* 具有很多的优点：

* 比 *Jekyll* 更快的生成静态网页的速度
* 支持 *[Markdown](http://wowubuntu.com/markdown/)* 格式
* 一行命令自动部署
* 强大的插件系统
* 有很多优秀且开源的主题可供选择

### 配置 Hexo

#### 安装并部署

安装 *Hexo* 只需要一行命令，打开终端，输入

```terminal
$ npm install hexo-cli -g
```

部署 *Hexo*，打开终端，输入

```terminal
$ hexo init blog
$ cd blog
$ npm install
$ hexo server
```

这样就已经搭建好本地的 *Hexo* 博客了，所有的文件都放在 `blog` 文件夹下。

目录结构如下所示

	.
	|--- _config.yml
	|--- db.json
	|--- public
	|--- source
	|   |--- _post
	|--- themes

各个文件(夹)的作用如下：

	* _config.yml: 站点配置文件，用来配置各种设置
	* db.josn: 缓存文件
	* public：部署后生成的静态网页文件都放在这个目录下
	* source: 存放网站内容的地方
	* _post: 存放文章(md文件）的地方
	* themes: 配置主题

更多详情请查看[官方介绍](https://hexo.io/docs/setup.html)

打开浏览器，输入`localhost:4000`，就可以看到Hexo自动生成的默认主页 *Hello World*

#### Hexo 常用命令

* `hexo init [folder]` 初始化网站
* `hexo new <title>` 新建文章，简写 `hexo n <title>`
* `hexo generate` 生成静态文件，简写 `hexo g`
* `hexo server` 开启本地服务器，简写 `hexo s`
* `hexo deploy` 自动部署，简写 `hexo d`
* `hexo clean` 清空缓存文件(`db.json`)和生成的静态文件(`public`)

可以在常用命令后面加些参数

* `hexo s --debug` 已debug模式启动本地服务器
* `hexo g -d` 生成静态文件后自动部署

更多命令请查看 [Commands](https://hexo.io/docs/commands.html)

### 配置 Github Pages

打开 *Github* ，新建一个 *repository*，命名为 `username.github.io` ，比如我的就是 `jumpingfrog0.github.io`

> 注意：*repository* 的名字中的 *username* 必须为你 *github* 账号的用户名，如果不匹配，那将没法正常工作

*Clone* 刚才新建的 *repository* 

```terminal
$ git clone https://github.com/username/username.github.io
```
#### 手动部署

* 运行部署命令：

	```terminal
	$ hexo g
	```
	
* 把 `blog`目录的`public`文件夹下的所有文件都拷贝到	`username.github.io` 代码仓库中
* push到远程仓库上

	```terminal
	$ git add --all
	$ git commit -m "Initial commit"
	$ git push -u origin master
	```
* 在浏览器中打开 `http://username.github.io`，就访问看到静态博客首页了

#### 自动部署

* 安装 `hexo-deployer-git`

	```terminal
	$ npm install hexo-deployer-git --save
	```

* 修改配置(`blog`目录下的`_config.yml`)

```css
deploy:
	type: git
  	repository: https://github.com/username/username.github.io.git
  	branch: master
```

* 打开终端，输入以下命令

```terminal
$ hexo clean
$ hexo g -d
```
运行完命令后，查看 *Github* 上 `username.github.io` *repository* 的提交记录，就可以有一个新的commit。在浏览器中打开 `http://username.github.io`，查看首页的变化。

### 配置主题

*Hexo* 可以很轻松的配置各种主题，官方[Themes](https://hexo.io/themes/)提供了很多个人开发者贡献的开源的主题,我们可以直接拿来使用。

当然，如果你想自己定制主题也可以，用js自己撸一套，请参照文档[Customization Themes](https://hexo.io/docs/themes.html)

我使用的是 `NexT` 主题，[NexT](https://github.com/iissnan/hexo-theme-next) 这套主题简洁优雅且易于使用，而且其作者目前依然在维护更新中，使其更优雅。具体配置可以看 `NexT` 的[主题配置文档](http://theme-next.iissnan.com/getting-started.html)，这里不再赘述。

在 *Hexo* 中有两份主要的配置文件，其名称都是 `_config.yml`。 其中，一份位于站点根目录下，主要包含 *Hexo* 本身的配置；另一份位于主题目录下，这份配置由主题作者提供，主要用于配置主题相关的选项。

为了描述方便，在以下说明中，将前者称为 `站点配置文件`， 后者称为 `主题配置文件`。

下面帖下我自己的配置

站点配置文件

```css
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: jumpingfrog0
subtitle:
description: Know a lot about a little, know a little about a lot
author: jumpingfrog0
language: zh-Hans
timezone:

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://jumpingfrog0.com
root: /
# permalink: :year/:month/:day/:title/
permalink: :year/:title/
permalink_defaults:

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next

# duoshuo
# duoshuo_shortname: jumpingfrog0

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
	type: git
  	repository: https://github.com/jumpingfrog0/jumpingfrog0.github.io.git
  	branch: master
```

主题配置文件

```css
# ---------------------------------------------------------------
# Site Information Settings
# ---------------------------------------------------------------

# Place your favicon.ico to /source directory.
favicon: /favicon.ico

# Set default keywords (Use a comma to separate)
keywords: "jumpingfrog0, Swift, iOS"

# Set rss to false to disable feed link.
# Leave rss as empty to use site's feed link.
# Set rss to specific value if you have burned your feed already.
rss:

# Specify the date when the site was setup
since: 2016

# ---------------------------------------------------------------
# Menu Settings
# ---------------------------------------------------------------

# When running hexo in a subdirectory (e.g. domain.tld/blog)
# Remove leading slashes ( "/archives" -> "archives" )
menu:
  home: /
  #categories: /categories
  #about: /about
  archives: /archives
  tags: /tags
  about: /about
  #commonweal: /404.html

# Enable/Disable menu icons.
# Icon Mapping:
#   Map a menu item to a specific FontAwesome icon name.
#   Key is the name of menu item and value is the name of FontAwsome icon.
#   When an question mask icon presenting up means that the item has no mapping icon.
menu_icons:
  enable: true
  # Icon Mapping.
  home: home
  about: user
  categories: th
  tags: tags
  archives: archive
  commonweal: heartbeat

# ---------------------------------------------------------------
# Scheme Settings
# ---------------------------------------------------------------

# Schemes
# scheme: Muse
#scheme: Mist
scheme: Pisces

# ---------------------------------------------------------------
# Sidebar Settings
# ---------------------------------------------------------------

# Social links
social:
  GitHub: https://github.com/jumpingfrog0
  # twitter: https://twitter.com/your-user-name
  微博: http://weibo.com/3290250165/profile?topnav=1&wvr=6&is_all=1
  # douban: https://www.douban.com/people/137739388/
  知乎: https://www.zhihu.com/people/huang-dong-hong
  #Others:

# Social Icons
social_icons:
  enable: true
  # Icon Mappings
  GitHub: github
  Twitter: twitter
  Weibo: weibo

# Sidebar Avatar
# in theme directory(source/images): /images/avatar.jpg
# in site  directory(source/uploads): /uploads/avatar.jpg
# default : /images/default_avatar.jpg
avatar: /images/minion.jpeg

# Links title, Chinese available
links_title: Links
# links
links:
  Bing: http://bing.com
  Objccn.io: http://objccn.io/

# TOC in the Sidebar
toc:
  enable: true

  # Automatically add list number to toc.
  number: true

# Creative Commons 4.0 International License.
# http://creativecommons.org/
# Available: by | by-nc | by-nc-nd | by-nc-sa | by-nd | by-sa | zero
#creative_commons: by-nc-sa
#creative_commons:

sidebar:
  # Sidebar Position, available value: left | right
  position: left
  #position: right

  # Sidebar Display, available value:
  #  - post    expand on posts automatically. Default.
  #  - always  expand for all pages automatically
  #  - hide    expand only when click on the sidebar toggle icon.
  #  - remove  Totally remove sidebar including sidebar toggle icon.
  display: post
  #display: always
  #display: hide
  #display: remove

# ---------------------------------------------------------------
# Misc Theme Settings
# ---------------------------------------------------------------

# Custom Logo.
# !!Only available for Default Scheme currently.
# Options:
#   enabled: [true/false] - Replace with specific image
#   image: url-of-image   - Images's url
custom_logo:
  enabled: false
  image:

# Code Highlight theme
# Available value:
#    normal | night | night eighties | night blue | night bright
# https://github.com/chriskempson/tomorrow-theme
highlight_theme: normal

# Automatically scroll page to section which is under <!-- more --> mark.
scroll_to_more: true

# Automatically Excerpt
auto_excerpt:
  enable: true
  length: 150

# Use Lato font
use_font_lato: true

# ---------------------------------------------------------------
# Third Party Services Settings
# ---------------------------------------------------------------

# MathJax Support
mathjax:

# Swiftype Search API Key
swiftype_key: _hsnBCSxyAH__Kvfd1uL

# Baidu Analytics ID
#baidu_analytics:

# Duoshuo ShortName
duoshuo_shortname: jumpingfrog0
duoshuo_hotartical: true

# Disqus
#disqus_shortname:

# Baidu Share
# Available value:
#    button | slide
#baidushare:
##  type: button

# Share
jiathis: true
#add_this_id:

# Share
duoshuo_share: true

# Google Webmaster tools verification setting
# See: https://www.google.com/webmasters/
#google_site_verification:

# Google Analytics
#google_analytics:

# CNZZ count
#cnzz_siteid:

# Make duoshuo show UA
# user_id must NOT be null when admin_enable is true!
# you can visit http://dev.duoshuo.com get duoshuo user id.
duoshuo_info:
  ua_enable: true
  admin_enable: false
  user_id: 0
  #admin_nickname: ROOT

# Facebook SDK Support.
# https://github.com/iissnan/hexo-theme-next/pull/410
facebook_sdk:
  enable: false
  app_id:       #<app_id>
  fb_admin:     #<user_id>
  like_button:  #true
  webmaster:    #true

# Show number of visitors to each article.
# You can visit https://leancloud.cn get AppID and AppKey.
leancloud_visitors:
  enable: false
  app_id: #<app_id>
  app_key: #<app_key>

# Tencent analytics ID
# tencent_analytics:

#! ---------------------------------------------------------------
#! DO NOT EDIT THE FOLLOWING SETTINGS
#! UNLESS YOU KNOW WHAT YOU ARE DOING
#! ---------------------------------------------------------------

# Motion
use_motion: true

# Fancybox
fancybox: true

# Static files
vendors: vendors
css: css
js: js
images: images

# Theme version
version: 0.5.0

```

### DNS 设置

还未配置，等购买域名后配置了再更...
