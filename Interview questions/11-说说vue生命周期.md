# 面试中如何简短精干的描述vue生命周期

出去面试，面试官看到你的简历上的项目经验写着使用Vue + Vue-router + Vuex。我相信，写上这些会让你简历提升一个档次，更会提升面试官对你好感。
接下来...

###### 面试官：什么是vue生命周期，详细说说你的理解。

如果你只是简单罗列出这几个钩子函数的名称，不具体深入阐述的话，你这样的回答很难令面试官满意。你可以直接把下面这张图扔给面试官（哈哈哈，开个玩笑啦）。



![img](https://upload-images.jianshu.io/upload_images/12462640-270ee3bcacbe6a50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

lifecycle.png

这张图是我从Vue官网上拷贝下来的，只要你能理解了这张图，也就对Vue的生命周期有了一个大致的了解。
但是如何简短精干，条理清晰的解释清楚vue生命周期，从而让面试官对你留下好印象呢？
vue生命周期总共分为8个阶段 创建前/后，载入前/后，更新前/后，销毁前/后。

> 创建/前后：在beforeCreated阶段，vue实例的挂载元素el还没有。

*在beforeCreated阶段可以添加loading事件，在created阶段发起后端请求，拿回数据*

> 载入前/后：在beforeMount阶段，vue实例的$el和data都初始化，但是挂载之前为虚拟的dom节点，data.message还未替换，页面无重新渲染。在mounted阶段，vue实例挂载完成，data.message成功渲染。

> 更新前/后：当data变化时，会触发beforeUpdate和updated方法。

> 销毁前/后：在执行destroy方法后，对data的改变不会再触发周期函数，说明此时vue实例已经解除了事件监听以及和dom的绑定，但是dom结构依然存在。

如果在面试中你能回答上这些一定会给面试官留下个好印象的