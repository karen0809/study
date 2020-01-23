你怎么理解vue中的diff算法？

概念：diff算法是一种优化手段，将前后两个模块进行差异化对比，修补（更新）差异的过程叫做patch(打补丁)。

vue的diff算法的核心是在 `vue/src/core/vdom/patch.js`中`updateChildren`方法。

Diff 算法的主要过程：

1. 通过对比树的边界结点，缩小两个树的大小。
2. 通过key或者遍历找到可复用的结点。
3. 通过对比最后的索引值，删除和新增结点。
