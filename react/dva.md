## dva 是什么

dva 是体验技术部开发的 React 应用框架，将上面三个 React 工具库包装在一起，简化了 API，让开发 React 应用更加方便和快捷。[dva](https://dvajs.com/guide/introduce-class.html#%E7%9B%AE%E5%89%8D%E6%9C%80%E6%B5%81%E8%A1%8C%E7%9A%84%E6%95%B0%E6%8D%AE%E6%B5%81%E6%96%B9%E6%A1%88)

dva = React-Router + Redux + Redux-saga

## dva 应用的最简结构

```js
import dva from 'dva';
const App = () => <div>Hello dva</div>;

// 创建应用
const app = dva();
// 注册视图
app.router(() => <App />);
// 启动应用
app.start('#root');
```

