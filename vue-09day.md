谈谈你对vue组件之间通信的理解？

组件通信的常用方式：

props

eventbus

vuex :创建唯一的全局数据管理者store，通过它管理数据并通知组件状态变更.

自定义事件

​	边界情况

​		$parent/$root  兄弟组件之间通信可通过共同祖辈搭桥

​		$children :父组件可以通过$children访问子组件实现父子通信。但是，$children不能保证子元素顺序

​		$refs：获取子节点引用

​		provide/inject ：能够实现祖先和后代之间传值

​	非prop特性：

​		$attrs  / $listeners  ：包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定 ( class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 ( class 和 style 除外)，并且可以通过 vbind="$attrs" 传入内部组件——在创建高级别的组件时非常有用。

插槽 插槽语法是Vue 实现的内容分发 API，用于复合组件开发。该技术在通用组件库开发中有大量应用。

