---
title: VUE（四）组件的使用
date: 2021-01-22 23:47:37
tags: [vue]
---

# 一、组件注册

组件 (Component) 是 Vue.js 最强大的功能之一。组件可以扩展 HTML 元素，封装可重用的代码。组件依照使用范围可以分为全局组件和局部组件；全局组件可以在所有的vue实例VUE实例中使用，而局部组件只能在注册了该组件的VUE实例中使用。

注意：
+	全局组件必须写在Vue实例创建之前，才在该根元素下面生效
+	局部组件注册方式，直接在Vue实例里面注册



1. 组件定义

```
// 组件命名规范：字母全小写且必须包含一个连字符（短横线分隔命名）；首字母大写定义组件（驼峰命名法）。
全局组件注册方式：Vue.component(组件名,{方法})

局部组件注册方式：在VUE实例的components中定义，即可
```


html 中attribute是大小写不敏感的，驼峰命名法的prop名需要使用与其等价的短横线分隔命名，例如：
```
Vue.component('blog-post', {
  // 在 JavaScript 中是 camelCase 的
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})
<!-- 在 HTML 中是 kebab-case 的 -->
<blog-post post-title="hello!"></blog-post>
```

2. 全局注册的组件

全局注册的组件也就是说它们在注册之后可以用在任何新创建的 `Vue 根实例 (new Vue)` 的模板中。比如：

```
// JS代码, component需定义在根实例前
Vue.component('my-component', {
	template: "<span style='color:red;'>通过component定义的全局组件</span>"
})

new Vue{(
	el:'#app'
)}

<!--html代码-->
<div id= 'app'>
	<my-component>

	</my-component>
<div id= 'app'>
```

3. 局部注册的组件

局部注册的组件只有在配置了`components`属性后才能被使用。

```
<!--html代码-->
<div id="app">
	<child-component><child-component>
</div>

// JS代码
// 局部注册的组件在其子组件中无法使用。
new Vue{(
	el:'#app',
	components: {
		"child-component":{
			template:"<p>根实例中components属性定义的局部组件</p>"
		}
	}
)}
```


如果其他组件想使用局部组件，也需要类似的定义。如下所示：

```
var com-b = Vue.component('my-component',{
	components: {
		//组件别称：实际调用的组件
		'component-a':com-a
	}
});
// 这样定义后，b组件就可以使用A组件了。
```

4. 组件中的数据和方法

+	方法
	+	父组件调用子组件：ref（在子组件中加上ref即可通过this.$refs.ref.method调用）
	+	子组件调用父组件：emit（this.$emit(调用的方法名，传递的参数)）
+	数据
	+	父组件传给子组件：props
	+	子组件传给父组件：emit
+	共享采用：vuex
+	导入使用：import


```

vm.$refs：一个对象，持有已注册过ref的所有子组件
ref：被用来给元素或子组件注册引用信息，引用信息将会注册在父组件的 $ref 对象上

let vm = new Vue{(
	el:'#app',
	components: {
		"child-component":{
			template:"<p>根实例中components属性定义的局部组件</p>"
		}
	}
)}

<span ref="my-span">空白行</span>

```
