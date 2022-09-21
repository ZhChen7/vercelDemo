
总体规划-TODO：
1. 编码规范建设  ✅
2. commit规范建设     ✅
3. 自动化拉取打tag功能（./tag.sh）  ✅
4. 分支命名规范
5. Issue 与 Merge Request 的模版 ( 添加 Merge Request Templates )
6. Code Review 合并条件（CR流程相关）
  1. 至少一条评论
  2. 至少两人点击MR通过（可用飞书文档收纳当天MR，指派对应MR人员）
  3. discussions 未解决时，无法点击 Merge 按钮
7. Eslint 个性化检查（语法等...）
8. 线上CRM系统 - 版本号、项目信息输出
  1. TODO：清理项目线上控制台输出（自定义loader）
9. 前端 Pipelines 失败/成功 飞书提醒功能 
10. node-sass 优化（解决锁node版本问题）
11. 加快打包build速度、Docker镜像、CI/CD赋能
  1. 拆包（优化包体积）
    1. 业务私包、工具函数包（todo: 整合资源）
  2. 资源CDN化（vue、elementUI...）
12. Webpack升级
  1. webpack升级5
  2. 剔除vuecli - 从vuecli集成中拆分出webpack，方便用webpack新特性及个性化配置
MR：https://git.corp.hetao101.com/CRM/crm-web-tutor-system-v2/merge_requests/704/diffs
---
一、开发遇到的痛点以及待优化问题
1. 缺少编码规范/自动检测修复功能，需制定合理的编码规范、统一编码代码风格
2. 缺少commit规范建设，需限制/规范commit message的格式，避免不规范的代码提交
3. 缺少自动化拉取tag功能（./tag.sh ），每次上线需手动填写tag，之前流程：前端项目上线流程 
4. 缺少前端 Pipelines 失败/成功 飞书提醒功能
5. 缺少上线系统版本号标识（无法区分线上跑的是哪一个版本，不方便问题快速定位）
待优化：
1. 优化测试发版流程、建立公共的testing测试分支，避免分支冲突造成影响测试流程
2. 部署流程（加快打包build速度、gitlab-runner、Docker镜像构建速度）
3. node-sass 锁node版本问题（后续可用dart-sass代替）， 影响范围（无法使用nodejs高版本以及涉及依赖的第三方库）
4. npm包构建方式（公共模块处理方式），可采用微前端方案优化
5. Webpack解耦（从vuecli集成中拆分出webpack，方便webpack升级5以及自定义化配置）

二、解决方案
1. 编码规范
解决方案：
eslint + prettier + yorkie + lint-staged
2. commit规范建设
解决方案：
commitlint + lint-staged + （commitizen cz-conventional-changelog）
[图片]
3. 自动化拉取tag功能
解决方案： 配合gitlab，写自动拉取tag脚本（限制master分支）
未来可实施部署方案：
- 测试环境（切换testing分支 -> 在testing分支打tag【接入自动化打tag脚本】 -> 发布测试tag ）
- 正式环境（切换master分支 -> 在master分支打tag【接入自动化打tag脚本】-> 发布正式tag  ）

使用方式：切换master分支，执行./tag.sh
[图片]

4. 前端 Pipelines 失败/成功 飞书提醒功能
解决方案：添加gitlab机器人，设置webhook

