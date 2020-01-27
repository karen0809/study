**谈谈你对MVC、MVP和MVVM的理解？**

**MVC（Model、view、Controller）**
 **Model（模型）**是应用程序中用于处理应用程序数据逻辑的部分。
 　通常模型对象负责在数据库中存取数据。

**View（视图）**是应用程序中处理数据显示的部分。
 　通常视图是依据模型数据创建的。

**Controller（控制器）**是应用程序中处理用户交互的部分。
 　通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。

在具体的业务场景中,  C作为M和V之间的连接, 负责获取输入的业务数据, 然后将处理后的数据输出到界面上做相应展示, 另外, 在数据有所更新时, C还需要及时提交相应更新到界面展示. 在上述过程中, 因为M和V之间是完全隔离的, 所以在业务场景切换时, 通常只需要替换相应的C, 复用已有的M和V便可快速搭建新的业务场景. 

![15226743-86c2d4be3b5833c3](C:\Users\ZHQ\Desktop\15226743-86c2d4be3b5833c3.webp)

​																		MVC框架模式图



**MVP**

​	MVP从视图层中分离了行为（事件响应）和状态（属性，用于数据展示），它创建了一个视图的抽象，也就是presenter层，而视图就是P层的『渲染』结果。P层中包含所有的视图渲染需要的动态信息，包括视图的内容（text、color）、组件是否启用（enable），除此之外还会将一些方法暴露给视图用于某些事件的响应。

![img](https://upload-images.jianshu.io/upload_images/15226743-947a7c01f8199148.png?imageMogr2/auto-orient/strip|imageView2/2/w/537/format/webp)

​																		MVP框架模式图

**MVVM**

​	MVVM是Model-View-ViewModel的简写。它本质上就是MVC 的改进版。MVVM 就是将其中的View 的状态和行为抽象化，让我们将视图 UI 和业务逻辑分开。当然这些事 ViewModel 已经帮我们做了，它可以取出 Model 的数据同时帮忙处理 View 中由于需要展示内容而涉及的业务逻辑。

![img](https://upload-images.jianshu.io/upload_images/15226743-1b2adc4a66e12c6e.png?imageMogr2/auto-orient/strip|imageView2/2/w/715/format/webp)MVC框架																		MVVM模式图
