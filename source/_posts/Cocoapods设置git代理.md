---
title: Cocoapods设置git代理
date:2020.07.16 10:03:28
tags:
  - Cocoapods
categories: 技术分享
---

查看代理
```
git config --global --get http.proxy
git config --global --get https.proxy
```
pod设置代理
```
//端口号可以自己更改
git config --global http.proxy 'socks5://127.0.0.1:1086' 
git config --global https.proxy 'socks5://127.0.0.1:1086'
//一般来说设置前两个就行了
git config --global https.proxy http://127.0.0.1:1086
git config --global https.proxy https://127.0.0.1:1086
```

取消代理
```
#取消
git config --global --unset http.proxy
git config --global --unset https.proxy
```
