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

### 日志中间件

```
npm install --save koa-logger
npm install --save-dev @types/koa-logger
```

引入时间格式化库MomentJS:
```
npm install --save moment
```

```
const loggerFormat = logger(str => {
  // 使用日志中间件
  console.log(Moment().format('YYYY-MM-DD HH:mm:ss') + str)
})

app.use(loggerFormat)
```

**Winston模块使用**

### dotenv，从文件加载环境变量

```
npm install --save dotenv
```

```
config/index.ts

import { IConfig } from '../types/config'
import dotenv from 'dotenv'

// Load environment variables from .env file
dotenv.config()
// dotenv.config({ path: '.env' });

// 关于 jwtExprisesIn: 例如：60，" 2天"，" 10h"，" 7d"。 数值解释为秒计数。
// 如果使用字符串，请确保提供时间单位（天，小时等），否则默认使用毫秒单位（" 120"等于" 120ms"）
const config: IConfig = {
    port: +process.env.NODE_PORT || 5000,
    isPrettyLog: process.env.NODE_ENV == 'development',
    routerBaseApi: '/api',
    jwtSecret: process.env.JWT_SECRET || 'pwd-jwt',
    jwtExprisesIn: 60 * 60, // jwtExprisesIn: '1h'
}

export { config }
```

### 安全中间件

```
npm install --save koa-helmet
npm install --save-dev @types/koa-helmet
```

helmet 通过增加如Strict-Transport-Security, X-Frame-Options, X-Frame-Options等HTTP头提高Express应用程序的安全性

### 定时任务

cron模块可以帮助我们在node中定时执行任务。如果你的定时需求是简单的setInterval()与setTimeout()计时器所无法满足的比较复杂的定时规则，推荐使用cron来配置。

```
npm install --save cron
npm install --save-dev @types/cron
```

```
new cronJob('* * * * * *', function () {
    //需要定时执行的任务代码写在这里
}, null, true);
```

第一个参数'* * * * * *'为cron表达式，例如：

```
Seconds: 0-59
Minutes: 0-59
Hours: 0-23
Day of Month: 1-31
Months: 0-11 (Jan-Dec)
Day of Week: 0-6 (Sun-Sat)
```

- "0 0 12 * * ? " ： 每天12点运行
- "0 15 10 * * ?" ：每天10:15运行
- "0 15 10 * * ? 2011" ： 2011年的每天10：15运行
- "0 * 14 * * ?" ：每天14点到15点之间每分钟运行一次，开始于14:00，结束于14:59
- "0 0/5 14 * * ?" ：每天14点到15点每5分钟运行一次，开始于14:00，结束于14:55
- "0 15 10 ? * MON-FRI" ：每周一，二，三，四，五的10:15分运行
- "0 15 10 15 * ?" ：每月15日10:15分运行
- "*/5 * * * * ?" ：每隔5秒执行一次
- "0 */1 * * * ?" ：每隔1分钟执行一次

### 连接数据库🔗

*请确保数据库已安装*

typeorm、mysql中间件

```
npm install typeorm --save
npm install mysql2 --save
```

为什么使用mysql2而不是经典的mysql:

1. 更高的性能
2. 支持PreparedStatement，多次查询性能更高，书写SQL更简单
3. 自带Promise包装器
4. 绝大部分API和mysql库兼容

##### 连接mysql

```
// ormconfig.json

[
  {
    "type": "mysql",
    "host": "localhost",
    "port": 3306,
    "username": "user1",
    "password": "user123%",
    "database": "galaxy",
    "synchronize": true,
    "logging": false,
    "entities": ["src/entity/*.ts"],
    "subscribers": ["src/subscriber/*.ts"],
    "migrations": ["src/migration/*.ts"],
    "timezone": "+08:00",
    "dateStrings": true,
    "cli": {
      "entitiesDir": "src/entity",
      "migrationsDir": "src/migration",
      "subscribersDir": "src/subscriber"
    }
  }
]
```
如果要创建多个连接，只需在数组中添加多个连接

