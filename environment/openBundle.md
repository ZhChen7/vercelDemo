## 优化规划：

1. 第一阶段，社区公共依赖拆分

1. 第二阶段，拆不带业务的包

1. 第三阶段，拆业务私包

目标： 26.06MB ->  小于10MB

## 准备工作

1. 前期CRM公共库统计：[CRM公共库统计](https://wrpnn3mat2.feishu.cn/wiki/wikcnhTax2zI3jLEZi7fApfEakc) 

## 第一阶段，社区公共依赖拆分

  我们使用社区依赖往往会锁定版本，所以没有必要每次发布都重复打包进来

  目前主要包含 vue vuex vue-router echarts momentjs element-ui @antv/g2 等

  改造完毕后，大概可以节省 10MB 左右的打包大小

- 手段：采用webpack - externals 外部扩展进行库拆分 

- souceMap - dev 调试

方案设计：

```JavaScript
# externals（vue.config.js） + html-CDN插入
config
  .set('externals', {
    "vue": 'Vue',
    'vue-router': 'VueRouter',
    "vuex": 'Vuex',
    "moment": "moment",
    "echarts": "echarts",
  })
  
  <script src="https://img.hetao101.com/resources/vue.runtime.min.js"></script>
  <script src="https://img.hetao101.com/resources/vue-router.min.js"></script>
  <script src="https://img.hetao101.com/resources/vuex.min.js"></script>
  <script src="https://img.hetao101.com/resources/moment.min.js"></script>
  <script src="https://img.hetao101.com/resources/echarts.min.js"></script>
  <script src="https://img.hetao101.com/resources/axios.min.js"></script>
```

使用 html-webpack-externals-plugin 插件

```JavaScript
    const HtmlWebpackExternalsPlugin = require('html-webpack-externals-plugin')
    const externalConfig = [
      {
        module: 'vue',
        entry: 'https://img.hetao101.com/resources/vue.runtime.min.js',
        global: 'Vue',
      },
      {
        module: 'vue-router',
        entry: 'https://img.hetao101.com/resources/vue-router.min.js',
        global: 'VueRouter',
      },
      {
        module: 'vuex',
        entry: 'https://img.hetao101.com/resources/vuex.min.js',
        global: 'Vuex',
      },
      {
        module: 'moment',
        entry: 'https://img.hetao101.com/resources/moment.min.js',
        global: 'moment',
      },
      {
        module: 'echarts',
        entry: 'https://img.hetao101.com/resources/echarts.min.js',
        global: 'echarts',
      },
      {
        module: 'axios',
        entry: 'https://img.hetao101.com/resources/axios.min.js',
        global: 'axios',
      },
    ]
    config.plugins.push(
      new HtmlWebpackExternalsPlugin({
        externals: externalConfig
      })
    )
```

- 上传工具：Oss-browser 

- 资源储存地址：oss://img-hetao/resources/

## 课导系统

### 优化前：26.06MB

分析报告：report.html

暂时无法在飞书文档外展示此内容

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=ODU3ZTRkMmY1NGNlM2Y0NTQ2MTk5M2JhNmU4ODA3Mjhfb2oyRDRISWxhMDY4U3dQTk5pUHM5M1JrMnRseDFrU2hfVG9rZW46Ym94Y25WdmdnT3lKVVliaE5CT3Rxd0FpMmNoXzE2NjUyMjgzMzg6MTY2NTIzMTkzOF9WNA)

### 优化后：16.61MB

分析报告：report1.html

暂时无法在飞书文档外展示此内容

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=MzRjYTQ1YzY0Y2YwMTNkOGUxZjM4MTEyMWFhZTg0OWFfRWtEQTJpR3FLNXgzSjROZkFuRHpBTEpGcW1IcGNOemdfVG9rZW46Ym94Y25uQzA1ZjlPb1dPb2dnRzBMajVHMEVjXzE2NjUyMjgzMzg6MTY2NTIzMTkzOF9WNA)

## 第二阶段，拆不带业务的包

主要是如下这些与业务无关的私包

1. @hetao/crm-basic  （ 基础业务组件）

1. @hetao/analysis-web-sdk （埋点SDK）

1. @hetao/npm-oss-sdk  （阿里云OSS上传插件）

1. @hetao/crm-package（工具包集合）
   1. @hetao/limbo-tools  （工具集）
   2. @hetao/package-common-js
   3. @hetao/package-vue-components （通用组件库封装）

1. @hetao/package-css （通用样式库、字体、reset样式）

1. @hetao/package-utils（与业务耦合的方法、枚举等）

**打成外部****CDN****：**

分类（重新架构）：

1. 工具库集合：@hetao/crm-package，@hetao/limbo-tools，@hetao/package-common-js，@hetao/package-utils

1. 含业务组件
   1. 业务UI库：@hetao/package-vue-components 、@hetao/package-css
   2. 基础业务组件：@hetao/crm-basic 

1. 埋点：@hetao/analysis-web-sdk 

暂时无法在飞书文档外展示此内容

### 工具库集合打包处理：

- 项目：hetaotool

- 打包CDN：https://img.hetao101.com/hetaotool/index.a2d6.bundle.js

- 上传平台：阿里云OSS（AliOSSPlugin）

@hetao/crm-package，@hetao/package-css，@hetao/limbo-tools，@hetao/package-common-js，@hetao/package-utils