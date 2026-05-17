# 一.React基础
`index.js`
```js
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
    <App />
);
```
将`index.html`中id为root的标签挂载到root上。
根组件`App.js`：
具体流程是：`App` -> `index.js` ->  `public/index.html(root)`。
# 二.JSX
## 1.JSX相关基础语法
**JSX中使用JS表达式**
	在JSX中可以使用 **大括号语法{}** 识别JS中的表达式，比如常见变量，函数调用，方法调用。

**JSX中进行循环遍历**
	注意，这里使用`map`时必须绑定一个`key`（React内部标识字段，用于提升渲染性能）。
```js
const list = [
  {id: 1, name: 'Vue'},
  {id: 2, name: 'React'},
  {id: 3, name: 'HTML'},
  {id: 4, name: 'CSS'},
  {id: 5, name: 'JavaScript'},
  {id: 6, name: 'TypeScript'},
  {id: 7, name: 'Node.js'},
]
function App() {
  return (
    <div className="App">
      <ul>
        {list.map((item) => <li key = {item.id}>{item.name}</li>)}
      </ul>
    </div>
  );
}
```

**JSX中使用条件渲染**
	可以通过 逻辑与 或 三元表达式 来使用条件渲染。

## 2.React基础使用
**React事件绑定**
```js
function App() {
  const clickHandler = () => {
    console.log('按钮被点击了');
  }
  return (
    <div className="App">
      <button onClick={clickHandler}>点击此按钮</button>
    </div>
  );
}
```
