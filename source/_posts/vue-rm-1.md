---
title: VUE（二）vue实例
date: 2021-01-22 15:21:03
tags: [vue]
---


# 一、基础

1. MVVM

VUE设计参考MVVM，虽然没有完全遵循 MVVM 模型，但是 Vue 的设计也受到了它的启发。因此在文档中经常会使用 vm (ViewModel 的缩写) 这个变量名表示 Vue 实例。那什么是 MVVM 模型？


它的设计参考后端的MVC模式，MVVM是Model-View-ViewModel的缩写。由于前端开发混合了HTML、CSS和JavaScript，而且页面众多，所以，代码的组织和维护难度其实更加复杂，这就是MVVM出现的原因。
+	Model用JS对象表示，存放数据；
+	View表示网页，负责显示；
+	ViewModel表示绑定，负责把Model的数据同步到View显示出来，还负责把View的修改同步回Model。

使用MVVM框架来实现数据修改，假定视图和模型是绑定的；JS中会将数据存储起来，由于存在绑定关系，所以修改数据就是修改页面。所以，我们只需要关注数据存储即可。

2. 起步

我们可以通过NPM的vue-cli使用vue，也可以通过网页引入vue.js进行练习。存在开发环境与生产环境两个版本，使用步骤如下：

+	引入vue.js
+	创建Vue实例对象，设置el属性和data属性
+	通过模板语法渲染数据


每个 Vue.js 应用都是通过构造函数 Vue 创建一个 Vue 的根实例来启动的：
```
var vm = new Vue({
  // 选项
})
```

一个 Vue 应用由一个通过 new Vue 创建的根 Vue 实例，以及可选的嵌套的、可复用的组件树组成。
一个 todo 应用的组件树如下：
+	根实例
	+	TodoList
		+	TodoItem
			+	TodoButtonDelete
			+	TodoButtonEdit
		+	TodoListFooter
			+	TodosButtonClear
			+	TodoListStatistics

当一个 Vue 实例被创建时，它将 data 对象中的所有的 属性 加入到 Vue 的响应式系统中。当这些 属性 的值发生改变时，视图将会产生“响应”，即匹配更新为新的值。

# 二、 el属性和data属性

有两个基本的属性需要先了解一下，分别是 el属性和data属性。

其中，el属性表示Vue实例挂载的元素节点，值可以是 CSS 选择符，或实际 HTML 元素，或返回 HTML 元素的函数；

+	Vue会管理el属性命中元素及其后代元素。
+	el属性中的参数可以用CSS的选择器，如ID选择器、类选择器等，但是建议使用ID选择器。
+	el元素不可以挂载到body标签上，建议挂载到div标签上。

而data属性存储的是前端的数据。通过模板语法渲染数据，如果后端的数据变化了，前台的页面也会有对应的修改。
+	data中定义数据
+	data中可以存在复杂类型的数据
+	渲染复杂类型数据时，遵守js语法即可

相关实例如下：
```
	<!DOCTYPE html>
	<html>
	<head>
	  <title>My first Vue app</title>
	  <!-- 1. 引入vue.js -->
	  <!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
	<!-- 生产环境版本，优化了尺寸和速度 -->
	<!-- <script src="https://cdn.jsdelivr.net/npm/vue"></script> -->
	</head>
	<body>
	  <div id="app">
		<!-- 3. 模板语法渲染数据 -->
		{{ message }}
		<p>{{ list[0].name}}    {{ list[0].phone}}<p>
	  </div>

	  <script>
		//2. 创建Vue实例对象，设置el属性和data属性  
		var app = new Vue({
		  el: '#app',
		  data: {
			message: 'Hello Vue!',
			list: [
				{
				name: 'jk',
				phone: 110
				}
			]
		  }
		})
	  </script>
	</body>
	</html>
```

由上可以知晓，el属性将实例挂载在css中ID为app的元素上；还设置了data，通过模板语法调用后台定义的数据。

# 三、 计算属性和侦听属性

计算属性和侦听属性即 computed与watch，它们都依赖值的变化被调用。

1. computed 计算属性

Vue中的computed属性称为计算属性：
+	计算属性允许我们对指定的视图，复杂的值计算。这些值将绑定到依赖项值，只在需要时更新。当computed依赖的属性值发生变化时，计算属性会重新计算；反之，则使用缓存中的属性值。
+	简化tempalte里面计算和处理props或$emit的传值


