> 对于一些需要记录用户行为的系统时，在进行网络请求的时候都会要求传递一下登录的token。不过，一般为了接口数据的安全，服务器的token一般不会设置太长，token过期之后就需要登录。但是频繁的登录会对用户造成不好的体验，因此，需要在后台定时去刷新token，并替换之前的token。我设置token默认存在30分钟，在token过期5分钟之内刷新

目前要做到token的无感刷新，主要有3种方案：

#### 方案一：

后端返回过期时间，前端每次请求就判断token的过期时间，如果快到过期时间，就需要去调用刷新token的接口。

**缺点**：需要后端额外提供一个token过期时间的字段；使用了本地时间判断，若本地时间被篡改，特别是本地时间比服务器时间慢时，拦截会失败。

#### 方案二：

写个定时器，然后定时刷新token接口。

**缺点**：浪费资源，消耗性能,不建议采用。

#### 方案三：

在请求响应拦截器中拦截，判断token 返回过期后，调用刷新token接口。



> 我们这里是在token即将过期的时候进行token刷新，而不是已经过期了才去刷新，这里即将过期的时间设置的是5分钟。
>
> 一般一个页面同时会有很多个请求，所以我们需要建一个数组先缓存起来这些请求；除此之外，还要建一个全局标志，绑在window上，用来判断当前是否正在刷新token；待token刷新完成后，再去执行数组中缓存的所有请求，此时，数组中的请求都是带着刷新后的最新的token值作为headers去请求的。

##### request.js

```js
import axios from 'axios'
import { Loading,MessageBox,Message } from 'element-ui'
import {
  getToken,
  removeToken,
  getExpires,
  removeExpires,
  setToken,
  setExpires
} from '@/utils/auth'
import {
  refreshToken
} from '@/api/users'

import router from '../router'

// #region 处理ajax效果
// 标记页面加载对象是否存在
let loadingService = null
// 当前请求数量
let ajaxCount = 0
// 检查当前是否所有ajax都结束方法
let checkAllAjaxDone = () => {
  if (ajaxCount === 0) {
    // console.log('所有ajax结束。。。。', loadingService);
    if (loadingService) {
      loadingService.close()
      loadingService = null
    }
  }
}

// 是否正在刷新的标志
window.isRefreshing = false;
// 存储请求的数组
let cacheRequestArr = [];

// 将所有的请求都push到数组中,其实数组是[function(token){}, function(token){},...]
function cacheRequestArrHandle(cb) {
  cacheRequestArr.push(cb);
}
// 数组中的请求得到新的token之后自执行，用新的token去重新发起请求
function afreshRequest(token) {
  cacheRequestArr.map(cb => cb(token));
  cacheRequestArr = [];
}
// 判断token是否即将过期
function isTokenExpired() {
  let curTime = new Date().getTime(); //获取系统当前时间
  let expiresTime = Number(getExpires()) - curTime; //获取token的过期时间，并减去当前时间
  // 还差5分钟即将过期或者已经过期了，但过期时间在5分钟内
  if (expiresTime >= 0 && expiresTime < 300000) {
    return true
  }
  return false
}

// #endregion

// 创建请求对象
const httpRequest = axios.create({
  // 统一的ajax请求前缀
  baseURL: process.env.VUE_APP_BASE_API, // url = base url + request url
  // withCredentials: true, // send cookies when cross-domain requests
  timeout: 5000 // 请求超时时间
})

// 请求封装
httpRequest.interceptors.request.use(
  config => {
    // do something before request is sent
    // console.log('request config data:', config.data)
    if (getToken()) {
      // 设置请求头token
      config.headers['Authorization'] = getToken()
      // 判断token是否即将过期
      if (isTokenExpired() && config.url !== '/users/refreshToken') {
        console.log('token即将过期，刷新token');
        //标志正在刷新token
        window.isRefreshing = true;
        // 重新请求token
        refreshToken(getToken()).then(res => {
          console.log('refreshToken res =>' + res);
          // 先保存token到cookie
          setToken(`${res.tokenHead}${res.token}`, res.expiration)
          //设置过期时间
          setExpires(res.expiration)
          // 重新设置请求头token
          config.headers['Authorization'] = getToken()
          //获取新的token
          console.log('获取新的token：', getToken());
          // 执行数组里的函数,重新发起被挂起的请求
          afreshRequest(getToken());
          // 重置标志
          window.isRefreshing = false;

        }).catch(err => {
          console.log('refreshToken err =>' + err);
          // 重置标志
          window.isRefreshing = false;
          // 删除本地缓存
          removeToken()
          removeExpires()
          // 跳转到登录页面
          router.push('/login')
        }).finally(() => {
          window.isRefreshing = false;
        })

        let retry = new Promise((resolve) => {
          cacheRequestArrHandle((token) => {
            config.headers['Authorization'] = token; // token为刷新完成后传入的token
            // 将请求挂起
            resolve(config)
          })
        })
        return retry;
      }
    }
    return config
  },
  error => {
    // do something with request error
    console.log(`发送请求失败${error}`) // for debug
    return Promise.reject(error)
  }
)

export default httpRequest

```

##### auth.js

```js
import Cookies from 'js-cookie'
// import { createUuid } from './common'

// 随机存储cookie的token key
// 2022-02-11 改为不随机了，因为每次刷新页面会重新创建uuid，所以每次刷新页面会导致无法获取到本地已有缓存
const TokenKey = '1D596CD8-8A20-4CEC-98DD-CDC12282D65C' // createUuid()

export function getToken () {
  return Cookies.get(TokenKey)
}

export function setToken (token,expires) {
  return Cookies.set(TokenKey, token,{'expires':expires}) //并设置超时时间
}

export function removeToken () {
  return Cookies.remove(TokenKey)
}

// 将expiresAt过期时间写入cookie
const expiresAtKey = 'ExpiresAt'

export function getExpires() {
    return Cookies.get(expiresAtKey)
}

export function setExpires(expiresAt, expires) {
    return Cookies.set(expiresAtKey, expiresAt, { expires: expires })
}

export function removeExpires() {
    return Cookies.remove(expiresAtKey)
}
```

##### users.js

```js
//刷新用户token
export function refreshToken(data) {
  return request({
    url: '/users/refreshToken',
    method: 'post',
    data
  })
}
```

转载：[https://blog.csdn.net/DLGDark/article/details/108824935] [https://blog.csdn.net/yu1431/article/details/130835868]
