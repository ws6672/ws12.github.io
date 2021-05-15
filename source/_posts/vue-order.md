---
title: VUE（三）指令
date: 2021-01-22 23:03:48
tags: [vue]
---

# 一、插值


*文本*
```
// 双大括号会将数据解释为普通文本，而非 HTML 代码
<span>Message: {{ msg }}</span>
```

*原始 HTML插入*
通过`v-html`将文本解析成html代码
```
<span v-html="rawHtml"></span>
```

*属性的绑定*
将属性`id`与`dynamicId`绑定
```
<div v-bind:id="dynamicId"></div>
```

*JS的支持*

模板中，绑定都能使用且能使用单个表达式：

```
{{ number + 1 }}
{{ ok ? 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}

<!-- 这是语句，不是表达式 -->
{{ var a = 1 }}

<!-- 流控制也不会生效，请使用三元表达式 -->
{{ if (ok) { return message } }}
```



# 二、常用指令

1. 基础

常用指令，【v-】前缀代表vue特有指令
+	v-bind用于绑定数据和元素属性
	+	`v-bind:title 表示绑定标签的title属性`
	+	 完整：`<a v-bind:href="url">...</a>`
	+	 缩写：`<a :[key]="url"> ... </a>`
+	v-on 事件监听器指令,用于用户交互
	+	完整：`<a v-on:click="doSomething">...</a>`
	+	缩写：`<a @[event]="doSomething"> ... </a>`
+	条件指令
	+	v-if
	+	v-else
	+	v-else-if
	+	设置key
		+	```
				-- 保证带key的标签唯一，切换后会被替换
				<template v-if="loginType === 'username'">
				  <label>Username</label>
				  <input placeholder="Enter your username" key="username-input">
				</template>
				<template v-else>
				  <label>Email</label>
				  <input placeholder="Enter your email address" key="email-input">
				</template>
		 	```
+	列表指令
	+	v-for 循环执行
+	v-model 设置数据双向绑定
+	v-text 用于渲染普通文本，相当于ele.innerText
+	v-html 用于渲染html代码，将html代码用于替换
+	v-show 类似于CSS中的`display:none`

2. 指令使用

```

<template>
  <div id="app-5">
      <!-- <button v-on:click="reverseMessage" v-bind:tittle="提示信息">反转消息</button> -->
      <ol v-show="flag">
        <li v-for="todo in todos">
          {{ todo.text }}
        </li>
      </ol>
      <button v-on:click="exchange">切换</button>
  </div>
</template>

 
 
<script>
export default {
  name: '#app-5',
  data () {
    return {
      message: 'Hello Vue.js!',
      flag:true,
      todos: [
        { text: '学习 JavaScript' },
        { text: '学习 Vue' },
        { text: '牛项目' }
      ]
    }
  },
  methods: {
	  exchange: function () {
		  this.flag = !this.flag;
	  }
  }
}
</script> 

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>

```


3. vue指令v-for报错：Elements in iteration expect to have 'v-bind:key' directives.eslint-plugin-vue

```
	v-for指令后加上:key="item"
```

4. 修饰符
修饰符 (modifier) 是以半角句号 . 指明的特殊后缀，用于指出一个指令应该以特殊方式绑定。例如，.prevent 修饰符告诉 v-on 指令对于触发的事件调用 event.preventDefault()
```
<form v-on:submit.prevent="onSubmit">...</form>
```

5. 指令缩写
*v-bind*
```
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>

<!-- 动态参数的缩写 (2.6.0+) -->
<a :[key]="url"> ... </a>
```

*v-on*
```
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>

<!-- 动态参数的缩写 (2.6.0+) -->
<a @[event]="doSomething"> ... </a>
```

> [指令](https://cn.vuejs.org/v2/guide/syntax.html#%E6%8C%87%E4%BB%A4)


# 三、Class 与 Style 绑定

操作元素的 class 列表和内联样式是数据绑定的一个常见需求。因为它们都是属性，所以我们可以用 v-bind 处理它们：只需要通过表达式计算出字符串结果即可。不过，字符串拼接麻烦且易错。因此，在将 v-bind 用于 class 和 style 时，Vue.js 做了专门的增强。表达式结果的类型除了字符串之外，还可以是对象或数组。



1. 对象语法
通过绑定class的值动态设置样式
```
<div v-bind:class="{ active: isActive }"></div>
data: {
  isActive: true,
  hasError: false
}
```

2. 数组语法
`<div v-bind:class="[activeClass, errorClass]"></div>`


3. 绑定内联样式-对象语法

通过绑定style的值动态设置样式：
```
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
data: {
  activeColor: 'red',
  fontSize: 30
}
```

4. 绑定内联样式-数组语法

```
<div v-bind:style="[baseStyles, overridingStyles]">
</div>
```
