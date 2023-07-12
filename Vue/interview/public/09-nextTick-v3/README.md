## 你知道nextTick吗，它是干什么的，实现原理是什么？

这道题考查大家对vue异步更新队列的理解，有一定深度，如果能够很好回答此题，对面试效果有极大帮助。



### 答题思路：

1. nextTick是啥？下一个定义
2. 为什么需要它呢？用异步更新队列实现原理解释
3. 我再什么地方用它呢？抓抓头，想想你在平时开发中使用它的地方
4. 下面介绍一下如何使用nextTick
5. 最后能说出源码实现就会显得你格外优秀



先看看官方定义

> Vue.nextTick( \[callback, context\] )
>
> 在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。
>
> ```js
> // 修改数据
> vm.msg = 'Hello'
> // DOM 还没有更新
> Vue.nextTick(function () {
> // DOM 更新了
> })
> ```

### 回答范例：

1. nextTick是Vue提供的一个全局API，由于   **vue的异步更新策略**  导致我们对数据的修改不会立刻体现在dom变化上，此时如果想要立即获取更新后的dom状态，就需要使用这个方法
2. **Vue 在更新 DOM 时是异步执行的。只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。nextTick方法会在队列中加入一个回调函数，确保该函数在前面的dom操作完成后才调用。**
3. 所以当我们想在修改数据后立即看到dom执行结果就需要用到nextTick方法。
4. 比如，我在干什么的时候就会使用nextTick，传一个回调函数进去，在里面执行dom操作即可。
5. 我也有简单了解nextTick实现，它会在callbacks里面加入我们传入的函数，然后用timerFunc异步方式调用它们，首选的异步方式会是Promise。这让我明白了为什么可以在nextTick中看到dom操作结果。



### 有趣的问题

```javascript
var vm = new Vue({
    el: '#example',
    data: {
        msg: 'begin',
    },
    mounted () {
      this.msg = 'end'
      console.log('1')
      setTimeout(() => { // macroTask
         console.log('3')
      }, 0)
      Promise.resolve().then(function () { //microTask
        console.log('promise!')
      })
      this.$nextTick(function () {
        console.log('2')
      })
  }
})
```

> 这个的执行顺序想必大家都知道先后打印：1、promise、2、3。
>
> 1. 因为首先触发了`this.msg = 'end'`，导致触发了`watcher`的`update`，从而将更新操作callback push进入vue的事件队列。
>
>    
>
> 2. `this.$nextTick`也为事件队列push进入了新的一个callback函数，他们都是通过`setImmediate` --> `MessageChannel` --> `Promise` --> `setTimeout`来定义`timeFunc`。而` Promise.resolve().then`则是microTask，所以会先去打印promise。
>
>    
>
> 3. 在支持`MessageChannel`和`setImmediate`的情况下，他们的执行顺序是优先于`setTimeout`的（在IE11/Edge中，setImmediate延迟可以在1ms以内，而setTimeout有最低4ms的延迟，所以setImmediate比setTimeout(0)更早执行回调函数。其次因为事件队列里，优先收入callback数组）所以会打印2，接着打印3
>
>    
>
> 4. 但是在不支持`MessageChannel`和`setImmediate`的情况下，又会通过`Promise`定义`timeFunc`,也是老版本Vue 2.4 之前的版本会优先执行`promise`。这种情况会导致顺序成为了：1、2、promise、3。因为this.msg必定先会触发dom更新函数，dom更新函数会先被callback收纳进入异步时间队列，其次才定义`Promise.resolve().then(function () { console.log('promise!')})`这样的microTask，接着定义`$nextTick`又会被callback收纳。我们知道队列满足先进先出的原则，所以优先去执行callback收纳的对象。

### 异步更新队列----2.7

1、promise

2、MutationObserver

3、setImmediate

4、setTimeOut
