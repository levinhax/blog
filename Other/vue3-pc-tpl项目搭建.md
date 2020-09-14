### vue create项目搭建

```
vue create vue3-pc-tpl
```

### 引入ant design vue

```
npm install --save ant-design-vue@next

npm install --save-dev babel-plugin-import
```

1. 配置ant-design-vue按需加载，修改babel.config.js
```
module.exports = {
  presets: ['@vue/cli-plugin-babel/preset'],
  plugins: [
    // 按需加载
    [
      'import',
      // style 为 true 加载 less文件
      { libraryName: 'ant-design-vue', libraryDirectory: 'es', style: 'css' }
    ]
  ]
}
```

修改about.vue页面
```
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <a-button type="primary">按钮</a-button>
  </div>
</template>

<script>
import { Button } from 'ant-design-vue'

export default {
  name: 'About',
  components: {
    [Button.name]: Button
  }
}
</script>
```

2. 完整引入

还原babel.config.js
```
module.exports = {
  presets: ['@vue/cli-plugin-babel/preset']
}
```

```
import { createApp } from 'vue'
import App from './App.vue'
import './registerServiceWorker'
import router from './router'
import store from './store'

import Antd from 'ant-design-vue'
import 'ant-design-vue/dist/antd.css'

createApp(App)
  .use(Antd)
  .use(store)
  .use(router)
  .mount('#app')
```

### Composition API

#### setup

setup函数有两个参数，分别是props和context

- props是setup函数的第一个参数，是组件外部传入进来的属性，与vue2.0的props基本一致
- context是setup函数的第二个参数，context是一个对象，里面包含了三个属性(都不能直接解构)
    - attrs与Vue2.0的this.$attrs是一样的，即外部传入的未在props中定义的属性
    - slots对应的是组件的插槽，与Vue2.0的this.$slots对应
    - emit对应的是Vue2.0的this.$emit, 即对外暴露事件

setup函数一般会返回一个对象，这个对象里面包含了组件模板里面要使用到的data与一些函数或者事件

#### reactive

reactive可以创建一个响应式的对象。

```
about.vue

<template>
  <div class="about">
    <h1>This is an about page</h1>
    <a-button type="primary">按钮</a-button>
    <h3>{{ state.text }}</h3>
  </div>
</template>

<script>
import { reactive } from 'vue'

export default {
  name: 'About',
  setup() {
    const state = reactive({
      text: '这是一段文字'
    })

    setTimeout(() => {
      state.text = '这是一段新的文字'
    }, 1000 * 5)

    return {
      state
    }
  }
}
</script>
```

如果直接把state解构出来
```
<template>
  <div class="about">
    <h3>{{ text }}</h3>
  </div>
</template>

<script>
import { reactive, toRefs } from 'vue'

export default {
  name: 'About',
  setup() {
    const state = reactive({
      text: '这是一段文字'
    })

    return {
      ...toRefs(state)
    }
  }
}
</script>
```

Vue3 中响应式对象是基于proxy实现的，通过reactive声明的对象添加新的属性可以实时反应出来。

*reactive通过传入一个对象然后返回了一个state,需要注意的是state与传入的对象是不用的，reactive对原始的对象并没有进行修改，而是返回了一个全新的对象，返回的对象是Proxy的实例。*

#### ref

```
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <a-button type="primary">按钮</a-button>
    <h3>{{ text }}</h3>
    <h3>{{ count }}</h3>
  </div>
</template>

<script>
import { reactive, toRefs, ref } from 'vue'

export default {
  name: 'About',
  setup() {
    const state = reactive({
      text: '这是一段文字'
    })
    const count = ref(10)

    setTimeout(() => {
      state.text = '这是一段新的文字'
      count.value = 12
    }, 1000 * 5)

    return {
      ...toRefs(state),
      count
    }
  }
}
</script>
```

reactive与ref的区别:

1. reactive传入的是一个对象，返回的是一个响应式对象，而ref传入的是一个基本数据类型（其实引用类型也可以），返回的是传入值的响应式值
2. reactive获取或修改属性可以直接通过state.prop来操作，而ref返回值需要通过name.value的方式来修改或者读取数据。但是需要注意的是，在template中并不需要通过.value来获取值，这是因为template中已经做了解套。

#### Vue3中使用v-model

在Vue3.0中为了实现统一，实现了让一个组件可以拥有多个v-model，同时删除掉了.sync。

```
<template>
  <!--在vue3.0中，v-model后面需要跟一个modelValue，即要双向绑定的属性名-->
  <a-input v-model:value="value" placeholder="Basic usage" />
</template>
```

使用Vue3.0自定义一个v-model示例

组件代码
```
<template>
  <div class="custom-input">
    <input :value="value" @input="_handleChangeValue" />
  </div>
</template>
<script>
export default {
  props: {
    value: {
      type: String,
      default: ""
    }
  },
  name: "CustomInput",
  setup(props, { emit }) {
    function _handleChangeValue(e) {
      // vue3.0 是通过emit事件名为 update:modelValue来更新v-model的
      emit("update:value", e.target.value);
    }
    return {
      _handleChangeValue
    };
  }
};
</script>
```

