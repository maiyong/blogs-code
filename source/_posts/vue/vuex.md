---
title: vuex
date: 2020-05-28 10:55:02
tags:
- vue
- 前端
---

###  vuex
Vuex是专门为Vue.js设计的状态管理模式。Vuex是全局的属性。
- Vuex的状态存储是响应式的
- 不可以直接修改，需要`commit`到`mutation`

#### State
`state`就是单一状态树，即用一个`state`对象保存整个应用的状态。

#### Getter
`getter`就像计算属性一样，会把结果缓存起来，当函数里的state变更时会重新执行。但也可以使用方法，但是使用方法就不会缓存。

#### Mutation
`mutation`类似于`redux`里的`reducer`，存函数，不能做异步操作，主要作用是更新state。

#### Action
`action`主要是为`mutation`服务，提交`mutation`到Mutation，而且`action`可以进行异步操作，解决`mutation`不能执行异步操作的问题

#### Module
由于使用单一状态树，会导致store变得非常臃肿，`module`就是解决这个问题的，把`store`分割成模块，每个模块有各自的`state`、`mutation`、`action`和`getter`

!['vuex'](https://vuex.vuejs.org/vuex.png)

#### plugins
类似`redux`的中间件，每次`mutation`后调用