**你知道nextTick的原理吗？**

vue中有一个较为特殊的API，nextTick。根据官方文档的解释，它可以在DOM更新完毕之后执行一个回调，用法如下：

> ```js
> // 修改数据
> 
> vm.msg = 'Hello'
> 
> // DOM 还没有更新
> 
> Vue.nextTick(function () {
> 
> // DOM 更新了
> 
> })
> ```
>
> 

尽管MVVM框架并不推荐访问DOM，但有时候确实会有这样的需求，尤其是和第三方插件进行配合的时候，免不了要进行DOM操作。而nextTick就提供了一个桥梁，确保我们操作的是更新后的DOM。

本文从这样一个问题开始探索：vue如何检测到DOM更新完毕呢？

检索一下自己的前端知识库，能监听到DOM改动的API好像只有MutationObserver了，后面简称MO.

### 理解MutationObserver

MutationObserver是HTML5新增的属性，用于监听DOM修改事件，能够监听到节点的属性、文本内容、子节点等的改动，是一个功能强大的利器，基本用法如下：

> ```js
> //MO基本用法
> var observer = new MutationObserver(function(){
> //这里是回调函数
> 
> console.log('DOM被修改了！');
> 
> });
> var article = document.querySelector('article');
> observer.observer(article);
> ```
>
> 

MO的使用不是本篇重点。这里我们要思考的是：vue是不是用MO来监听DOM更新完毕的呢？

那就打开vue的源码看看吧，在实现nextTick的地方，确实能看到这样的代码：

> //vue@2.2.5 /src/core/util/env.js
>
> 
>
> ```js
> if (typeof MutationObserver !== 'undefined' && (isNative(MutationObserver) || MutationObserver.toString() === '[object MutationObserverConstructor]')) {
> 
> var counter = 1
> 
> var observer = new MutationObserver(nextTickHandler)
> 
> var textNode = document.createTextNode(String(counter))
> 
> observer.observe(textNode, {
> characterData: true
> })
> 
> timerFunc = () => {
> 
> counter = (counter + 1) % 2
> textNode.data = String(counter)
> 
> }
> 
> }
> ```
>
> 

简单解释一下，如果检测到浏览器支持MO，则创建一个文本节点，监听这个文本节点的改动事件，以此来触发nextTickHandler（也就是DOM更新完毕回调）的执行。后面的代码中，会执行手工修改文本节点属性，这样就能进入到回调函数了。

大体扫了一眼，似乎可以得到实锤了：哦！vue是用MutationObserver监听DOM更新完毕的！

难道不感觉哪里不对劲吗？让我们细细想一下：

1. 我们要监听的是模板中的DOM更新完毕，vue为什么自己创建了一个文本节点来监听，这有点说不通啊！

    

2. 难道自己创建的文本节点更新完毕，就能代表其他DOM节点更新完毕吗？这又是什么道理！

看来我们上面得出的结论并不对，这时候就需要讲讲js的事件循环机制了。

### 事件循环（Event Loop）

在js的运行环境中，我们这里光说浏览器吧，通常伴随着很多事件的发生，比如用户点击、页面渲染、脚本执行、网络请求，等等。为了协调这些事件的处理，浏览器使用事件循环机制。

简要来说，事件循环会维护一个或多个任务队列（task queues），以上提到的事件作为任务源往队列中加入任务。有一个持续执行的线程来处理这些任务，每执行完一个就从队列中移除它，这就是一次事件循环了

我们平时用setTimeout来执行异步代码，其实就是在任务队列的末尾加入了一个task，待前面的任务都执行完后再执行它。

关键的地方来了，每次event loop的最后，会有一个UI render步骤，也就是更新DOM。标准为什么这样设计呢？考虑下面的代码：

> ```js
> for(let i=0; i<100; i++){
> dom.style.left = i + 'px';
> }
> ```
>
> 

浏览器会进行100次DOM更新吗？显然不是的，这样太耗性能了。事实上，这100次for循环同属一个task，浏览器只在该task执行完后进行一次DOM更新。

那我们的思路就来了：只要让nextTick里的代码放在UI render步骤后面执行，岂不就能访问到更新后的DOM了？

vue就是这样的思路，并不是用MO进行DOM变动监听，而是用队列控制的方式达到目的。那么vue又是如何做到队列控制的呢？我们可以很自然的想到setTimeout，把nextTick要执行的代码当作下一个task放入队列末尾。

然而事情却没这么简单，vue的数据响应过程包含：数据更改->通知Watcher->更新DOM。而数据的更改不由我们控制，可能在任何时候发生。如果恰巧发生在repaint之前，就会发生多次渲染。这意味着性能浪费，是vue不愿意看到的。

所以，vue的队列控制是经过了深思熟虑的（也经过了多次改动）。在这之前，我们还需了解event loop的另一个重要概念，microtask.

### microtask

从名字看，我们可以把它称为微任务。对应的，task队列中的任务也被叫做macrotask。名字相似，性质可不一样了。

每一次事件循环都包含一个microtask队列，在循环结束后会依次执行队列中的microtask并移除，然后再开始下一次事件循环。

在执行microtask的过程中后加入microtask队列的微任务，也会在下一次事件循环之前被执行。也就是说，macrotask总要等到microtask都执行完后才能执行，microtask有着更高的优先级。

microtask的这一特性，简直是做队列控制的最佳选择啊！vue进行DOM更新内部也是调用nextTick来做异步队列控制。而当我们自己调用nextTick的时候，它就在更新DOM的那个microtask后追加了我们自己的回调函数，从而确保我们的代码在DOM更新后执行，同时也避免了setTimeout可能存在的多次执行问题。

常见的microtask有：Promise、MutationObserver、Object.observe(废弃)，以及nodejs中的process.nextTick.

咦？好像看到了MutationObserver，难道说vue用MO是想利用它的microtask特性，而不是想做DOM监听？对喽，就是这样的。核心是microtask，用不用MO都行的。事实上，vue在2.5版本中已经删去了MO相关的代码，因为它是HTML5新增的特性，在iOS上尚有bug。

那么最优的microtask策略就是Promise了，而令人尴尬的是，Promise是ES6新增的东西，也存在兼容问题呀~ 所以vue就面临一个降级策略。

### vue的降级策略

上面我们讲到了，队列控制的最佳选择是microtask，而microtask的最佳选择是Promise.但如果当前环境不支持Promise，vue就不得不降级为macrotask来做队列控制了。

macrotask有哪些可选的方案呢？前面提到了setTimeout是一种，但它不是理想的方案。因为setTimeout执行的最小时间间隔是约4ms的样子，略微有点延迟。还有其他的方案吗？

不卖关子了，在vue2.5的源码中，macrotask降级的方案依次是：setImmediate、MessageChannel、setTimeout.

setImmediate是最理想的方案了，可惜的是只有IE和nodejs支持。

MessageChannel的onmessage回调也是microtask，但也是个新API，面临兼容性的尴尬...

所以最后的兜底方案就是setTimeout了，尽管它有执行延迟，可能造成多次渲染，算是没有办法的办法了。

1. vue用异步队列的方式来控制DOM更新和nextTick回调先后执行

    

2. microtask因为其高优先级特性，能确保队列中的微任务在一次事件循环前被执行完毕

    

3. 因为兼容性问题，vue不得不做了microtask向macrotask的降级方案