5. 分支命名规范
master 分支： 
- master 为主分支，也是用于部署生产环境的分支，确保 master 分支稳定性
- master 分支一般由 develop 以及 hotfix 分支合并，任何时间都不能直接修改代码
 feature 分支： （feature/*）
- 开发新功能时，以 develop 为基础创建 feature 分支
- 分支命名：feature/ 开头的为特性分支；命名规则：feature/user_module、 feature/cart_module
hotfix 分支： （hotfix/*）
- 分支命名：hotfix/ 开头的为修复分支，它的命名规则与 feature 分支类似
- 线上出现紧急问题时，需要及时修复，以 master 分支为基线，创建 hotfix 分支，修复完成后，需要合并到 master 分支和 develop 分支

6. 添加 Merge Request Templates
- .gitlab/merge_request_templates/
[图片]
[图片]
7. CR 流程
- Code Review 合并条件
  1. 至少一条评论
  2. 至少两人点击MR通过（可建立文档收纳当天MR，指派对应MR人员）
  3. discussions 未解决时，无法点击 Merge 按钮
[图片]

8. 控制台赋能 - 版本号、项目信息输出
- 用途：方便定位线上系统版本 - 查找问题
方案：

TODO：清理项目线上控制台输出（防止打印敏感信息）
方案一：使用 uglifyjs-webpack-plugin / terser-webpack-plugin 插件
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')
module.exports = {
  // 省略...
  mode: "production",
  optimization: {
    minimizer: [
      new UglifyJsPlugin({
        uglifyOptions: {
          // 删除注释
          output: {
            comments: false
          },
          compress: {
            drop_console: true, // 删除所有调式带有console的
            drop_debugger: true,
            pure_funcs: [ 'console.log' ] // 删除console.log
          }
        }
      })
    ]
  }
}
方案二：webpack - 自定义loader
// source：表示当前要处理的内容
const reg = /(console.log()(.*)())/g
module.exports = function(source) {
  // 通过正则表达式将当前处理内容中的console替换为空字符串
  source = source.replace(reg, "")
  // 再把处理好的内容return出去，坚守输入输出都是字符串的原则，并可达到链式调用的目的供下一个loader处理
  return source
}

9. 拆包
- CRM业务组件包&工具包做资源聚合cdn化 
拆包问题记录：
1. 拆element：私包对element有非标准化模块引用（图片放大器等...），需要单独处理
# externals
config
  .set('externals', {
    "vue": 'Vue',
    "element-ui": "ELEMENT",
  })
  
     
<script src="https://cdn.bootcdn.net/ajax/libs/vue/2.6.11/vue.runtime.min.js"></script>
<script src="https://unpkg.com/element-ui@2.10.1/lib/index.js"></script>
      
[图片]

---
三、常见易混淆问题
1. eslint与prettier有什么区别
2. eslint与airbnb，standard代码规范有什么区别
3. eslint与vscode eslint插件有什么区别

Eslint与Prettier的区别
ESLint 是一个 javascript 代码检测工具，它主要检测的是下面这些问题：
- 例如未使用的变量
- 未定义的引用
- 比较时使用三等
- 禁止不必要的括号
- ...等等
以上这些问题，都是我们需要去统一约定的，这就是Eslint所做的事情。
那Prettier呢，我们都知道Prettier是一款纯粹的代码格式化工具，很显然，它的作用就是：格式化，不关系代码语法等问题。 比如常见格式化问题：
- 代码缩紧
- 单引号与双引号
- ...等鞥
因此，我们可以理解Eslint和Prettier的作用都是为了让我们项目中的代码规范能够统一，只是他们的专攻的方向不一样，而在实际项目开发中，最佳实践就是将Eslint与Prettier都引入到我们的项目中。 不过要注意一点：Eslint与Prettier虽然专攻的方向不一样，但是还是有一部分重叠的规则，那如果项目中亮着都引入啦，就需要我们额外处理这些这部分冲突的规则

Eslint与airbnb，standard代码规范的区别
我们可以把Eslint理解为一个单纯的代码检测工具，那具体检测的标准是什么呢？那就需要定义一系列的规则，比如：单引号双引号问题，我们可以定义一条规则，因此，Eslint默认集成了一套自己的规则列表。
而airbnb，standard代码规范又是什么呢？其实就是各个公司把自己常使用的一些规则列表开源了出来，从而我们在自己的项目中，就可以直接引入这些开源的规则列表，就没必要我们自己去定义规则啦。
常见开源的规则标准有：
- eslint-config-airbnb
- eslint-config-standard
- eslint-config-google
当然，国内的大公司其实也开源了自己的代码规范，比如：腾讯的 eslint-config-tencent等。

eslint与vscode eslint插件有什么区别
我们常说的Eslint，其实就是一个npm包，我们可以在项目中像安装其他依赖包一样，直接安装eslint这个包，并且使用他内置的一些命令，我们也可以修改其对应的配置文件等。
npm init
npm i eslint --save-dev
npx eslint --init // 初始化eslint配置文件
npx eslint lint  // 检测
npx eslint lint --fix // 检测并修复
也就是说，安装完之后，我们就可以手动去执行npx eslint lint命令去对我们的代码进行检测是否符合规范的，同时还可以使用--fix参数去手动修复。
但是试想一下，我们在实际代码开发的过程中，每写几行代码，就得手动执行一下这个命令，那得多麻烦呀，很显然，这不是我们的最佳实践。那到底如何做呢？答案就是在编辑器中引入eslint插件.
这里以vscode为例，如下图所示，安装eslint插件：
[图片]
[图片]
[图片]
安装完成以后，打开我们自己的项目中，我们就可以右击使用vscode的格式化功能去直接对我们的项目中代码进行格式化啦。要注意的一点：当vscode安装eslint插件以后，就会自动优先以我们项目中引入的eslint规则去进行格式化，这样可以保证如果我们切换到其他项目，使用了不同的规则，vscode格式化的时候，依然是以当前项目的规则为准。，当然前提是我们项目需要引入eslint。
除此之外，还可以进一步优化，每次都得右击才能进行格式化是不是也很麻烦，是的，这时我们也可以修改vscode eslint插件的配置，从而实现每次我们保存代码的时候，就可以自动格式化。

四、技术使用说明
1、初始化 ESLint 配置文件
npm i -D eslint
npx eslint --init
[图片]
.eslintrc.js：
module.exports = {
  root: true,
  parser: 'vue-eslint-parser',
  env: {
    browser: true,
    es2021: true
  },
  globals: {
    // 自定义的全局变量
    window: true,
    require: true,
    process: true,
    localStorage: true,
    chrome: true,
  },
  extends: [
    'plugin:vue/essential',
    'standard',
    'prettier'
  ],
  parserOptions: {
    ecmaVersion: 12,
    sourceType: 'module'
  },
  plugins: [
    'vue',
    'prettier'
  ],
  rules: {
    "prettier/prettier": "error",
    "arrow-body-style": "off",
    "prefer-arrow-callback": "off",
    "no-console": "warn",
    "no-debugger": "warn",
    // ----- standard and others 相关规则 -----
    'semi': [ 'error', 'never' ],
    'indent': [
      'error',
      2,
      {
        SwitchCase: 1,
        VariableDeclarator: 'first',
        FunctionDeclaration: { parameters: 'first' },
        FunctionExpression: { parameters: 'first' },
        CallExpression: { arguments: 'first' },
        MemberExpression: 1,
        ArrayExpression: 'first',
        ObjectExpression: 'first',
        ImportDeclaration: 'first',
        flatTernaryExpressions: false,
        offsetTernaryExpressions: true,
        ignoreComments: false,
      },
    ],
    'key-spacing': [
      'error'
    ],
    'no-trailing-spaces': 'error',
    'comma-dangle': [ 'error', 'only-multiline' ],
    'comma-spacing': [ 'error', { before: false, after: true } ],
    'space-infix-ops': 'error',
    'array-bracket-spacing': [ 'error', 'always' ],
    'object-curly-spacing': [ 'error', 'always' ],
    'object-curly-newline': [
      'error',
      {
        ObjectExpression: { consistent: true },
        ObjectPattern: { consistent: true },
        ImportDeclaration: { multiline: true, minProperties: 3 },
        ExportDeclaration: { multiline: true, minProperties: 3 },
      },
    ],
    'object-property-newline': [
      'error',
      { allowAllPropertiesOnSameLine: true },
    ],
    'operator-linebreak': [
      'error',
      'after',
      {
        overrides: {
          '?': 'before',
          ':': 'before',
        },
      },
    ],
    'multiline-ternary': [ 'error', 'always-multiline' ],
    'no-new': 'error',
    'no-use-before-define': 'off', // React import, service interface
    'no-var': 'error',
    'no-eval': 'error',
    'prefer-regex-literals': 'off', // RegExp
    'jsx-quotes': [ 'error', 'prefer-single' ],
    'quote-props': [ 'error', 'consistent-as-needed' ],
    "vue/multi-word-component-names": [ "error", {
      ignores: [ "template" ]
    } ]
  }
}



2、初始化 Prettier 配置
npm install prettier eslint-config-prettier eslint-plugin-prettier -D
- eslint-config-prettier 用来关闭eslint和 prettier冲突的规则，我们要将这个放置在extends的最后，这样它就有机会覆盖其他配置。
- eslint-plugin-prettier前面我们关闭了eslint的规则，现在我们开启prettier的规则。
在根目录下新建 .prettierrc.js 文件：
- useTabs：使用tab缩进还是空格缩进，选择false；
- tabWidth：tab是空格的情况下，是几个空格，选择2个；
- printWidth：当行字符的长度，推荐80，也有人喜欢100或者120；
- singleQuote：使用单引号还是双引号，选择true，使用单引号；
- trailingComma：在多行输入的尾逗号是否添加，设置为 none；
- semi：语句末尾是否要加分号，默认值true，选择false表示不加；
module.exports = {
  eslintIntegration: true,
  stylelintIntegration: true,
  bracketSpacing: true,
  semi: false, // 不添加行尾分号
  singleQuote: true, // 文件使用单引号
  jsxSingleQuote: true, // jsx 使用单引号
  jsxBracketSameLine: true, // jsx 标签结束符'>'换行
  printWidth: 80,
  proseWrap: 'preserve',
  quoteProps: 'as-needed',
  trailingComma: 'es5', // 末尾逗号（type 定义除外）
  requirePragma: true,
  overrides: [
    {
      files: '*.md',
      options: {
        tabWidth: 2,
      },
    },
  ],
}
 创建.prettierignore忽略文件
/dist/*
.local
.output.js
/node_modules/**

**/*.svg
**/*.sh

/public/*

3、初始化 lint-staged、husky/yorkie 配置
husky:
npm install husky --save-dev
npx husky install

// package.json
{
  "scripts": {
    "prepare": "husky install"
  }
}
yorkie：
npm i -D yorkie

  "gitHooks": {
    "pre-commit": "lint-staged",
    "commit-msg": "commitlint -e $GIT_PARAMS"
  }
  
  "lint-staged": {
    "*.{ts,js,tsx,jsx,vue}": [
      "eslint --fix"
    ]
  }
  
  "commitlint": {
    "extends": [
      "@commitlint/config-conventional"
    ]
  }
创建脚本 .husky/pre-commit：
npx husky add .husky/pre-commit "npm run lint-staged"

npx husky add .husky/commit-msg 'npx commitlint --edit $1'
lint-staged：
npm install --save-dev lint-staged
  "lint-staged": {
    "*.{ts,js,tsx,jsx,vue}": [
      "eslint --fix",
      "git add"
    ],
    "**/*.less": "stylelint --syntax less",
    "**/*.{js,jsx}": "npm run lint-staged:js"
  }
  "scripts": {
    "lint-staged": "lint-staged"
  },
