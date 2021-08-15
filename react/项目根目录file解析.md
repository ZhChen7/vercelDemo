# 使用ESLint+Prettier规范React+Typescript项目

## 一、概述

**[EditorConfig](https://link.zhihu.com/?target=https%3A//editorconfig.org/)**：不同编辑器和IDE之间定义和维护一致的代码风格；`.editorconfig`

**[ESlint](https://link.zhihu.com/?target=https%3A//github.com/eslint/eslint)**：代码检查工具； `.eslintrc.js`

**[prettier](https://link.zhihu.com/?target=https%3A//prettier.io/docs/en/)**：一个 Opinionated 的代码格式化工具； `.prettierrc`

**[husky](https://link.zhihu.com/?target=https%3A//github.com/typicode/husky/tree/master)**：一个让配置 git hooks 变得更简单的工具； `.huskyrc`

**[lint-staged](https://link.zhihu.com/?target=https%3A//github.com/okonet/lint-staged)**：只对 git 中变更的文件进行 lint 操作；`.lintstagedrc`

## 二、详解

**2.1 EditorConfig**

- **官网：[EditorConfig](https://link.zhihu.com/?target=https%3A//editorconfig.org/)**
- **参考：[【译】EditorConfig 介绍 | AlloyTeam](https://link.zhihu.com/?target=http%3A//www.alloyteam.com/2014/12/editor-config/)**

`EditorConfig` 可以帮助开发者在不同的编辑器和 IDE 之间定义和维护一致的代码风格。`EditorConfig` 包含一个用于定义代码格式的文件和一批编辑器插件，这些插件可以让编辑器读取配置文件并依此格式化代码。`EditorConfig` 的配置文件十分易读，并且可以很好的在 VCS（`Version Control System`）下工作。

.editorconfig

```js
# http://editorconfig.org
root = true

[*]
charset = utf-8
end_of_line = lf
indent_size = 2
indent_style = space
insert_final_newline = true
max_line_length = 80
trim_trailing_whitespace = true

[*.md]
max_line_length = 0
```



**2.2 ESLint**

- ***github：\*[eslint/eslint](https://link.zhihu.com/?target=https%3A//github.com/eslint/eslint)**
- ***参考：[深入理解 ESlint](https://link.zhihu.com/?target=https%3A//juejin.im/post/6844903901292920846)\***

ESLint 是一个Javascript Linter，帮助我们规范代码质量，提高团队开发效率。

![img](https://pic1.zhimg.com/80/v2-cda7dd631f85be0ec0cdd882f42c80c8_1440w.jpg)

1. **[避免代码错误](https://link.zhihu.com/?target=https%3A//eslint.org/docs/rules/%23possible-errors)**
2. **[写出最佳实践的代码](https://link.zhihu.com/?target=https%3A//eslint.org/docs/rules/%23best-practices)**
3. **[规范变量使用方式](https://link.zhihu.com/?target=https%3A//eslint.org/docs/rules/%23variables)**
4. **[规范代码格式](https://link.zhihu.com/?target=https%3A//eslint.org/docs/rules/%23stylistic-issues)**
5. **[更好的使用新的语法](https://link.zhihu.com/?target=https%3A//eslint.org/docs/rules/%23ecmascript-6)**

社区比较知名的代码规范，eslint 配合这些代码规范，能够检测出代码潜在问题，从而提高代码质量。

- [standardjs](https://link.zhihu.com/?target=https%3A//standardjs.com/readme-zhcn.html)
- [airbnb](https://link.zhihu.com/?target=https%3A//github.com/airbnb/javascript)

ESLint 是完全插件化的，每一个规则都是一个插件并且你可以在运行时添加更多的规则。

.eslintrc.js

```js
module.exports = {
  env: {
    browser: true,
    es6: true,
    node: true,
  },
  parser: "babel-eslint", // 解析器
  extends: [], // 扩展
  plugins: [], // 插件
  globals: {
    Atomics: "readonly",
    SharedArrayBuffer: "readonly",
  },
  parserOptions: {
    ecmaVersion: 2018,
    sourceType: "module",
  },
  rules: {},
};
```



**2.3 Prettier**

- ***Github：[prettier/eslint-plugin-prettier](https://link.zhihu.com/?target=https%3A//github.com/prettier/eslint-plugin-prettier)\***
- ***参考：[陈龙：Prettier看这一篇就行了](https://zhuanlan.zhihu.com/p/81764012)\***

eslint 虽然能帮助我们提高代码质量，但并不能完全统一编码风格，因为这些代码规范的重点并不在代码风格上，虽然有一定的限制。

**prettier 是一个能够统一团队编码风格的工具**，能够极大的提高团队执行效率，统一的编码风格能很好的保证代码的可读性。

.prettierrc

```js
{
  "quotes": true,
  "semi": true,
  "tabWidth": 2
}
```



## 2.4 husky

- ***参考：[Git - Git 钩子](https://link.zhihu.com/?target=https%3A//git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)\***
- ***Github：[typicode/husky](https://link.zhihu.com/?target=https%3A//github.com/typicode/husky)\***

**git hook**

Git 能在特定的重要动作发生时触发自定义脚本。

为了防止一些不规范的代码`commit` 并`push`到远端，我们可以在`git`命令执行前用一些钩子来检测并阻止。

![img](https://pic3.zhimg.com/80/v2-821425e428848527be2c3e3eaccc1cb6_1440w.jpg).git/hooks/*

上图为各种钩子的案例脚本，可以把sample去掉，直接编写shell脚本来执行（可以参考git 官方 ）。git hooks 这种方式需要把脚本放在 `.git/hooks`目录下面，团队合作会比较麻烦，需要有工具自动把脚本安装进目录。husky就是为了解决这个问题。

```
husky` 是一个 npm 包，安装后可以很方便的在 `package.json` 配置git hook，也可以创建独立的 `.huskyrc
```

原理：`husky`会根据 package.json里的配置，在`.git/hooks`目录生成[所有](https://link.zhihu.com/?target=https%3A//git-scm.com/docs/githooks)的 hook 脚本（如果你已经自定义了一个hook脚本，`husky`不会覆盖它）

.huskyrc

```js
{
  "hooks": {
    "pre-commit": "lint-staged"
  }
}
```

## 2.5 lint-staged

- **GitHub：[okonet/lint-staged](https://link.zhihu.com/?target=https%3A//github.com/okonet/lint-staged)**
- **参考：[lint-staged 使用教程](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/jiaoshou/p/12250278.html)**
- **参考：[https://segmentfault.com/a/1190000009546913](https://link.zhihu.com/?target=https%3A//segmentfault.com/a/1190000009546913)**

`lint-staged` 针对暂存的 git 文件运行 linters，不要让不符合规则的代码溜进代码库。`lint-staged`总是将 **所有暂存文件的列表传递给任务**，忽略任何文件都应该在任务本身中配置，比如：`.prettierignore` / `.eslintignore` 。lint-stage 总是配合 husky一起使用。

.lintstagedrc

```js
{
  "src/**/*.js": [
    "eslint --fix",
    "prettier --write",
    "git add"
  ]
}
```

## 三、React + TypeScript 项目中该怎么选择 Linting？

最近在做 react + typescript 项目过程中，想通过合适的工具对代码进行规范以及统一编码规则。很自然的想到了 `eslint` 、`tslint`

### 3.1 TSLint

TSLint 对TypeScript 支持得很好，并且如果你使用的是 VsCode IDE，还有出色的插件支持。

可能有人会有疑问：JavaScript 语言非常灵活，所以需要相应的代码检测，而TypeScript 有强大的 **类型系统** 和 **对ES6+** 的支持。 在编译过程中已经可以检测出很多问题。为什么还需要？

因为TypeScript 关注的是类型的匹配，而不是代码风格。

比如：

```js
// 1.缩进应该是四个空格还是两个空格？
// 2.是否应该禁用 var？
// 3.接口名是否应该以 I 开头？
// 4.是否应该强制使用 === 而不是 ==？
```

TypeScript 不会关注上面的问题，但这些问题能够影响到团队协作效率、代码的可理解、可维护性。

例子：

```js
let myName = 'Tom';
console.log(`My name is ${myNane}`);
console.log(`My name is ${myName.toStrng()}`);
console.log(`My name is ${myName}`)
```

![img](https://pic4.zhimg.com/80/v2-f997638ab5f911aae114ffecf25f1ef3_1440w.jpg)

![img](https://pic4.zhimg.com/80/v2-ccb91ce0628923c9d4dd99208d3f68b7_1440w.jpg)



### **3.2 应该使用哪种代码检查工具？**

TSLint 的优点：

- 专为 TypeScript 服务，bug 比 ESLint 少
- 不受限于 ESLint 使用的语法树 [ESTree](https://link.zhihu.com/?target=https%3A//github.com/estree/estree)
- 能直接通过 `tsconfig.json` 中的配置编译整个项目，使得在一个文件中的类型定义能够联动到其他文件中的代码检查

ESLint 的优点：

- 基础规则比 TSLint 多很多
- 社区繁荣，插件众多

![img](https://pic2.zhimg.com/80/v2-e7a7f09842e8332c71c162451b9a6ad9_1440w.jpg)



### 3.3 BUT

参考：

[TypeScript 官方决定全面采用 ESLintwww.oschina.net![图标](https://pic1.zhimg.com/v2-6b2745affbe649932fbceb1a75b519e4_180x120.jpg)](https://link.zhihu.com/?target=https%3A//www.oschina.net/news/103818/future-typescript-eslint)

由于性能问题，[TypeScript 官方决定全面采用 ESLint](https://link.zhihu.com/?target=https%3A//www.ithome.com.tw/news/128382)，甚至把仓库（Repository）作为测试平台，而 ESLint 的 TypeScript 解析器也成为独立项目，专注解决双方兼容性问题。

JavaScript 代码检测工具 ESLint 在 TypeScript 团队发布全面采用 ESLint 之后，发布[typescript-eslint](https://link.zhihu.com/?target=https%3A//github.com/typescript-eslint/typescript-eslint) 项目，以集中解决TypeScript 与 ESLint兼容性问题。而ESLint不再维护typescript-eslint-parser，也不会在npm 上做任何发布。TypeScript 解析器转移至Github 的 typescript-eslint/parser。

[typescript-eslint/typescript-eslintgithub.com![图标](https://pic3.zhimg.com/v2-bf9c8e384d804e905bb3404b8c5496fa_ipico.jpg)](https://link.zhihu.com/?target=https%3A//github.com/typescript-eslint/typescript-eslint)[TypeScript 2019上半年发展规划github.com](https://link.zhihu.com/?target=https%3A//github.com/Microsoft/TypeScript/issues/29288)

### 3.4 SO

虽然TSLint 很长一段时间 是Linting TypeScript 的标准，但ESLint会很快完全取代 TSLint，TSLint将被抛弃。



## 四、React + TypeScript 项目 Linting 搭建

### 4.1 使用`npx`创建项目

```js
$ npx create-react-app eslint-react-intro --typescript
```

说明：[npx 参考](https://link.zhihu.com/?target=https%3A//medium.com/%40maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b)

### 4.2 安装 ESLint 解析 TypeScript 的依赖

1. `eslint`：javascript代码检测工具，使用[espree](https://link.zhihu.com/?target=https%3A//github.com/eslint/espree)解析器
2. `@typescript-eslint/parser`：将 TypeScript 转换为 ESTree，使 eslint 可以识别
3. `@typescript-eslint/eslint-plugin`：只是一个可以打开或关闭的规则列表

```js
$ yarn add eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin -D
```

### 4.3 配置 ESLint

**1.使用eslint 交互式工具配置**

```js
$ ESLint --init
```

它会问你一系列问题，以帮助你配置它。

可以参考：

[https://larrylu.blog/improve-code-quality-using-eslint-742cf1f384f1larrylu.blog](https://link.zhihu.com/?target=https%3A//larrylu.blog/improve-code-quality-using-eslint-742cf1f384f1)

**2.自定义配置，添加配置文件 `.eslintrc.js`**

```js
module.exports = {
  parser: {},  // 解析器
  extends: [], // 继承的规则 [扩展]
  plugins: [], // 插件
  rules: {}    // 规则
};
```

**plugin 与 extend 的区别：**

- extend 提供的是 eslint 现有规则的一系列预设
- 而 plugin 则提供了除预设之外的自定义规则，当你在 eslint 的规则里找不到合适的的时候就可以借用插件来实现了

为了使配置能够正常运行，我们需要配置 解析器、插件、规则集等。

```js
module.exports = {
  parser: "@typescript-eslint/parser",
  extends: ["plugin:@typescript-eslint/recommended"],
  plugins: ["@typescript-eslint"],
  rules: {}
};
```

我们已经告诉ESLint 怎样正确解析 TypeScript 代码，并且使用了一组推荐的插件规则（`extends`字段选项中的配置）。

**3. 接下来我们将为 React 添加基本规则**

此规则，由`Create React App`团队提供

```js
module.exports = {
  parser: "@typescript-eslint/parser",
  extends: ["plugin:@typescript-eslint/recommended", "react-app"],
  plugins: ["@typescript-eslint", "react"],
  rules: {}
};
```

我们对 TypeScript 和 React 进行了 规范(linting)，此时需要选择一种代码格式化程序。前面提到的Prettier 将是首选工具，因为它在检测和修复样式错误方面做的很出色，并且和ESLint有很好的集成。

**4. 安装 prettier 依赖**

```js
$ yarn add prettier eslint-config-prettier eslint-plugin-prettier -D
```

1. `prettier`: 格式化规则程序
2. `eslint-config-prettier`: 禁用所有和 Prettier 产生冲突的规则
3. `eslint-plugin-prettier`: 把 Prettier 应用到 Eslint，配合 rules "prettier/prettier": "error" 实现 Eslint 提醒。

```js
module.exports = {
  parser: '@typescript-eslint/parser',
  extends: [
    'plugin:@typescript-eslint/recommended',
    'react-app',
    'plugin:prettier/recommended',
  ],
  plugins: ['@typescript-eslint', 'react'],
  rules: {},
};
```



**5. Visual Studio Code 集成 ESLint 与 Prettier**

![img](https://pic3.zhimg.com/80/v2-ce7c603e5750cca37053c9629eaf755a_1440w.jpg)

![img](https://pic3.zhimg.com/80/v2-62712f572a76075a25ba8195f6f203de_1440w.jpg)

最后，虽然默认配置（`setting.json`）已经很好，但默认只检测**`.js`和** `*.jsx`文件，我们还需要配置`typescripe`，同时我们也希望有自动修复功能（有些是不可能自动修复）。

**user settings** 与 **workspace settings：**user settings 里面是更通用的设置，workspace settings 是跟随项目存在，可以做到团队统一。

![img](https://pic4.zhimg.com/80/v2-e8cd3a98bc701de462ead3a80a1f5247_1440w.jpg)

```json
"eslint.validate": [
    "javascript",
    "javascriptreact",
    {
      "language": "typescript",
      "autoFix": true
    },
    {
      "language": "typescriptreact",
      "autoFix": true
    }
]
```

**6.使用 husky，提交检测（可选）**

[https://github.com/typicode/huskygithub.com](https://link.zhihu.com/?target=https%3A//github.com/typicode/husky)

结合自己团队实际情况，是否选择，建议使用。

```js
$ yarn add husky -D
```



```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "npm test",
      "pre-push": "npm test",
      "...": "..."
    }
  }
}
```





## 五、总结

ESList：@typescript-eslint项目取代 tslint

TSLint：逐渐被舍弃

Prettier：统一代码风格

VSCode：在`setting.json`中设置，可以集成 ESLint、Prettier 等规则，并能自动修复。

------

[Linting Your React+Typescript Project with ESLint and Prettier!medium.com](https://link.zhihu.com/?target=https%3A//medium.com/%40dors718/linting-your-react-typescript-project-with-eslint-and-prettier-2423170c3d42)[自然醒：使用ESLint+Prettier来统一前端代码风格zhuanlan.zhihu.com![图标](https://pic4.zhimg.com/v2-3832770c55dc52cdcfe9430da2bd2eeb_180x120.jpg)](https://zhuanlan.zhihu.com/p/38267286)[https://standardjs.com/readme-zhcn.htmlstandardjs.com](https://link.zhihu.com/?target=https%3A//standardjs.com/readme-zhcn.html)[https://larrylu.blog/improve-code-quality-using-eslint-742cf1f384f1larrylu.blog](https://link.zhihu.com/?target=https%3A//larrylu.blog/improve-code-quality-using-eslint-742cf1f384f1)[TypeScript 官方决定全面采用 ESLintwww.oschina.net![图标](https://pic1.zhimg.com/v2-6b2745affbe649932fbceb1a75b519e4_180x120.jpg)](https://link.zhihu.com/?target=https%3A//www.oschina.net/news/103818/future-typescript-eslint)[https://medium.com/@oliver.grack/using-eslint-with-typescript-and-react-hooks-and-vscode-c583a18f0c75medium.com](https://link.zhihu.com/?target=https%3A//medium.com/%40oliver.grack/using-eslint-with-typescript-and-react-hooks-and-vscode-c583a18f0c75)