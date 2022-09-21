总体规划-TODO：

1. 编码规范建设 
2. commit规范建设   
3. 自动化拉取打tag功能（./tag.sh） 
4. 分支命名规范
5. Issue 与 Merge Request 的模版 ( 添加 Merge Request Templates )
6. Code Review 合并条件（CR流程相关）
   1. 至少一条评论
   2. 至少两人点击MR通过（可用飞书文档收纳当天MR，指派对应MR人员）
   3. discussions 未解决时，无法点击 Merge 按钮

7. Eslint 个性化检查（语法等...）
8. 线上CRM系统 - 版本号、项目信息输出
   1. TODO：清理项目线上控制台输出（自定义loader）

9. 前端 **[Pipelines](https://git.corp.hetao101.com/CRM/crm-web-tutor-system-v2/pipelines)** **失败/成功** 飞书提醒功能 

10. node-sass 优化（解决锁node版本问题）

11. 加快打包build速度、CI/CD赋能（执行个性化任务）

12. 拆包（优化包体积）

    1. 业务私包、工具函数包（todo: 整合资源）

    2. 资源CDN化（vue、elementUI...）

13. Webpack升级

    1. webpack升级5

    2. 剔除vuecli - 从vuecli集成中拆分出webpack，方便用webpack新特性及个性化配置

MR：https://git.corp.hetao101.com/CRM/crm-web-tutor-system-v2/merge_requests/704/diffs

## 一、开发遇到的痛点以及待优化问题

1. 缺少编码规范/自动检测修复功能，需制定合理的编码规范、统一编码代码风格

1. 缺少commit规范建设，需限制/规范commit message的格式，避免不规范的代码提交

1. 缺少自动化拉取tag功能（./tag.sh ），每次上线需手动填写tag，之前流程：[前端项目上线流程](https://wrpnn3mat2.feishu.cn/wiki/wikcnZG9JFURECS8HluqkSs7oJb) 

1. 缺少前端 **[Pipelines](https://git.corp.hetao101.com/CRM/crm-web-tutor-system-v2/pipelines)** **失败/成功** 飞书提醒功能

1. 缺少上线系统版本号标识（无法区分线上跑的是哪一个版本，不方便问题快速定位）

**待优化：**

1. 优化测试发版流程、建立公共的testing测试分支，避免分支冲突造成影响测试流程

1. 部署流程（加快打包build速度、gitlab-runner、Docker镜像构建速度）

1. node-sass 锁node版本问题（后续可用dart-sass代替）， 影响范围（无法使用nodejs高版本以及涉及依赖的第三方库）

1. npm包构建方式（公共模块处理方式），可采用微前端方案优化

1. Webpack解耦（从vuecli集成中拆分出webpack，方便webpack升级5以及自定义化配置）

1. CR体系建设

1. 性能优化，打包体积过大（拆私包、拆第三方库）

## 二、解决方案/使用说明

1. ### 编码规范

**解决方案**：

```undefined
eslint + prettier + yorkie + lint-staged
```

**使用方式：**

1. 拉取最新master，删掉node_modules，重新 `npm install`

1. 安装vscode插件

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=Y2U4ZGJhOTU0NWQ0NjNmMGRhY2ZhYzlkYTNjNjU2ZTdfVk4zMnF1S1BGR0ZYTVhnWnY2QkNjbzZGY2k4NDk5VHpfVG9rZW46Ym94Y25DVUNLZVBtamhtaUVaTUJ1c2RaODNjXzE2NjM3MzIxMTU6MTY2MzczNTcxNV9WNA)

1. ### commit规范建设

**解决方案**：

```undefined
commitlint + lint-staged + （commitizen cz-conventional-changelog）
```

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=ODdiZDQzZWVkYmI5MjA0YWNlZmU5MDczZjFiNzFkMDFfZjJVeDVqN1J2M1hZc1BxREdpd01KVVhzUk9rZ1lobGdfVG9rZW46Ym94Y24wem85Zmg0NmJEdjJWRlhkRlRHMDFjXzE2NjM3MzIxMTU6MTY2MzczNTcxNV9WNA)

**使用方式：**

> 参考：https://github.com/conventional-changelog/commitlint/tree/master/@commitlint/config-conventional

```PowerShell
$ <commit-type>[(commit-scope)]: <commit-message>

<commit-type> 常见为： 
- chore：构建配置相关。
- docs：文档相关。
- feat：添加新功能。
- fix：修复 bug。
- pref：性能相关。
- refactor：代码重构，一般如果不是其他类型的 commit，都可以归为重构。
- revert：分支回溯。
- style：样式相关。
- test：测试相关。

[(commit-scope)] 可选，表示范围，例如：refactor(cli)，表示关于 cli 部分的代码重构。
<commit-message> 提交记录的信息

# correct example
feat: add a new feature xxx
fix: fix issue xxx
refactor: rewrite the code of xxx
fix(testing): correct testing code of xxx
```

1. ### 自动化拉取tag功能

**解决方案**： 配合gitlab，写自动拉取tag脚本（限制master分支）

**未来可实施****部署****方案**：

- 测试环境（切换testing分支 -> 在testing分支打tag【接入自动化打tag脚本】 -> 发布测试tag ）

- 正式环境（切换master分支 -> 在master分支打tag【接入自动化打tag脚本】-> 发布正式tag  ）

**使用方式**：

- 切换master分支，执行`./tag.sh`

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=MzI5MTNkNDUzMzk2NDIzZjY5MzEwZWQyMWJlNWRkMGRfMlZYa09GSXhzYk9lWjFpRWRuQUhUc3k4WFVYRXV1aVpfVG9rZW46Ym94Y25ld1hCeHlLakRhdG1uWTNISUh0bTlkXzE2NjM3MzIxMTU6MTY2MzczNTcxNV9WNA)

