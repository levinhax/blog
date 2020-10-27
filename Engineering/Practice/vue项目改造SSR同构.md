## 实践背景

项目优化改造

## SSR优势

- 减少首页白屏：服务端直出页面
- SEO：直出的页面包含了页面关键数据信息

现在的 SSR 渲染一般指的都是同构渲染，可以兼顾客户端渲染的大部分优势，所以这里是做改造。

## SSR缺陷

- 服务端压力：
- 开发体验：前端团队可能对服务层代码的把控力

## 方案

![SSR改造方案](./images/001.png)
FCP：首次内容绘制时间，TTI：可交互时间

Vue3 优化的一大原因是尽可能将部分 VDOM 的渲染改为字符串拼接，我们可以按照同样的思路，改造 Vue2。

### 常规优化

**渲染前**
缓存：数据、组件、页面
请求：http、keep-alive
降级：客户端渲染

第一，多级缓存。接口数据、组件和最终吐出的页面均可缓存。这一步的核心是继续把 CPU 压力转移到内存，前者可以缩短请求链路，后两个可以减少渲染计算量。缓存的方式非常灵活，简陋一点就直接用内存缓存，配合 **LRU 算法**基本够用。复杂的场景就需要上 Redis 等内存数据库。

第二，请求复用。我们通常使用封装好的 Request、Axios 等库完成请求，最值得留意的选项就是使用开启了 keep-alive 的 http-agent，它能让后续的请求复用之前建立的连接，减少重复的握手次数。

第三点，降级熔断。如果没有降级，虽然 Node.js 节点比较稳定，不至于因为压力而宕机，但却会出现请求堆积，导致 Node.js 请求后端接口超时，服务将呈现不可用状态。

**渲染后**

- CDN加速静态资源
- 压缩响应体

## 实现

store/index.js
```
export function createStore() {
  return new Vuex.Store({
    modules,
    getters
  });
}
```

router/index.js
```
export function createRouter() {
  return new VueRouter({
    mode: "history",
    routes
  });
}
```

src/main.js
```
import Vue from 'vue'
import App from './App.vue'
import { createRouter } from './router'
import { createStore } from './store'
import { sync } from 'vuex-router-sync'

// import './styles/global.scss';
// 引入Vant
import Vant from "vant";
import "vant/lib/index.css";
Vue.use(Vant);
// 按需引入
// import '@/plugins/vant';

export function createApp () {
  // 创建 router 和 store 实例
  const router = createRouter()
  const store = createStore()

  // 同步路由状态(route state)到 store
  sync(store, router)

  const app = new Vue({
    // 注入 router 到根 Vue 实例
    router,
    store,
    render: h => h(App)
  })

  // 返回 app 和 router
  return { app, router, store }
}
```

src/entry-client.js
```
import { createApp } from "./main";

// const { app, router } = createApp();

// router.onReady(() => {
//   app.$mount("#app");
// });

const { app, router, store } = createApp();

if (window.__INITIAL_STATE__) {
  store.replaceState(window.__INITIAL_STATE__);
}

router.onReady(() => {
  // 添加路由钩子函数，用于处理 asyncData.
  // 在初始路由 resolve 后执行，
  // 以便我们不会二次预取(double-fetch)已有的数据。
  // 使用 `router.beforeResolve()`，以便确保所有异步组件都 resolve。
  router.beforeResolve((to, from, next) => {
    const matched = router.getMatchedComponents(to);
    const prevMatched = router.getMatchedComponents(from);

    // 我们只关心非预渲染的组件
    // 所以我们对比它们，找出两个匹配列表的差异组件
    let diffed = false;
    const activated = matched.filter((c, i) => {
      return diffed || (diffed = prevMatched[i] !== c);
    });

    if (!activated.length) {
      return next();
    }

    // 这里如果有加载指示器 (loading indicator)，就触发

    Promise.all(
      activated.map(c => {
        if (c.asyncData) {
          return c.asyncData({ store, route: to });
        }
      })
    )
      .then(() => {
        // 停止加载指示器(loading indicator)

        next();
      })
      .catch(next);
  });

  app.$mount("#app");
});
```

