# 对vuex的理解

一、概念

　　vuex是一个专为vue.js应用程序开发的状态管理模式（它采用集中式存贮管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化）。

二、五大核心属性

　　核心属性为：state，getter，mutation，action，module

- **state**：存储数据，存储状态；在根实例中注册了store 后，用 `this.$store.state` 来访问；对应vue里面的data；存放数据方式为响应式，vue组件从store中读取数据，如数据发生变化，组件也会对应的更新。
- **getters**：可以认为是 store 的计算属性，相当于 vue中的 computed，依赖于 state里面的值。它的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。
- **mutations**：用于修改状态，store里面的数仅能通过`mutations`里面的方法改变,但是必须是同步的。更改 vuex 的 store 中的状态的唯一方法是提交 mutation，也就是$store.commit。
- **actions**：包含任意异步操作，用它处理完后再触发mutations来改变状态。
- **module**：将 store 分割成模块，每个模块都具有state、mutation、action、getter、甚至是嵌套子模块。

三、vuex的数据传递流程

　　当组件进行数据修改的时候我们需要调用Dispatch来触发Actions里面的方法。

　　actions里面的每个方法中都会有一个commit方法，当方法执行的时候会通过commit来触发mutations里面的方法进行数据的修改。　　

　　mutations里面的每个函数都会有一个state参数，这样就可以在mutations里面进行state的数据修改，当数据修改完毕后，会传导给页面。页面的数据也会发生改变。