1. ### 前端 **[Pipelines](https://git.corp.hetao101.com/CRM/crm-web-tutor-system-v2/pipelines)** **失败/成功** 飞书提醒功能

解决方案：添加gitlab机器人，设置webhook

1. ### 分支命名规范

**master 分支**： 

- master 为主分支，也是用于部署生产环境的分支，确保 master 分支稳定性

- master 分支一般由 develop 以及 hotfix 分支合并，任何时间都不能直接修改代码

 **feature 分支**： （feature/*）

- 开发新功能时，以 develop 为基础创建 feature 分支

- 分支命名：feature/ 开头的为特性分支；命名规则：feature/user_module、 feature/cart_module

**hotfix 分支**： （**hotfix/\***）

- 分支命名：hotfix/ 开头的为修复分支，它的命名规则与 feature 分支类似

- 线上出现紧急问题时，需要及时修复，以 master 分支为基线，创建 hotfix 分支，修复完成后，需要合并到 master 分支和 develop 分支

1. ### 添加 Merge Request Templates

- .gitlab/merge_request_templates/

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=MzE3NDJiYjcxZDM0YWIyNWRjYmVjZjc0MTM4NmVhNWZfZERhaHp1YnhqMk9VN2ltTExjSnYxcktydzJPNkJTM0RfVG9rZW46Ym94Y251M241NWlYamVKRk1WYld3NVlranpmXzE2NjM3MzIxMTU6MTY2MzczNTcxNV9WNA)

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjA3MTc0OGUwNjc2Yjc3OTgyYTI4OWQ1NGVkZjBmZjVfTE5udWFIbVVGUElOUzVaTFhlYktUQjg4Z3MwRHBFeFVfVG9rZW46Ym94Y256aVpQMmdLS3hMbXZKWWJ3cm1oNzdiXzE2NjM3MzIxMTU6MTY2MzczNTcxNV9WNA)

1. ### CR 流程

- Code Review 合并条件
  - 至少一条评论
  - 至少两人点击MR通过（可建立文档收纳当天MR，指派对应MR人员）
  - discussions 未解决时，无法点击 Merge 按钮

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=MWFlMjkzNTYxMTVmYzY0NTA1M2ZmYTE3OGVjMzkxYjRfanlMRXFMdURLQ2JLSUZmcEVwb3NSTlRaZU1abkpyVlFfVG9rZW46Ym94Y25pNWl5c1c4OGRQVFIzM2ZCUnM3U1FjXzE2NjM3MzIxMTU6MTY2MzczNTcxNV9WNA)

1. ### 控制台赋能 - 版本号、项目信息输出

- 用途：方便定位线上系统版本 - 查找问题

方案：

**TODO：清理项目线上控制台输出（防止打印敏感信息）**

方案一：使用 uglifyjs-webpack-plugin / terser-webpack-plugin 插件

```JavaScript
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
```

方案二：webpack - 自定义loader

```JavaScript
// source：表示当前要处理的内容
const reg = /(console.log()(.*)())/g
module.exports = function(source) {
  // 通过正则表达式将当前处理内容中的console替换为空字符串
  source = source.replace(reg, "")
  // 再把处理好的内容return出去，坚守输入输出都是字符串的原则，并可达到链式调用的目的供下一个loader处理
  return source
}
```

1. ### 拆包

- [CRM业务组件包&工具包做资源聚合cdn化](https://wrpnn3mat2.feishu.cn/docx/doxcn6OOiNkp7VoiXamLHlcGQWc) 

拆包问题记录：

1. 拆element：私包对element有非标准化模块引用（图片放大器等...），需要单独处理

```JavaScript
# externals
config
  .set('externals', {
    "vue": 'Vue',
    "element-ui": "ELEMENT",
  })
  
     
<script src="https://cdn.bootcdn.net/ajax/libs/vue/2.6.11/vue.runtime.min.js"></script>
<script src="https://unpkg.com/element-ui@2.10.1/lib/index.js"></script>
      
```

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=NzliYTEzYWI3ZTk5ODQyZDhmZDNjOTMzMTk3NDcyZDBfYVRsc0VDdWpYbUREYXp6TmtWcWdncUcxZHFNV3g2QmlfVG9rZW46Ym94Y25vSE5zNWJReU56c1oyR1FvSXI0ODhiXzE2NjM3MzIxMTU6MTY2MzczNTcxNV9WNA)

## 三、改造课导系统

- 课导分支MR：https://git.corp.hetao101.com/CRM/crm-web-tutor-system-v2/merge_requests/704/diffs