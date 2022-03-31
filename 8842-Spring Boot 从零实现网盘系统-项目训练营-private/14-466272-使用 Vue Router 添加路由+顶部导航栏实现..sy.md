---
show: step
version: 1.0
enable_checker: true
---

# 使用 Vue Router 添加路由+顶部导航栏实现

## 实验介绍

本次实验将讲解页面及路由的添加，并实现网盘顶部导航栏。

#### 知识点

- Vue Router
- Element UI 的 NavMenu 导航菜单

#### 开发计划

- 开发内容：路由插件 Vue Router 的使用，登录、注册、网盘页面的添加，采用 Element UI 的 NavMenu 组件构建顶部导航栏，并实现点击导航栏跳转到相应页面的效果。
- 开发耗时：实验预计完成时间为 1 ~ 1.5 小时
- 开发难点：
  1. Vue Router 的使用
  2. NavMenu 组件的使用

## 使用 Vue Router 添加路由

Vue Router 是 Vue.js 官方的路由管理器（[Vue Router](https://router.vuejs.org/zh/)），和 Vue.js 的核心深度集成，可以帮我们快速创建、配置路由，例如路由和页面的对应、路由参数获取和修改、HTML5 历史模式或 hash 模式等。

### 路由相关文件解析

先来看下自动生成的路由文件 `src/router/index.js`，首先需要引入 `vue-router` 模块（在上个实验中创建项目时，已经安装了 Vue Router 依赖）、挂载在 Vue 上，接着需要创建路由列表，包含路径、路由名称、页面文件等配置，之后创建路由实例，并导出，然后在 `src/main.js` 中引入并使用即可。

![11-1](https://doc.shiyanlou.com/courses/3472/1557563/86571992803593a744e16c9f8d0f08bc-0/wm)

基于上一个实验，对此文件中的部分代码做了注释说明：

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router' //  引入vue-router模块
import Home from '../views/Home.vue' //  引入Home页面对应的文件

Vue.use(VueRouter) //  将VueRouter挂载在Vue上

// 创建路由列表
const routes = [
  {
    path: '/', //  路由路径，即浏览器地址栏中显示的URL
    name: 'Home', //  路由名称
    component: Home //  路由所使用的页面
  },
  {
    path: '/about',
    name: 'About',
    /**
     * 1. route level code-splitting
     * 2. this generates a separate chunk (about.[hash].js) for this route :
     *    这会为该路由生成一个单独的块(about.[hash].js)，打包时对应的css文件、js文件将会以 webpackChunkName 的值拼接hash值命名。
     * 3. which is lazy-loaded when the route is visited.
     *    当此路由被加载时，才加载对应的页面文件。这样做可以加快页面访问速度。
     */
    component: () => import(/* webpackChunkName: "about" */ '../views/About.vue')
  }
]

// 创建路由实例
const router = new VueRouter({
  mode: 'history', //  HTML5 History 模式
  base: process.env.BASE_URL,
  routes
})

export default router
```

这里特别要说明的是 HTML5 History 模式。如果不使用 history 模式，例如现有的关于页面，路由路径将会是 `http://oursite.com/#/about` 这种形式。如果不想要就很丑的 hash， 可以用路由的 **history 模式**，那么路由路径就会是 `http://oursite.com/about`。这种模式充分利用 `history.pushState` API 来完成 URL 跳转而无须重新加载页面。如果使用了这种模式，在部署时需要后台配置支持，因为我们的应用是个单页客户端应用，如果后台没有正确的配置，当用户在浏览器直接访问 `http://oursite.com/about` 就会返回 404。所以需要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 `index.html` 页面，这个页面就是你 app 依赖的页面。

例如此次课程部署用到的 nginx，应当添加如下配置：

```nginx
location / {
  try_files $uri $uri/ /index.html;
}
```

这里先暂做了解，后续实验将会在部署相关章节中，详细说明配置项。

之后，我们可以在 `*.vue` 文件中通过 `$router` 访问路由实例，调用 `this.$router` 来操作路由（[编程式的导航 | Vue Router](https://router.vuejs.org/zh/guide/essentials/navigation.html)），例如后续实验将会用到的 `this.$router.push`，用于向 history 栈添加新的记录；使用 `this.$route` 获取路由信息，例如接下来将会用到的获取当前页面路由路径 `this.$route.path`。

### 首页、登录、注册页面及路由添加

首先来做一点清洁工作：删除 `src/views/about.vue` 文件，删除 `src/router/index.js` 中的路由 About，在 `src/App.vue` 的\<style\>中，删除 `text-align center`、`color #2c3e50`、`margin-top 60px`。

在 `src/assets` 下新建 `style` 文件夹，然后在该文件夹下新建 `base.styl` 文件，键入以下内容，设置所有元素的外边距和内边距均为 `0px`：

```stylus
* {
  margin: 0;
  padding: 0;
}
```

然后在 `src/main.js` 中引入此样式文件：

```javascript
import './assets/style/base.styl'
```

我们将沿用 `src/router/index.js` 中的路由 Home 来作为首页，可以看到此路由的 component 中引入的页面文件为 `src/views/Home.vue`，先来清空下 `src/views/Home.vue` 中的代码，以便进行后续的代码编写，并删除 `src/components/HelloWorld.vue` 文件。

清空后，键入以下内容：

```vue
<template>
  <div class="home">这是网盘主页</div>
</template>

<script>
export default {
  name: 'Home',
  data() {
    return {}
  },
  methods: {}
}
</script>
```

在 `src/views` 中新建登录页面文件 `Login.vue`，键入以下内容：

```vue
<template>
  <div class="login">这是登录页面</div>
</template>

<script>
export default {
  name: 'Login',
  data() {
    return {}
  },
  methods: {}
}
</script>
```

在 `src/router/index.js` 创建登录页面路由：

```javascript
const routes = [
  ...
  {
    path: '/login', //  登录页面
    name: 'Login',
    component: () => import(/* webpackChunkName: "login" */ '../views/Login.vue')
  }
]
```

同理，在 `views` 目录下新建注册页面文件 `Register.vue` 并添加路由：

```vue
<template>
  <div class="login">这是注册页面</div>
</template>

<script>
export default {
  name: 'Login',
  data() {
    return {}
  },
  methods: {}
}
</script>
```

在 `router/index.js` 文件下添加路由：

```javascript
const routes = [
  ...
  {
    path: '/register', //  注册页面
    name: 'Register',
    component: () => import(/* webpackChunkName: "register" */ '../views/Register.vue')
  }
]
```

页面和路由添加好了，我们需要在 `src/App.vue` 中添加上述三个页面的跳转链接。

### 跳转链接添加

在 `src/App.vue` 中的 \<template\> 中添加首页、登录、注册页面的跳转链接：

```vue
<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">首页</router-link> | <router-link to="/login">登录页面</router-link> |
      <router-link to="/register">注册页面</router-link>
    </div>
    <router-view />
  </div>
</template>
```

现在启动项目：（之后将不再提示）

```bash
cd file-web
npm run serve
```

启动成功之后，仍然是点击右侧的 Web 服务（之后将不再提示）打开页面，检验下上述三个页面是否可以显示并成功跳转：

首页页面及路径：

<img src="https://doc.shiyanlou.com/courses/3472/1557563/0c9454b3417ec27cd3f5c74254cdc076-0/wm" alt="11-2" style="zoom:80%;" />

登录页面及路径：

<img src="https://doc.shiyanlou.com/courses/3472/1557563/371b16f3d40b2fbd12b3b4a13ef61e99-0/wm" alt="11-3" style="zoom:80%;" />

注册页面及路径：

<img src="https://doc.shiyanlou.com/courses/3472/1557563/8914914fa600c0c4c55a7e8ed55a35f2-0/wm" alt="11-4" style="zoom:80%;" />

可以看到三个页面均可以成功跳转。

### 404 页面添加

为了防止用户在地址栏输入错误的路径而导致页面加载出错，我们需要用一个 404 页面来拦截，页面添加方式与登录、注册页面相同，在 `src/views` 目录下新建 `Error_404.vue` 文件：

```vue
<template>
  <div class="error-404">此页面不存在……</div>
</template>

<script>
export default {
  name: 'Error_404'
}
</script>
```

重点是 404 页面路由的添加位置，需要添加在所有路由之后（[404 Not found 路由](https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html#%E6%8D%95%E8%8E%B7%E6%89%80%E6%9C%89%E8%B7%AF%E7%94%B1%E6%88%96-404-not-found-%E8%B7%AF%E7%94%B1)）：

```javascript
const routes = [
  ...
  {
    path: '/register', //  注册页面
    name: 'Register',
    component: () => import(/* webpackChunkName: "register" */ '../views/Register.vue')
  },
  {
    path: '*', //  404页面
    name: 'Error_404',
    component: () => import(/* webpackChunkName: "error_404" */ '../views/Error_404.vue')
  }
]
```

可以看到在浏览器地址栏里输入不存在的路径，就会自动打开 404 页面，有 CSS 基础的同学，可以给 404 页面添加一些样式，或者去 Iconfont（[阿里巴巴矢量图标库](https://www.iconfont.cn/home/index)）上寻找一些图标或插画，来使页面内容更加丰富。

<img src="https://doc.shiyanlou.com/courses/3472/1557563/a19fcda53e85b2645d5416c237bf8608-0/wm" alt="11-5" style="zoom: 67%;" />

## 使用 Element UI 中的 NavMenu 导航菜单

前面我们已经成功的添加了页面，接下来我们将使用 [NavMenu 导航菜单](https://element.eleme.cn/#/zh-CN/component/menu) 添加顶部导航栏，帮助我们快速构建页面，免去不必要的样式修改、事件添加等工作。

在 `src/components` 下创建文件 `Header.vue`，键入以下内容：

```vue
<template>
  <el-menu :default-active="activeIndex" :router="true" mode="horizontal">
    <el-menu-item index="Home" :route="{ name: 'Home' }">首页</el-menu-item>
    <el-menu-item index="Login" :route="{ name: 'Login' }">登录</el-menu-item>
    <el-menu-item index="Register" :route="{ name: 'Register' }">注册</el-menu-item>
  </el-menu>
</template>

<script>
export default {
  name: 'Header',
  data() {
    return {}
  },
  computed: {
    // 当前激活菜单的 index
    activeIndex() {
      return this.$route.name //  获取当前路由名称
    }
  }
}
</script>
```

其中的 el-menu 即为 Element UI 中的 NavMenu 导航菜单，`:router="true"` 表示使用 vue-router 的模式， index 是每个导航菜单的唯一标志，这里配置为各个页面对应的路由名称 name，default-active 为当前激活菜单的 index，为了刷新页面时也可以保证停留在当前页面，这里采用计算属性的方式给 activeIndex 赋值。\<el-menu-item\> 中的属性 route 为 Vue Router 路径对象，即要跳转到的页面的路由对象，这里依次配置为首页、登录、注册页面的路由对象。

在 `src/App.vue` 中引入、注册并使用此组件：

```vue
<template>
  <div id="app">
    <!-- 3. 使用组件 -->
    <Header></Header>
    <router-view />
  </div>
</template>

<script>
import Header from '@/components/Header.vue' //  1. 引入组件

export default {
  name: 'App',
  //  2. 注册组件
  components: {
    Header
  }
}
</script>
```

来看下页面效果：

<img src="https://doc.shiyanlou.com/courses/3472/1557563/69f3034b991cab3b0b6b6faf92b8819b-0/wm" alt="11-6" style="zoom:80%;" />

<img src="https://doc.shiyanlou.com/courses/3472/1557563/9d6ba7844f0e0c88ad2f411cc24bf732-0/wm" alt="11-7" style="zoom:80%;" />

<img src="https://doc.shiyanlou.com/courses/3472/1557563/b6500c870def93beccbc5401d67c0bda-0/wm" alt="11-8" style="zoom:80%;" />

至此我们的顶部导航栏就实现了，最终的文件目录结构如下：

<img src="https://doc.shiyanlou.com/courses/3472/1557563/ed801569722c303ede36097c8f0a00a2-0/wm" alt="11-9" style="zoom:67%;" />

有兴趣的同学可以自己参考官方文档给的组件属性，结合 stylus 调整下导航栏的样式，例如将登录、注册靠右放置、当前激活菜单底部高亮颜色修改等。

## 实验总结

此次实验带领大家了解了 Vue Router 的相关知识，实践了路由的使用，页面的引入，结合 Element UI 的 NavMenu 导航菜单，快速构建出了页面的主体效果，有兴趣的同学可以尝试给页面添加上底部栏，优化顶部导航栏样式、在顶部导航栏添加所需要展示的 logo 等。

本次实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3472/code11.zip
```
