---
show: step
version: 1.0
enable_checker: true
---

# 采用 Vue CLI@4 + Element UI 搭建前端工程

## 实验介绍

本实验及后续章节将介绍网盘项目的前端工程，本次实验主要是介绍采用 Vue CLI@4 脚手架搭建前端项目的流程和组件库 Element UI 的使用，我会尽可能的给大家讲清楚每个安装步骤。如果你对 Vue CLI@4 和 Element UI 已经有一定的了解，可以在学习的过程中适当跳过一些简单的步骤。

#### 知识点

- Vue CLI@4 环境要求和安装步骤
- 采用 Vue CLI@4 搭建前端工程的两种方式及依赖安装
- Element UI 的安装和引入

#### 开发计划

- 开发内容：本次实验带领大家从零开始，采用 Vue CLI@4 脚手架搭建前端工程，采用组件库 Element UI 快速构建页面，并介绍项目文件结构。
- 开发耗时：实验预计完成时间为 0.5 ~ 1 小时
- 开发难点：无

## 检测环境要求

打开 Vue CLI 官方文档（[安装 | Vue CLI](https://cli.vuejs.org/zh/guide/installation.html)），看到版本 4.x 对 Node.js 的要求和旧版本的 Vue CLI 升级到 4.x 的方法。

![10-1](https://doc.shiyanlou.com/courses/3472/1557563/8d27651920d694ef6e22cf591bc18994-0/wm)

执行以下命令，检测环境的 Node.js 版本：

```bash
node --version
```

可以看到符合要求

![10-2](https://doc.shiyanlou.com/courses/3472/1557563/35c410a841db01b984ec5363b00be507-0/wm)

执行以下命令，检测 Vue CLl 的安装情况：

```bash
vue --version
```

可以看到环境安装的是 Vue CLI@4

![10-3](https://doc.shiyanlou.com/courses/3472/1557563/5ca14596cdadccfc5762110cb0d8767b-0/wm)

## 创建并启动项目

创建项目有两种方式：终端方式和图形化界面。

下面将介绍创建项目的具体步骤，过程中会安装 Vue Router、Vuex、Axios、Stylus 等项目必须的依赖，之后启动项目。

### 终端方式

执行以下命令来创建项目，`vue create` 加上项目名称，此次实验项目命名为 `file-web`：

```bash
vue create file-web
```

可以看到如下结果：

![10-4](https://doc.shiyanlou.com/courses/3472/1557563/f930607f5589536479a5e0c915fd306c-0/wm)

`?Please pick a presset（Use arrow keys）` 这一行是提示，之后的每一个依赖安装都会以这样的提示开头，紧接着的是选项列表。用键盘上下键可切换选中项（高亮蓝色），回车键确定选中。

第一个问题：请选择预设，这里我们采用 `Manually select features` 来手动安装一些依赖。

选中之后会自动将你选中的项拼接在提示之后，并出现下一个依赖的安装提问语句和选项列表。

![10-5](https://doc.shiyanlou.com/courses/3472/1557563/d8b14183715e0bc0bb6cb6dda48beff1-0/wm)

第二个提示：为你的项目选择你需要的功能。仍然是键盘上下键控制选中项，这里采用空格键选中或取消选中当前项。这里我们选中下图中的几个功能：选择 Vue 版本，安装 Babel、Router、Vuex、CSS 预编译器和格式化检查，回车键。

![10-6](https://doc.shiyanlou.com/courses/3472/1557563/c0cb464b9fee428c4a4e3fb0072b2777-0/wm)

第三个提示：选择 Vue.js 版本，此实验采用 `2.x`。

![10-7](https://doc.shiyanlou.com/courses/3472/1557563/30be76ea1f0dfb7e5d75e19f87a40901-0/wm)

第四个提示：路由是否采用 history 模式（[HTML5 History 模式 | Vue Router](https://router.vuejs.org/zh/guide/essentials/history-mode.html#html5-history-%E6%A8%A1%E5%BC%8F)），选择是。输入 Y，回车键。

![10-8](https://doc.shiyanlou.com/courses/3472/1557563/4b9331bc4c61a15d8900ed2c0f5f71de-0/wm)

第五个提示：选择一个 CSS 预编译器，可以选择任意一个，此实验采用 `Stylus`。

![10-9](https://doc.shiyanlou.com/courses/3472/1557563/6223c40ac5e1891d9cb4fc555e481769-0/wm)

第六个提示：选择一个插件化的 JavaScript 代码检测和格式化工具 ，结合 Visual Studio Code 提供的扩展程序 ESLint、Prettier 等，你可以选择自己习惯用的代码检测工具和格式化工具，此实验采用 `ESLint width error prevention only`。

![10-10](https://doc.shiyanlou.com/courses/3472/1557563/a0bbc89d542ba8bd3ab4185d8a195c18-0/wm)

第七个提示：何时检测代码，此实验采用 `Lint on save` 保存文件时测试。

![10-11](https://doc.shiyanlou.com/courses/3472/1557563/dee7f4d7cbf7256ae455479f8a095336-0/wm)

第八个提示：如何存放配置，选择保存到专用的配置文件中。

![10-12](https://doc.shiyanlou.com/courses/3472/1557563/8fa8588f74dad694b19e6c16c5631bce-0/wm)

第九个提示：是否保存本次配置（y：记录本次配置，然后需要给配置命名，N：不记录本次配置），这里选择 N。

![10-13](https://doc.shiyanlou.com/courses/3472/1557563/abc2fd97de31630a28f0200d655e6048-0/wm)

至此配置完成，所有的配置如下图所示：

![10-14](https://doc.shiyanlou.com/courses/3472/1557563/048360176e9b779e82e53c875a42b3fe-0/wm)

自动开始安装刚才所选的依赖：

![10-15](https://doc.shiyanlou.com/courses/3472/1557563/da04c364d8396903fd158c5dfca0691e-0/wm)

等待进度条 100%，项目创建成功：

![10-16](https://doc.shiyanlou.com/courses/3472/1557563/c4950a2790652ce08147ed0506a9c605-0/wm)

可以看到启动项目的提示：

```bash
cd file-web
npm run serve
```

执行以上命令，启动成功之后可以看到 `App running at: - Local: http://localhost:8080/`

![10-17](https://doc.shiyanlou.com/courses/3472/1557563/2f8a2795502e8c67893f25680966c171-0/wm)

本地环境运行时，在浏览器中打开此地址即可；在本课程环境中，需要在 `file-web` 根目录下新建文件 `vue.config.js`，写入以下内容：

```javascript
module.exports = {
  publicPath: '/',
  devServer: {
    host: '0.0.0.0',
    open: true,
    disableHostCheck: true
  }
}
```

重新启动项目，点击实验环境右侧的 Web 服务，即可在本地浏览器打开页面，至此，我们的项目就创建并启动成功了。

<img src="https://doc.shiyanlou.com/courses/3472/1557563/4f76944a697a57cfb7befec70bf123f1-0/wm" alt="10-18" style="zoom:50%;" />

### 图形化界面方式

执行以下命令，会启动 8000 端口，并自动在浏览器中打开新的页面，你可以在页面中创建、启动、停止、管理多个项目。

```bash
vue ui
```

![10-19](https://doc.shiyanlou.com/courses/3472/1557563/aca7a6e4fc1e37ef47f636b44bbea226-0/wm)

由于环境的特殊性，无法打开图形化界面，大家可以在本地进行尝试，依赖安装步骤和终端方式类似。

### 项目目录结构介绍

如图所示，可以看到生成的项目默认包含一些目录，下面来介绍几个重点目录和文件：

<img src="https://doc.shiyanlou.com/courses/3472/1557563/cd1c0f29c98141de9b057463b44a1704-0/wm" alt="10-20" style="zoom:80%;" />

1. `node_modules`：存放项目的各种依赖。
2. `public`：存放静态资源，其中的 index.html 是项目的入口文件，浏览器访问项目的时候默认打开的是生成后的 index.html。
3. `src`：存放项目主体文件，具体介绍如下：
   - `assets`：存放各种静态文件，包括图片、CSS 文件、JavaScript 文件、各类数据文件等。
   - `components`：存放公共组件，比如此课程后续将会用到的顶部导航栏 Header.vue。
   - `router/index.js`：vue-router 安装时自动生成的路由相关文件，主要用来为每个路由设置路径、名称和对应页面的 .vue 文件等。
   - `store/index.js`：vuex 安装时自动生成的状态相关文件，后续章节会详细介绍，用来让多个页面或组件共享数据。
   - `views`：存放页面文件，比如默认生成的 Home.vue 首页、About.vue 关于页面。
   - `App.vue`：是主 vue 模块，主要是使用 router-link 引入其他模块，所有的页面都是在 App.vue 下切换的。
   - `main.js`：是入口文件，主要作用是初始化 vue 示例、引用某些组件库或挂载一些变量。
4. `.eslintrc`：配置代码校验 ESLint 规则。
5. `.gitignore`：配置 git 上传时想要忽略的文件。
6. `babel.config.js`：一个工具链，主要用于兼容低版本的浏览器。
7. `package.json`：配置项目名称、版本号，记录项目开发所安装的依赖的名称、版本号等。
8. `package-lock.json`：记录项目安装依赖时，各个依赖的具体来源和版本号。

### 格式化代码配置

为了保持代码整洁，我们需要对编辑器做一定的设置。在编辑器根目录下创建文件夹 .vscode，创建 settings.json 文件，写入以下内容：

```json
{
  // 控制是否在打开文件时，基于文件内容自动检测 #editor.tabSize# 和 #editor.insertSpaces#。
  "editor.detectIndentation": false,
  // 一个制表符等于的空格数
  "editor.tabSize": 2,
  // 每次保存的时候是否自动格式化
  "editor.formatOnSave": true,
  // 函数(名)和后面的括号之间是否加空格
  "javascript.format.insertSpaceBeforeFunctionParenthesis": false,
  "vetur.format.defaultFormatterOptions": {
    "prettier": {
      "semi": false, //  代码结尾不加分号
      "singleQuote": true, //  使用单引号
      "trailingComma": "none" //  不自动添加逗号
    }
  }
}
```

![10-21](https://doc.shiyanlou.com/courses/3472/1557563/a60b83afc76dbea4ac0f2d1eff7f62e4-0/wm)

配置项的各个说明都在代码注释中，大家可以根据自己的习惯更改配置项。

## Element UI 的安装和使用

Element UI 是一套基于 Vue 2.0 的桌面端组件库，其中的很多组件可以帮助我们快速搭建页面，例如后续实验将要用到的 NavMenu 导航菜单、Table 表格、Breadcrumb 面包屑等。来跟随官方文档（[组件 | Element](https://element.eleme.cn/#/zh-CN/component/installation)）学习 Element UI 的安装和使用。

### 安装

先通过 `Ctrl + C` 组合快捷键停止项目。或者新建终端，通过 `cd file-web` 命令进入项目根目录。

使用 `npm` 的方式安装 Element UI，目前最新版本是 v2.15.0，执行以下命令，会自动安装最新版本：

```bash
npm i element-ui -S
```

等待安装，成功之后如下图所示。

![10-22](https://doc.shiyanlou.com/courses/3472/1557563/01e5a8faf35cb9da40706c649e7c8d58-0/wm)

### 引入

此次实验采用完整引入 Element UI 的方式，在 `src/main.js` 中补充以下内容：

```javascript
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'

Vue.use(ElementUI)
```

<img src="https://doc.shiyanlou.com/courses/3472/1557563/5ccb193e3efe766bb7dcc0eaf1a88392-0/wm" alt="10-23" style="zoom: 80%;" />

需要注意的是，样式文件需要单独引入。

### 组件使用介绍

先来通过 `npm run serve` 重新启动项目。使用最常见的按钮组件 Button 来验证我们的组件库是否正确引入了（此次课程所有组件均来自 Element UI，可以参照官方示例文档使用），后续实验用到的组件，使用方法均和此组件相同。

我们来使用官方示例代码中的主要按钮：

![10-24](https://doc.shiyanlou.com/courses/3472/1557563/6c9fda222905fbcf01a57caabda66229-0/wm)

在 `src/views/Home.vue` 中的写入以下内容：

```html
<el-button type="primary">主要按钮</el-button>
```

![10-25](https://doc.shiyanlou.com/courses/3472/1557563/2e234becace146f120987f6559917584-0/wm)

仍然是点击右侧的 Web 服务器，打开页面，看到按钮已经渲染出来了，证明 Element UI 组件库引入成功。

<img src="https://doc.shiyanlou.com/courses/3472/1557563/b659a09cd89a94b913162e6aae1cb78c-0/wm" alt="10-26" style="zoom:80%;" />

### 组件属性介绍

Element UI 的每个组件介绍最下方，都有相关的文档说明，包括此组件的属性、事件等。

![10-27](https://doc.shiyanlou.com/courses/3472/1557563/ab0ac8d8d025299be82e80413dae95e9-0/wm)

例如 Button 按钮组件的属性介绍部分：参数名对应 HTML 元素中的属性名，参数值是可选值中的一个，对应 HTML 元素中每一个属性的属性值，部分参数有默认值。

例如刚才使用的 `<el-button type="primary">主要按钮</el-button>`，参数 type，参数值 primary。可以加上参数是否圆角按钮 round，看下效果：

```html
<el-button type="primary" round>主要按钮</el-button>
```

看到按钮变成了圆角：

<img src="https://doc.shiyanlou.com/courses/3472/1557563/33234936c78020e3ce5fbfad7fe60763-0/wm" alt="10-28" style="zoom: 67%;" />

### 组件事件介绍

我们再来看下 Input 输入框的事件介绍：

![10-29](https://doc.shiyanlou.com/courses/3472/1557563/bfd6227e3c3c6de6198b48d0f40f2378-0/wm)

可以看到 Element UI 提供了组件最常用的一些事件，包括事件名称、说明和回调参数的格式，比如输入框的失焦事件，继续向 `src/views/Home.vue` 文件中写入如下代码：

```html
<el-input v-model="testValue" placeholder="请输入内容" @blur="handleBlur" style="width: 200px;"></el-input>
```

```vue
<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'

export default {
  name: 'Home',
  components: {
    HelloWorld
  },
  data() {
    return {
      testValue: '' //  输入框的值
    }
  },
  methods: {
    // 输入框失焦事件
    handleBlur() {
      console.log(this.testValue) //  控制台打印出输入框的值
    }
  }
}
</script>
```

![10-30](https://doc.shiyanlou.com/courses/3472/1557563/509332865627f502e657bbb9d81ada71-0/wm)

在页面上的输入框中输入 `this is test value`，点击输入框之外的区域，触发输入框的失焦事件，可以看到控制台（F12 打开）打印出了输入框的值：

![10-31](https://doc.shiyanlou.com/courses/3472/1557563/2224767101d17d4968284bd5f2466ee6-0/wm)

其他组件的使用，大家有兴趣可以自行学习和研究。

## 实验总结

本实验向大家演示了 Vue CLI@4 搭建项目的流程，并介绍了 Element UI 的安装和使用方法，有兴趣的同学可以尝试使用 Element UI 中的一些组件搭建页面。

本次实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3472/code10.zip
```
