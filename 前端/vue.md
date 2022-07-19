# 概念

- MVVM：M + V + VM（双向绑定）
- 虚拟DOM：利用内存管理DOM
- axios：vue里的ajax
- ElementUI + vue 最佳组合 ==> vue-element-admin
- 指令：v-开头
- vue 生命周期 https://cn.vuejs.org/v2/guide/instance.html

https://www.bilibili.com/video/BV18E411a7mC?p=15



# 基础应用

## 杂七杂八

- $amount

```javascript
new Vue({}).$amount('#app') 相当于 new Vue({el: '#app'})
```

- $watch 侦听属性

```javascript
let vm = new Vue({el: '#app', data: {txt: 'haha'}});
// 当 txt 值发生变化会触发该事件
vm.$watch('txt', (n, o) => console.log(n + ':' + o));
```

- ==不要==在选项属性或回调上使用==箭头函数==，否则this会报undefind。如：

```javascript
// 页面初始化方法
created: () => console.log(this.a)
vm.$watch('a', newValue => this.myMethod())
```

## 指令

- v-model：双向绑定（value，selected，checked ===> data: {xxx: "value"}）
- v-bind/v-on 传参：方括号 [ ] 内为属性名/方法名表达式

```javascript
// :[attr] 相当于 v-bind:[attr]
<input type="text" :[attr]="txt" />
// 相当于 v-bind:value="txt"
new Vue({data: {txt: 'haha', attr: 'value'}});
```

- v-bind:value 缩写 :value
- v-on:click 缩写 @click
- 对于input来说 v-model=xxx 相当于 v-bind:value=xxx

## 组件

- 定义：把模板（html）和操作（js逻辑）组合成一个可复用的标签
- ==切记：组件名除首字母外不能有大写的，否则页面不显示==

```html
<div id="app">
    <!-- item要往组件里传的参数 -->
	<my-name v-for="s in list" v-bind:item="s"></my-name>
</div>
```

```javascript
Vue.component('my-name', {
    // props：接收参数关键定
    // item：可利用标签传进来的变量
    props: ['item'],
    template: '<h1>{{item}}</h1>'
});
new Vue({
    el: '#app', 
    data: {list: ['aaa', 'bbb', 'ccc']}
});
// 结果：
//<h1>aaa</h1>
//<h1>bbb</h1>
//<h1>ccc</h1>
```

## slot 槽

- 简单理解为组件内的其它标签：<组件><组件1/2/3/...n></组件1/2/3/...n></组件>

```html
<main>
    <t-title slot="title"></t-title>
    <t-body slot="body"></t-body>
</main>
<!--结果：-->
<div>
	<h1>title</h1>
    <p>body</p>
</div>
```

```javascript
Vue.component('main', {
    // 上面的 slot="title" 和 slot="body"
    // 这样确保两个 slot 是一一对应的
    template: '<div><slot name="title"></slot><slot name="body"></slot></div>'
});
Vue.component('t-title', {template: '<h1>title</h1>'});
Vue.component('t-body', {template: '<p>body</p>'});
```

## 自定义事件

- 用于组件之间通讯
- this.$emit()

> 一个删除组件里删除 app 里数组的例子：

```html
<!-- app.list 和 app.rm -->
<div id="app"><t-main :items="list" @remove="rm"></t-main></div>
```

```javascript
Vue.component('t-main', {
    props: ['items'],
    // 模板最外层的 div 不能加 v-if 或 v-for
    template: '<div>\
					<p v-for="(v, i) in items">\
						{{v}}<button @click="del(i)">delete</button>\
					</p>\
				</div>',
    // $emit('remove') => @remove="rm"
    methods: {del: function(i) {this.$emit('remove', i)}}
});
new Vue({
    el: '#app',
    data: {list: ['a', 'b', 'c', 'd']},
    methods: {
        rm: function(i) {
            this.list.splice(i, 1)
        }
    }
});
```



## 计算属性

- 计算出来的结果**缓存**在内存中

```javascript
// 普通方法：每次调返回的值都不一样
methods: {fun1: () => new Date().getTime()},
// 计算属性：每次调用时，如果方法里的值没有被改变，则返回的值是不变的（缓存）
computed: {fun2: () => new Date().getTime()}

// 普通方法，带()
{{fun1()}} <br>
// 计算属性，不带()
{{fun2}}
```

## 侦听属性

```html
<!-- v-bind不好用 只有 v-model -->
<input type="text" v-model="txt" />
```

