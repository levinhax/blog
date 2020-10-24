utils/request.js
```
import axios from 'axios'
import { message } from 'ant-design-vue'
import { getToken, removeToken } from '@/utils/auth'

const service = axios.create({
  baseURL: process.env.VUE_APP_BASE_API, // 开发环境代表代理的路径，生产环境用后端域名
  // withCredentials: true, // send cookies when cross-domain requests
  timeout: 20000 // request timeout
})

// request interceptor
service.interceptors.request.use(
  config => {
    if (getToken()) {
      config.headers['token'] = getToken()
    }
    return config
  },
  error => {
    console.log(error)
    return Promise.reject(error)
  }
)

// response interceptor
service.interceptors.response.use(
  response => {
    const res = response.data

    if (res.isError) {
      message.error(res.error.message)
      return Promise.reject(new Error(res.message || 'Error'))
    } else {
      return res
    }
  },
  error => {
    console.log('err: ' + error) // for debug
    if (error.response.status === 400 && error.response.data.error.code === 'AUTHORIZE_FAILED') {
      message.error(error.response.data.error.message)
      removeToken()
      location.reload()
    }

    if (error.response.status === 500) {
      message.error(error.response.data.error.message)
    }

    return Promise.reject(error)
  }
)

export default service
```

api/login.js
```
import request from '@/utils/request'
// import qs from 'qs'

// 账号密码登录
export function loginByPassword(data) {
  return request({
    url: '/login/login-on',
    method: 'post',
    data
  })
}
```

api/home.js
```
import request from '@/utils/request'

// 产业布局图
export function IndustryDistribution(params) {
  return request({
    url: '/index/industry/distribution',
    method: 'get',
    params
  })
}

// 企业码访问情况
export function getVisit(area) {
  return request({
    url: `/cockpit/company/visit?area=${area}`,
    method: 'get'
  })
}

// 问卷调查导出
export const questionnairesExport = (params) => {
    return axios
      .get(`${apiHost}/manage/question/summaryExport/${params.questionId}`, params, {
        responseType: 'blob',
      })
      .then((res) => {
        return httpHelp(res);
      })
      .catch(catchError);
};
```
