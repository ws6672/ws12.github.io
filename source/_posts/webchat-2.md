---
title: 小程序速读（二）小实例
date: 2020-01-30 22:59:00
tags: [微信小程序]
---

# 存储
数据存储对象包括数据流在加工过程中产生的临时文件或加工过程中需要查找的信息。小程序有时候需要存储数据，可以通过全局变量、本地存储以及可以通过浏览器存储。

### 全局变量（app.globalData）
数据可以保存到小程序关闭。
在App.js 初始化变量
```
App({
  globalData:{
    appid: '111',//
    secret: '111',//
    openid:'111'
  }
});
```

*JS*
```
// 建议写在开头，方便与App.js文件关联
var app = getApp();
// 获取 全局变量
var globalData = app.globalData.openid;
this.globalData.openid = e.detail.openid;
 
// 设置 全局变量
getApp().globalData.openid = "12345";
```



### 本地存储
以Sync（同步，同时）结尾的都是都是同步缓存，二者的区别是，异步不会阻塞当前任务，同步缓存直到同步方法处理完才能继续往下执行。


##### 异步数据
数据保存后，可以持续到下次启动。

通过key-value的方式存储数据，可以对本地缓存进行设置、获取和清理。本地缓存最大为10MB

wx.setStorageSync(KEY,DATA)
wx.getStorageSync(KEY)
wx.getStorageInfoSync
wx.removeStorageSync(KEY)
wx.clearStorageSync()

```
//  添加数据
	let taskList = wx.getStorageSync('taskList');
	taskList.unshift({
        id: new Date(),
        text: this.data.work,
        status: false
	}); 
	wx.setStorageSync({
	  key: 'taskList',
	  data: taskList
	})
	
// 遍历数据
    let taskUndone = wx.getStorageSync('taskList').filter((item) => {
      return !item.status;
    });
```


##### 同步数据
数据保存后，可以持续到下次启动。

wx.setStorage
wx.getStorage  
wx.getStorageInfo
wx.removeStorage(KEY)
wx.clearStorage

```
wx.setStorage({
  key: 'hasUserInfo',
  data: false,
})
```



# tabBar

小程序 的 tabBar 默认只可以设置为跳转的设置，但我们可以对她进行样式设置、显示设置、展示文字或者红点的设置。

### tabbar右上角的红点显示与隐藏
通过【wx.showTabBarRedDot(Object object)】可以显示红点, 通过【wx.hideTabBarRedDot(Object object)】隐藏红点。设置参数如下：

|属性|类型|必填|说明|
|:--|:--|:--|:--|
|index|number|是|定位tabBar中的位置|
|success|func|否|调用成功|
|fail|func|否|调用失败|
|complete|func|否|调用完成|


实例
```
wx.showTabBarRedDot({
  // 从0开始,表示显示红点的tabBar位置
  index:1,
  success:function(){
  },
  fail:function(){
  },
  complete:function(){}
});
 wx.hideTabBarRedDot({
	// 从0开始,表示隐藏红点的tabBar位置，建议写在【onShow】函数中，在切换后红点消失
	index: 1,
	success: function () {
	},
	fail: function () {
	},
	complete: function () { }
  });
```

### tabbar右上角文字的显示与隐藏
通过【wx.setTabBarBadge(Object object)】可以显示文字。设置参数如下：

|属性|类型|必填|说明|
|:--|:--|:--|:--|
|index|number|是|定位tabBar中的位置|
|text|string|是|显示的文本|
|success|func|否|调用成功|
|fail|func|否|调用失败|
|complete|func|否|调用完成|



通过【wx.removeTabBarBadge(Object object)】隐藏文字。设置参数如下：

|属性|类型|必填|说明|
|:--|:--|:--|:--|
|index|number|是|定位tabBar中的位置|
|success|func|否|调用成功|
|fail|func|否|调用失败|
|complete|func|否|调用完成|

 
> 微信官方api要求text属性应该为number类型，但是使用过程中发现必须为string类型，并且为数字字符串、字母字符串、和汉字字符串时最多都显示4个字符，多于四个时不再显示，汉字时竖直显示。


实例
```
	// app.js 设置全局变量
	App({
		globalData: {
		  undoSize:0
		}
	});
	// index.js
	let undoSize = wx.getStorageSync('undoSize');
	undoSize++;
	wx.setStorageSync('undoSize', undoSize); 
	wx.setTabBarBadge({
	  index:1,
	  text: String(undoSize)
	});
	
	// undo.js
	wx.removeTabBarBadge({index:1});
```


### 显示和隐藏 tabBar
在开发中， 地图或者拍照等功能，需要隐藏底部的导航条。

wx.showTabBar(Object object) 显示
wx.hideTabBar(Object object) 隐藏 

 wx.showTabBar({
	//  tabBar显示
	animation: true,
	success: function () {
	},
	fail: function () {
	},
	complete: function () { }
  });

 wx.hideTabBar({
	//  tabBar隐藏
	animation: true,
	success: function () {
	},
	fail: function () {
	},
	complete: function () { }
  });

设置参数如下：

|属性|类型|必填|说明|
|:--|:--|:--|:--|
|animation|boolean|否|是否展示动画|
|success|func|否|调用成功触发函数|
|fail|func|否|调用失败触发函数|
|complete|func|否|调用完成触发函数|

```
 wx.hideTabBarRedDot({
	// 从0开始,表示隐藏红点的tabBar位置，建议写在【onShow】函数中，在切换后红点消失
	index: 1,
	success: function () {
	},
	fail: function () {
	},
	complete: function () { }
  });
```

### 其它 tabBar 对应的全局函数


`wx.setTabBarStyle(Object object)`
	动态设置 tabBar 的整体样式
	
|属性|类型|默认值|必填|说明|
|:--|:--|:--|:--|:--|
|color|string||否|tab 上的文字默认颜色，HexColor|
|selectedColor|string||否|tab 上的文字选中时的颜色，HexColor|
|backgroundColor|string||否|tab 的背景色，HexColor|
|borderStyle|string||否|tabBar上边框的颜色， 仅支持 black/white|
|success|function||否|接口调用成功的回调函数|
|fail|function||否|接口调用失败的回调函数|
|complete|function||否|接口调用结束的回调函数（调用成功、失败都会执行）|

`wx.setTabBarItem(Object object)`
	动态设置 tabBar 某一项的内容，2.7.0 起图片支持临时文件和网络文件
	> 基础库 1.9.0 开始支持，低版本需做兼容处理。
|属性|类型|默认值|必填|说明|
|:--|:--|:--|:--|:--|
|index|number||是|tabBar 的哪一项，从左边算起|
|text|string||否|tab 上的按钮文字|
|iconPath|string||否|图片路径，icon 大小限制为 40kb，建议尺寸为 81px * 81px，当 postion 为 top 时，此参数无效|
|selectedIconPath|string||否|选中时的图片路径，icon 大小限制为 40kb，建议尺寸为 81px * 81px ，当 postion 为 top 时，此参数无效|
|success|function||否|接口调用成功的回调函数|
|fail|function||否|接口调用失败的回调函数|
|complete|function||否|接口调用结束的回调函数（调用成功、失败都会执行）|

### 导入 Iconfont 图片库

可以通过[Iconfont图片库](https://www.iconfont.cn/) 获取适合的图片。
