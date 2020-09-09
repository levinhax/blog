## 概述

本文将以 react 为切入点，记录打造一个基于React生态系统的后台管理系统模板的过程，以此加深对 react 技术栈以及项目实战的理解。

项目由 [create-react-app](https://create-react-app.dev/) 脚手架快速构建

```
npx create-react-app react-ts-admin-tpl --template typescript
```

## 代码规范

### ESLint+Prettier规范

**ESLint关注代码质量, Prettier关注代码风格**

**版本升级后可能会有风格变化，故可锁定版本号**

ESLint 是一个Javascript Linter，帮助我们规范代码质量，提高团队开发效率。社区比较知名的代码规范：[standardjs](https://standardjs.com/readme-zhcn.html) 和 [airbnb](https://github.com/airbnb/javascript)

Prettier 是一个能够统一团队编码风格的工具，能够极大的提高团队执行效率，统一的编码风格能很好的保证代码的可读性。

使用 Create React App 创建的 React 应用，默认安装了 ESLint 依赖。

package.json 文件中的 eslintConfig 属性只提供了用于发现常见错误的最小规则集：
```
"eslintConfig": {
    "extends": "react-app"
},
```

```
npm install --save-dev @typescript-eslint/parser @typescript-eslint/eslint-plugin eslint-config-airbnb eslint-plugin-react eslint-plugin-promise eslint-plugin-unicorn eslint-plugin-react-hooks
```

- eslint: ESLint 的核心代码
- @typescript-eslint/parser：ESLint 的解析器，用于解析typescript ，从而检查和规范 Typescript代码
- @typescript-eslint/eslint-plugin：这是一个 ESLint 插件，包含了各类定义好的检测 Typescript 代码的规范，在 extends 中添加 'plugin:@typescript-eslint/recommended' 可开启针对 ts 语法推荐的规则定义
- eslint-config-airbnb: airbnb规则校验，如果要开启 React Hooks 的检查，需要在 extends 中添加一项 'airbnb/hooks'
- eslint-plugin-react: 规范React代码
- eslint-plugin-react-hooks：hooks校验
- eslint-plugin-promise ：让你把 Promise 语法写成最佳实践
- eslint-plugin-unicorn ：提供了更多有用的配置项，比如我会用来规范关于文件命名的方式

*先检查node_modules里是否有eslint*

使用推荐配置的两种方法：

1. 修改 package.json 文件中的 eslintConfig 属性
2. 创建 .eslintrc.* 文件

在项目根目录下新建 .eslintrc.js 文件
```
module.exports = {
  parser: '@typescript-eslint/parser', //定义ESLint的解析器
  extends: [
    'plugin:@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:promise/recommended',
    'plugin:unicorn/recommended'
  ],
  plugins: ['@typescript-eslint', 'react', 'react-hooks', 'promise', 'unicorn'],
  settings: {
    //自动发现React的版本，从而进行规范react代码
    react: {
      pragma: 'React',
      version: 'detect'
    }
  },
  parserOptions: {
    // 指定ESLint可以解析JSX语法
    ecmaVersion: 2020, // Allows for the parsing of modern ECMAScript features
    sourceType: 'module', // Allows for the use of imports
    ecmaFeatures: {
      jsx: true // Allows for the parsing of JSX
    }
  },
  rules: {
    'no-console': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    'no-unused-vars': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',

    'unicorn/filename-case': [
      'error',
      {
        cases: {
          // 中划线
          kebabCase: true,
          // 小驼峰
          camelCase: true,
          // 下划线
          snakeCase: false,
          // 大驼峰
          pascalCase: true
        }
      }
    ],
    'unicorn/prefer-query-selector': 'off'
  },
  env: {
    // 指定代码的运行环境
    browser: true,
    node: true
  },
  globals: {
    __REDUX_DEVTOOLS_EXTENSION__: true
  }
}
```

结合 Prettier 规范代码
```
npm install --save-dev prettier eslint-config-prettier eslint-plugin-prettier
```

- Prettier：Prettier插件的核心代码
- eslint-config-prettier：解决 ESLint 中的样式规范和 Prettier 中样式规范的冲突，以 Prettier的样式规范为准
- eslint-plugin-prettier：将 Prettier 作为 ESLint 规范来使用

根目录下创建 .prettierrc 文件
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

修改 .eslintrc.js 文件，引入 Prettier
```
module.exports = {
  parser: '@typescript-eslint/parser', //定义ESLint的解析器
  extends: [
    'plugin:@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:promise/recommended',
    'plugin:unicorn/recommended',
    'prettier/@typescript-eslint',
    'plugin:prettier/recommended'
  ],
  plugins: ['@typescript-eslint', 'react', 'react-hooks', 'promise', 'unicorn'],
  settings: {
    //自动发现React的版本，从而进行规范react代码
    react: {
      pragma: 'React',
      version: 'detect'
    }
  },
  parserOptions: {
    // 指定ESLint可以解析JSX语法
    ecmaVersion: 2020, // Allows for the parsing of modern ECMAScript features
    sourceType: 'module', // Allows for the use of imports
    ecmaFeatures: {
      jsx: true // Allows for the parsing of JSX
    }
  },
  rules: {
    'no-console': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'off',
    'no-unused-vars': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',

    'unicorn/filename-case': [
      'error',
      {
        cases: {
          // 中划线
          kebabCase: true,
          // 小驼峰
          camelCase: true,
          // 下划线
          snakeCase: false,
          // 大驼峰
          pascalCase: true
        }
      }
    ],
    'unicorn/prefer-query-selector': 'off'
  },
  env: {
    // 指定代码的运行环境
    browser: true,
    node: true
  },
  globals: {
    __REDUX_DEVTOOLS_EXTENSION__: true
  }
}
```

'prettier' 及之后的配置要放到原来添加的配置的后面，这样才能让 prettier 禁用之后与其冲突的规则。
```
{
  extends: [
    // other configs ...
    'prettier',
    'prettier/@typescript-eslint',
    'prettier/react',
    'prettier/unicorn',
  ]
}
```

*可以在根目录下新建 .eslintignore 文件，忽略对指定文件的校验*

```
# 不用校验的文件
src/react-app-env.d.ts
src/serviceWorker.ts
src/setupTests.ts
/public/*
*.js
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

修改package.json

```
"scripts": {
    "lint": "eslint --ext .js,.jsx,.ts,.tsx --ignore-path .eslintignore .",
    "lint:fix": "eslint --fix --ext .js,.jsx,.ts,.tsx --ignore-path .eslintignore .",
    "format": "prettier --write \"src/**/*.ts\" \"src/**/*.tsx\""
}
```

### stylelint

js(ts)代码风格统一后，样式也要统一一下。

```
npm install --save-dev stylelint stylelint-config-standard
```

在目录下新建 .stylelintrc.js

```
module.exports = {
  extends: ['stylelint-config-standard'],
  rules: {
    'comment-empty-line-before': null,
    'declaration-empty-line-before': null,
    'function-name-case': 'lower',
    'no-descending-specificity': null,
    'no-invalid-double-slash-comments': null,
    'rule-empty-line-before': 'always',
  },
  ignoreFiles: ['node_modules/**/*', 'public/**/*', 'dist/**/*'],
}
```

- stylelint-config-rational-order: 用于按照以下顺序将相关属性声明进行分组来对它们进行排序

1. Positioning
2. Box Model
3. Typography
4. Visual
5. Animation
6. Misc

```
.declaration-order {
  /* Positioning */
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 10;

  /* Box Model */
  display: block;
  float: right;
  width: 100px;
  height: 100px;
  margin: 10px;
  padding: 10px;

  /* Typography */
  color: #888;
  font: normal 16px Helvetica, sans-serif;
  line-height: 1.3;
  text-align: center;

  /* Visual */
  background-color: #eee;
  border: 1px solid #888;
  border-radius: 4px;
  opacity: 1;

  /* Animation */
  transition: all 1s;

  /* Misc */
  user-select: none;
}
```

- stylelint-declaration-block-no-ignored-properties: 用于提示我们写的矛盾样式，比如下面的代码中 width 是会被浏览器忽略的，这可以避免我们犯一些低级错误

```
{ display: inline; width: 100px; }
```

```
npm install --save-dev stylelint-order stylelint-config-rational-order stylelint-declaration-block-no-ignored-properties
```

修改.stylelintrc.js
```
module.exports = {
  extends: ['stylelint-config-standard', 'stylelint-config-rational-order'],
  plugins: ['stylelint-order', 'stylelint-declaration-block-no-ignored-properties'],
  rules: {
    'comment-empty-line-before': null,
    'declaration-empty-line-before': null,
    'function-name-case': 'lower',
    'no-descending-specificity': null,
    'no-invalid-double-slash-comments': null,
    'rule-empty-line-before': 'always',
    'order/properties-order': [],
    'plugin/rational-order': [
      true,
      {
        'border-in-box-model': false,
        'empty-line-between-groups': false,
      },
    ],
    'plugin/declaration-block-no-ignored-properties': true,
  },
  ignoreFiles: ['node_modules/**/*', 'public/**/*', 'dist/**/*'],
}
```

stylelint 的冲突解决也是一样的，先安装插件 stylelint-config-prettier :

```
npm install --save-dev stylelint-config-prettier
```

.stylelintrc.js中添加以下配置：
```
{  
    extends: [
    // other configs ...
    'stylelint-config-prettier'
  ]
}
```

package.json中新增名令：
```
"lint-stylelint": "stylelint --config .stylelintrc.js src/**/*.{css,scss,less}"
```

### 使用 husky lint-staged 来控制 git 提交之前的校验

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

### CHANGELOG

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

参考：

- https://zhuanlan.zhihu.com/p/180491533
