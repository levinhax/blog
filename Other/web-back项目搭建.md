# 概述

本文将以 react 为切入点，记录打造一个基于React生态系统的后台管理系统模板的过程，以此加深对 react 技术栈以及项目实战的理解。

最终项目目录如下：
```
web-back
├── config                     webpack配置文件
    ├──...
    ├──webpack.config.dev.js   开发环境配置
    ├──webpack.config.prod.js  生产环境配置
├── public                      模板文件目录
    ├──index.html               模板
├── src                        源码文件
    ├──...
    ├──index.js                 入口
├── README.md                  README
```

## webpack安装

- 执行 npm init , 创建package.json
- 安装webpack基本包

```
npm install --save-dev webpack webpack-cli
```