4、commitlint
    "commitizen": "^4.2.4",
    "cz-conventional-changelog": "^3.2.0",
vim智能提示
npm install --save-dev @commitlint/config-conventional @commitlint/cli
新建 commitlint.config.js
module.exports = { 
    extends: ["@commitlint/config-conventional"] 
}

5、配置 stylelint
npm install stylelint --save-dev

# Could not find "stylelint-config-standard". Do you need a `configBasedir`?
npm install stylelint-config-standard --save-dev

6、配置 package.json 中的脚本命令
"scripts": {
  "lint": "npm run lint:js && npm run lint:style && npm run lint:prettier",
  "lint-staged": "lint-staged",
  "lint-staged:js": "eslint --ext .js,.jsx,.ts,.tsx ",
  "lint:fix": "eslint --fix --cache --ext .js,.jsx,.ts,.tsx --format=pretty ./src && npm run lint:style",
  "lint:js": "eslint --cache --ext .js,.jsx,.ts,.tsx --format=pretty ./src",
  "lint:prettier": "check-prettier lint",
  "lint:style": "stylelint --fix \"src/**/*.less\" --syntax less",
  "prettier": "prettier -c --write \"**/*\""
},

7、.editorconfig
EditorConfig 有助于为不同 IDE 编辑器上处理同一项目的多个开发人员维护一致的编码风格。
# http://editorconfig.org

