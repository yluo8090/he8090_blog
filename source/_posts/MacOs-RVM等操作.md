---
title: MacOs RVM等操作
date: 2018-09-07 11:00:12
tags:
  - Mac操作
categories: Mac技术
---

Mac OS X 系统自带的 Ruby,但是如果不管理它，它是不会自动升级滴，所以，我们需要一个ruby版本及安装工具管理它，那是什么呢？

Ruby的管家婆登场-----》RVM全称Ruby Version Manager，是一个非常好用的Ruby版本以及安装工具。就是用来安装和控制Ruby版本的工具。

RVM也是需要我们自己安装的。

一、rvm

1、安装：curl -L get.rvm.io | bash -s stable

2、指定源：source ~/.rvm/scripts/rvm

3、查看版本：rvm -v 

4、列出指定源中所有ruby版本：rvm list known

二、ruby

1、安装ruby： rvm install 2.4.0

2、更新rubyGems版本：gem update --system

三、检查ruby

1、检查ruby源：gem sources -l 

2、移除多余的源：gem sources --remove https://rubygems.org/

3、添加源：gem sources --add https://gems.ruby-china.org

四、cocoapods

1、安装：sudo gem install cocoapods

注意：

　　OS X 10.11之前系统的安装cocoapods 指令：$ sudo gem install cocoapods

　　OS X 10.11以后系统的安装cocoapods 指令：$ sudo gem install -n /usr/local/bin cocoapods

2、cd 到工程目录，创建podfile文件：vim podfile（也可以使用pod init 会创建一个文件）

3、更新pod库：pod update --verbose --no-repo-update  或者   pod install --verbose --no-repo-update

gem相关命令使用

1.显示gem的帮助和版本

gem –h/--help

#显示gem的帮助

gem –v /--version

#显示gem的版本号

2\. 列出远程库的所有可用软件

gem query --remote        

# 短命令: gem q -r

你可以看到一个关于远程主机上所有软件的详细列表。

3\. 查找远程主机上的特定软件

gem query --remote --name-matches doom

# 短命令: gem q -rn doom

你将看到一个匹配doom的详细列表。

gem list –remote --d

#用子命令list列出远程安装的gems

4.1 安装一个远程软件

gem install --remote progressbar

# 短命令: gem i -r progressbar –y

远程安装progressbar到你的主机，-y的意思是无条件的安装依赖包

gem install rails –remote

#从远程服务器安装rails包，其中rails可以被替换成任何一个gem list –remote –d中显示的软件包

4.2 安装软件的特定版本

gem ins -r progressbar-0.0.3

安装progressbar的0.0.3版本

gem ins -r progressbar --version '> 0.0.1'

将安装progressbar的大于0.0.1的最新版本

5\. 查看一个已安装的软件

gem specification progressbar

# 短命令: gem spec progressbar

你会看到关于已安装的包progressbar的详细信息。

6\. 卸载一个软件

gem uninstall progressbar

卸载了progressbar

7.1 将所有安装的软件列表

gem query --local

# 短命令: 'gem q -l'

7.2 查看某个已安装的软件

gem query --local --name-matches doom

# 短命令: 'gem q -ln doom'

或：gem list --local

CocoaPods相关命令

# 第一次使用安装框架    //只用安装一次,之后使用 添加删除都用 pod update --no-repo-update

$ pod install

# 安装框架，不更新本地索引，速度快

$ pod install --no-repo-update

# 今后升级、添加、删除框架，或者框架不好用

$ pod update

# 更新框架，不更新本地索引，速度快

$ pod update --no-repo-update

# 搜索框架

$ pod search XXX#

帮助

$ pod --help

​