相关实例如下：
```
	<!DOCTYPE html>
	<html>
	<head>
	  <title>My first Vue app</title>
	  <!-- 1. 引入vue.js -->
	  <!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
	</head>
	<body>
	  <div id="app">
			<div>
				<p>科目  分数</p>
				<p v-for="item in results">{{item.name}}  {{item.score}}</p>
				<p>总分：{{this.computedAvg }}</p>
			</div>
	  </div>

	  <script>
		//2. 创建Vue实例对象，设置el属性和data属性  
		var app = new Vue({
			el: '#app',
			data: {
				count:1,
				results: [
					{name: 'english', score: 70},
					{name: 'math', score: 80},
					{name: 'chinese', score: 90}
				]
			},
			computed: {
				computedAvg: function() {
					let sum=0
					let rst = this.results;
					for (let i=0; i < rst.length; i++) {
						sum+=rst[i].score
					}
					return sum/rst.length
				}
			}
		})
	  </script>
	</body>
	</html>
```

如果results中的数据发生变动，计算属性中定义的computedAvg 的返回值也会被修改。


计算属性的 setter：

```
	computed: {
		// getter 获取值时触发，计算属性中默认使用get
		// setter 设置值时触发
		fullName: {
			get: function () {
			  return this.firstName + ' ' + this.lastName
			},
			set: function (newValue) {
			  var names = newValue.split(' ')
			  this.firstName = names[0]
			  this.lastName = names[names.length - 1]
			}
		}
	}
```

通过`vm.fullName = 'John Doe'`的方式调用`setter`设置值。

2. watch 侦听属性

watch用于监测一个值的变化，并调用因为变化涉及的方法，可以通过它动态改变状态。但是，在watch中不要使用箭头函数，因为箭头函数中的this是指向当前作用域。

相关实例如下：

```
	<!DOCTYPE html>
	<html>
	<head>
	  <title>My first Vue app</title>
	  <!-- 1. 引入vue.js -->
	  <!-- 开发环境版本，包含了有帮助的命令行警告 -->
		<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
	</head>
	<body>
	  <div id="app">
			<div>
				<p><input type="number" v-model = "yc"/> 英寸==> 厘米</P>
				<p>{{ yc }} 英寸 = {{  this.exchangeYC }} 厘米 </P>
			</div>
	  </div>

	  <script>
		//2. 创建Vue实例对象，设置el属性和data属性  
		var app = new Vue({
			el: '#app',
			data: {
				yc: 0
			},
			
			computed: {
				exchangeYC: function() {
					return this.yc * 2.54
				}
			},
			watch: {
				yc: {
					// yc值修改后触发
					handler(nv, ov) {
						// 后台输出值
						console.log(ov + "==>" + nv);
					},
					immediate: false, //网页加载完后不会立即调用一次
					deep: true //会进行深度监听，如果绑定的是变量，内部属性的变动也会触发
				}
			}
		})
	  </script>
	</body>
	</html>
```


Computed与Watch的异同：

+	Computed
	+	需要主动调用
	+	支持缓存、不支持异步、依赖数据变化才会重新计算
	+	适合多对一或者一对一
+	Watch 
	+	不支持缓存、支持异步、依赖数据变化才会重新计算
	+	监听的函数接收两个参数，第一个参数是最新的值；第二个参数是输入之前的值
	+	适合一对多


# 四、函数

1. 定义

VUE支持函数的定义。函数必须在 Vue.js 中的 methods 属性下添加，类似于计算属性（computed），在 Vue.js 中，methods 被命名为方法，是调用对象上下文中的函数，还可以操作对象中包含的数据。

相关语法：
```
// 函数定义
add:function(num){}

// 传递参数
<button @click="add(10)">add</button>

```

例如：
```
<!DOCTYPE html>
<html>
<head>
  <title>My first Vue app</title>
  <!-- 1. 引入vue.js -->
  <!-- 开发环境版本，包含了有帮助的命令行警告 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
  <div id="app">
		<p>计数：{{ count }}</p>
		<button @click="add(10)">add</button>
  </div>

  <script>
    //2. 创建Vue实例对象，设置el属性和data属性  
	var app = new Vue({
		el: '#app',
		data: {
			count:1
		},
		methods: {
			add: function(num) {
				if(num!='') {
					this.count+=num
				} else {
					this.count++
				}
			}
		}
	})
  </script>
</body>
</html>
```

2. $event 参数

传递的$event参数都是关于点击鼠标的一些事件和属性

3. `.native修饰器`

在实际开发中经常需要把某个按钮封装成组件，然后反复使用，如何让组件调用构造器里的方法，而不是组件里的方法。就需要用到我们的.native修饰器了。

add按钮封装成组件：
```
	//声明btn对象：
	var btn={
		template:`<button>组件Add</button>`    
	}

	//在构造器里声明：
	components:{
		"btn":btn
	}

	//用.native修饰器来调用构造器里的add方法
	<p><btn @click.native="add(3)"></btn></p>
```
