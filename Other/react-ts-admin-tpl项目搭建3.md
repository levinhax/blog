### 路由配置

```
npm install --save react-router-dom
npm install --save react-router-config

npm install --save-dev @types/react-router-dom
npm install --save-dev @types/react-router-config
```

router/index.ts

```
import { lazy, LazyExoticComponent } from 'react'
// import { Redirect } from 'react-router-dom'
// import { RouteProps } from 'react-router-dom'

const Layout = lazy(() => import('../layout'))
const BlankLayout = lazy(() => import('../layout/BlankLayout'))
const NotFound = lazy(() => import('../views/Error/NotFound'))
const NoPermission = lazy(() => import('../views/Error/NoPermission'))

const Dashboard = lazy(() => import('../views/Dashboard'))
const About = lazy(() => import('../views/About'))

export interface IRouteMeta {
  title: string
  icon?: string
}

export interface IRouteBase {
  // 路由路径
  path?: string
  exact?: boolean
  // 路由组件
  component?: LazyExoticComponent<any>
  // 路由信息
  meta: IRouteMeta
  // 302 跳转
  redirect?: string
  // 是否校验权限, false 为不校验, 不存在该属性或者为true 为校验, 子路由会继承父路由的 auth 属性
  auth?: boolean
}

export interface IRoute extends IRouteBase {
  routes?: IRoute[]
}

/**
 * routes 第一级路由负责最外层的路由渲染，比如 userLayout 和 Layout 的区分
 * 所有系统内部存在的页面路由都要在此地申明引入，菜单栏的控制是支持异步请求获取的数据
 */

const routes: IRoute[] = [
  //   以下为错误页面路由
  {
    path: '/error',
    meta: {
      title: '错误页面',
    },
    component: BlankLayout,
    redirect: '/error/404',
    routes: [
      {
        path: '/error/404',
        component: NotFound,
        meta: {
          title: '页面不存在',
        },
        auth: false,
      },
      {
        path: '/error/403',
        component: NoPermission,
        meta: {
          title: '暂无权限',
        },
        auth: false,
      },
    ],
  },

  // 登录注册页面
  {
    path: '/login',
    component: Login,
    meta: {
      title: '用户登录',
      icon: ''
    }
  },

  // 管理系统
  {
    path: '/',
    // exact: true,
    component: Layout,
    meta: {
      title: '后台管理系统',
      icon: '',
    },
    // redirect: '/dashboard',
    routes: [
      {
        path: '/',
        exact: true,
        component: Dashboard,
        redirect: '/dashboard',
        meta: {
          title: '首页',
          icon: 'dashboard'
        },
        // render: () => <Redirect to={'/dashboard'} />,
      },
      {
        path: '/dashboard',
        component: Dashboard,
        meta: {
          title: '首页',
          icon: 'dashboard',
        },
      },
      {
        path: '/about',
        component: About,
        meta: {
          title: '关于',
          icon: '',
        },
      },

      // //   以下为错误页面路由
      // {
      //   path: '/error',
      //   meta: {
      //     title: '错误页面',
      //   },
      //   component: BlankLayout,
      //   redirect: '/error/404',
      //   routes: [
      //     {
      //       path: '/error/404',
      //       component: NotFound,
      //       meta: {
      //         title: '页面不存在',
      //       },
      //       auth: false,
      //     },
      //     {
      //       path: '/error/403',
      //       component: NoPermission,
      //       meta: {
      //         title: '暂无权限',
      //       },
      //       auth: false,
      //     },
      //   ],
      // },

      {
        path: '/*',
        meta: {
          title: '错误页面',
        },
        redirect: '/error/404',
      },
    ],
  },
]

export default routes
```

layout/BlankLayout.tsx
```
import React from 'react'
import { renderRoutes } from 'react-router-config'
import { RouteComponentProps } from 'react-router-dom'
import { IRoute } from '../router'

interface Props extends RouteComponentProps {
  route: IRoute
}

function BlankLayout(props: Props) {
  const { route } = props

  return <>{renderRoutes(route.routes)}</>
}

export default BlankLayout
```

layout/index.tsx
```
import React from 'react'
import { Layout } from 'antd'
import { renderRoutes } from 'react-router-config'
import { RouteComponentProps } from 'react-router-dom'
import { IRoute } from '../router'
import './index.scss'

const { Header, Sider, Content } = Layout

interface Props extends RouteComponentProps {
  route: IRoute
}

function LayoutIndex(props: Props) {
  const { route } = props

  console.log(props)
  console.log('route')
  console.log(route)

  return (
    <Layout className="layout-wrapper">
      <Sider>Sider</Sider>
      <Layout>
        <Header>Header</Header>
        <Content>{renderRoutes(route.routes)}</Content>
        {/* <Footer style={{ textAlign: 'center' }}>levinhax @2020</Footer> */}
      </Layout>
    </Layout>
  )
}

export default LayoutIndex
```

App.tsx
```
import React, { Suspense } from 'react'
import { HashRouter as Router, Switch, Route, Redirect } from 'react-router-dom'
// import { BrowserRouter as Router } from 'react-router-dom'
import { renderRoutes } from 'react-router-config'
import routes from './router'
import { Spin } from 'antd'
import './styles/App.scss'

function App() {
  return (
    <Suspense fallback={<Spin size="large" className="app-loading" />}>
      <Router>
        <Switch>
          {/* 精确匹配是不是在首页 */}
          <Route path="/" exact render={() => <Redirect to="/dashboard" />} />

          {/* 登录注册 */}
          {/* <Route exact path="/login" component={Login} /> */}

          {/* UI页面 */}
          {renderRoutes(routes)}
        </Switch>
      </Router>
    </Suspense>
  )
}

export default App
```
