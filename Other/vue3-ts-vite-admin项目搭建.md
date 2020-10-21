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

shim.d.ts: (这个其实不太需要，因为项目中全是通过tsx开发的，如果用到.vue文件需要声明)
```
// declare module '*.vue' {
//   import Vue from 'vue'
//   export default Vue
// }

declare module '*.vue' {
  import { Component } from 'vue'
  const component: Component
  export default component
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

## 添加 Sass 样式表

```
npm install --save-dev sass
```

```
import { createApp } from 'vue'
import router from '/@/router'
import store from '/@/store'
import App from './App.vue'
// import '/@/index.css'
import './style/index.scss'

const app = createApp(App)
app.use(router).use(store).mount('#app')
```

.variable.scss
```
$layoutBg: #2273cf;

:export {
  layoutBg: $layoutBg;
}
```

style/index.scss
```
@import './variable.scss';

body {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: $layoutBg;
}
```

修改shim.d.ts
```
declare module '*.vue' {
  import Vue from 'vue'
  export default Vue
}

// declare module '*.vue' {
//   import { Component } from 'vue'
//   const component: Component
//   export default component
// }

//解决scss文件报错问题
declare module '*.scss' {
  const sass: any
  export default sass
}
```

新建postcss.config.js，后续会用得到
```
module.exports = {
  autoprefixer: {},
}
```

normalize.css
```
npm install --save normalize.css
```

```
import 'normalize.css/normalize.css'
```

## Ant Design of Vue

```
npm install --save ant-design-vue@next
```

main.js
```
import Antd from 'ant-design-vue'
import 'ant-design-vue/dist/antd.css'

app.use(Antd)
```

views/About/index.vue
```
<template>
  <div>
    <a-button type="primary">About</a-button>
  </div>
</template>

<script>
export default {
  name: 'About',
}
</script>
```

views/About/index.tsx
```
import { defineComponent } from 'vue'
import Logo from '../../assets/logo.png'
// import HelloWorld from '../../components/HelloWorld.vue'
import { Input, Divider } from 'ant-design-vue'

export default defineComponent({
  name: 'About',
  setup() {
    return () => (
      <div class="about">
        <h3>This is about page.</h3>
        <img src={Logo}/>

        {/* 这里使用tsx写法，就不建议引入.vue文件了 */}
        {/* <HelloWorld /> */}

        <Input style="width: 200px;" />
        <Divider />
      </div>
    )
  },
})
```

## 代码提交校验

package.json修改
```
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "eslint --ext .js,.jsx,.ts,.tsx,.vue --ignore-path .eslintignore .",
    "lint:fix": "eslint --fix --ext .js,.jsx,.ts,.tsx,.vue --ignore-path .eslintignore .",
    "format": "prettier --write \"src/**/*.ts\" \"src/**/*.tsx\" \"src/**/*.vue\""
  },
```

eslint补充包：
```
npm install @typescript-eslint/eslint-plugin@latest @typescript-eslint/parser --save-dev
```

我们使用husky这个库来为我们添加pre-commit功能: npm install husky lint-staged --save-dev

```
// package.json

"husky": {
    "hooks": {
      "pre-commit": "npm run lint"
    }
},
```
这里直接使用了.huskyrc配置文件

```
.huskyrc

{
  "hooks": {
    "pre-commit": "lint-staged",
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
  }
}
```

使用commitlint规范git提交信息
```
npm install --save-dev @commitlint/cli @commitlint/config-conventional
```

在根目录新建一个 commitlint.config.js 文件，用作自定义一些commit提交规则，当然这一步也可以不做，直接使用常规校验也行
```
// commitlint.config.js

module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'build',
        'ci',
        'chore',
        'docs',
        'feat',
        'fix',
        'perf',
        'refactor',
        'revert',
        'style',
        'test',
      ],
    ],
    'type-case': [0],
    'type-empty': [0],
    'scope-empty': [0],
    'scope-case': [0],
    'subject-full-stop': [0, 'never'],
    'subject-case': [0, 'never'],
    'header-max-length': [0, 'always', 72],
  },
}
```

在之前的husky配置中，把commitlint的校验命令添加进去:
```
"husky": {
    "hooks": {
      "pre-commit": "npm run lint-staged",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
},
```

在package.json中新增：
```
  "lint-staged": {
    "src/**/*.{ts,tsx}": [
      "npm run format",
      "npm run lint:fix"
    ]
  },
```
此时git提交就会先执行校验。

## CHANGELOG

```
npm install --save-dev conventional-changelog-cli
```

在package.json中新增:
```
"changelog": "conventional-changelog -p angular -i CHANGELOG.md -s"
```

生成日志: npm run changelog

conventional-changelog-cli不会覆盖任何以前的变更日志。 新增的日志基于自上一个commit的

如果这是您第一次使用此工具，并且想要生成所有以前的变更日志，则可以执行：
```
conventional-changelog -p angular -i CHANGELOG.md -s -r 0
```

## 环境变量

```
.env

# loaded in all cases
VITE_HOST = '0.0.0.0'
VITE_PORT = 8080
VITE_BASE_URL = './'
VITE_OUTPUT_DIR = 'dist'

.env.staging

# 测试环境
VITE_APP_ENV = 'staging'

# Whether to open mock
VITE_USE_MOCK = false

# base api
VITE_APP_BASE_API = '/api/'
```

修改package.json
```
  "scripts": {
    "dev": "NODE_ENV=development vite",
    "build": "NODE_ENV=production vite build",
    "build:stage": "NODE_ENV=staging vite build",
    "lint": "eslint --ext .js,.jsx,.ts,.tsx,.vue --ignore-path .eslintignore .",
    "lint:fix": "eslint --fix --ext .js,.jsx,.ts,.tsx,.vue --ignore-path .eslintignore .",
    "format": "prettier --write \"src/**/*.ts\" \"src/**/*.tsx\" \"src/**/*.vue\"",
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s"
  },
```

vite.config.ts
```
import { SharedConfig } from 'vite'
import path from 'path'
import fs from 'fs'
import dotenv from 'dotenv'

const pathResolve = (pathStr: string) => {
  return path.resolve(__dirname, pathStr)
}

// console.log('config log: ')
// console.log(process.env.NODE_ENV)

const envFiles = [
  /** default file */ `.env`,
  /** mode file */ `.env.${ process.env.NODE_ENV }`
]

for (const file of envFiles) {
  const envConfig = dotenv.parse(fs.readFileSync(file))
  for (const k in envConfig) {
    process.env[k] = envConfig[k]
  }
}

const config: SharedConfig = {
  alias: {
    '/@/': pathResolve('./src'),
  },
}

module.exports = config
```

## Mock数据

这里借助 vite-plugin-mock
```
npm install --save-dev vite-plugin-mock
```
