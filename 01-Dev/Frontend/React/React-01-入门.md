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
`JSX`是一种结合了`JavaScript`和`XML`的语法。
## 1.JSX相关基础语法
**JSX中使用JS表达式**
	在JSX中可以使用 **大括号语法{}** 识别JS中的表达式，比如常见变量，函数调用，方法调用。

**JSX中进行循环遍历**
	注意，这里使用`map`时**必须绑定**一个`key`（React内部标识字段，用于提升渲染性能）。
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
# 三.组件
一个组件就是用户界面的一部分，它可以由自己的逻辑和外观，多个组件可以互相嵌套，也可以使用多次。
组件内部存放了**逻辑**和**视图**。
```js
//函数组件
function Button() {
  return <button>点击此按钮</button>
}
//箭头函数组件
const Button = () => {
  return <button>点击此按钮</button>
}

function App() {
  return (
    <div className="App">
      <Button />
      <Button></Button>
    </div>
  );
}
```

## 1.useState状态变量
### 1.1.基本使用
状态变量可以用来**同步显示**某些数值，它发生了变化之后，页面会立刻发生同步渲染。
`const [count, setCount] = useState(初始值)`，`count`为我们设置的状态变量，`setCount`为设置`count`的函数，基本使用如下：
```js
import{useState} from 'react';
function App(){
  const [count, setCount] = useState(0);
  function clickHandler(){
    setCount(count + 1);
  }
  return (
    <div>
      <button onClick={clickHandler}>{count}</button>
    </div>
  )
}
export default App;
```
注意，只能使用其**回调函数**`setCount()`来设置新值，单纯的`count++`不可以进行赋值。

### 1.2.修改对象状态
对于对象的修改，使用**展开运算符**`...`然后选择要修改的属性进行修改，如下：
```js
import{useState} from 'react';
function App(){
  const [person, setPerson] = useState({name: 'Fish', age: 20});
  function clickHandler(){
    setPerson({...person, 
      name: 'Duck',
      age: person.age + 1});
  }
  return (
    <div>
      <button onClick={clickHandler}>
        {person.name} is {person.age} years old
      </button>
    </div>
  )
}
export default App;
```

## 2.组件基础样式
React组件基础的样式控制有两种方式：行内样式（不推荐），class类名控制
行内样式（不推荐）：
```js
const style = {
  color: 'red',
  fontSize: '20px'
}
<div style={{color : 'red'}}> this is div</div>
```
class类名控制：
`index.css`
```css
.foo{
    color: red;
    font-size: 20px;
}
```
`App.js`
```js
import './index.css'

function App() {
  return (
    <div>
      <span className='foo'> this is span </span>
    </div>
  )
}
```

## 3.受控表单绑定
将输入框与`useState`状态变量进行双向绑定
```js
import{useState} from 'react';
import './index.css';

function App() {
  const [person, setPerson] = useState({ name: 'Fish', age: 20 });

  function handleChange(e) {
    const { name, value } = e.target;//解构运算，name为标签里面的name属性，value为对应的文本
    setPerson(prev => ({
      ...prev,
      [name]: name === 'age' ? Number(value) : value//name属性可能对应着'name'或'age'
    }));
  }

  return (
    <div>
      <div>
        <label>
          Name:
          <input
            type="text"
            name="name"
            value={person.name}
            onChange={handleChange}
            className='foo'
          />
        </label>
      </div>
      <div>
        <label>
          Age:
          <input
            type="number"
            name="age"
            value={person.age}
            onChange={handleChange}
            className='foo'
          />
        </label>
      </div>
      <div>
        {person.name} is {person.age} years old
      </div>
    </div>
  );
}

export default App;
```

## 4.React中获取DOM
在`React`组件中获取/操作DOM，需要使用`useRef`钩子函数，分为两步：
1. 使用`useRef`创建`ref`对象，与`JSX`绑定。
```js
const inputRef = useRef(null)
<input type='text' ref={inputRef} />
```
2. 在DOM可用时通过`inputRef.current`拿到DOM对象
```js
function handleClick() {
  alert(inputRef.current.value);
}
```
