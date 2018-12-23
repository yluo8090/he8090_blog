---
title: iOS自建分发
date: 2018-05-14 10:24:41
tags:
  - 发布
categories: iOS技术
---
1、首先满足具有https证书的域名和空间。
2、通常使用github或者国内第三方托管平台。
3、上传ipa文件到空间内，获取ipa文件的下载地址。
4、然后编辑plist文件（注意：ipa文件和plist文件名要保持一致），编辑完成之后继续上传到空间内，获取到plist文件的下载地址。
5、直接访问或者间接访问 ```itms-services://?action=download-manifest&url=https://code.aliyun.com/FFPlayer/FFplayFolder/raw/master/ffplay.plist```
大功告成！

```
//plist文件
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
					<string>xxx.ipa</string>
				</dict>
			</array>
			<key>metadata</key>
			<dict>
				<key>bundle-identifier</key>
				<string>bundle id</string>
				<key>bundle-version</key>
				<string>版本号</string>
				<key>kind</key>
				<string>software</string>
				<key>title</key>
				<string>应用名称</string>
			</dict>
		</dict>
	</array>
</dict>
</plist>
```
