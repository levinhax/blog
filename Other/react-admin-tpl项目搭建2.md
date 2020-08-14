## 使用Antd

```
npm install --save antd
```
修改 src/App.js，引入 antd 的按钮组件。
```
import React from 'react';
import { Button } from 'antd';
import './App.css';

const App = () => (
  <div className="App">
    <Button type="primary">Button</Button>
  </div>
);

export default App;
```

修改 src/App.css，在文件顶部引入 antd/dist/antd.css。
```
@import '~antd/dist/antd.css';
```

## 引入scss

```
npm install --save node-sass
```
不需要eject，不需要改webpack配置，安装后即可使用。

Adding a CSS Modules Stylesheet: 文件命名 - [name].module.css，如 Button.module.css，Button.module.scss

## 配置webpack

原package.json内容:
```
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "lint": "eslint --ext .js,.jsx,.ts,.tsx --ignore-path .eslintignore .",
    "lint:fix": "eslint --fix --ext .js,.jsx,.ts,.tsx --ignore-path .eslintignore .",
    "format": "prettier --write \"src/**/*\"",
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s",
    "changelog:all": "conventional-changelog -p angular -i CHANGELOG.md -s -r 0"
  },
```

使用create-react-app脚手架搭建的react工程，webpack和相关的依赖都已经配置好了，开发编译打包都很省心。但是随着项目的深入，难免会遇到要改webpack配置的情况。

### 执行npm run eject命令

使用 create-react-app 提供的 [yarn run eject](https://create-react-app.dev/docs/available-scripts/#npm-run-eject) 命令可以将所有内建的配置暴露出来

### 使用 react-scripts-rewired 和 customize-cra 来自定义 create-react-app 的 webpack 配置

```
npm install react-app-rewired customize-cra --save-dev
```

修改package.json的scripts为：
```
  "scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-scripts eject"
}
```

在根目录下新增config-overrides.js：
```
/* config-overrides.js */

module.exports = function override(config, env) {
  //do stuff with the webpack config...
  return config;
}
```

#### 配置别名@

```
// config-overrides.js

const { override, addWebpackAlias } = require('customize-cra')
const path = require('path')

module.exports = override(
  console.log('env : ', process.env.NODE_ENV),

  // 配置路径别名
  addWebpackAlias({
    '@': path.resolve(__dirname, 'src'),
  })
)
```

在根目录创建一个 tsconfig.extend.json 文件，添加以下配置（如果直接修改tsconfig.json中的path属性时，项目跑起来时会将path属性删除，不知道原因）
```
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

在tsconfig.json中最后一行追加
```
"extends": "./tsconfig.extend.json"
```

此时可以使用@别名：
```
index.tsx

import '@/styles/App.css'
import '@/styles/index.scss'
```
