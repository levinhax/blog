## 概述

本文将以 react 为切入点，记录打造一个基于React生态系统的后台管理系统模板的过程，以此加深对 react 技术栈以及项目实战的理解。

项目由 [create-react-app](https://create-react-app.dev/) 脚手架快速构建

```
npx create-react-app react-admin-tpl --template typescript
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
npm install --save-dev @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

- eslint: ESLint 的核心代码
- @typescript-eslint/parser：ESLint 的解析器，用于解析typescript ，从而检查和规范 Typescript代码
- @typescript-eslint/eslint-plugin：这是一个 ESLint 插件，包含了各类定义好的检测 Typescript 代码的规范

使用推荐配置的两种方法：

1. 修改 package.json 文件中的 eslintConfig 属性
2. 创建 .eslintrc.* 文件

在项目根目录下新建 .eslintrc.js 文件
```
module.exports = {
  parser: "@typescript-eslint/parser", //定义ESLint的解析器
  extends: ["plugin:@typescript-eslint/recommended"], //定义文件继承的子规范
  plugins: ["@typescript-eslint"], //定义了该eslint文件所依赖的插件
  parserOptions: {
    ecmaVersion: 2020, // Allows for the parsing of modern ECMAScript features
    sourceType: "module", // Allows for the use of imports
    ecmaFeatures: {
      jsx: true // Allows for the parsing of JSX
    }
  },
  rules: {
    "no-console": "off",
    "no-debugger": "off",
    "no-unused-vars": "off"
  },
  env: {
    //指定代码的运行环境
    browser: true,
    node: true
  }
};
```

使用 Eslint 规范 React 代码
```
npm install --save-dev eslint-plugin-react
```

```
module.exports = {
  parser: "@typescript-eslint/parser", //定义ESLint的解析器
  extends: ['plugin:react/recommended', "plugin:@typescript-eslint/recommended"], //定义文件继承的子规范
  plugins: ["@typescript-eslint"], //定义了该eslint文件所依赖的插件
  settings: {
    //自动发现React的版本，从而进行规范react代码
    react: {
      pragma: "React",
      version: "detect"
    }
  },
  parserOptions: {
    // 指定ESLint可以解析JSX语法
    ecmaVersion: 2020, // Allows for the parsing of modern ECMAScript features
    sourceType: "module", // Allows for the use of imports
    ecmaFeatures: {
      jsx: true // Allows for the parsing of JSX
    }
  },
  rules: {
    "no-console": "off",
    "no-debugger": "off",
    "no-unused-vars": "off"
  },
  env: {
    //指定代码的运行环境
    browser: true,
    node: true
  }
};
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
    'plugin:react/recommended',
    'plugin:@typescript-eslint/recommended',
    'prettier/@typescript-eslint',
    'plugin:prettier/recommended'
  ], //定义文件继承的子规范
  plugins: ['@typescript-eslint'], //定义了该eslint文件所依赖的插件
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
    'no-unused-vars': 'off'
  },
  env: {
    //指定代码的运行环境
    browser: true,
    node: true
  }
}
```

*可以在根目录下新建 .eslintignore 文件，忽略对指定文件的校验*

```
# 不用校验的文件
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
