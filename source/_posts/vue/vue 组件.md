---
title: vue 组件
date: 2020-05-26 10:55:02
tags:
- vue
- 前端
---

###  vue 组件

#### 组件注册

##### 全局注册
使用`Vue.component('name')`进行注册的组件都是全局注册，可以在程序任意位置使用

##### 局部注册
直接在组件的`components`属性中进行声明即可


#### Prop

##### 单向数据流
所有的`prop`都使得其父子`prop`之间形成了一个单向下行绑定，目的是防止数据流混乱，每次父组件更新时，子组件的所有`prop`都会更新到最新，也就是说不能在组件中修改`prop`。

##### 类型检查
可以对`prop`类型进行检查验证，当类型验证失败时，`Vue`会发出一个警告，类型有
- String
- Number
- Boolean
- Array
- Object
- Date
- Function
- Symbol

##### 非Prop的Attribute
组件与根标签`class`与`style`attribute会对传进来的值进行合并，而其他会被替换。如果不想继承组件的attribute，可以使用`inheritAttrs: false`，但是这个声明对`class`与`style`无效

#### 自定义事件
可以使用`this.$emit('name')`与父组件进行事件通信。

##### 编译作用域
就是说父模版的所有内容都是在父级进行编译，因此插槽是编译后才被显示在子模版。如果需要引用子组件的props，可以使用作用域插槽，意思就是通过v-bind与v-slot:defalut来进行绑定。

##### 后备内容
有个场景是无数据时显示无数据内容，有传数据就显示传进来的数据，`slot`就可以实现，如
```html
<!-- 这种情况如果组件没传模版内容进来就显示Submit，否则显示传进来的内容 -->
<button type="submit">
  <slot>Submit</slot>
</button>
```

##### 具名插槽
每个插槽都有名称，没有名称的叫`default`，使用的时候就可以这样
```html
<template v-slot:header>
  <h1>title</h1>
<template/>
```

##### 插槽的缩写
与`v-on`和`v-bind`一样，`v-slot`也有缩写，#就可以


#### 动态组件

##### keep-alive
`keep-alive`是一个保存组件状态的一个标签，如
```html
<keep-alive>
  <!-- 失活的组件会缓存起来 -- >
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

#### 处理边界

##### 访问元素
- 访问根实例
可以使用`this.$root`来访问根实例中的属性，适用小规模的程序。

- 访问子组件实例
可以使用`ref`来访问子组件的实例
```html
<input ref="input"></input>
```

- 访问父级组件实例
和`$root`一样，可以使用`$parent`访问父组件的属性

- 依赖注入
如果所有的子组件都需要访问父组件的一个方法，那么就可以使用`provide`与`inject`，`provide`在父组件声明方法，`inject`在子组件中接收。
```html
// 父组件
provide: function() {
  return {
    getMap: this.getMap
  } 
}

// 子组件
inject: ['getMap']
```

#### 程序化的事件侦听器
其实就是事件侦听，通过`$emit`与`$on`，`$once`，`$off`三个事件
```html
mounted: function() {
  var picker = new Pikaday({
    field: this.$ref.input,
    format: 'YYYY-MM-DD'
  }) 
  this.$once('hook:beforeDestroy', function() {
    picker.destroy()
  })
}
```

#### 进入/离开&列表过度
Vue提供了`transition`的封装组件，在`条件渲染(v-if)`，`条件展示(v-show)`，`动态组件`，`组件根节点`会添加过渡。当插入或删除包含在`transition`组件中的元素，Vue会做以下处理：
- 判断目标元素是否应用了`CSS过渡或动画`，如果是，会在恰当的时机添加/删除CSS类名
- 如果过渡组件提供了`钩子函数`，则这些函数会被调用。
- 如果没有`钩子函数`和`CSS过渡动画`，则下一帧会立即执行。
##### 过渡的类名
6个
- v-enter
开始过渡状态
- v-enter-active
进入过渡动画生效状态
- v-enter-to
结束过渡状态
- v-leave
开始离开过渡状态
- v-leave
离开过渡生效状态
- v-leave-to
离开过渡结束状态

##### 列表的过渡
使用`<transition-group>`组件进行过渡
