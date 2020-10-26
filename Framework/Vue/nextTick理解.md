## 介绍

Vue官网：在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。

```
Vue.nextTick( [callback, context] )

Vue.nextTick(function () {
  // DOM 更新了
})

Vue.nextTick()
  .then(function () {
    // DOM 更新了
  })
```

在 vue 中 created 函数钩子函数执行的时候DOM 其实并未进行任何渲染，所以得放在 nextTick 中去获取 dom，与其对于得生命周期钩子函数是 mounted

## 事件循环

js 是单线程执行，当然，现在又有了一个 worker 创造了多线程环境，但是 worker 受限很多， js 执行是有一个执行栈，主要分了，宏任务（macro-task）和 微任务（micro-task）

宏任务有那些

- setTimeout
- I/O
- setInterval
- setImmediate
- 主线程
- MessageChannel

微任务有那些

- Promise 系列 .then .catch .finally
- process.nexttick
- MutationObserver

```
// 执行流程
              （执行一个宏任务，产生宏任务，入栈）
                    ↑
--------↑------- 宏任务 ←--—--
        |           |         |
        |           |         |
        |           |         |
        |           ↓         | 没有
        |         微任务 ------
        |           |↘
        |           |（执行所有微任务过程中产生微任务，继续执行）
        |           |
        |           | 有 执行完所以微任务
        |           |
        |___________↓

--------------------↓------------------
                直到栈为空

```

看一个实例：
```
console.log(1)

setTimeout(() => {
  console.log(8)
}, 2000)

setTimeout(() => {
  console.log(3)
  Promise.resolve().then(() => {
    console.log(4)
  })
  setTimeout(() => {
    console.log(6)
  }, 3000)
}, 1000)

new Promise((resolve, reject) => {
  console.log(5)
  resolve()
}).then(() => {
  console.log(7)
})

console.log(2)

// 结果：1 5 2 7 3 4 8 6
```

- 主线程 (宏任务) 打印 1 - 5 - 2 【new Promise 会立即执行，不属于，微任务】
- 执行所有微任务 promise.then 打印 7
- 在执行栈中抛出一个【可以执行的 -> 到时间，虽然，第一个 setTimeout 首先注册，在任务队列栈底】宏任务执行 3 - 4
- 在执行所有微任务 【没有】
- 在抛出一个宏任务执行 8
- 在执行所有微任务 【没有】
- 在抛出一个宏任务执行 6
- 任务执行完毕
- 1 5 2 7 3 4 8 6

## nextTick

在 Vue 整个 nextTick 的作用
```
  Vue.component('example', {
  template: '<span>{{ message }}</span>',
  data: function () {
    return {
      message: '未更新'
    }
  },
  methods: {
    updateMessage: function () {
      this.message = '已更新'
      console.log(this.$el.textContent) // => '未更新'
      this.$nextTick(function () {
        console.log(this.$el.textContent) // => '已更新'
      })
    }
  }
})
// 在 updateMessage 方法中，更新数据，立即获取更新后的 dom 是获取不到的，所以得把获取 dom 加到事件队列的栈，异步获取更新后的dom  
```

主线程更新前 ---> 遇到宏任务或微任务 ---> 放入栈 ---> 主线程执行完成，更新完成 ----> 执行栈 ---- > 获取更新后的dom
