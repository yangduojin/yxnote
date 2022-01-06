# nodejs

- [nodejs](#nodejs)
  - [ES](#es)
    - [Promise](#promise)
    - [axios两种写法](#axios两种写法)
    - [拦截器](#拦截器)
    - [模块化](#模块化)

## ES

```js
let | const 
// 解构赋值
const F4 = ['小沈阳','刘能','赵四']
let [xiao, liu, zhao] = F4

// 模板字符串
`我喜欢${lovest}` 

// 声明对象简写
let person2 = {// 前面定义过值和方法
    age,
    username,
    sing,}

//定义方法简写
let person2 = {
sayHi() {
    console.log('Hi')
},}

//参数默认值 
function add(a, b, c = 0) { return a + b + c}

//对象扩展运算符 
let someone = { ...person }//对像拷贝

//箭头函数
let fn = (a) => { return a + 100}
let fn = a => a + 100
```

### Promise

```js
// 将错误和正常分开catch,then 执行
const fs = require('fs')
const p = new Promise((resolve, reject) => {
    fs.readFile('./xx.txt', (err, data) => {
    if (err) reject(err)
                        resolve(data)
    })})
p.then(response => {
    console.log(response.toString())
}).catch(error => {
    console.log('出错了')
    console.error(error)})
```

### axios两种写法

```js
axios({method: 'get',
     url: 'http://localhost:8080/user/list',})
     .then().catch()
axios.get('http://localhost:8080/user/list')
        .then().catch()
```

### 拦截器

```js
// 两个拦截器要放在ajax请求前
const request = axios.create({
        baseURL: 'http://localhost:8080', 
        timeout: 1000, 
        headers: {'token': 'helen123456'} 
request({method:'get',url:'/user/list'}).then().catch()
request.interceptors.request.use(
function (config) {
        config.headers.token = 'helen123456'
        return config
},
function (error) {
        return Promise.reject(error)})
request.interceptors.response.use(
        function (response) {
        return response.data
},
function (error) {
        return Promise.reject(error)})
```

### 模块化

```js
// es6之前 node.js中的 require 引入模块，exports 导出模块
// es6  export 导出模块  import 引入模块

//js暴露模块
export let star = '王力宏';或
let star = '王力宏';
function sing(){console.log('大城小爱')}
export {star, sing}

// js引入模块
import * as m1 from './m1.js'
import * as m2 from './m2.js'
import {star, sing} from './m1.js'
import {star as star2} from './m2.js' //使用as防止重名

//注意js引入别的js模块需要type
<script src="app.js" type="module"></script>
```