```javascript
let vm = new Vue({
    el: '#app', 
    data: {txt: 'haha'},
    watch: {txt: (nv, ov) => console.log(nv + ':' + ov)}
});
```

## 路由

- 官方教程：https://router.vuejs.org/zh/
- api 文档：https://router.vuejs.org/zh/api/#router-view

### 起步

```html
<!-- 使用 router-link 组件来导航. -->
<!-- 通过传入 `to` 属性指定链接. -->
<!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
<router-link to="/foo">Go to Foo</router-link>
<router-link to="/bar">Go to Bar</router-link>

<!-- 路由出口 -->
<!-- 路由匹配到的组件将渲染在这里 -->
<router-view></router-view>
```

```javascript
// 导入路由
import VueRouter from "vue-router";
// 必须要use一下
Vue.use(VueRouter);
// 1. 定义组件。可以从其他文件 import 进来
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }
// 2. 定义路由
// 每个路由应该映射一个组件（属性：component）
const routes = [{ path: '/foo', component: Foo }, { path: '/bar', component: Bar }];
// 3. 创建 router 实例，然后传 `routes` 配置。还可以传别的配置参数
const router = new VueRouter({routes}); // routes(缩写) 相当于 routes: routes
// 4. 创建和挂载根实例。
// 记得要通过 router 配置参数注入路由，
// 从而让整个应用都有路由功能
const app = new Vue({el: '#app', router});
```

- this.$router 和 new VueRouter 是一样的 好处是不需要 import
- ==this.route 是当前路由信息，和 this.​router 差了个 'r'==

### 动态路由匹配

```javascript
// 动态路径参数 以冒号开头
{ path: '/user/:id', component: User }
// 比如访问路径为：/user/abc
// 则模板里就显示：user abc
template: '<div>User {{ $route.params.id }}</div>'
```

- 多断路径参数

| 模式                      | 匹配路径           | $route.params                 |
| ------------------------- | ------------------ | ----------------------------- |
| /user/:name               | /user/abc          | {name: 'abc'}                 |
| /user/:name/post/:post_id | /user/abc/post/123 | {name: 'abc', post_id: '123'} |

- 响应路由的参数变化

> 假如从 /user/aaa 跳转到 /user/bbb，由于是同一个组件，所以aaa的实例会被bbb复用，也就意味着组件生命周期的勾子函数不会被调用。此时==想对路由变化做出响应==，可以通过 watch 实现

```javascript
watch: { $route: (to, from) => { console.log(to) } }
```

### 嵌套路由（子路由/子组件）

> ==注意：==以 / 开头的路由为==根路由==，子路由（children）里不能以 / 开头

```javascript
// children的path不能以/开头
// path: '' 表示默认就显示子路由（组件）
children: [{path: 'ccc', component: () => import("../views/Haha.vue")}]
```

```html
<!--如果父路由和子路由是同一个组件（/user），那么 to="ccc"就可以-->
<!--如果父路由和子路由不是同一个组件，那么 to="/user/ccc"-->
<router-link to="ccc">ccc</router-link>
<router-view>ccc</router-view>
```

### 编程式导航（代码控制跳转）

```javascript
// <router-link to="/home"> 等同于 this.$router.push('/home')
// 字符串
this.$router.push('home')
// 对象
this.$router.push({ path: 'home' })
// 命名的路由
this.$router.push({ name: 'user', params: { userId: '123' }})
// 带查询参数，变成 /register?plan=private
this.$router.push({ path: 'register', query: { plan: 'private' }})
// 如果提供了 path 则会忽略 params ，这里的 params 不生效
this.$router.push({ path: '/user', params: { userId }}) // -> /user
```

### 命名路由

```javascript
// name 标识名称
routes: [{path: '/user/:userId', name: 'user'}];
// <router-link 跳转
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
// 代码跳转
this.$router.push({ name: 'user', params: { userId: 123 }})
```

### 滚动条位置

```javascript
router = new VueRouter({
  routes: [...],
  scrollBehavior: (to, from, savedPosition) => {
      // return 期望滚动到哪个的位置
      return { x: 0, y: 0 }
  }
})
```

### 路由懒加载

- 打包时 js 包会很大且影响页面加载。实行懒加载后，路由被访问时才加载组件；更加高效

```javascript
const Foo = () => import('./Foo.vue');
router = new VueRouter({routes: [{ path: '/foo', component: Foo }]})
```

