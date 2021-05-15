---
title: VUE（一）入门
date: 2020-02-27 22:43:06
tags: [vue]
---

# 一、基础

1. Vue.js 是什么
Vue (读音 /vjuː/，类似于 view) 是一套用于构建用户界面的渐进式框架。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与现代化的工具链以及各种支持类库结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。


2. 特点

+	轻量级的框架
+	数据驱动视图
	+	当一个 Vue 实例被创建时，它将 data 对象中的所有的属性加入到 Vue 的响应式系统中。当这些属性的值发生改变时，视图将会产生“响应”，即匹配更新为新的值。
+	声明式渲染（声明式地将数据渲染进 DOM）
	+	```
		-- 元素绑定
		<div id="app">
		  {{ message }}
		</div>
		var test = new Vue({
		  el: '#app',
		  data: {
			message: 'Hello Vue!'
		  }
		})
	```
+	指令：存在内置指令支持Vue.js与页面交互
+	组件化：可以扩展 HTML 元素，封装可重用的代码
+	客户端路由：提供路由插件Vue-router，用于构建单页面应用
+	状态管理：状态管理实际就是一个单向的数据流，State 驱动 View 的渲染，而用户对 View 进行操作产生 Action，使 State 产生变化，从而使 View 重新渲染，形成一个单独的组件。


3. 组件化的思想

组件系统是 Vue 的另一个重要概念，因为它是一种抽象，允许我们使用小型、独立和通常可复用的组件构建大型应用。在vue在中，一个工具条、弹出框、一个按钮甚至于一个标题，都可以抽象为一个组件，构成类似树的结构。

VUE组件本质上是一个拥有预定义选项的一个 Vue 实例
```
// 定义名为 todo-item 的新组件
Vue.component('todo-item', {
  template: '<li>这是个待办项</li>'
})

var app = new Vue(...)
```

一个 Vue 应用由一个通过 new Vue 创建的根 Vue 实例，以及可选的嵌套的、可复用的组件树组成。
![组件树](/image/vue/vue-0-tree.png)


4. Vue 组件与自定义元素的关系

> Vue 组件非常类似于自定义元素——它是 Web 组件规范的一部分，这是因为 Vue 的组件语法部分参考了该规范。例如 Vue 组件实现了 Slot API 与 is attribute。但是，还是有几个关键差别：

+	Web Components 规范已经完成并通过，但未被所有浏览器原生实现。目前 Safari 10.1+、Chrome 54+ 和 Firefox 63+ 原生支持 Web Components。相比之下，Vue 组件不需要任何 polyfill，并且在所有支持的浏览器 (IE9 及更高版本) 之下表现一致。必要时，Vue 组件也可以包装于原生自定义元素之内。
+	Vue 组件提供了纯自定义元素所不具备的一些重要功能，最突出的是跨组件数据流、自定义事件通信以及构建工具集成。
+	虽然 Vue 内部没有使用自定义元素，不过在应用使用自定义元素、或以自定义元素形式发布时，依然有很好的互操作性。Vue CLI 也支持将 Vue 组件构建成为原生的自定义元素。




# 二、使用

1. 使用VSCODE编辑器

