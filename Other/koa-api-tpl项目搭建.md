## 目录

```
koa-api-tpl
├─ .eslintrc.js            // eslint配置
├─ .prettierrc             // prettierrc配置
├─ .huskyrc                // 钩子Husky
├─ commitlint.config.js    // commitlint配置
├─ ormconfig.js            // typeorm数据库配置
├─ package.json            // 项目配置
├─ src
│    ├─ controller         // 业务逻辑
│    ├─ entity             // 数据库模型
│    ├─ index.ts           // 入口
│    ├─ middlewares        // 中间件
│    ├─ routes             // 路由
│    ├─ types              // 数据的类型
│    └─ utils              // 常用工具
└─ tsconfig.json
```

### 快速构建一个项目

```
mkdir koa-api-tpl

npm init   // 生成package.json文件

npm install koa --save

npm install --save-dev typescript @types/koa
```

生成tsc配置文件(tsconfig.json):

```
tsc --init
```
文件详情：[tsconfig.json](https://github.com/levinhax/koa-api-tpl/blob/master/tsconfig.json)

在node中采用CommonJS 模块规范，而我们使用TypeScript之后，将会使用es6的import进行模块的导入导出，而typescript文件最终编译回javascript文件的时候，需要将这部分代码转换成CommonJS规范，否则项目出错，所以module选项应为commonjs。

### 编写并启动文件

```
src/index.ts

const Koa = require('koa')
const app = new Koa()

import { Context } from 'koa'

app.use(async (ctx: Context) => {
    ctx.body = 'Hello World'
})

app.listen(5000, () => {
    console.log(`Server is running on port 5000`)
})
```

### 开发热更新

1. ts-node-dev

```
npm install --save-dev ts-node-dev cross-env
```

```
"scripts": {
  "dev": "cross-env NODE_ENV=development ts-node-dev --inspect -- src/index.ts",
}
```

2. ts-node + nodemon实现Typescript项目热更新

- ts-node: 直接运行ts项目
- nodemon: 监听文件改变自动重启Node Server服务的工具

```
"scripts": {
  "start": "nodemon -e ts,tsx --exec ts-node src/index.ts"
}
```

上述参数是设置在命令行中的，也可以在nodemon.json中设置
```
"scripts": {
  "start": "nodemon src/index.ts"
}
```

```
// nodemon.json

{
    "restartable": "rs",
    "verbose": true,
    "execMap": {
        "ts": "ts-node"
    },
    "ignore": ["**/*.test.ts", ".git", "node_modules"],
    "watch": ["src"],
    "env": {
        "TS_NODE_FILES": "true",
        "TS_NODE_PROJECT": "./tsconfig.json",
        "APP_ENV": "dev",
        "NODE_ENV": "qa",
        "DBNAME": "galaxy",
        "DBACCOUNT": "root",
        "DBPASSWORD": "pass123456"
    },
    "ext": "js,json,ts"
}
```

### 代码规范

#### eslint结合prettier规范代码并格式化

```
npm install --save-dev eslint prettier @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-config-airbnb eslint-config-prettier eslint-plugin-prettier
```

- eslint-config-airbnb: airbnb是我们主要的代码检测规则
- eslint-config-prettier: 解决eslint和prettier对于同一个错误多次报错的问题
- eslint-plugin-prettier: 让eslint调用prettier对你的代码风格进行检查
- 在根目录下新建 .eslintrc.js 文件
- 根目录添加 .prettierrc 文件，用作该项目的prettierrc格式化配置

```
.eslintrc.js

module.exports = {
  parser: '@typescript-eslint/parser', // Specifies the ESLint parser
  extends: [
    'plugin:@typescript-eslint/recommended', // Uses the recommended rules from @typescript-eslint/eslint-plugin
    'prettier/@typescript-eslint',
    'plugin:prettier/recommended'
  ],
  parserOptions: {
    ecmaVersion: 2020, // Allows for the parsing of modern ECMAScript features
    sourceType: 'module', // Allows for the use of imports
    ecmaFeatures: {
      jsx: true // Allows for the parsing of JSX
    }
  },
  rules: {
    // Place to specify ESLint rules. Can be used to overwrite rules specified from the extended configs
    // e.g. '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-var-requires': 'off',
    'no-console': 'off',
    'no-debugger': 'off',
    'no-unused-vars': 'off'
  },
  env: {
    browser: true,
    node: true
  }
}
```

```
.prettierrc

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

.prettierrc和.eslintrc.js配置中，如果module出现了警告，在在根目录新建.eslintignore，在该文件里面把 .eslintrc.js和.prettierrc文件添加为检测对象即可
```
// .eslintignore

!.prettierrc    // ! 是添加进检测的意思
```

#### 使用husky做pre-commit

- 添加一个命令到package.json中去，用作代码格式检测

```
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "lint": "eslint src --ext .ts,.tsx"
},
```

- 添加pre-commit

我们使用husky这个库来为我们添加pre-commit功能: npm install husky --save-dev

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
{
  "hooks": {
    "pre-commit": "lint-staged",
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
  }
}
```

- 使用commitlint规范git提交信息

```
npm install --save-dev lint-staged @commitlint/cli @commitlint/config-conventional
```

在根目录新建一个 commitlint.config.js 文件，用作自定义一些commit提交规则，当然这一步也可以不做，直接使用常规校验也行
```
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
      "pre-commit": "npm run lint",
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
},
```

pageage.json中添加:

```
"lint-staged": {
    "src/**/*.{ts,tsx}": [
      "prettier --write \"src/**/*.ts\"",
      "eslint --fix"
    ]
},
```

### 使用pm2跑后端项目

```
"scripts": {
    "dev": "cross-env NODE_ENV=development ts-node-dev --inspect -- src/index.ts",
    "build": "tsc",
    "test": "echo \"Error: no test specified\" && exit 1",
    "lint": "eslint src --ext .ts,.tsx"
},
```

pm2是node进程管理工具，可以利用它来简化很多node应用管理的繁琐任务，如性能监控、自动重启、负载均衡等，而且使用非常简单

安装并使用pm2: npm install -g pm2

启动后端项目: pm2 start dist/src/index.js --watch -i 2

访问: http://localhost:5000/

### 静态资源目录

使用koa-static中间件，只需简单几步就可以搭建一个静态服务器，让我们获取到项目中的图片等静态资源。
```
npm install --save koa-static
npm install --save-dev @types/koa-static
```

```
// 配置静态web服务的中间件
import koaStatic from 'koa-static';
const path = require('path');

// app.use(koaStatic(__dirname + '/static/'));
app.use(koaStatic(path.join(__dirname + '/static/')));
```

### 跨域

```
npm install --save koa2-cors
npm install --save-dev @types/koa2-cors
```

### 获取请求参数

```
npm install --save koa-bodyparser
npm install --save-dev @types/koa-bodyparser
```
打印 ctx.request对象发现可以获取到传的参数

### 连接数据库

*请确保数据库已安装*

typeorm、mysql中间件

``````
npm install typeorm --save
npm install mysql --save
```
