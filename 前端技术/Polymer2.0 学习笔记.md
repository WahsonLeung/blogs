# Polymer 2.0 学习笔记

标签（空格分隔）： 前端技术

---

#### 1. dom-repeat渲染时机
```
<template is="dom-repeat" items="{{arr}}">
    [[compute(item)]]
</template>
...
<script>
...
ready(){
    super.ready();
    this.arr.push(1); //1
    setTimeout(() => {
        this.arr.push(2); //2
        this.notifySplices('arr'); //3
    },0);
}
compute(item){
    return item;
}
</script>
```
现象描述：当只执行//1处代码，polymer能够重新渲染dom-repeat，但是如果在setTimeout里面，必须在执行//2后执行//3代码才能生效。
原因分析：
1. 首先，我们试着通过打断点的方式，查明dom-repeat的渲染时机，那么首先办法构造一个切入点，如图，当dom-repeat渲染的时候，它必定会调用``compute``方法，我们在compute里面打上断点。

![此处输入图片的描述][1]
&nbsp;
2. 此时，通过调用栈大致能分析出dom-repeat的渲染发生在ready()执行的时候。

![此处输入图片的描述][2]
&nbsp;
3. 我们再查看``__debounceRender``的代码，可以看出这里异步调用了`` __render``函数，然后大家不用猜也知道``__render`在dom-repeat里是干嘛的了`。

![此处输入图片的描述][3]
&nbsp;
4. 那么答案已经浮出水面了。正因为``__render``是异步执行，它会在ready()执行后执行，在ready中对arr push的值会在``__render``中渲染出来。而setTimeout里面的任务则在``__render``之后执行，因此只能通过``this.notifySplices``触发dom-repeat重新渲染。
&nbsp;

  [1]: http://om0gfsbon.bkt.clouddn.com/201704251493132233975.png
  [2]: http://om0gfsbon.bkt.clouddn.com/20170425149313004957543.png
  [3]: http://om0gfsbon.bkt.clouddn.com/20170425149313013936431.png