+	[下载](https://code.visualstudio.com/docs/?dv=win)
+	安装插件快捷键——[ctrl+Shift+X ]
+	安装必要插件
	+	```
	-- 中文支持
	configure display language
	-- 实现支持vue文件的代码高亮
	vetur
		-- 配置
		文件->首选项->设置->用户设置
		"emmet.syntaxProfiles": {"vue-html":"html","vue":"html"

		}
	—— 自动完成另一侧标签的同步修改
	Auto Rename Tag 
	—— 自动闭合HTML/XML标签
	Auto Close Tag
	—— 自动路劲补全
	Path Intellisense
	——— 快捷键显示
	vue vscode snippets 
	```
+	修改vue模板，输入vue即可触发模板
	```
		--Ctrl+Shift+P打开命令输入 snippets; 在搜索框输入vue选择‘vue.json’
		{
			"Print to console": {
				"prefix": "vue",
				"body": [
					"<template>",
					" <div>\n",
					" </div>",
					"</template>\n",
					"<script>",
					" export default {",
					"   data () {",
					"     return {\n",
					"     }",
					"   },",
					"   components: {\n",
					"   }",
					" }",
					"</script>\n",
					"<style>\n",
					" ",
					"</style>",
					"$2"
				],
				"description": "Log output to console"
			}
		}
	```

2. npm 安装 VUE

```
-- vue.js
$ npm install -g vue

-- 如果下载不了，可以使用淘宝源
$ npm install -g cnpm --registry=https://registry.npm.taobao.org

-- 安装vue客户端，用于生成项目
cnpm install --global vue-cli

-- 生成初始化项目
vue init webpack test
```


3. 初始化项目
```
-- test是项目名称，不能用中文
--创建项目 
	-- webpack是打包方式
vue init webpack test
	? Project name test
	? Project description (A Vue.js project)
	? Project description A Vue.js project 
	? Vue build (Use arrow keys)
	? Vue build standalone
	? Install vue-router? (Y/n) y	路由管理器
	? Install vue-router? Yes
	? Use ESLint to lint your code? (Y/n) y	代码检测工具
	? Use ESLint to lint your code? Yes
	? Pick an ESLint preset (Use arrow keys)
	? Pick an ESLint preset Standard
	? Set up unit tests (Y/n) y		单元测试框架
	? Set up unit tests Yes
	? Pick a test runner (Use arrow keys)	测试执行过程管理平台
	? Pick a test runner jest
	? Setup e2e tests with Nightwatch? (Y/n) n	自动化测试框架
	? Setup e2e tests with Nightwatch? No
	
-- 启动
npm run dev

-- 打包项目
npm run build

```

4. 新建项目遇到的问题


*【webpack-dev-server'】不是内部或外部命令，也不是可运行的程序*
```
--webpack-dev-server版本过高

npm remove webpack-dev-server
npm install webpack-dev-server@2.9.1
npm run dev
```

*【npm ERR! Maximum call stack size exceeded】错误*

1. npm异常,最大回调函数栈数量超出，全局更新npm
`npm install npm -g`
2. 如果无效，修改仓库源
`npm set registry https://registry.npm.taobao.org/`

*【npm ERR! code EINTEGRITY】错误*
`npm cache verify`

5. vue更新

```
-- 先删除vue
npm uninstall vue-cli -g

-- 重新下载
npm install vue-cli -g
```

# 三、生命周期

1. 实例的生命周期钩子

每个 Vue 实例在被创建时都要经过一系列的初始化过程.例如，
+	需要设置数据监听
+	编译模板
+	将实例挂载到 DOM
+	在数据变化时更新 DOM 等。

在这个过程中也会运行一些叫做生命周期钩子的函数，这给了用户在不同阶段添加自己的代码的机会。


2. 钩子分类
+	beforeCreate 在实例初始化之后，数据观测 (data observer) 和 event/watcher 事件配置之前被调用。
+	created 在实例创建完成后被立即调用。在这一步，实例已完成以下的配置：数据观测 (data observer)，属性和方法的运算，watch/event 事件回调。然而，挂载阶段还没开始，$el 属性目前尚不可用。
+	beforeMount 在挂载开始之前被调用：相关的 render 函数首次被调用。
+	mounted 实例被挂载后调用，这时 el 被新创建的 vm.$el 替换了
+	beforeUpdate 数据更新时调用，
+	updated 
+	activated 被 keep-alive 缓存的组件激活时调用。
+	deactivated 被 keep-alive 缓存的组件停用时调用。
+	beforeDestroy 实例销毁之前调用。在这一步，实例仍然完全可用。
+	destroyed 实例销毁时调用
+	errorCaptured 当捕获一个来自子孙组件的错误时被调用

> 所有的生命周期钩子自动绑定 this 上下文到实例中，因此你可以访问数据，对属性和方法进行运算。这意味着你不能使用箭头函数来定义一个生命周期方法 (例如 created: () `=>` this.fetchTodos())。这是因为箭头函数绑定了父上下文， 它并没有this，this 会作为变量一直向上级词法作用域查找，直至找到为止，经常导致 Uncaught TypeError: Cannot read property of undefined 或 Uncaught TypeError: this.myMethod is not a function 之类的错误。


3. 钩子的使用
```
new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` 指向 vm 实例
    console.log('a is: ' + this.a)
  }
})
```

4. 生命周期图示

![生命周期图示](/image/vue/vue-1-lifecycle.png)


# 四、参考文章

> [vue-cli + webpack 新建项目出错的解决方法](https://blog.csdn.net/fungleo/article/details/79016838)
[Vue 实例](https://cn.vuejs.org/v2/guide/instance.html)