root = true

[*] # 表示所有文件适用
charset = utf-8 # 设置文件字符集为 utf-8
indent_style = space # 缩进风格（tab | space）
indent_size = 2 # 缩进大小
end_of_line = lf # 控制换行类型(lf | cr | crlf)
trim_trailing_whitespace = true # 去除行首的任意空白字符
insert_final_newline = true # 始终在文件末尾插入一个新行

[*.md] # 表示仅 md 文件适用以下规则
max_line_length = off
trim_trailing_whitespace = false

[Makefile]
indent_style = tab

---
五、改造课导系统
    "@vue/cli-plugin-eslint": "3.12.1",
    "@vue/eslint-config-standard": "4.0.0",
    "babel-eslint": "10.1.0",
   
    "eslint": "5.16.0",
    "eslint-plugin-vue": "5.2.3",
卸载vue-cli脚手架默认的eslint校验规则
解耦（方便后续可持续迭代）
npm un @vue/cli-plugin-eslint @vue/eslint-config-standard babel-eslint eslint eslint-plugin-vue
解决方案：
课导分支：https://git.corp.hetao101.com/CRM/crm-web-tutor-system-v2/merge_requests/704/diffs

使用方式：
1. 拉去最新master，删掉node_modules，重新 npm install
2. 安装vscode插件
[图片]


六、Error 问题记录
[图片]
- 问题描述：node.js 版本过低 > 14