使用组件
```
<template>
  <!--在使用v-model需要指定modelValue-->
  <custom-input v-model:value="state.inputValue"></custom-input>
</template>
<script>
import { reactive } from "vue";
import CustomInput from "../components/custom-input";
export default {
  name: "Home",
  components: {
    CustomInput
  },
  setup() {
    const state = reactive({
      inputValue: ""
    });
    return {
      state
    };
  }
};
</script>
```

#### watch的使用

```
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <a-button type="primary">按钮</a-button>
    <h3>{{ text }}</h3>
    <h3>{{ count }}</h3>
  </div>
</template>

<script>
import { reactive, toRefs, ref, watch } from 'vue'

export default {
  name: 'About',
  setup() {
    const state = reactive({
      text: '这是一段文字'
    })
    const otherState = reactive({
      city: '浙江',
      area: '杭州'
    })
    const count = ref(10)

    watch(count, (newValue, oldValue) => {
      // 输出 12 10
      console.log(newValue, oldValue)
    })

    // watch 可以监听一个函数的返回值
    watch(
      () => {
        return otherState.city + otherState.area
      },
      value => {
        // 当otherState中的 city或者area发生变化时，都会进入这个函数
        console.log(`地点：${value}`)
      }
    )

    setTimeout(() => {
      state.text = '这是一段新的文字'
      count.value = 12
      otherState.area = '宁波'
    }, 1000 * 3)

    return {
      ...toRefs(state),
      ...toRefs(otherState),
      count
    }
  }
}
</script>
```

watch除了可以监听单个值或者函数返回值之外，也可以同时监听多个数据源
```
export default {
  setup() {
    const text = ref('')
    const count = ref(10')
    watch([text, count], ([text, count], [prevText, prevCount]) => {
      console.log(prevText, text)
      console.log(prevCount, count)
    })
  }
}
```

#### watchEffect的用法

watchEffect的用法与watch有所不同，watchEffect会传入一个函数，然后立即执行这个函数，对于函数里面的响应式依赖会进行监听，然后当依赖发生变化时，会重新调用传入的函数

```
import { ref, watchEffect } from 'vue'
export default {
  setup() {
    const id = ref('0')
    watchEffect(() => {
      // 先输出 0 然后两秒后输出 1
      console.log(id.value)
    })

    setTimeout(() => {
      id.value = '1'
    }, 2000)
  }
}
```

在Vue3.0中，watch与watchEffect同样也会返回一个unwatch函数，用于取消执行
```
export default {
  setup() {
    const id = ref('0')
    const unwatch = watchEffect(() => {
      // 仅仅输出0
      console.log(id.value)
    })

    setTimeout(() => {
      id.value = '1'
    }, 2000)
    // 1秒后取消watch，所以上面的代码只会输出0
    setTimeout(() => {
      unwatch()
    }, 1000)
  }
}
```

#### 计算属性

```
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <a-button type="primary">按钮</a-button>
    <h3>{{ text }}</h3>
    <h3>{{ count }}</h3>
    <h3>{{ place }}</h3>
  </div>
</template>

<script>
import { reactive, toRefs, ref, watch, computed } from 'vue'

export default {
  name: 'About',
  setup() {
    const state = reactive({
      text: '这是一段文字'
    })
    const otherState = reactive({
      city: '浙江',
      area: '杭州'
    })
    const count = ref(10)

    const place = computed(() => otherState.city + otherState.area)

    watch(count, (newValue, oldValue) => {
      // 输出 12 10
      console.log(newValue, oldValue)
    })

    // watch 可以监听一个函数的返回值
    watch(
      () => {
        return otherState.city + otherState.area
      },
      value => {
        // 当otherState中的 city或者area发生变化时，都会进入这个函数
        console.log(`地点：${value}`)
      }
    )

    setTimeout(() => {
      state.text = '这是一段新的文字'
      count.value = 12
      otherState.area = '宁波'
    }, 1000 * 3)

    return {
      ...toRefs(state),
      ...toRefs(otherState),
      count,
      place
    }
  }
}
</script>
```

Vue3.0的计算属性也可以设置getter和setter
```
export default {
  setup() {
    const info = reactive({
      firstName: '刘',
      lastName: '一刀'
    })

    const name = computed({
      get: () => info.firstName + '-' + info.lastName,
      set(val) {
        const names = val.split('-')
        info.firstName = names[0]
        info.lastName = names[1]
      }
    })

    return {
      name
    }
  }
}
```

### vue-router使用

```
import { createRouter, createWebHashHistory } from 'vue-router'
const router = createRouter({
  // vue-router有hash和history两种路由模式，可以通过createWebHashHistory和createWebHistory来指定
  history: createWebHashHistory(),
  routes
})

router.beforeEach(() => {
  
})

router.afterEach(() => {
  
})
export default router
```