## 插件

- 在用插件之前都需要use。如：

```javascript
Vue.use(Vuex);
Vue.use(VueRouter);
```



## 状态管理（Vuex）

- https://vuex.vuejs.org/zh/

### 概念

- 专为 vue 开发的**状态管理模式**。采用集中式管理所有组件的状态。

- 状态管理模式包含：
  - **state**：驱动应用的数据源
  - **view**：以声明方式将 view 映射到视图
  - **actions**：响应在 view 上的用户输入导致的状态变化

```javascript
new Vue({
  // state
  data: () => {return {count: 0}},
  // view
  template: `<div>{{ count }}</div>`,
  // actions
  methods: {increment: () => {this.count++}}
});
```

- 问题：
  - 传参的方法对于多层嵌套的组件将会非常繁琐，并且对于兄弟组件间的状态传递无能为力。
  - 我们经常会采用父子组件直接引用或者通过事件来变更和同步状态的多份拷贝。
  - 以上会导致代码无法维护
- 解决：
  - 把组件的共享状态抽取出来，以一个全局单例模式管理。
  - 任何组件不管在任何位置都能获取或触发状态
  - 代码变得更好维护
- 简单的应用不要用

### State

```javascript
// 把 store 对象提供给 “store” 选项，这样就把 store 的实例注入所有的子组件
new Vue({el: '#app', store: store, components: {A}});
// 通过根实例中注册的 store
// 根实例下的所有组件都会被注入一个 store
// 且能通过 this.$store 访问
const A = {computed: {count: () => this.$store.state.count}}
```

- mapState 辅助函数：为解决组件获取多个状态时产生的冗余计算属性

```javascript
// 导入, 注意 {} 不能丢
import { mapState } from 'vuex'
export default {
  computed: {
    fun1: () => 'fun1',
    fun2: () => 'fun2',
    // 和别的计算方法混入时 mapState 前必须加 ...
    ...mapState({
      // 箭头函数可使代码更简练
      count: state => state.count,
      // 传字符串参数 'count' 等同于 `state => state.count`
      countAlias: 'count',
      // 为了能够使用 `this` 获取局部状态，必须使用常规函数
      countPlusLocalState: function(state) {
        return state.count + this.localCount
      }
    })
  }
}
```

### Getter

- Vuex 允许在 store 中定义 getter。像计算属性一样，返回的值缓存起来，当依赖发生改变时才会重新计算

```javascript
export default new Vuex.Store({
    // 数据源
  	state: {ds: [{id:1},{id:2},{id:3},{id:4},{id:5}]},
    // s == state
  	getters: {list: s => s.ds.filter(x => x.id > 3)}
});
// 调用
// 要用 this 所以不能 lambda
// 因为是计算属性 所以是.list 不能.list()
methods: {gt: function() {return this.$store.getters.list}}
```

- 通过方法访问

```javascript
// id: 调用方传的参数
getters: {find: s => (id: number) => s.ds.find(x => x.id == id)}
// 调用
this.$store.getters.find(4)
// 结果：{id: 4}
```

- mapGetters 辅助函数 仅仅将 store 中的 getter 映射到局部计算属性中

### Mutation

> 修改 store 中的状态唯一方法是提交 mutation。mutation 类似事件，每个都有**事件类型（type）**和一个**回调函数（handler）**。回调函数是更改状态的地方，且接受 state 作为第一个参数。

```javascript
new Vuex.Store({
  state: { a: 0 },
  // 定义 mutation 事件 fun1
  mutations: {fun1: function(s, o) {s.a = o.a}}
});
// 调用：this.$store.commit('方法名', 参数)
methods: {click: function() {this.$store.commit('fun1', {a: 1})}}
```

### Action

- Action 类似于 mutation，不同在于
  - Action 提交的是 mutation，而不是直接变更状态
  - Action 可以包含任意异步操作



## Axios 异步通信

- 安装：npm install axios

- https://www.kancloud.cn/yunye/axios/234845

```javascript
import axios from 'axios';
axios.post('http://localhost:8888').then(r => console.log(r.data))
```



# 前后分离

## 安装

- https://cli.vuejs.org/zh/
- **安装cli**：cnpm install -g @vue/cli
- 查看版本：vue -V

## 创建项目

