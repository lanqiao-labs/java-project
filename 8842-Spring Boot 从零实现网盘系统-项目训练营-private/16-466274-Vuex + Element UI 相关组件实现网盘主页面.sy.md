---
show: step
version: 1.0
enable_checker: true
---

# Vuex + Element UI 相关组件实现网盘首页

## 实验介绍

本实验将使用 Element UI 的 Table 表格、NavMenu 导航菜单、BreadCrumb 面包屑导航、Checkbox 多选框等组件实现网盘的主页面，并使用 Vuex 来保存表格列筛选的结果，实现页面组件间的数据联动。

#### 知识点

- Vuex
- Element UI 的 NavMenu 导航菜单、BreadCrumb 面包屑导航栏、Table 表格、Pagination 分页和 Checkbox 多选框组件的使用

#### 开发计划

- 开发内容：网盘主页面的实现，主要包括以下几点
  1. 使用 NavMenu 导航组件实现左侧文件分类栏，并添加收缩、展开的效果
  2. 使用 BreadCrumb 面包屑导航组件展示当前查看的文件路径
  3. 使用 Table 表格组件实现文件列表
  4. 使用 Pagination 分页组件实现文件列表的分页
  5. 使用 Checkbox 组件实现文件表格某些列的显隐控制
  6. 使用 Vuex 来保存表格列筛选的结果
  7. 添加获取文件列表的接口
  8. 左侧文件分类栏和右侧表格、右侧面包屑导航栏的数据联动
- 开发耗时：实验预计完成时间为 2~2.5 小时
- 开发难点：
  1. Vuex 的使用
  2. 组件间的数据联动

## 实现网盘首页

### 使用 NavMenu 实现左侧菜单

根据前面的实验，我们知道了项目启动的完整命令，如下所示：

```bash
# 启动后端
sudo service mysql start
cd /home/project/qiwen-file
mvn spring-boot:run

# 新开一个终端，启动前端
cd /home/project/file-web
npm run serve
```

在之前的实验中，实现顶部导航栏时已经用到过 NavMenu 导航菜单，这次我们主要来实现菜单的收缩展开功能。先来对项目的文件目录做点调整：

1. 在 `src/views` 下新建文件夹 Home，将 `src/views/Home.vue` 重命名为 `src/views/index.vue`，并移动到刚才创建好的文件夹 `src/views/Home` 中。
2. 修改 `src/router/index.js` 中对原有的 `Home.vue` 的引入，保证路由和页面可以正常匹配：将 `import Home from '../views/Home.vue'` 改为 `import Home from '../views/Home/index.vue'`。

刷新首页，可以正常显示。

在 `src/views/Home` 下创建新的文件夹 `components`，存放独属于首页的组件，后续的面包屑导航栏、表格列筛选组件都会放置于此文件夹下。

在 `src/views/Home/components` 下创建新文件 `SideMenu.vue`，在 `src/views/Home/index.vue` 中添加以下内容来引入、注册并使用此组件：

```vue
<template>
  <div class="home">
    <!-- 3. 使用组件 -->
    <SideMenu></SideMenu>
  </div>
</template>

<script>
import SideMenu from './components/SideMenu.vue' //  1.引入左侧菜单组件

export default {
  name: 'Home',
  components: {
    SideMenu //  2. 注册组件
  },
  data() {
    return {}
  },
  methods: {}
}
</script>
```