src/entry-server.js
```
import { createApp } from "./main";

// export default context => {
//   // 因为有可能会是异步路由钩子函数或组件，所以我们将返回一个 Promise，
//   // 以便服务器能够等待所有的内容在渲染前，
//   // 就已经准备就绪。
//   return new Promise((resolve, reject) => {
//     const { app, router } = createApp();

//     // 设置服务器端 router 的位置
//     router.push(context.url);

//     // 等到 router 将可能的异步组件和钩子函数解析完
//     router.onReady(() => {
//       const matchedComponents = router.getMatchedComponents();
//       // 匹配不到的路由，执行 reject 函数，并返回 404
//       if (!matchedComponents.length) {
//         return reject({ code: 404 });
//       }

//       // Promise 应该 resolve 应用程序实例，以便它可以渲染
//       resolve(app);
//     }, reject);
//   });
// };

export default context => {
  return new Promise((resolve, reject) => {
    const { app, router, store } = createApp();

    router.push(context.url);

    router.onReady(() => {
      const matchedComponents = router.getMatchedComponents();
      if (!matchedComponents.length) {
        return reject({ code: 404 });
      }

      // 对所有匹配的路由组件调用 `asyncData()`
      Promise.all(
        matchedComponents.map(Component => {
          if (Component.asyncData) {
            return Component.asyncData({
              store,
              route: router.currentRoute
            });
          }
        })
      )
        .then(() => {
          // 在所有预取钩子(preFetch hook) resolve 后，
          // 我们的 store 现在已经填充入渲染应用程序所需的状态。
          // 当我们将状态附加到上下文，
          // 并且 `template` 选项用于 renderer 时，
          // 状态将自动序列化为 `window.__INITIAL_STATE__`，并注入 HTML。
          context.state = store.state;

          resolve(app);
        })
        .catch(reject);
    }, reject);
  });
};
```

server/index.template.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <!-- <title>Document</title> -->
    <!-- 使用双花括号(double-mustache)进行 HTML 转义插值(HTML-escaped interpolation) -->
    <title>{{ title }}</title>

    <!-- 使用三花括号(triple-mustache)进行 HTML 不转义插值(non-HTML-escaped interpolation) -->
    {{{ meta }}}
</head>
<body>
    <!--vue-ssr-outlet-->
</body>
</html>
```

server/ssr.js
```
const Koa = require("koa");
const koaStatic = require("koa-static");
const koaMount = require("koa-mount");
const path = require("path");

const resolve = file => path.resolve(__dirname, file);
const app = new Koa();

const isDev = process.env.NODE_ENV !== "production";
const router = isDev ? require("./dev.ssr") : require("./server");

app.use(router.routes()).use(router.allowedMethods());
// 开放目录
app.use(koaMount("/dist", koaStatic(resolve("../dist"))));
app.use(koaMount("/public", koaStatic(resolve("../public"))));

// const port = process.env.PORT || 5000;
const port = 5000;

app.listen(port, () => {
  console.log(`server started at localhost:${port}`);
});

module.exports = app;
```

server/dev.ssr.js
```
const webpack = require("webpack");
const axios = require("axios");
const MemoryFS = require("memory-fs");
const send = require("koa-send");
const Router = require("koa-router");
const fs = require("fs");
const path = require("path");

const resolve = file => path.resolve(__dirname, file);

const router = new Router();

// 1、webpack配置文件
const webpackConfig = require("@vue/cli-service/webpack.config");
const { createBundleRenderer } = require("vue-server-renderer");

// 2、编译webpack配置文件
const serverCompiler = webpack(webpackConfig);
const mfs = new MemoryFS();
// 指定输出到的内存流中
serverCompiler.outputFileSystem = mfs;

// 3、监听文件修改，实时编译获取最新的 vue-ssr-server-bundle.json
let bundle;
serverCompiler.watch({}, (err, stats) => {
  if (err) {
    throw err;
  }
  stats = stats.toJson();
  stats.errors.forEach(error => console.error(error));
  stats.warnings.forEach(warn => console.warn(warn));
  const bundlePath = path.join(
    webpackConfig.output.path,
    "vue-ssr-server-bundle.json"
  );
  bundle = JSON.parse(mfs.readFileSync(bundlePath, "utf-8"));
});

function renderToString(context, renderer) {
  return new Promise((resolve, reject) => {
    // 这里无需传入一个应用程序，因为在执行 bundle 时已经自动创建过。
    // 现在我们的服务器与应用程序已经解耦！
    renderer.renderToString(context, (err, html) => {
      err ? reject(err) : resolve(html);
    });
  });
}

const handleRequest = async ctx => {
  if (!bundle) {
    ctx.body = "等待webpack打包完成后在访问在访问";
    return;
  }
  const url = ctx.path;
  if (url.includes("favicon.ico")) {
    return await send(ctx, url, { root: resolve("../public") });
  }

  // 4、获取最新的 vue-ssr-client-manifest.json
  const clientManifestResp = await axios.get(
    "http://localhost:8080/vue-ssr-client-manifest.json"
  );
  const template = fs.readFileSync(resolve("./index.template.html"), "utf-8");
  const clientManifest = clientManifestResp.data;

  // createBundleRenderer 支持热重载
  const renderer = createBundleRenderer(bundle, {
    runInNewContext: false,
    template,
    clientManifest
  });

  const html = await renderToString(ctx, renderer);
  ctx.body = html;
};

