---
title: iOS工程之间依赖
date: 2016-12-07 9:52:22
tags:
    - 工程依赖
categories: 技术分享
---

在开发过程中公司采用的是SDK化，对基础和同样的功能进行封装为.a静态库，在使用时通过工程依赖的方式直接引用.a的这个工程，有时候会出现许多问题。

1、为了让引入的工程的文件编译和新的工程文件之间不产生冲突，需要注意别在Framework Search Paths中去添加依赖路径。
2、为了直接使用引入工程的文件可以在Framework Search Paths中添加
{% codeblock lang:objc %}
$(PROJECT_DIR)/实体工程路径名/实体.a文件名
{% endcodeblock %}
3、Header Seach Paths路径添加
{% codeblock lang:objc %}
"$(SRCROOT)/实体.a工程路径名/实体.a工程路径名/"
"${TARGET_BUILD_DIR)/.a工程名"
{% endcodeblock %}
