### Vite

Vite 是一个由原生 ESM 驱动的 Web 开发构建工具。在开发环境下基于浏览器原生 ES imports 开发，在生产环境下基于 Rollup 打包。

它主要具有以下特点：

- 快速的冷启动
- 即时的模块热更新
- 真正的按需编译

```
npm init vite-app <project-name>
cd <project-name>
npm install
npm run dev
```

### vite 原理解析

当浏览器识别 type="module" 引入js文件的时候，内部的 import 就会发起一个网络请求，尝试去获取这个文件。

然后可以通过通过拦截路由 / 和 .js 结尾的请求，再通过 node 去加载对应的 .js 文件
```
    const fs = require('fs')
    const path = require('path')
    const Koa = require('koa')
    const app = new Koa()

    app.use(async ctx=>{
        const {request:{url} } = ctx
        // 首页
        if(url === '/') {
            ctx.type="text/html"
            ctx.body = fs.readFileSync('./index.html','utf-8')
        } else if (url.endsWith('.js')) {
            // js文件
            const p = path.resolve(__dirname,url.slice(1))
            ctx.type = 'application/javascript'
            const content = fs.readFileSync(p,'utf-8')
            ctx.body = content
        }
    })

    app.listen(3001, ()=>{
        console.log('听我口令，3001端口，起~~')
    })
```

如果只是简单的代码，这样加载就可以了。完全是按需加载，比起 webpack 的语法解析性能当然会快非常多。

#### 第三方库

但是遇到第三方库以上代码就会找不到 .js 文件的位置了，此时 vite 会用 es-module-lexer 把文件解析成 ast，拿到 import 的地址。

通过分析 import 的内容，识别是不是第三方库（这个主要是看前面是不是相对路径）

如果是第三方库就去 node_modules 中查找，vite 中通过在第三方库中添加前缀 /@modules/，然后发现了 /@modules/ 后走 第三方库逻辑
```
    if(url.startsWith('/@modules/')){
        // 这是一个node_module里的东西
        const prefix = path.resolve(__dirname,'node_modules',url.replace('/@modules/',''))
        const module = require(prefix+'/package.json').module
        const p = path.resolve(prefix,module)
        const ret = fs.readFileSync(p,'utf-8')
        ctx.type = 'application/javascript'
        ctx.body = rewriteImport(ret)
    }
```

#### .vue 单文件解析

首先 xx.vue 返回的格式大概是这样的
```
const __script = {
    setup() {
        ...
    }
}
import {render as __render} from "/src/App.vue?type=template&t=1592389791757"
__script.render = __render
export default __script
```

然后可以用 @vue/compiler-dom 把 html 解析成 render

#### CSS

通过 document.createElement('style') 然后再注入就好了