router.get("*", handleRequest);

module.exports = router;
```

server/server.js
```
const fs = require("fs");
const path = require("path");
const Router = require("koa-router");
const send = require("koa-send");

const resolve = file => path.resolve(__dirname, file);

const router = new Router();

// 第 2 步：获得一个createBundleRenderer
const { createBundleRenderer } = require("vue-server-renderer");
const template = fs.readFileSync(resolve("./index.template.html"), "utf-8");
const serverBundle = require("../dist/vue-ssr-server-bundle.json");
const clientManifest = require("../dist/vue-ssr-client-manifest.json");

// createBundleRenderer 支持热重载
const renderer = createBundleRenderer(serverBundle, {
  runInNewContext: false,
  template,
  clientManifest
});

function renderToString(context) {
  return new Promise((resolve, reject) => {
    // 这里无需传入一个应用程序，因为在执行 bundle 时已经自动创建过。
    // 现在我们的服务器与应用程序已经解耦！
    renderer.renderToString(context, (err, html) => {
      err ? reject(err) : resolve(html);
    });
  });
}

// 第 3 步：添加一个中间件来处理所有请求
const handleRequest = async ctx => {
  const url = ctx.path;

  if (url.includes(".")) {
    return send(ctx, url, { root: path.resolve(__dirname, "../dist/") });
  }

  ctx.res.setHeader("Content-Type", "text/html");

  const context = {
    title: "vue ssr",
    url,
    meta: `
    <meta name="viewport" content="initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no, width=device-width" />
    `
  };
  // 将 context 数据渲染为 HTML
  const html = await renderToString(context);
  ctx.body = html;
};
router.get("*", handleRequest);
module.exports = router;
```

vue.config.js
```
const VueSSRServerPlugin = require("vue-server-renderer/server-plugin");
const VueSSRClientPlugin = require("vue-server-renderer/client-plugin");
const merge = require("webpack-merge");
const nodeExternals = require("webpack-node-externals");

const TARGET_NODE = process.env.WEBPACK_TARGET === "node";
const targetEntry = TARGET_NODE ? "server" : "client";
const isDev = process.env.NODE_ENV !== "production";

console.log("TARGET_NODE: ", TARGET_NODE);

module.exports = {
  publicPath: isDev ? "/" : "/",
  devServer: {
    port: 8080,
    open: true,
    historyApiFallback: true,
    headers: { "Access-Control-Allow-Origin": "*" }
  },
  // css extract 很重要，否则报document的错误，设置为false不会单独打包出css文件
  css: {
    extract: false
  },
  configureWebpack: () => ({
    // 将 entry 指向应用程序的 server / client 文件
    entry: `./src/entry-${targetEntry}.js`,
    // 对 bundle renderer 提供 source map 支持
    devtool: "source-map",
    target: TARGET_NODE ? "node" : "web",
    node: TARGET_NODE ? undefined : false,
    output: {
      libraryTarget: TARGET_NODE ? "commonjs2" : undefined
    },
    // https://webpack.js.org/configuration/externals/#function
    // https://github.com/liady/webpack-node-externals
    // 外置化应用程序依赖模块。可以使服务器构建速度更快，
    // 并生成较小的 bundle 文件。
    externals: TARGET_NODE
      ? nodeExternals({
          // 不要外置化 webpack 需要处理的依赖模块。
          // 你可以在这里添加更多的文件类型。例如，未处理 *.vue 原始文件，
          // 你还应该将修改 `global`（例如 polyfill）的依赖模块列入白名单
          whitelist: [/\.css$/, /\.scss$/]
        })
      : undefined,
    optimization: {
      splitChunks: TARGET_NODE ? false : undefined
    },
    plugins: [TARGET_NODE ? new VueSSRServerPlugin() : new VueSSRClientPlugin()]
  }),
  chainWebpack: config => {
    config.module
      .rule("vue")
      .use("vue-loader")
      .tap(options => {
        return merge(options, {
          optimizeSSR: false
        });
      });

    // fix ssr hot update bug
    if (TARGET_NODE) {
      config.plugins.delete("hmr");
    }
  }
};
```

项目地址：[vue-ssr-template](https://gitee.com/levinhax/vue-ssr-template)
