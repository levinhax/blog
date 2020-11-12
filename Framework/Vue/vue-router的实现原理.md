## vue-router的使用

router/index.js
```
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
```

实例化router，并配置router的配置对象
router/index.js
```
const Home = () => import('../views/Home')

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    // route level code-splitting
    // this generates a separate chunk (about.[hash].js) for this route
    // which is lazy-loaded when the route is visited.
    component: () => import(/* webpackChunkName: "about" */ '../views/About.vue')
  }
]

const router = new VueRouter({
  routes
})

export default router
```

在vue实例上配置router实例
main.js
```
import router from "@/router";

new Vue({
  router,
  render: h => h(App),
}).$mount('#app')
```

使用
```
<template>
  <div id="app">
    <router-link to="/">home</router-link>|
    <router-link to="/about">about</router-link>|
    <router-view></router-view>
  </div>
</template>
```

router-link和router-view是两个Vue全局组件，他们分别用来跳转路由和展示路由对应的组件内容。

我们点击了router-link时导致路由变了，vue-router内部必然是在监听路由变化，根据路由规则找到匹配的组件，然后在router-view中渲染。所以，**路由切换最终是页面的不同组件的展示，而没有真正去刷新页面**。

## hash与history模式

默认hash模式，如果浏览器不支持history新特性,则采用hash方式

源码中的实现
```
// 根据mode确定history实际的类并实例化
switch (mode) {
  case 'history':
    this.history = new HTML5History(this, options.base)
    break
  case 'hash':
    this.history = new HashHistory(this, options.base, this.fallback)
    break
  case 'abstract':
    this.history = new AbstractHistory(this, options.base)
    break
  default:
    if (process.env.NODE_ENV !== 'production') {
      assert(false, `invalid mode: ${mode}`)
    }
}
```

### HashHistory

HashHistory.push() 和 HashHistory.replace()

- HashHistory.push() 将新路由添加到浏览器访问历史的栈顶
- HashHistory.replace() 它并不是将新路由添加到浏览器访问历史的栈顶，而是替换掉当前的路由

### History

History interface是浏览器历史记录栈提供的接口，通过back(), forward(), go()等方法，我们可以读取浏览器历史记录栈的信息，进行各种跳转操作。

## vue-router核心实现原理

1. 实现一个静态install方法，因为作为插件都必须有这个方法，给Vue.use()去调用；
2. 可以监听路由变化；
3. 解析配置的路由，即解析router的配置项routes，能根据路由匹配到对应组件；
4. 实现两个全局组件router-link和router-view；

```
let Vue;
class CVueRouter {
    constructor(options){
        this.$options=options;
        this.$routerMap={};//{"/":{component:...}}
        // url 响应式，当值变化时引用的地方都会刷新
        this.app = new Vue({
            data:{
                current:"/"
            }
        });
    }
    // 初始化
    init(){
        // 监听事件
        this.bindEvent();
        // 解析路由
        this.createRouteMap();
        // 声明组件
        this.initComponent();
    }
    bindEvent(){
        window.addEventListener('hashchange',this.onHashchange.bind(this));
    }
    onHashchange(){
        this.app.current = window.location.hash.slice(1) || "/";
    }
    createRouteMap(){
        this.$options.routes.forEach(route=>{
            this.$routerMap[route.path]=route;
        })
    }
    initComponent(){
        Vue.component('router-link',{
            props:{
                to:String,
            },
            render(h){
                return h('a',{attrs:{href:'#'+this.to}},[this.$slots.default])
            }
        });
        Vue.component('router-view',{
            render:(h)=>{
                const Component = this.$routerMap[this.app.current].component;
                return h(Component)
            }
        });
    }
}

// 参数是vue构造函数，Vue.use(router)时,执行router的install方法并把Vue作为参数传入
CVueRouter.install = function(_vue){
    Vue = _vue;
    //全局混入
    Vue.mixin({
        beforeCreate(){//拿到router的示例，挂载到vue的原型上
            if (this.$options.router) {
                Vue.prototype.$router=this.$options.router;
                this.$options.router.init();
            }
        }
    })
}
export default CVueRouter;
```

- Vue.use(Router)时，会调用router的install方法并把Vue类传入，混入beforeCreate方法，即在Vue实例化后挂载前在vue原型上挂个$router方法(因为这样后面才能用this.$router.push()...)，然后调用router实例的init方法；
- 在init中把三件事情都干了，监听路由，解析路由（路由mapping匹配），定义组件；
- 存储当前路由的变量this.app.current非一般的变量，而是借用Vue的响应式定义的，所以当路由变化时只需要给这个this.app.current赋值，而router-view组件刚好引用到这个值，当其改变时所有的引用到的地方都会改变，则得到的要展示的组件也就响应式的变化了。
