# vue3-ts-vite-admin项目搭建

基于vite打包的vue3 + ts + tsx后台管理项目 -- 项目搭建过程

## 开始

### 项目初始化

Vite 是一个 web 开发构建工具，由于其原生 ES 模块导入方法，它允许快速提供代码。
通过在终端中运行以下命令，可以使用 Vite 快速构建 Vue 项目。

```
$ npm init vite-app <project-name>
$ cd <project-name>
$ npm install
$ npm run dev
```

或

```
$ yarn create vite-app <project-name>
$ cd <project-name>
$ yarn
$ yarn dev
```

### 集成TypeScript

```
npm install --save-dev typescript
```

项目根目录创建配置文件：tsconfig.json：
```
{
  "include": ["./**/*.ts"],
  "compilerOptions": {
    "jsx": "react",
    "target": "es2020" /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017','ES2018' or 'ESNEXT'. */,
    "module": "commonjs" /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'. */,
    // "lib": ["es2017.object"] /* Specify library files to be included in the compilation. */,
    // "declaration": true /* Generates corresponding '.d.ts' file. */,
    // "declarationMap": true,                /* Generates a sourcemap for each corresponding '.d.ts' file. */
    "sourceMap": true /* Generates corresponding '.map' file. */,
    // "outFile": "./",                       /* Concatenate and emit output to single file. */
    "outDir": "./dist" /* Redirect output structure to the directory. */,

    "strict": true /* Enable all strict type-checking options. */,
    "noUnusedLocals": true /* Report errors on unused locals. */,
    "noImplicitReturns": true /* Report error when not all code paths in function return a value. */,

    "moduleResolution": "node" /* Specify module resolution strategy: 'node' (Node.js) or 'classic' (TypeScript pre-1.6). */,
    "esModuleInterop": true /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */
  }
}
```

### 集成eslint

```
npm install --save-dev eslint eslint-plugin-vue
```

项目根目录创建配置文件.eslintrc.js：
```
module.exports = {
  parser: 'vue-eslint-parser',
  parserOptions: {
    parser: '@typescript-eslint/parser', // Specifies the ESLint parser
    ecmaVersion: 2020, // Allows for the parsing of modern ECMAScript features
    sourceType: 'module', // Allows for the use of imports
    ecmaFeatures: {
      // tsx: true, // Allows for the parsing of JSX
      jsx: true,
    },
  },
  // settings: {
  //   tsx: {
  //     version: "detect" // Tells eslint-plugin-react to automatically detect the version of React to use
  //   }
  // },
  extends: [
    'plugin:vue/vue3-recommended',
    'plugin:@typescript-eslint/recommended', // Uses the recommended rules from the @typescript-eslint/eslint-plugin
    'prettier/@typescript-eslint', // Uses eslint-config-prettier to disable ESLint rules from @typescript-eslint/eslint-plugin that would conflict with prettier
    'plugin:prettier/recommended', // Enables eslint-plugin-prettier and eslint-config-prettier. This will display prettier errors as ESLint errors. Make sure this is always the last configuration in the extends array.
  ],
  rules: {
    // Place to specify ESLint rules. Can be used to overwrite rules specified from the extended configs
    // e.g. "@typescript-eslint/explicit-function-return-type": "off",
  },
};
```

.eslintignore
```
/public/*
```

### 集成pritter

```
npm install --save-dev prettier eslint-config-prettier eslint-plugin-prettier
```

项目根目录创建配置文件：.prettierrc：
```
{
  "printWidth": 100,
  "tabWidth": 2,
  "singleQuote": true,
  "semi": false,
  "proseWrap": "always",
  "arrowParens": "avoid",
  "bracketSpacing": true,
  "endOfLine": "lf"
}
```

.prettierignore
```
**/*.svg
*.lock
*.png
*.toml
package.json
LICENSE
/dist
commitlint.config.js
docker
Dockerfile*
.dockerignore
.DS_Store
.editorconfig
.eslintignore
.gitignore
.history
.huskyrc
.prettierignore
yarn-error.log
```

### 修改入口文件

因为默认项目模板是以src/main.js为入口的，我们需要把它修改为src/main.ts。
在根目录的index.html中修改入口文件的引用即可：
```
... ...
<body>
  ... ...
  <script type="module" src="/src/main.ts"></script>
</body>
</html>
```

*这里也可以不做修改，继续使用 main.js*

### 优化TS类型推断

在src目录下，创建shim.d.ts、source.d.ts

shim.d.ts: (这个其实不太需要，因为项目中全是通过tsx开发的)(对这句话我有疑问)
```
declare module '*.vue' {
  import Vue from 'vue';
  export default Vue;
}
```

source.d.ts: (优化编译器提示，声明静态资源文件)
```
declare const React: string;
declare module '*.json';
declare module '*.png';
declare module '*.jpg';
```

## vue-router

```
npm install --save vue-router@next
```

views/About/index.tsx
```
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'About',
  setup() {
    return () => (
      <>
        <h3>This is about page.</h3>
      </>
    )
  },
})
```

views/Home/index.tsx
```
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'Home',
  setup() {
    return () => (
      <>
        <h3>This is home page.</h3>
      </>
    )
  },
})
```

router/index.ts
```
import { RouteRecordRaw, createRouter, createWebHistory } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    name: 'Home',
    component: () => import('../views/Home'),
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('../views/About'),
  },
]

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes,
})

export default router
```

App.vue
```
<template>
  <router-view v-slot="{ Component }">
    <keep-alive>
      <component :is="Component" />
    </keep-alive>
  </router-view>
</template>

<script>
export default {
  name: 'App'
}
</script>
```
或App.tsx
```
import { defineComponent } from 'vue';
import {RouterLink, RouterView} from 'vue-router';
import './style/main.scss'

export default defineComponent({
  name: 'App',
  setup() {
    return () => (
      <>
        <div id="nav">
          <RouterLink to="/">Home</RouterLink> |
          <RouterLink to="/about">About</RouterLink>
        </div>
        <RouterView/>
      </>
    );
  }
});
```

main.js
```
import { createApp } from 'vue'
import router from './router'
import App from './App.vue'
import './index.css'

createApp(App).use(router).mount('#app')
```

## vuex

```
npm install --save vuex@next
```

在src目录下，新建store文件夹，并在文件夹内创建index.ts
```
import { createStore } from 'vuex'
import { state } from './state'

export default createStore({
  state,
  mutations: {},
  actions: {},
  modules: {},
})
```

```
export interface State {
  title: string
}

export const state: State = {
  title: 'Vue(v3) 与 tsx 的结合~',
}
```

main.js
```
import { createApp } from 'vue'
import router from './router'
import store from './store'
import App from './App.vue'
import './index.css'

createApp(App).use(router).use(store).mount('#app')
```

## 配置

vite.config.ts
```
import { SharedConfig } from 'vite'
import path from 'path'

const pathResolve = (pathStr: string) => {
  return path.resolve(__dirname, pathStr)
}

const config: SharedConfig = {
  alias: {
    '/@/': pathResolve('./src'),
  },
}

module.exports = config
```

main.js
```
import { createApp } from 'vue'
import router from '/@/router'
import store from '/@/store'
import App from './App.vue'
import '/@/index.css'

createApp(App).use(router).use(store).mount('#app')
```
