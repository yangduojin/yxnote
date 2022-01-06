# Vue

- [Vue](#vue)
  - [钩子函数](#钩子函数)
  - [Router 要引入vue-router](#router-要引入vue-router)

## 钩子函数

- beforeCreate, ==created==
- beforeMount, ==mounted==
- beforeUpdate, updated
- beforeDestroy, destroyed

## Router 要引入vue-router

```js
<div id="app">
<h1>Hello App!</h1>
<p><router-link to="/">首页</router-link>
    <router-link to="/invest">我要投资</router-link>
    <router-link to="/user">用户中心</router-link></p>
<router-view></router-view>
</div>  
<script>
const welcome = { template: '<div>尚融宝主页</div>' }
const invest = { template: '<div>投资产品</div>' }
const user = { template: '<div>用户信息</div>' }
const routesVariable = [
    { path: '/', redirect: '/welcome' }, //设置默认指向的路径
    { path: '/welcome', component: welcome },
    { path: '/invest', component: invest },
    { path: '/user', component: user },
]
const router = new VueRouter({
    routes:routesVariable, // （缩写）相当于 routes: routes
})
new Vue({
    el: '#app',
    router,//(引入常量router,也可直接写入new VueRouter)
})
</script>
```