- 1- vue create 项目名
- 2- 选择 Manually select features（手动选择）
- 3- 除了两个测试的都先（按空格键选）
- 若选了测试环境就选 jest，测试代码比 mocha 简洁
- 4- Use class-style component syntax?：y
- 5- Use Babel alongside TypeScript (required for modern mode, auto-detected polyfills, transpiling JSX)：y
- 6- Use history mode for router：y
- 7- Less
  - Less基于javascript，而Sass基于ruby，Stylus也基于非javascript
- 8- ESLint + Prettire 代码风格
  - 为什么选择？少数服从多数
  - 文章说明 https://segmentfault.com/p/1210000011670866/read
- 9- Lint on save（保存就检测）
- 10- Indedicated config files（单独保存在各自的配置文件中）
- 11- Save this as a preset for future projects?：n
  - 将此作为将来项目的预置吗？y，下次可以直接用
- 12- 运行：npm run serve==（不是server，没有r）==

## 打包

- npm run build
- 参考：https://cli.vuejs.org/zh/config/

- 如果发布到服务上不在根目录下，则会报某些 js/css 404 问题。解决：

```javascript
// 1. 在vue项目根目录下添加文件 vue.config.js
// 2. 写上以下内容
module.exports = {publicPath: process.env.NODE_ENV == 'production' ? '/dist' : '/'};
// publicPath 是 vue cli 3.3以后版本用的 以前的版本用 baseUrl
// process 是一个全局变量 用来描述当前项目的环境信息 所以不需要导入就可以直接使用
// 当 npm run build 时 process.env.NODE_ENV 的值为 'production'

// publicPath == '/dist' 表示打完包后访问：http://xxx:xx/dist
```

# webpack

## 安装

```bash
# 打包工具
cnpm install webpack -g
# 客户端工具
cnpm install webpack-cli -g
```

## 实例

```javascript
// modules/hello.js
exports.sayHi = function() {
    document.write('<h1>hi</h1>')
}
exports.sayHi1 = function() {
    document.write('<h1>hi1</h1>')
}
exports.sayHi2 = function() {
    document.write('<h1>hi2</h1>')
}
exports.sayHi3 = function() {
    document.write('<h1>hi3</h1>')
}
```

```javascript
// modules/main.js
let o = require('./hello');
o.sayHi();
```

```javascript
// webpack.config.js
module.exports = {
    // 入口
    entry: './modules/main.js',
    // 打包后生成的文件
    output: {
        filename: './bundle.js'
    }
};
```

```bash
# 打包命令
webpack
# 打包后生成文件：./dist/bundle.js
```

# popper.js

一款功能强大的 js 定位引擎，比如 tooltip 弹窗口可以用到

# 引入 bootStrap

```bash
# i = install，-S = --save，-D = --save-dev
# -S / -D 区别：前者发布环境也会用，后者只在开发环境用
cnpm i bootstrap-vue -S
```

```typescript
// main.ts
import BootstrapVue from 'bootstrap-vue'
import 'bootstrap/dist/css/bootstrap.css'
import 'bootstrap-vue/dist/bootstrap-vue.css'

Vue.config.productionTip = false;
Vue.use(BootstrapVue)
```

# Element-UI

## 1. 引入

```bash
cnpm i element-ui -S
```

```typescript
// main.ts
import element from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

Vue.config.productionTip = false;
Vue.use(element)
```

官方文档：https://element.eleme.cn/#/zh-CN/

# 引入 ant-design

```ba
cnpm i ant-design-vue -S
```

```typescript
import 'ant-design-vue/dist/antd.min.css';
// 然后按需加载组件
```

## 按需引入

```bash
# -D = --save-dev
cnpm i babel-plugin-import -D
```

```javascript
module.exports = {
  presets: ["@vue/cli-plugin-babel/preset"],
  // 官方推荐按需引入方案
  plugins: [
    ["import", { "libraryName": "ant-design-vue", "libraryDirectory": "es", "style": "css" }]
  ]
};
```

```typescript
// 在 .vue 文件里
import Vue from "vue";
import {Button} from 'ant-design-vue';

Vue.use(Button)
```

# 填坑

## 1. console.log 报错

```javascript
// package.json
"eslintConfig": {
    "rules": {
        "no-console": "off"
    }
}
```

## 2. Invalid Host header

```js
// vue.config.js
// 当通过内网穿透报页面上显示 Invalid Host header
devServer: {
    port: 8000,
    // webpack-dev-server出于安全考虑，默认检查hostname
    disableHostCheck: true // 跳过检查
}
```

