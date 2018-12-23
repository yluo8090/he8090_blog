---
title: 使用Hexo+gitHub搭建个人博客
date: 2017-05-29 11:32:54
tags:
    - Hexo
    - gitHub
    - 博客
categories: 技术分享
---
#操作命令

sudo npm install -g hexo
hexo init
hexo server

hexo clean
hexo generate
hexo deploy

#关联github

vim _config.yml
deploy:

type: git

repo: https://github.com/misszero8090/misszero8090.github.io.git

branch: master

<!-- more -->

# 然后执行命令：

npm install hexo-deployer-git --save

#一些常用命令：

hexo new"postName" #新建文章

hexo new page"pageName" #新建页面

hexo generate #生成静态页面至public目录

hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）

hexo deploy #将.deploy目录部署到GitHub

hexo help # 查看帮助

hexo version #查看Hexo的版本


# Site #站点信息
title: blog Name #标题
subtitle: Subtitle #副标题
description: my blog desc #描述
author: me #作者
language: zh-CN #语言
timezone: Asia/Shanghai #时区

# URL
url: http://yoururl.com   #用于绑定域名, 其他的不需要配置
root: /
#permalink: :year/:month/:day/:title/
permalink: posts/title.html
permalink_defaults:

# Directory #目录
source_dir: source #源文件
public_dir: public #生成的网页文件
tag_dir: tags #标签
archive_dir: archives #归档
category_dir: categories #分类
code_dir: downloads/code
i18n_dir: :lang #国际化
skip_render:

# Writing #写作
new_post_name: :title.md #新文章标题
default_layout: post #默认模板(post page photo draft)
titlecase: false #标题转换成大写
external_link: true #新标签页里打开连接
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight: #语法高亮
enable: true
line_number: true #显示行号
auto_detect: true
tab_replace:

# Category & Tag #分类和标签
default_category: uncategorized #默认分类
category_map:
tag_map:

# Date / Time format #日期时间格式
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination #分页
per_page: 10 #每页文章数, 设置成 0 禁用分页
pagination_dir: page

# Extensions #插件和主题
## 插件: http://hexo.io/plugins/
## 主题: http://hexo.io/themes/
theme: next

# Deployment #部署, 同时发布在 GitHub 和 GitCafe 上面
deploy:
- type: git
repo: git@gitcafe.com:username/username.git,gitcafe-pages
- type: git
repo: git@github.com:username/username.github.io.git,master

# Disqus #Disqus评论系统
disqus_shortname:

#plugins: #插件，例如生成 RSS 和站点地图的
- hexo-generator-feed
- hexo-generator-sitemap