参考 Element UI 的 NavMenu（[NavMenu 垂直菜单](https://element.eleme.cn/#/zh-CN/component/menu#ce-lan)），先来实现一个区分文件类型的垂直菜单，在 `SideMenu.vue` 文件中添加以下内容：

```vue
<template>
  <el-menu
    class="side-menu"
    :default-active="activeIndex"
    :router="true"
    :collapse="isCollapse"
    background-color="#545c64"
    text-color="#fff"
    active-text-color="#ffd04b"
  >
    <el-menu-item index="0" :route="{ name: 'Home', query: { fileType: 0 } }">
      <!-- 图标均来自 Element UI 官方图标库 https://element.eleme.cn/#/zh-CN/component/icon -->
      <i class="el-icon-menu"></i>
      <span slot="title">全部</span>
    </el-menu-item>
    <el-menu-item index="1" :route="{ name: 'Home', query: { fileType: 1 } }">
      <i class="el-icon-picture"></i>
      <span slot="title">图片</span>
    </el-menu-item>
    <el-menu-item index="2" :route="{ name: 'Home', query: { fileType: 2 } }">
      <i class="el-icon-document"></i>
      <span slot="title">文档</span>
    </el-menu-item>
    <el-menu-item index="3" :route="{ name: 'Home', query: { fileType: 3 } }">
      <i class="el-icon-video-camera-solid"></i>
      <span slot="title">视频</span>
    </el-menu-item>
    <el-menu-item index="4" :route="{ name: 'Home', query: { fileType: 4 } }">
      <i class="el-icon-headset"></i>
      <span slot="title">音乐</span>
    </el-menu-item>
    <el-menu-item index="5" :route="{ name: 'Home', query: { fileType: 5 } }">
      <i class="el-icon-takeaway-box"></i>
      <span slot="title">其他</span>
    </el-menu-item>
  </el-menu>
</template>

<script>
export default {
  name: 'SideMenu',
  data() {
    return {}
  },
  computed: {
    // 当前激活菜单的 index
    activeIndex() {
      return String(this.$route.query.fileType) //  获取当前路由参数中包含的文件类型
    }
  }
}
</script>
```

接着我们来使用 NavMenu 的 collapse 属性（[NavMenu 折叠](https://element.eleme.cn/#/zh-CN/component/menu#zhe-die)）把左侧菜单的收缩展开功能添加上，并调整样式：

```vue
<template>
  <!-- collapse 属性：控制菜单收缩展开 -->
  <el-menu
    class="side-menu"
    :default-active="activeIndex"
    :router="true"
    :collapse="isCollapse"
    background-color="#545c64"
    text-color="#fff"
    active-text-color="#ffd04b"
  >
    <div class="fold-wrapper">
      <!-- click事件 当点击时切换菜单的收缩状态 -->
      <i
        class="el-icon-s-unfold"
        v-if="isCollapse"
        title="展开"
        @click="isCollapse = false"
      ></i>
      <i
        class="el-icon-s-fold"
        v-else
        title="收缩"
        @click="isCollapse = true"
      ></i>
    </div>
    <el-menu-item index="0" :route="{ name: 'Home', query: { fileType: 0 } }">
      <i class="el-icon-menu"></i>
      <span slot="title">全部</span>
    </el-menu-item>
    <!-- 剩余代码不再赘述 -->
  </el-menu>
</template>

<script>
export default {
  name: 'SideMenu',
  data() {
    return {
      isCollapse: false //  控制菜单收缩展开
    }
  },
  // 已有代码不再赘述
  watch: {
    // 监听收缩状态变化，存储在sessionStorage中，保证页面刷新时仍然保存设置的状态
    isCollapse(newValue) {
      sessionStorage.setItem('isCollapse', newValue)
    }
  },
  created() {
    this.isCollapse = sessionStorage.getItem('isCollapse') === 'true' //  读取保存的状态
    if (!this.$route.query.fileType) {
      this.$router.replace({ query: { fileType: 0 } })
    }
  }
}
</script>

<style lang="stylus" scoped>
@import '~@/assets/style/mixins.styl';

.side-menu {
  // 高度设置为屏幕高度减去顶部导航栏的高度
  height: calc(100vh - 61px);
  overflow: auto;
  // 调整滚动条样式
  setScrollbar(6px, #909399, #EBEEF5);

  // 折叠图标调整样式
  .fold-wrapper {
    height: 56px;
    line-height: 56px;
    padding: 0 20px;
    text-align: right;
    color: #fff;
    font-size: 24px;

    .el-icon-s-unfold, .el-icon-s-fold {
      cursor: pointer;

      &:hover {
        opacity: 0.5;
      }
    }
  }
}

// 对展开状态下的菜单设置宽度
.side-menu:not(.el-menu--collapse) {
  width: 200px;
}
</style>
```

`<style>` 中引入的 styl 文件需要在 `src/assets/style` 中新建 `mixins.styl`，内容如下：

```stylus
setScrollbar(scrollbarWidth, trackColor = #EBEEF5, thumbColor = #909399)
	//  修改滚动条下面的宽度
	&::-webkit-scrollbar
		width scrollbarWidth
	//  修改滚动条的下面的样式
	&::-webkit-scrollbar-track
		background-color trackColor
		-webkit-border-radius 2em
		-moz-border-radius 2em
		border-radius 2em
	//  修改滑块
	&::-webkit-scrollbar-thumb
		background-color thumbColor
		-webkit-border-radius 2em
		-moz-border-radius 2em
		border-radius 2em
```

现在刷新首页，看到左侧菜单已经可以手动控制收缩展开了，并且滚动条样式也做了修改：

![13-1](https://doc.shiyanlou.com/courses/3472/1557563/c9be036dcb3901717f969486594e414e-0)

### 使用 BreadCrumb 实现面包屑导航

参考官方示例（[BreadCrumb 面包屑导航](https://element.eleme.cn/#/zh-CN/component/breadcrumb#ji-chu-yong-fa)）来实现面包屑导航栏，以显示当前查看的文件的路径。在 `views/Home/components` 目录下新建文件 `BreadCrumb.vue`，添加以下内容：

```vue
<template>
  <el-breadcrumb class="bread-crumb" separator="/">
    <el-breadcrumb-item :to="{ path: '/' }">全部</el-breadcrumb-item>
    <el-breadcrumb-item>文件夹1</el-breadcrumb-item>
    <el-breadcrumb-item>文件夹1-1</el-breadcrumb-item>
  </el-breadcrumb>
</template>

<script>
export default {
  name: 'BreadCrumb',
  data() {
    return {}
  }
}
</script>

<style lang="stylus" scoped>
.bread-crumb {
  height: 28px;
  line-height: 28px;
}
</style>
```

在 `index.vue` 中引入 `BreadCrumb.vue`，调整布局，使菜单居左，右侧内容区域自适应宽度：

```vue
<template>
  <div class="home">
    <!-- 3. 使用组件 -->
    <!-- 左侧菜单 - 区分文件类型 -->
    <SideMenu class="home-left"></SideMenu>
    <!-- 右侧内容区 -->
    <div class="home-right">
      <!-- 面包屑导航栏 - 显示文件路径 -->
      <BreadCrumb></BreadCrumb>
    </div>
  </div>
</template>

<script>
import SideMenu from './components/SideMenu.vue' //  1.引入左侧菜单组件
import BreadCrumb from './components/BreadCrumb.vue' //  引入面包屑导航栏

export default {
  name: 'Home',
  components: {
    SideMenu, //  2. 注册组件
    BreadCrumb
  },
  data() {
    return {}
  }
}
</script>

<style lang="stylus" scoped>
.home {
  // 使用flex布局 菜单居左固定宽度 右侧内容区域自适应宽度
  display: flex;

  .home-left {
    box-sizing: border-box;
  }

  .home-right {
    box-sizing: border-box;
    width: calc(100% - 200px);
    padding: 8px 24px;
    flex: 1;
  }
}
</style>
```

刷新首页，看下页面布局，收缩展开左侧菜单时，右侧内容区域可以自适应宽度：

<img src="https://doc.shiyanlou.com/courses/3472/1557563/c2941412bb333fddd211158ef8f3cb7e-0/wm" alt="13-2" style="zoom:67%;" />

### 使用 Table 表格组件实现文件展示

参考官方示例（[Table 表格](https://element.eleme.cn/#/zh-CN/component/table)）来实现文件展示区域。仍然在 `views/Home/components` 下创建文件 `FileTable.vue`，添加以下内容：

```vue
<template>
  <el-table :data="tableData" style="width: 100%">
    <el-table-column prop="fileName" label="文件名"> </el-table-column>
    <el-table-column prop="extendName" label="类型" width="100"> </el-table-column>
    <el-table-column prop="fileSize" label="大小" width="60"> </el-table-column>
    <el-table-column prop="uploadTime" label="修改日期" width="180"> </el-table-column>
    <el-table-column label="操作" width="60"> </el-table-column>
  </el-table>
</template>

<script>
export default {
  name: 'FileTable',
  data() {
    return {
      // 表格数据先模拟
      tableData: [
        {
          fileName: 'markdown样式文件',
          extendName: 'markdown',
          fileSize: '10KB',
          uploadTime: '2020-10-28 16:33:33'
        },
        {
          fileName: '项目源码',
          extendName: 'zip',
          fileSize: '7MB',
          uploadTime: '2020-12-28 20:00:50'
        }
      ]
    }
  }
}
</script>
```

在 `index.vue` 中引入：

```vue
<template>
  <div class="home">
    <!-- 3. 使用组件 -->
    <!-- 左侧菜单 - 区分文件类型 -->
    <SideMenu class="home-left"></SideMenu>
    <!-- 右侧内容区 -->
    <div class="home-right">
      <!-- 面包屑导航栏 - 显示文件路径 -->
      <BreadCrumb></BreadCrumb>
      <!-- 表格组件 - 文件展示区 -->
      <FileTable></FileTable>
    </div>
  </div>
</template>

<script>
import SideMenu from './components/SideMenu.vue' //  1.引入左侧菜单组件
import BreadCrumb from './components/BreadCrumb.vue' //  引入面包屑导航栏
import FileTable from './components/FileTable.vue' //  引入文件表格展示区

export default {
  name: 'Home',
  components: {
    SideMenu, //  2. 注册组件
    BreadCrumb,
    FileTable
  },
  data() {
    return {}
  }
}
</script>
...
```

刷新首页，看下文件展示区域和页面布局：

![13-3](https://doc.shiyanlou.com/courses/3472/1557563/c2077568eb9aa346721939d0d47aa3ce-0/wm)

表格的操作列需要添加对文件操作的按钮：删除、移动、重命名、下载，同时需要支持操作列的展开和收缩两种形态，先来添加下操作列展开状态下的按钮，并为每个按钮先创建好点击事件，写法可以参考官方示例（[Table 自定义列模板](https://element.eleme.cn/#/zh-CN/component/table#zi-ding-yi-lie-mo-ban)），继续编辑 `FileTable.vue` 文件：

```vue
<template>
  <el-table :data="tableData" style="width: 100%">
    <!-- 已有代码不再赘述 -->
    ...
    <el-table-column label="操作" width="220">
      <template slot-scope="scope">
        <!-- 操作列展开状态下的按钮 -->
        <div class="opera-unfold">
          <el-button
            type="text"
            size="small"
            @click.native="handleClickDelete(scope.row)"
            >删除</el-button
          >
          <el-button
            type="text"
            size="small"
            @click.native="handleClickMove(scope.row)"
            >移动</el-button
          >
          <el-button
            type="text"
            size="small"
            @click.native="handleClickRename(scope.row)"
            >重命名</el-button
          >
          <el-button type="text" size="small">
            <a target="_blank" style="display: block; color: inherit">下载</a>
          </el-button>
        </div>
      </template>
    </el-table-column>

  </el-table>
</template>

<script>
export default {
  name: 'FileTable',
  data() {
    return {
      // ……
    }
  },
  methods: {
    // 删除按钮 - 点击事件
    handleClickDelete(row) {
      console.log('删除', row.fileName)
    },
    // 移动按钮 - 点击事件
    handleClickMove(row) {
      console.log('移动', row.fileName)
    },
    // 重命名按钮 - 点击事件
    handleClickRename(row) {
      console.log('重命名', row.fileName)
    }
  }
}
</script>
```

刷新页面，点击操作列的按钮，可以看到控制台打印出相应的信息：

![13-4](https://doc.shiyanlou.com/courses/3472/1557563/19eff73b5fa17880bb19c1a36860b922-0/wm)

再来添加操作列收缩状态下的按钮，用 Element UI 的下拉菜单（[Dropdown 下拉菜单](https://element.eleme.cn/#/zh-CN/component/dropdown)）来实现：

```vue
<template>
  <el-table :data="tableData" style="width: 100%">
    <!-- 已有代码不再赘述 -->
    ...
    <!-- 表格操作列 -->
    <el-table-column label="操作" width="260">
      <template slot-scope="scope">
        <!-- 操作列展开状态下的按钮 -->
        <!-- 已有代码不再赘述 -->
        ...
        <!-- 操作列收缩状态下的按钮 -->
        <el-dropdown trigger="click">
          <el-button size="mini">
            操作
            <i class="el-icon-arrow-down el-icon--right"></i>
          </el-button>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item @click.native="handleClickDelete(scope.row)"
              >删除</el-dropdown-item
            >
            <el-dropdown-item @click.native="handleClickMove(scope.row)"
              >移动</el-dropdown-item
            >
            <el-dropdown-item @click.native="handleClickRename(scope.row)"
              >重命名</el-dropdown-item
            >
            <el-dropdown-item>
              <a target="_blank" style="display: block; color: inherit">下载</a>
            </el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </template>
    </el-table-column>
  </el-table>
</template>
```

刷新页面，看下效果：

![13-5](https://doc.shiyanlou.com/courses/3472/1557563/0d5bf2843217e1847b6cbc214d05aedb-0/wm)

来添加下表格操作列收缩和展开状态的切换，将控制切换的入口添加在表格头上，这里需要用到 Element UI 中 Table 组件的自定义表头（[Table 自定义表头](https://element.eleme.cn/#/zh-CN/component/table#zi-ding-yi-biao-tou)），同时将表格列的宽度设置为动态变化，随收缩状态而改变：

```vue
<template>
  <el-table :data="tableData" style="width: 100%">
    <!-- 已有代码不再赘述 -->
    ...
    <!-- 表格操作列 自定义表格头，原有的 label="操作" 需要删除，宽度动态变化 -->
    <el-table-column :width="operaColumnIsFold ? 200 : 100">
      <!-- 自定义表格头 -->
      <template slot="header">
        <span>操作</span>
        <i class="el-icon-circle-plus" title="展开" @click="operaColumnIsFold = true"></i>
        <i class="el-icon-remove" title="折叠" @click="operaColumnIsFold = false"></i>
      </template>
      <template slot-scope="scope">
        <!-- 操作列展开状态下的按钮 通过v-if控制 -->
        <div class="opera-unfold" v-if="operaColumnIsFold">
          <!-- 已有代码不再赘述 -->
          ...
        </div>
        <!-- 操作列收缩状态下的按钮 通过v-else控制 -->
        <el-dropdown trigger="click" v-else>
          <!-- 已有代码不再赘述 -->
          ...
        </el-dropdown>
      </template>
    </el-table-column>
  </el-table>
</template>

<script>
export default {
  name: 'FileTable',
  data() {
    return {
      // data中已有的数据不再赘述
      ...
      operaColumnIsFold: false //  表格操作列-是否收缩
    }
  },
  watch: {
    // 监听收缩状态变化，存储在sessionStorage中，保证页面刷新时仍然保存设置的状态
    operaColumnIsFold(newValue) {
      sessionStorage.setItem('operaColumnIsFold', newValue)
    }
  },
  created() {
    this.operaColumnIsFold = sessionStorage.getItem('operaColumnIsFold') === 'true' //  读取保存的状态
  },
  ...
}
</script>

<style lang="stylus" scoped>
// 表格操作列-表头图标样式调整
.el-icon-circle-plus, .el-icon-remove {
  margin-left: 8px;
  cursor: pointer;
  font-size: 16px;

  &:hover {
    opacity: 0.5;
  }
}
</style>
```

刷新首页，点击表格操作列表头的图标，切换收缩状态：

<img src="https://doc.shiyanlou.com/courses/3472/1557563/2a938704ff8a338c432f406a388976f1-0/wm" alt="13-6" style="zoom:67%;" />

### 使用 Pagination 组件实现分页

由于网盘中存储的文件会很多，一次性加载所有的文件会降低文件加载速度，且在表格内拖动滚动条的方式在交互上也不够友好，所以需要常见的分页组件来提升查看文件的效率。参考官方示例（[Pagination 分页组件的附加功能](https://element.eleme.cn/#/zh-CN/component/pagination#fu-jia-gong-neng)）中的完整功能，来实现分页组件。仍然在 `views/Home/components` 下创建新文件 `FilePagination.vue`，添加以下内容：

```vue
<template>
  <el-pagination
    class="file-pagination"
    :current-page="pageData.currentPage"
    :page-size="pageData.pageCount"
    :total="pageData.total"
    :page-sizes="[10, 20, 50, 100]"
    layout="sizes, total, prev, pager, next"
    @current-change="handleCurrentChange"
    @size-change="handleSizeChange"
  >
  </el-pagination>
</template>

<script>
export default {
  name: 'FilePagination',
  data() {
    return {
      pageData: {
        currentPage: 1, //   页码
        pageCount: 20, //  每页显示条目个数
        total: 0 //  总数
      }
    }
  },
  methods: {
    // 分页组件 当前页码改变
    handleCurrentChange(currentPage) {
      this.pageData.currentPage = currentPage
    },
    // 分页组件 每页显示条目个数改变
    handleSizeChange(pageCount) {
      this.pageData.pageCount = pageCount
    }
  }
}
</script>

<style lang="stylus" scoped>
// 分页组件内容居右显示
.file-pagination {
  margin-top: 16px;
  text-align: right;
}
</style>
```

然后在 `views/Home/index.vue` 文件中添加如下代码：

```vue
<template>
  <div class="home">
  ...
    <div class="home-right">
      ...
      <FilePagination
        :pageData="pageData"
        @changePageData="changePageData"
      ></FilePagination>
    </div>
  </div>
</template>  

<script>
...
import FilePagination from './components/FilePagination.vue' //  引入分页组件
...
export default {
  name: 'Home',
  components: {
    ...
    FilePagination
  },
...
```

刷新首页，查看分页组件：

![13-7](https://doc.shiyanlou.com/courses/3472/1557563/bc7766a238253f375e81e0ea4d94708a-0/wm)

分页功能启用时，文件数据的回显会在稍后添加完接口之后讲解。

### 使用 Checkbox 组件实现列显隐控制

参考官方示例，结合 Checkbox（[Checkbox 多选框](https://element.eleme.cn/#/zh-CN/component/checkbox)）和 Dialog（[Dialog 对话框](https://element.eleme.cn/#/zh-CN/component/dialog)）来控制表格列的显隐。仍然在 `views/Home/components` 下新建文件 `SelectColumn.vue`，添加以下内容：

```vue
<template>
  <div class="select-column">
    <el-button type="info" size="mini" plain icon="el-icon-s-operation" @click="handleClickSelectColumn"
      >设置显示列</el-button
    >
    <!-- 对话框 当点击"设置显示列"按钮时弹出对话框 -->
    <el-dialog title="设置表格列显隐" width="700px" :visible.sync="dialogVisible">
      <!-- 多选框组件 勾选需要在表格中显示的列 -->
      <el-checkbox-group v-model="selectedColumn">
        <el-checkbox v-for="item in columnOptions" :key="item.value" :label="item.value">{{ item.label }}</el-checkbox>
      </el-checkbox-group>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogOk()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'SelectColumn',
  props: {},
  data() {
    return {
      dialogVisible: false,
      selectedColumn: [], //  被选中的表格需要显示的列
      columnOptions: [
        {
          value: 'extendName',
          label: '类型'
        },
        {
          value: 'fileSize',
          label: '大小'
        },
        {
          value: 'uploadTime',
          label: '修改日期'
        }
      ]
    }
  },
  methods: {
    // 设置显示列按钮 - 点击事件
    handleClickSelectColumn() {
      this.dialogVisible = true
    },
    // 对话框 确定按钮
    dialogOk() {
      this.dialogVisible = false
      console.log(this.selectedColumn)
    }
  }
}
</script>
```

然后在 `views/Home/index.vue` 文件中添加如下代码：

```vue
<template>
  <div class="home">
  ...
    <div class="home-right">
      <SelectColumn></SelectColumn>
      ...
</template>  

<script>
...
import SelectColumn from './components/SelectColumn.vue' //  引入控制列显隐组件
...
export default {
  name: 'Home',
  components: {
    ...
    SelectColumn
  },
...
```

刷新首页，点击设置显示列按钮，勾选显示列，确定后可以看到控制台打印了勾选的值：

![13-8](https://doc.shiyanlou.com/courses/3472/1557563/e458d0e785263f813f77a345aa532b9a-0/wm)

列显隐的控制接下来会结合 Vuex 的讲解一起实现。

## 使用 Vuex 存储界面设置

前面我们已经实现了左侧菜单、表格操作列的收缩状态切换，接上一节，来实现表格列的显隐控制。需要用到 Vue.js 中的核心插件 Vuex（[Vuex 是什么？ | Vuex](https://vuex.vuejs.org/zh/)）结合 sessionStorage 来实现。

Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。它采用集中式存储，管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。每一个 Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的**状态(state)**。Vuex 和单纯的全局对象有以下两点不同：

1. Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。
2. 不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地**提交 (commit) mutation**。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。

为了便于管理各个功能模块的状态，通常会在 `src/store` 目录下新建文件夹 `module`，在此文件夹中创建各个功能模块对应的 JavaScript 文件，例如之前创建的用户管理模块对应的 `user.js`，然后在 `src/store/index.js` 中引入各个模块， 每个模块拥有自己的 state、mutation、action、getter 等，可以参考官方解释（[Module | Vuex](https://vuex.vuejs.org/zh/guide/modules.html)）。

现在来创建首页功能模块对应的文件 `src/store/module/file.js`，参考 Vuex 官方示例（[开始 | Vuex](https://vuex.vuejs.org/zh/guide/)），提供一个初始 state 对象和一些 mutation ：

```javascript
export default {
  state: {
    allColumnList: ['extendName', 'fileSize', 'uploadTime'], // 所有可供选择的表格列
    selectedColumnList: sessionStorage.getItem('selectedColumnList') //  需要显示的表格列
  },
  mutations: {
    // 改变需要显示的表格列
    changeSelectedColumnList(state, data) {
      sessionStorage.setItem('selectedColumnList', data.toString())
      state.selectedColumnList = data.toString()
    }
  },
  actions: {}
}
```

因为 Vuex 中保存的值在页面刷新时也会重置，所以需要结合 sessionStorage 来实现。

接着需要在 `src/store/index.js` 中导入并注册此模块，并通过 getters 派生出 selectedColumnList 的值：

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

import user from './module/user' // 引入 user.js
import file from './module/file' // 1. 引入 file 模块

Vue.use(Vuex)

export default new Vuex.Store({
  state: {},
  getters: {
    // 是否登录
    isLogin: (state) => state.user.isLogin,
    // 用户名
    username: (state) => state.user.username,
    // 用户id
    userId: (state) => state.user.userId,
    // 用户详细信息
    userInfoObj: (state) => state.user.userInfoObj,
    // 需要显示的表格列
    selectedColumnList: (state) =>
      state.file.selectedColumnList === null ? state.file.allColumnList : state.file.selectedColumnList.split(',')
  },
  mutations: {
    //
  },
  actions: {
    //
  },
  modules: {
    user,
    file // 2. 注册模块
  }
})
```

接下来我们就可以在 `*.vue` 中使用 `this.$store.getters.selectedColumnList` 来获取存储的值，使用 `this.$store.commit('changeSelectedColumnList', [])` 来提交 mutation，其中 `[]` 为 mutation 的 **载荷（payload）**，传递给 `src/store/module/file.js` 中的 changeSelectedColumnList 参数中的 data ，载荷可以是任何类型，包括字符串、数组、对象等。

在 `FileTable.vue` 中通过计算属性 `computed` 来获取存储的值，并通过 `v-if` 来控制表格列是否渲染：

```vue
<template>
  <el-table :data="tableData" style="width: 100%">
    <!-- 已有代码不再赘述 -->
    ...
    <el-table-column prop="fileName" label="文件名"> </el-table-column>
    <!-- 通过 v-if 来控制 类型 列是否显示 -->
    <el-table-column
      prop="extendName"
      label="类型"
      width="100"
      v-if="selectedColumnList.includes('extendName')"
    >
    </el-table-column>
    <!-- 通过 v-if 来控制 大小 列是否显示 -->
    <el-table-column
      prop="fileSize"
      label="大小"
      width="60"
      v-if="selectedColumnList.includes('fileSize')"
    >
    </el-table-column>
    <!-- 通过 v-if 来控制 修改日期 列是否显示 -->
    <el-table-column
      prop="uploadTime"
      label="修改日期"
      width="180"
      v-if="selectedColumnList.includes('uploadTime')"
    >
    </el-table-column>
    <!-- 已有代码不再赘述 -->
    ...
  </el-table>
</template>

<script>
export default {
  data() {
    return {
      // 已有代码不再赘述
      ...
    }
  },
  computed: {
    // 表格显示列
    selectedColumnList() {
      return this.$store.getters.selectedColumnList
    }
  },
  ....
}
</script>
```

现在需要在 `SelectColumn.vue` 中来控制表格列的显隐，点击按钮时，获取 store 中存储的表格显示列，对应的多选框处于勾选状态，点击对话框确定按钮时，提交 mutation 更新表格显示列：

```vue
...
<script>
export default {
  name: 'SelectColumn',
  ...
  methods: {
    // 设置显示列按钮 - 点击事件
    handleClickSelectColumn() {
      // 1. 获取store中存储的表格显示列
      this.selectedColumn = this.$store.getters.selectedColumnList
      this.dialogVisible = true
    },
    // 对话框 确定按钮
    dialogOk() {
      // 2. 通过提交 mutation 更新表格显示列
      this.$store.commit('changeSelectedColumnList', this.selectedColumn)
      this.dialogVisible = false
      //console.log(this.selectedColumn)
    }
  }
}
</script>
```

刷新首页，表格显示列的显隐已可以控制：

<img src="https://doc.shiyanlou.com/courses/3472/1557563/23091ee1366ad7fe7da63bdce2097c22-0/wm" alt="13-9" style="zoom: 67%;" />

## 页面接口添加并实现数据联动

在 `src/request` 下新建文件 `file.js`，后续与文件有关的接口均可放在此文件中，添加接口：

```javascript
import { get } from './http'

// 左侧菜单选择的为 全部 时，根据路径，获取文件列表
export const getFileListByPath = (p) => get('/file/getfilelist', p)
// 左侧菜单选择的为 除全部以外 的类型时，根据类型，获取文件列表
export const getFileListByType = (p) => get('/file/selectfilebyfiletype', p)
```

当左侧菜单选择**全部**时，右侧的面包屑导航栏将会显示当前所处的路径，需调用接口——根据路径获取文件列表；当左侧菜单选择**除全部以外**的菜单时，右侧的面包屑导航栏会显示当前所展示的文件类型，右侧表格区域显示相应类型的文件，需调用接口——根据类型获取文件列表。

### 左侧菜单和右侧表格数据联动

表格数据获取要考虑三点：文件类型、文件路径、分页。在前面实现左侧菜单组件时，已经把文件类型添加到了路由参数上，文件类型可以通过 `this.$route.query.fileType` 来获取；文件路径在面包屑导航栏组件中；分页数据也在子组件中；查询结果需要在表格中显示。

综合考量，将调用接口函数写在 `src/views/Home/index.vue` 中比较合适，父子组件通讯使用 `props/$emit`，在 `index.vue` 中添加以下内容：

```vue
<template>
  <div class="home">
      ...
      <!-- 表格组件 - 文件展示区 -->
      <FileTable :tableData="tableData" :loading="loading"></FileTable>
      ...
  </div>
</template>

<script>
//	已有代码不再赘述
...
import { getFileListByPath, getFileListByType } from '@/request/file.js' //  引入获取文件列表接口

export default {
  name: 'Home',
  ...
  data() {
    return {
      loading: false,
      tableData: [], //  文件列表
      pageData: {
        currentPage: 1, //   页码
        pageCount: 20, //  每页显示条目个数
        total: 0 //  总数
      }
    }
  },
  computed: {
    // 左侧菜单选中的文件类型
    fileType() {
      return this.$route.query.fileType ? Number(this.$route.query.fileType) : 0
    }
  },
  watch: {
    fileType() {
      this.getFileData() //  获取文件列表
    }
  },
  mounted() {
    this.getFileData() //  获取文件列表
  },
  methods: {
    // 获取文件列表
    getFileData() {
      this.loading = true // 打开表格loading状态
      if (this.fileType === 0) {
        // 左侧菜单选择的为 全部 时，根据路径，获取文件列表
        this.loading = false
        this.getFileDataByPath()
      } else {
        // 左侧菜单选择的为 除全部以外 的类型时，根据类型，获取文件列表
        this.getFileDataByType()
      }
    },
    // 根据路径获取文件列表
    getFileDataByPath() {
      getFileListByPath({
        filePath: '/',
        currentPage: this.pageData.currentPage,
        pageCount: this.pageData.pageCount
      }).then(
        (res) => {
          this.loading = false //  关闭表格loading状态
          if (res.success) {
            this.tableData = res.data.list // 表格数据赋值
            this.pageData.total = res.data.total //  分页组件 - 文件总数赋值
          } else {
            this.$message.error(res.message)
          }
        },
        (error) => {
          this.loading = false
          console.log(error)
        }
      )
    },
    // 根据类型获取文件列表
    getFileDataByType() {
      getFileListByType({
        fileType: this.fileType, //  文件类型
        currentPage: this.pageData.currentPage, //  当前页码
        pageCount: this.pageData.pageCount //  每页条目数
      }).then(
        (res) => {
          this.loading = false //  关闭表格loading状态
          if (res.success) {
            this.tableData = res.data.list // 表格数据赋值
            this.pageData.total = res.data.total //  分页组件 - 文件总数赋值
          } else {
            this.$message.error(res.message)
          }
        },
        (error) => {
          this.loading = false
          console.log(error)
        }
      )
    },
    // 分页组件 - 页码或当页条目数改变时
    changePageData(pageData) {
      this.pageData.currentPage = pageData.currentPage // 页码赋值
      this.pageData.pageCount = pageData.pageCount //  每页条目数赋值
      this.getFileData() // 获取文件列表
    }
  }
}
</script>
```

在 `FilePagination.vue` 中添加以下内容：

```vue
...
<script>
export default {
  name: 'FilePagination',
  props: {
    pageData: {
      type: Object,
      required: true
    }
  },
  data() {
    return {}
  },
  methods: {
    // 分页组件 当前页码改变
    handleCurrentChange(currentPage) {
      this.$emit('changePageData', {
        ...this.pageData,
        currentPage
      })
    },
    // 分页组件 每页显示条目个数改变
    handleSizeChange(pageCount) {
      this.$emit('changePageData', {
        ...this.pageData,
        pageCount
      })
    }
  }
}
</script>
...
```

在 `FileTable.vue` 中添加以下内容：

```vue
<template>
  <el-table :data="tableData" style="width: 100%" v-loading="loading">
    <!-- 已有代码不再赘述 -->
    ...
  </el-table>
</template>

<script>
export default {
  name: 'FileTable',
  props: {
    // 表格数据，同时需要删除原本在 data( return { } ) 中的 tableData，否则会报错
    tableData: {
      type: Array,
      required: true
    },
    // 表格加载状态
    loading: {
      type: Boolean,
      required: true
    }
  },
  data() {
    return {
      operaColumnIsFold: sessionStorage.getItem('operaColumnIsFold') || false //  表格操作列-是否收缩
    }
  },
  //  已有代码不再赘述
  ...
}
</script>
```

刷新首页，左侧菜单选中**全部**、**图片**时看下运行结果：

![13-10](https://doc.shiyanlou.com/courses/3472/1557563/28947b8c2ded3fd3ad5628fc87d96f05-0/wm)

![13-11](https://doc.shiyanlou.com/courses/3472/1557563/4f142a8bd1bae4c585cacf29b957ef2b-0/wm)

下一节介绍完上传文件后，上述两个接口返回值就会有数据了。

### 左侧菜单和右侧面包屑导航栏数据联动

在 `src/views/Home/index.vue` 中给面包屑导航栏组件传值 `fileType`：

```vue
<!-- 面包屑导航栏 - 显示文件路径 -->
<BreadCrumb :fileType="fileType"></BreadCrumb>
```

由于要区分当前查看的文件是按类型还是按路径，面包屑导航栏要显示的信息不同，在 `BreadCrumb.vue` 中添加以下内容：

```vue
<template>
  <div class="bread-crumb-wrapper">
    <div class="current-path">当前位置：</div>
    <!-- 按类型查看文件时 -->
    <el-breadcrumb class="bread-crumb" v-if="fileType" separator="/">
      <el-breadcrumb-item>{{ fileTypeMap[fileType] }}</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 按路径查看文件时 -->
    <el-breadcrumb class="bread-crumb" v-else separator="/">
      <!-- 当点击面包屑导航栏中的某一级时，改变路由 -->
      <el-breadcrumb-item
        v-for="(item, index) in breadCrumbList"
        :key="index"
        :to="{ query: { fileType: 0, filePath: item.path } }"
        >{{ item.name }}</el-breadcrumb-item
      >
    </el-breadcrumb>
  </div>
</template>

<script>
export default {
  name: 'BreadCrumb',
  props: {
    fileType: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      fileTypeMap: {
        1: '全部图片',
        2: '全部文档',
        3: '全部视频',
        4: '全部音乐',
        5: '其他'
      },
      // 依据路径查看时 先模拟路径数据
      breadCrumbList: [
        { name: '全部文件', path: '/' },
        { name: '实验楼', path: '/实验楼/' }
      ]
    }
  }
}
</script>

<style lang="stylus" scoped>
.bread-crumb-wrapper {
  height: 32px;
  line-height: 32px;
  display: flex;
  align-items: center;
}
</style>
```

在下个实验讲解完上传文件和创建文件夹后，将会回到这个文件，完善面包屑导航栏的功能：当点击面包屑导航栏中的某一级时，改变路由，在 `index.vue` 中监听 filePath 变化重新获取表格数据。

刷新首页，看到左侧菜单切换时，路由变化，右侧面包屑导航栏也会随之变化：

![13-12](https://doc.shiyanlou.com/courses/3472/1557563/f80864eb4b454f8a17e830a6c21ced96-0/wm)

## 实验总结

本次实验介绍了使用 Element UI 的部分组件来实现首页页面，使用 Vuex 保存数据、共享状态，并实现了组件间的数据联动。

本次实验完整代码可以通过如下命令进行下载：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3472/code13.zip
```
