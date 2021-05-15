---
title: 小程序速读（一）入门
date: 2020-01-17 23:27:51
tags: 微信小程序
---

# 一、基础 

### 1. 概述

##### 什么是小程序
微信小程序，简称小程序，英文名Mini Program，是一种不需要下载安装即可使用的应用，它实现了应用“触手可及”的梦想，用户扫一扫或搜一下即可打开应用。

##### 特点

+ 无需下载、卸载
+ 扫码获取
 
### 2. 资源
+ [编译器下载](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html)
+ [开发者网站](https://open.weixin.qq.com/)
+ [微信公众平台——可以获取个人开发者的appID、发布小程序](https://mp.weixin.qq.com/)
+ [数据分析](https://developers.weixin.qq.com/miniprogram/analysis/#%E5%B8%B8%E8%A7%84%E5%88%86%E6%9E%90)
+ [运营规范](https://developers.weixin.qq.com/miniprogram/product/)
+ [接入指南](https://developers.weixin.qq.com/miniprogram/introduction/)
+ [设计指南](https://developers.weixin.qq.com/miniprogram/design/v)
+ [支付开发](https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=7_3&index=1)


### 3. 云开发
+ 数据库：一个既可在小程序前端操作，也能在云函数中读写的 JSON 文档型数据库
+ 文件存储：在小程序前端直接上传/下载云端文件，在云开发控制台可视化管理
+ 云函数：在云端运行的代码，微信私有协议天然鉴权，开发者只需编写业务逻辑代码
[参考文档](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/basis/getting-started.html)


### 4. 小程序减负
微信小程序在发布的时候，对提交的代码有1M大小的限制

不使用大图片
	使用小图片或者设置纯颜色
使用工具压缩优化代码
	使用Gulp，结合一些功能插件，如：uglify, cssnano, htmlmin等。使用这些工具
	+	Gulp是一个自动化构建工具,开发者可以使用它在项目开发过程中自动执行常见任务
	+	uglifyJs是一个js 解释器、最小化器、压缩器、美化器工具集
	+	cssnano是CSS优化和分解插件
	+	[htmlmin](https://www.npmjs.com/package/htmlmin)可以压缩html
	
	
### 5. 快捷键列表

##### 5.1 格式调整
+	Ctrl+S：保存文件（必须保存才可以看到效果）
+	`Ctrl+[， Ctrl+]`：代码行缩进
+	`Ctrl+Shift+[， Ctrl+Shift+]`：折叠打开代码块
+	Ctrl+C Ctrl+V：复制粘贴，如果没有选中任何文字则复制粘贴一行
+	Shift+Alt+F：代码格式化
+	Alt+Up，Alt+Down：上下移动一行
+	Shift+Alt+Up，Shift+Alt+Down：向上向下复制一行
+	Ctrl+Shift+Enter：在当前行上方插入一行
+	Ctrl+Shift+F：全局搜索

##### 5.2 光标相关
+	Ctrl+End：移动到文件结尾
+	Ctrl+Home：移动到文件开头
+	Ctrl+i：选中当前行
+	Shift+End：选择从光标到行尾
+	Shift+Home：选择从行首到光标处
+	Ctrl+Shift+L：选中所有匹配
+	Ctrl+D：选中匹配
+	Ctrl+U：光标回退

##### 5.3 界面相关
+	`Ctrl + \\`：隐藏侧边栏
+	`Ctrl + m`: 打开或者隐藏模拟器 	



# 二、实例入门 

### 1. 新建一个项目

1. 新建一个小程序
![小程序添加](/image/webchat/wechat-add.png)

其中，需要一个appID，可以在 [微信公众平台—【可以获取个人开发者的appID、发布小程序】](https://mp.weixin.qq.com/) 处获取
![wechat-appid](/image/webchat/wechat-appid.png)

> 小程序的 AppID 相当于小程序平台的一个身份证，后续你会在很多地方要用到 AppID （注意这里要区别于服务号或订阅号的 AppID）

第一个项目想了解一下大致的功能，不打算使用微信提供的云开发功能，语言选用【TypeScript】。


![编译器界面](/image/webchat/webchat-jm.png)
[真机预览](http://www.wxapp-union.com/forum.php?mod=viewthread&tid=1812)

2. 代码结构

+ `.json` 后缀的 JSON 配置文件
 + 数据格式，静态配置
 + app.json 是当前小程序的全局配置，包括了小程序的所有页面路径、界面表现、网络超时时间、底部 tab 等
  + pages字段 —— 用于描述当前小程序所有页面路径
  + window字段 —— 定义小程序所有页面的顶部背景颜色，文字颜色定义等
 + project.config.json 编译器工具的配置信息
 + page.json   pages目录下的【.json】文件，页面的特定配置
 + JSON 语法
  + JSON文件都是被包裹在一个大括号中 `{}`，通过key-value的方式来表达数据。JSON的Key必须包裹在一个`双引号`中
 + 数据格式
   + 数字，包含浮点数和整数
   + 字符串，需要包裹在双引号中
   + Bool值，true 或者 false
   + 数组，需要包裹在方括号中 `[]`
   + 对象，需要包裹在大括号中 `{}`
   + Null
   
> JSON 文件中无法使用注释，试图添加注释将会引发报错。

  
+ `.wxml` 后缀的 WXML 模板文件
	+ 与html的不同之处
		+ 【view, button, text】代替【div, p, span】
		+ `支持【wx:if】，支持双向绑定`
+ `.wxss` 后缀的 WXSS 样式文件
+ `.js` 后缀的 JS 脚本逻辑文件



# 三、app.json 
有五个部分可以配置：`pages, window, tarBar, networkTimeout和debug`

### 1 实例

```
{
  "pages": [
    "pages/index/index",
    "pages/logs/logs"
  ],
  "window": {
    "backgroundTextStyle": "light",
    "navigationBarBackgroundColor": "#fff",
    "navigationBarTitleText": "WeChat",
    "navigationBarTextStyle": "black"
  },
  "style": "v2",
  "sitemapLocation": "sitemap.json",
  "tabBar": {
  	// 背景色
    "backgroundColor": "#fbfbfb",
	// 边框颜色
    "borderStyle": "white",
	// 选中底部的颜色
    "selectedColor": "#50e3c2",
	// 未选中颜色
    "color": "#aaa",
	// 底部跳转的链接及文字描述
    "list": [
      {
        "pagePath": "pages/index/index",
		"iconPath": "images/homepage_off.png",
　　　　"selectedIconPath": "images/homepage_on.png" ,//选中图标
        "text": "首页"
		
      },
      {
        "pagePath": "pages/index/index",
		"iconPath": "images/homepage_off.png",
　　　　"selectedIconPath": "images/homepage_on.png" ,//选中图标
        "text": "我"
      }
    ]
  }
}
```

### 2 window

定义窗口的配置信息。

|属性|类型|默认值|描述|
|:--|:--|:--|:--|
|navigationBarBackgroundColor|HexColor|#000000|导航栏背景颜色，如"#000000"|
|navigationBarTextStyle|String|white|导航栏标题颜色，仅支持 black/white|
|navigationBarTitleText|String|a|导航栏标题文字内容|
|backgroundColor|HexColor|#ffffff|窗口的背景色|
|backgroundTextStyle|String|dark|下拉背景字体、loading 图的样式，仅支持 dark/light|
|enablePullDownRefresh|Boolean|False|是否开启下拉刷新|

### 3 tarBar
 用来设置底部的标签页。

### 4 networkTimeout
 用来设置各种网络请求的超时时间


|属性|类型|必填|说明|
|:--|:--|:--|:--|
|request|Number|否|wx.request的超时时间，单位毫秒|
|connectSocket|Number|否|wx.connectSocket的超时时间，单位毫秒|
|uploadFile|Number|否|wx.uploadFile的超时时间，单位毫秒|
|downloadFile|Number|否|wx.downloadFile的超时时间，单位毫秒|


# 四、常用组件 
### 1. 视图容器

+ cover-image 覆盖在原生组件之上的图片视图
+ cover-view 覆盖在原生组件之上的文本视图
+ movable-area movable-view的可移动区域
+ movable-view 可移动的视图容器，在页面中可以拖拽滑动
+ scroll-view 可滚动视图区域
+ swiper 滑块视图容器
+ swiper-item 仅可放置在swiper组件中，宽高自动设置为100%
+ view 视图容器

### 2. 基础内容

+ icon 图标
+ progress 进度条
+ rich-text 富文本
+ text 文本

*表单组件*
 
+ button 按钮
+ checkbox 多选项目
+ checkbox-group 多项选择器，内部由多个checkbox组成
+ editor 富文本编辑器，可以对图片、文字进行编辑
+ form 表单
+ input 输入框
+ label 用来改进表单组件的可用性
+ picker 从底部弹起的滚动选择器
+ picker-view 嵌入页面的滚动选择器
+ picker-view-column 滚动选择器子项
+ radio 单选项目
+ radio-group 单项选择器，内部由多个 radio 组成
+ slider 滑动选择器
+ switch 开关选择器
+ textarea 多行输入框

*导航 *
+ functional-page-navigator 仅在插件中有效，用于跳转到插件功能页
+ navigator 页面链接

*媒体组件*
 
+ audio 音频
+ camera 系统相机
+ image 图片
+ live-player 实时音视频播放（v2.9.1 起支持同层渲染）
+ live-pusher 实时音视频录制（v2.9.1 起支持同层渲染）
+ video 视频（v2.4.0 起支持同层渲染）

*地图*
+ map 地图（v2.7.0 起支持同层渲染）

*画布*

+ canvas 画布（2d 接口 v2.9.0 起支持同层渲染）
*开放能力*
 
+ web-view 承载网页的容器
+ ad Banner 广告
+ official-account 公众号关注组件
+ open-data 用于展示微信开放的数据

*原生组件说明*
+ native-component ## 原生组件


# 参考资料

> [从零开始：微信小程序新手入门宝典](http://www.wxapp-union.com/forum.php?mod=viewthread&tid=1989)
[一斤代码 ：如何为你的微信小程序瘦身？](http://www.wxapp-union.com/portal.php?mod=view&aid=934)