```
// index.ts

// 连接数据库
import { createConnection } from 'typeorm'

// createConnection方法会自动读取来自ormconfig文件或环境变量中的连接选项
// const connection = await createConnection();
createConnection().then(() => {
  app.listen(5000, () => {
    console.log(`Server is running on port 5000`)
  })
})
.catch((error) => console.log('TypeOrm连接失败： ', error))
```

##### 实体Entity

实体是一个映射到数据库表（或使用 MongoDB 时的集合）的类。 你可以通过定义一个新类来创建一个实体，并用@Entity()来标记

```
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";
@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id!: number; // !: 强制解析，告诉编译器，这里一定有值
    @Column()
    userName!: string;
    @Column()
    userPass!: string;
    @Column()
    isActive!: boolean;
    @Column()
    userAvatar?: string
}
```

这将创建以下数据库表：
```
+-------------+--------------+----------------------------+
|                          user                           |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| userName    | varchar(255) |                            |
| userPass    | varchar(255) |                            |
| isActive    | boolean      |                            |
+-------------+--------------+----------------------------+
```
基本实体由列和关系组成。 每个实体必须有一个主列（如果使用 MongoDB，则为 ObjectId 列）。

每个实体都必须在连接选项中注册，或者你可以指定包含所有实体的整个目录， 该目录下所有实体都将被加载，如上配置中的entities。

##### 接口逻辑

###### Controller控制器

Controller 负责解析用户的输入，处理后返回相应的结果。

Controller 层主要对用户的请求参数进行处理（校验、转换），然后调用对应的 service 方法处理业务，得到业务结果后封装并返回：

- 获取用户通过 HTTP 传递过来的请求参数。
- 校验、组装参数。
- 调用 Service 进行业务处理，必要时处理转换 Service 的返回结果，让它适应用户的需求。
- 通过 HTTP 将结果响应给用户。

###### Service服务

Service 就是在复杂业务场景下用于做业务逻辑封装的一个抽象层

- 保持 Controller 中的逻辑更加简洁。
- 保持业务逻辑的独立性，抽象出来的 Service 可以被多个 Controller 重复调用。
- 将逻辑和展现分离，更容易编写测试用例

### 路由

```
npm install --save koa-router
npm install --save-dev @types/koa-router
```

```
import Router from 'koa-router'
import { UserController } from '../controller'

const router = new Router({
    prefix: '/api',
})

router.get('/', async (ctx) => {
    ctx.body = 'Hello World!'
})

router.get('/test', async (ctx) => {
    ctx.throw(401, '请求出错, 登录已过期')
})

router.get('/users', UserController.getUsers)
router.post('/register', UserController.createUser)
router.post('/login', UserController.login)
router.get('/users/:id', UserController.getUser)
router.put('/users/:id', UserController.updateUser)
router.delete('/users/:id', UserController.deleteUser)

export const routes = router.routes()
```

### JWT校验身份

```
npm install --save jsonwebtoken
npm install --save-dev @types/jsonwebtoken
```

登录时生成token，请求时对token进行校验(排除公共路由)

### koa-swagger-decorator

```
npm install --save koa-swagger-decorator
```

swagger接口文档

```
const swaggerRouter = new SwaggerRouter()

// user routes
swaggerRouter.get('/users', UserController.getUsers)
swaggerRouter.post('/register', UserController.createUser)

// Swagger endpoint
swaggerRouter.swagger({
  title: 'koa-api-tpl',
  description: 'koa api doc',
  version: '1.0.0',
  // if you are using koa-swagger-decorator within nested router, using this param to let swagger know your current router point
  // prefix: '/api',
  // [optional] default is /swagger-html
  swaggerHtmlEndpoint: '/swagger-html',
  // [optional] default is /swagger-json
  swaggerJsonEndpoint: '/swagger-json'
})

// 查找对应目录下的controller类
swaggerRouter.mapDir(path.resolve(__dirname, '../controller/'))

export { swaggerRouter }
```

controller接口做js-doc注释

访问：http://localhost:5000/swagger-html
