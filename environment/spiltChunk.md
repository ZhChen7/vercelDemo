## 背景：

目前CRM系统，全采用vuecli脚手架搭建，是大型SPA单页应用，相关依赖包全部打进一个bundle文件app.js被浏览器加载，导致单文件体积过大，影响加载速度。需要进行合理的进行拆包，性能优化。

**目前存在的问题：**

1. 打包后单文件过大，需要合理的进行拆包

1. 开发过程中编译、构建时间过长，因为全量处理了私包文件

## 收益

1. 最初的比较大的打包文件，拆分成若干小文件，并行加载，提高加载速度

1. 因为不用在开发阶段处理私包，能极大的提高打包和编译速度

1. 单独发布私包后，不用额外打包上线关联依赖的项目（如有需要锁定具体版本的需求，可以指定版本，但在我司的业务场景下，很少有这样的场景）

## 优化规划：

1. 第一阶段，社区公共依赖拆分

1. 第二阶段，拆不带业务的包

1. 第三阶段，拆业务私包

*目标： 26.06MB ->  小于10MB

## 准备工作：

1. 前期CRM公共库统计：[CRM公共库统计](https://wrpnn3mat2.feishu.cn/wiki/wikcnhTax2zI3jLEZi7fApfEakc) 

## 第一阶段，社区公共依赖拆分

  我们使用社区依赖往往会锁定版本，所以没有必要每次发布都重复打包进来

  目前主要包含 vue vuex vue-router echarts momentjs element-ui @antv/g2 等

  改造完毕后，大概可以节省 10MB 左右的打包大小

- 手段：采用webpack - externals 外部扩展进行库拆分 

- souceMap - dev 调试

**技术方案设计：**

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
    entry: isProduction ?
    'https://img.hetao101.com/resources/2.6.12/vue.runtime.min.js' :
    'https://img.hetao101.com/resources/2.6.12/vue.js',
    global: 'Vue',
  },
  {
    module: 'vue-router',
    entry: 'https://img.hetao101.com/resources/3.5.1/vue-router.min.js',
    global: 'VueRouter',
  },
  {
    module: 'vuex',
    entry: 'https://img.hetao101.com/resources/3.6.2/vuex.min.js',
    global: 'Vuex',
  },
  {
    module: 'moment',
    entry: 'https://img.hetao101.com/resources/2.24.0/moment.min.js',
    global: 'moment',
  },
  {
    module: 'echarts',
    entry: 'https://img.hetao101.com/resources/5.1.2/echarts.min.js',
    global: 'echarts',
  },
  {
    module: 'axios',
    entry: 'https://img.hetao101.com/resources/0.19.0/axios.min.js',
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

## 课导系统（为例）

### 优化前包大小：26.06MB

分析报告：report.html

暂时无法在飞书文档外展示此内容

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTBjZTM4NDE4NjZlOTgxN2M5YWNiNGE0YWFiMzA1M2RfVVl6VVNXc1BDWVZrQW1pczlPbG90TEdVMm1BQjhPRlJfVG9rZW46Ym94Y25WdmdnT3lKVVliaE5CT3Rxd0FpMmNoXzE2Njc0NDYyNTM6MTY2NzQ0OTg1M19WNA)

### 优化后包大小：16.61MB

分析报告：report1.html

暂时无法在飞书文档外展示此内容

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=Yjc5NmFmODgwNzY5ZWE2OGUxMjQ1NjIyZjhjNDBjYTFfUG5XUVNSR21sOThpZUtacXRoSkZDM29OM3JlNGk5em5fVG9rZW46Ym94Y25uQzA1ZjlPb1dPb2dnRzBMajVHMEVjXzE2Njc0NDYyNTM6MTY2NzQ0OTg1M19WNA)

### 成果分析：

1. #### 体积大小

- 结果分析： **26.06****MB** **-> 16.61MB**,  优化 **36.26%**

1. #### 构建速度

- **结果分析**
  - 项目本地启动速度🆚：27s -> 18s ，提升 33%
  - build构建时间优化后：40s ->  32s，提升 20%

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=YzhkY2NlZWQwM2U3ODc2YWNkOGRkZWY5ZjgwNWNhNzBfMzlCb3RiWXRXOW1XdkJGelZLdVQ5bVIyc2lRRkF4QzJfVG9rZW46Ym94Y256SHRBR1diSjVrcmRqcDVQM0xQdFlHXzE2Njc0NDYyNTM6MTY2NzQ0OTg1M19WNA)

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=ZWJmNzY3YzUyZGM5YTRjODFhMzAzNTAxZTUxODVkZDJfMFBuNmVqSG5TTnZsMXlNR0hwTmpxNmlkYW12YkxJbG1fVG9rZW46Ym94Y24zbTdaeG1tZXdpUlU0RlNDcUh0b21jXzE2Njc0NDYyNTM6MTY2NzQ0OTg1M19WNA)

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

https://img.hetao101.com/hetaotool/index.a2d6.bundle.js

### 第三阶段，拆业务私包

待开始...