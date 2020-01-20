问题：v-if和v-for哪个优先级高？如果两个同时出现，应该怎么优化得到更好的性能？

答案：当 Vue 处理指令时，v-for 比 v-if 具有更高的优先级。

1.为了过滤一个列表中的项目 (比如 v-for="user in users" v-if="user.isActive")，在这种情形下，哪怕我们只渲染出一小部分用户的元素，也得在每次重渲染的时候遍历整个列表，不论活跃用户是否发生了变化。

方案：将 users 替换为一个计算属性 (比如 activeUsers)，让其返回过滤后的列表。

2.为了避免渲染本应该被隐藏的列表 (比如 v-for="user in users" v-if="shouldShowUsers")。这种情形下，可以将 v-if 移动至容器元素上，这样当不满足shouldShowUsers时，也就不会循环users。
