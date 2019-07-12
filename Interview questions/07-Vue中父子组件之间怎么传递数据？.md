### 1、父组件向子组件传值

（1）第一个就是要明白怎么在父页面中向子组件中传值？

- 你可以给子组件传入一个静态的值：



![img](https:////upload-images.jianshu.io/upload_images/2891127-60092a004be0dbba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/602/format/webp)

图片来自vue官网

- 但我们一般都是需要传动态的值，所以需要v-bind绑定：



![img](https:////upload-images.jianshu.io/upload_images/2891127-aaf51402e79ff193.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/599/format/webp)

图片来自vue官网

当然，你传的值可以是数字、对象、数组等等，[参见vue官网](https://cn.vuejs.org/v2/guide/components-props.html)。

（2）第二个就是要知道如何在子组件中接受父页面传过来的值，有几点需要了解：

- 组件实例的作用域是孤立的；
- 子组件要显式的用`props`选项声明它预期的数据，如：

```
// 某个子组件中：
export default {
  props: {
    title: {
      type: String,
      default: 'hello world'
    }
  }
}
```

然后就可以在页面中使用了，具体我们在项目中体现。

> 以上就是`单向数据流`的一般表现了： 父级 prop 的更新会向下流动到子组件中，但是反过来则不行。

### 2、子组件向父组件传值

那么我们延伸一下，子组件怎么向父组件传值？
 （1）基本概念

- 在Vue中，父子组件的关系可以总结为`prop`向下传递，`事件`向上传递。父组件通过`prop`给子组件下发数据，子组件通过`事件`给父组件发送信息。



![img](https:////upload-images.jianshu.io/upload_images/2891127-591b88f49fb05f19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/246/format/webp)

props down , events up

- 每个Vue实例都实现了事件接口：使用`$on(evntName)`监听事件；使用`$emit(eventName,optionalPayload)`触发事件。另外，父组件可以在使用子组件的地方直接用`v-on`来监听子组件触发的事件。

（2）举例说明

- 父组件在组件上定义了一个自定义事件`childFn`，事件名为`parentFn`用于接受子组件传过来的`message`值。

```
<!-- 父组件 -->
<template>
    <div class="test">
      <test-com @childFn="parentFn"></test-com>
      <br/> 
      子组件传来的值 : {{message}}
    </div>
</template>

<script>
export default {
    // ...
    data: {
        message: ''
    },
    methods: {
       parentFn(payload) {
        this.message = payload;
      }
    }
}
</script>
```

- 子组件是一个`buttton`按钮，并为其添加了一个`click`事件，当点击的时候使用`$emit()`触发事件，把`message`传给父组件。

```javascript
<!-- 子组件 -->
<template> 
<div class="testCom">
    <input type="text" v-model="message" />
    <button @click="click">Send</button>
</div>
</template>
<script>
export default {
    // ...
    data() {
        return {
          // 默认
          message: '我是来自子组件的消息'
        }
    },
    methods: {
      click() {
            this.$emit('childFn', this.message);
        }
    }    
}
</script>
```

- 这样我们就基本实现了子组件向父组件发送值了。
   大家可以在[git仓库](https://github.com/Ewall1106/mall)(chapter23)中于`test.vue`和`testCom.vue`这两个组件查看具体的代码。



![img](https:////upload-images.jianshu.io/upload_images/2891127-c5db663c172ea735.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/388/format/webp)

点击send发送



![img](https:////upload-images.jianshu.io/upload_images/2891127-129823130a431ace.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/391/format/webp)

父组件接收组件传过来的值

### 3、小结

通过`"props down , events up"`我们就简单的实现了父子组件之间的双向传值，这是很基本的知识点，其它还有$invoke等等方法，大家可以去官网好好看看。
 关于通信和状态管理就必须了解`vuex`了，后面我们在项目中当涉及了再好好讲。