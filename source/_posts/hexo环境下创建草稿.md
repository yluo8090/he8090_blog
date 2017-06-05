---
title: hexo环境下创建草稿
date: 2017-06-02 10:51:25
tags:
    - 创建草稿
categories: 技术分享
---
```
//新建
$ hexo new draft "new draft"
```
######会在source/_drafts目录下生成一个new-draft.md文件。但是这个文件不被显示在页面上，链接也访问不到。也就是说如果你想把某一篇文章移除显示，又不舍得删除，可以把它移动到_drafts目录之中。

```
//如果你希望强行预览草稿，更改配置文件：
render_drafts: true

//或者，如下方式启动server：
$ hexo server --drafts

//把草稿变成文章，或者页面：
$ hexo publish [layout] <filename>
```
