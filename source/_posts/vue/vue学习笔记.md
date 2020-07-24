---
title: vue 实例/模版语法
date: 2020-05-25 10:55:02
tags:
- vue
- 生命周期
- 前端
---

### vue 实例/模版语法

#### vue 实例

##### 数据与方法
当一个Vue实例被创建时，会把`data`中所有的property加入到`响应式系统`中，意思就是property的值变动，那么视图也会跟着相应。
- 当实例创建后，如果添加新的property的话，被创建的值改动不会导致视图更新。
- 如果data对象使用了`Object.freeze()`的话，响应系统也无法追踪变化

##### 生命周期
理解生命周期是对一个框架最基本也是最重要的部分，不可以用箭头函数形式声明生命周期函数，因为箭头函数没this，而this会从上级词法作用域查找，就会出现不可预估的错误。
- beforeCreate：实例初始化后调用，数据观测(data observer) 和 event/watcher 事件配置之前被调用
- created：vue实例被创建后被调用，数据观测已经开始，但是还没挂载到dom，$el目前不可以使用
- beforeMount：挂载前被调用
- mounted：实例被挂载完后被调用，不保证所有的子组件都被挂载完，如果希望所有子组件都更新完可以调用vm.$nextTick
- beforeUpdate：数据更新时被调用，发生在virtual Dom 打补丁前，适合访问现有的dom，移除事件监听器
- updated：虚拟Dom重新渲染和打补丁后调用，不建议在这个函数更新状态，建议使用computed或者watcher更新
- actived：被keep-alive缓存的组件激活时调用
- deactivated：被keep-alive缓存的组件停用时被调用
- beforeDestroy：实例被摧毁前调用，实例仍然可以用
- destroyed：实例被摧毁后调用
- errorCaptured：当捕获一个来自子孙组件的错误时被调用

![image](https://cn.vuejs.org/images/lifecycle.png)


#### 模版语法
Vue主要利用模板语法把模版编译成`虚拟DOM`，结合响应式系统，计算出需要重新渲染多少组件，并把DOM操作数减至最少。

##### 差值
- 文本：双大括号Mustache语法，
- 原始HTML：使用v-html指令输出，但是容易被XSS攻击
- Attribute：使用v-bind可以动态绑定attribute值
- 使用Javascript表达式：在大括号以及attribute都可以使用一个表达式，但是不建议，建议使用computed属性。模板表达式都是被放在沙盒中，只能访问全局变量的一个白名单，无法访问用户自己定义的全局变量。

##### 指令
是指带有`v-`的attribute，指责是当表达式值变更时会响应式作用于DOM
- 动态参数：如v-bind:[someAttr]
- 修饰符：使用.表达式指定修饰符，如.prevent，v-on:submit.prevent告知调用event.preventDefault()。


#### 计算属性(computed)
计算属性会缓存计算出来的值，如果计算属性中计算的值发生变化，那计算属性的函数会重新被调用，重新计算，即基于`响应式依赖`进行缓存。

#### 侦听属性(watch)
侦听器与计算属性类似，就是属性变更时会重新调用侦听器。一般用于属性变更时重新发请求这些场景。