#### 在setup中使用vue-router

```
import { useRoute, useRouter } from 'vue-router'

export default {
  setup() {
    // 获取当前路由
    const route = useRoute()
    // 获取路由实例
    const router = useRouter()
    // 当当前路由发生变化时，调用回调函数
    watch(() => route, () => {
      // 回调函数
    }, {
      immediate: true,
      deep: true
    })

    // 路由跳转
    const getHome = () => {
      router.push({
        path: '/'
      })
    }

    return {
      getHome
    }
  }
}
```

上面代码我们使用watch来监听路由是否发生变化，除了watch之外，我们也可以使用vue-router提供的生命周期函数

```
import { onBeforeRouteUpdate, onBeforeRouteLeave, useRoute } from 'vue-router'
export default {
  setup() {
    onBeforeRouteUpdate(() => {
      // 当当前路由发生变化时，调用回调函数
    })
  }
}
```

### 使用vuex

store/index.js
```
import { createStore } from 'vuex'

export default createStore({
  state: {},
  mutations: {},
  actions: {}
})
```

utils/auth.js
```
// import Cookies from "js-cookie";
// const TokenKey = "token";

export function getToken() {
  //   return Cookies.get(TokenKey);
  return localStorage.getItem('token') || 'test-token'
}

export function setToken(token) {
  //   return Cookies.set(TokenKey, token, { expires: 0.5 });
  return localStorage.setItem('token', token)
}

export function removeToken() {
  localStorage.removeItem('token')
  //   Cookies.remove(TokenKey);
}
```

store/modules/user.js
```
import { getToken } from '@/utils/auth'

const state = {
  token: getToken() || '',
  userInfo: {}
}

const mutations = {
  SET_TOKEN: (state, token) => {
    state.token = token
  }
}

const actions = {
  updateToken({ commit }, userToken) {
    commit('SET_TOKEN', userToken)
  }
}

export default {
  namespaced: true,
  state,
  mutations,
  actions
}
```

store/getters.js
```
const getters = {
  token: state => state.user.token
}

export default getters
```

store/index.js
```
import Vuex from 'vuex'
import getters from './getters'

const modulesFiles = require.context('./modules', true, /\.js$/)

const modules = modulesFiles.keys().reduce((modules, modulePath) => {
  // set './app.js' => 'app'
  const moduleName = modulePath.replace(/^\.\/(.*)\.\w+$/, '$1')
  const value = modulesFiles(modulePath)
  modules[moduleName] = value.default
  return modules
}, {})

export default Vuex.createStore({
  modules,
  getters
})
```

About.vue
```
import { useStore } from 'vuex'

setup() {
    const store = useStore()
    const token = store.getters['token']

    return {
        token
    }
}
```

### axios

```
utils/request.js

import axios from 'axios'
// import { Message, MessageBox } from 'element-ui'
import store from '@/store'
import { getToken } from '@/utils/auth'

const service = axios.create({
  baseURL: process.env.VUE_APP_BASE_API, // 开发环境代表代理的路径，生产环境用后端域名
  // withCredentials: true, // send cookies when cross-domain requests
  timeout: 6000 // request timeout
})

// request interceptor
service.interceptors.request.use(
  config => {
    // do something before request is sent

    if (store.getters.token) {
      // let each request carry token
      // ['X-Token'] is a custom headers key
      // please modify it according to the actual situation
      config.headers['token'] = getToken()
    }
    return config
  },
  error => {
    // do something with request error
    console.log(error) // for debug
    return Promise.reject(error)
  }
)

// response interceptor
service.interceptors.response.use(
  response => {
    const res = response.data

    if (res.code !== 200) {
      //   Message({
      //     message: res.message || 'Error',
      //     type: 'error',
      //     duration: 5 * 1000
      //   })

      //   if (res.code === 401) {
      //     // to re-login
      //     MessageBox.confirm(
      //       'You have been logged out, you can cancel to stay on this page, or log in again',
      //       'Confirm logout',
      //       {
      //         confirmButtonText: 'Re-Login',
      //         cancelButtonText: 'Cancel',
      //         type: 'warning'
      //       }
      //     ).then(() => {
      //       store.dispatch('user/resetToken').then(() => {
      //         location.reload()
      //       })
      //     })
      //   }
      return Promise.reject(new Error(res.message || 'Error'))
    } else {
      return res
    }
  },
  error => {
    console.log('err' + error) // for debug
    // Message({
    //   message: error.message,
    //   type: 'error',
    //   duration: 5 * 1000
    // })
    return Promise.reject(error)
  }
)

export default service
```

api/user.js
```
import request from '@/utils/myRequest'

// 获取测试数据列表
export function testDataList(params) {
  return request({
    url: '/api/testdata/list',
    method: 'get',
    params
  })
}
```
