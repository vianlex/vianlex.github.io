# React 基础知识

## JSX 基础语法

JSX 是 JavaScript 语法扩展，可以让你在 JavaScript 文件中书写类似 HTML 的标签。

1. JSX 嵌入表达式

在 JSX 中，你可以使用大括号 {} 嵌入任何有效的 JavaScript 表达式

```jsx

const name = 'React' 
const Welcome = <h1> Hello {name}! </h1>

const items = [<li> Item1 </li>, <li> Item2 </li>, <li> Item3 </li>]
const List = <ui>{items}</ui>

const numbers = [1, 2, 3, 4, 5]
const element = <ul>{numbers.map((number) =><li key={number.toString()}>{number}</li>)}</ul>;

const isLoginIn = true
const element = <div> {isLoginIn ? <h1>Welcome!</h1> : <h1>Please Sign In!</h1>} </div>

const user = {name: 'React'}
let getName = function(user){ return user.name}
const element =  <h1> Hello {getName(user)} </h1>

```

2. JSX 属性和样式表达式

JSX 的属性，必须使用驼峰式命名法，不能使用 `-` 符号。

注意：元素的 class 在 JSX 中要写成 className，因为 class 是 JavaScript 的保留字。

```jsx

let class2 = 'main'
const element = <div className={class2}> Content </div>

const props = { id: "main", className: "box" }
const element = <div {...props}> Spread Example </div>

let style= {width: '100%'}
const element = <a src={style}> Hello World </a>

// 如果是字面量对象需要双 {} 符号
const element = <a src={{width: '100%'}}> Hello World </a>

```

3. JSX 注释

```jsx

const d1 = <div>{/** 多行注释*/}</div>

const d2 = <div>
  {/** 
    多行注释
    多行注释*/}
</div>

const d3 =  <div> 
  {
    // 单行注释必须这么写
  } 
</div>

```

4. 多行 JSX 

当 JSX 代码较长时，可以使用括号 () 将代码包裹起来，以保持代码的可读性

```jsx

const element = <div><h1>Hello, world!</h1><p>This is a paragraph.</p></div>

// 分成多行写时，必须使用 ()
const element = (
  <div>
    <h1>Hello, world!</h1>
    <p>This is a paragraph.</p>
  </div>
)

```

5. 单一的根元素

JSX 元素组件，必须是单一的根元素，当有多个元素时，必须将它们包裹在一个元素中。

```jsx

// 必须返回单一元素
const element = <div><h1>Hello, world!</h1><p>This is a paragraph.</p></div>

// 通过 React 提供了 React.Fragment 包裹多个元素时，实际渲染结果不会有多余的外层容器
const element = <React.Fragment> <h1>Hello, world!</h1><p>This is a paragraph.</p> </React.Fragment>

// 支持空标签
const element = <><h1>Hello, world!</h1><p>This is a paragraph.</p></>

```


## React 元素和组件

### React 元素

React 元素一旦创建，就不能修改其属性或子元素，React 元素是普通的 JavaScript 对象，不包含复杂的逻辑。

####  创建 React 元素 

1. 通过 JSX 创建元素

```jsx

const element = <h1> Hello World! </h1>

// 自定义的组件，也可以一个元素
const element = <Welcome name='Hello React'/>

// 判断一个变量是否 JSX 元素，可以通过 React 内置的 React.isValidElement(object) 方法来实现
const flag =  React.isValidElement(<h1> Hello World </h1>) // 返回 True
const flag =  React.isValidElement(element) // 返回 True
const flag = React.isValidElement("Hello World") // 返回 Flase

```

2. 通过 React.createElement() 创建元素

```js
// 第一个参数是创建元素的标签，第二参数是元素的属性，第三个参数元素的内容（文本或者子元素）
const element = React.createElement('h1', null, 'Hello, World!');

// 返回的是一个 json 对象
const element = React.createElement('h1',{className: 'greeting'},'Hello, world!')

// React.createElement 本质上是创建一个对象，所以可以通过直接编写对象的方式来实现等价的效果
const element = { type: 'h1',props: {className: 'greeting', children: 'Hello, world!'}}

```

### React 组件

React 是可重用的代码块，用于封装UI逻辑和结构，组件包含多个元素，React 支持函数组件和类组件，它们的主要区别如下：

- 函数组件：没有生命周期方法和无法使用状态，但可以通过 useEffect Hook 来模拟生命周期行为和通过 useState Hook 来管理状态。
- 类组件：可以使用 componentDidMount、componentDidUpdate 和 componentWillUnmount 等生命周期方法和通过 this.state、this.setState 来管理状态。

#### 创建 React 组件

注意：通过函数创建组件时，必须引入 React 对象 `import React from 'react' `, 因为 JSX 本质上只是一种语法糖，最终需要转化成 React.createElement 来创建组件。 

1. 通过函数方式创建 React 组件

当 React 渲染 Welcome 组件时，React 会将 Welcome 组件的属性，封装成一个对象，传递给组件函数的 props 参数，
从而达到将数据从父组件传递到子组件中，使组件可以根据 Props 渲染出不同的内容。

Props 的默认值：是通过给函数设置 defaultProps 属性来实现的。

注意：Props 是只读的，在组件内部不能修改。

```jsx

function Welcome(props) {
    return <h1>Hello, {props.name}!</h1>;
}
// 设置 Welcome 组件的默认属性
Welcome.defaultProps = {name: 'React'}
// 属性校验，通过 propTypes 设置组件的 name 属性必须传递的
Welcome.propTypes = {
  name: PropTypes.string.isRequired
};


let Greet = function(props){
    return <h1>Hello, {props.name}!</h1>;
}

// 可将 Greeting 文本组件
let Greeting = function(props) {
    // 非 JSX 元素不能使用 {} 嵌入式表达式
    return "Hello " + props.name
}

```

2. 通过类方式创建 React 组件

通过类方式创建 React 组件时，必须继承 React.Component 并且实现 render 方法，React.Component 中默认定义有 this.props 和 this.state 

```js
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}

```

## state 组件状态

State 是 React 组件中的一个对象，用于存储组件内部的数据。与 Props（属性）不同，State 是组件私有的，完全由组件自身管理。当 State 发生变化时，React 会自动重新渲染组件，以反映最新的数据状态。

注意：State 是 React 组件内部的可变数据，而 Props 是从父组件传递下来的不可变数据。

### 函数组件的 state 

在函数组件中，State 通常通过 useState 钩子（Hook）来管理。useState 是 React 提供的一个内置钩子，用于在函数组件中添加 State。

注意：由于 State 更新是异步的，更新 State 后状态不会立即刷新，如果更新状态后，立即访问可能得到的还是旧值，同时 React 会将多个 State 更新合并，以提高性能。

```js

import React, { useState } from 'react';

function InputForm() {

  /**
   * inputValue 表示当前的状态值
   * setInputValue 是更新状态的函数，当通过 setInputValue 会更新 inputValue 的值，React 会自动重新渲染组件
   * 注意：State 的更新是异步的，当调用 setInputValue 函数后，inputValue 的值不会立即更新，立即访问 inputValue 可能会得到旧值。
   */
  const [inputValue, setInputValue] = useState('');

  const handleChange = (event) => {
    setInputValue(event.target.value);
  };

  return (
    <div>
      <input type="text" value={inputValue} onChange={handleChange} />
      <p>You typed: {inputValue}</p>
    </div>
  );
}


```

### 类组件的 state 

1. 通过构成方法或者类属性初始化 State

```js

// 通过构造方法初始化 state
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
      user: {name:'John', age: 18},
      items: []
    };
  }
}

// 通过类型属性方法初始化 state（需要 Babel 支持）
class MyComponent extends React.Component {
  state = {
    count: 0,
    user: {name:'John', age: 18},
    items: []
  };
}


```

2. 更新 state 

更新 state 必须使用 this.setState() 方法来更新 state，直接修改 this.state 不会触发重新渲染。

setState 函数说明：`setState(Object|function(必须返回对象))`

setState() 是异步的，React 可能会批量执行多个 setState() 调用以提高性能。

setState 是异步的，setState 属性后，立即访问 this.state 属性可能得到的旧值，
我们可以通过 useEffect 监听状态变化来解决该问题， 通过 useEffect Hook 函数能保证我们获取到的数据是最新的。

注意：setState() 更新值是浅合并，setState() 只会将你提供的对象到当前 state，因此我们更新数组对象或者对象时，先合并旧值，再更新。

```jsx
// 错误的写法，不会触发组件重新渲染，无法达到双向绑定的效果
this.state.count = 1

// 正确的写法
this.setState({count : 2}) 
this.setState({count : this.state.count + 2})
// 更新前获取旧值
this.setState(oldState =>{
    // 打印旧的值
    console.log(oldState.count == this.state.count)
    let newValue = this.state.count + 1
    // 返回要更新的值
    return {count: newValue}
})

// 更新对象，注意要返回新的实例对象
let newUser = {this.state.user, ...{age: 20}}
this.setState({user: newUser})
this.setState(oldState=>{
    let newUser = {oldState.user, ...{age: 20}}
    return {user: newUser}
})


//使用回调函数，更新状态时，通过回调函数能保证我们获取到的数据是最新的。
this.setState({count:2}, ()=>{
    console.log(this.state.count ) 
})

```

### State 与 Props 的区别

State 和 Props 都是 React 中用于管理数据的机制，但它们有本质的区别：

- State：组件内部的可变数据，由组件自身管理。
- Props：从父组件传递下来的不可变数据，用于组件之间的通信。


## React 组件生命周期

在 React 中，组件的生命周期是指组件从创建、更新到销毁的整个过程。

### 生命周期方法

React 组件的生命周期可以分为三个阶段：

挂载阶段（Mounting）：组件被创建并插入到 DOM 中。
更新阶段（Updating）：组件的状态或属性发生变化，导致组件重新渲染。
卸载阶段（Unmounting）：组件从 DOM 中移除。

### 类组件生命周期

1. 挂载阶段 (Mounting)

- `constructor()` - 初始化 state 和绑定方法
- `static getDerivedStateFromProps()` - 在渲染前根据 props 计算 state
- `render()` - 渲染组件
- `componentDidMount()` - 组件挂载完成后调用，适合执行副作用(如 API 调用)

2. 更新阶段 (Updating)

- `static getDerivedStateFromProps()` - props 或 state 变化时调用
- `shouldComponentUpdate()` - 决定组件是否需要重新渲染
- `render()` - 重新渲染组件
- `getSnapshotBeforeUpdate()` - 在 DOM 更新前捕获一些信息
- `componentDidUpdate()` - 组件更新后调用，适合执行基于DOM的操作

3. 卸载阶段 (Unmounting)

- `componentWillUnmount()` - 组件卸载前调用，用于清理(如取消订阅、定时器)
- `static getDerivedStateFromError()` - 捕获后代组件抛出的错误
- `componentDidCatch()` - 捕获后代组件抛出的错误并记录错误信息


### 函数组件生命周期

函数组件本身没有生命周期方法，但可以通过Hooks实现类似功能。

1. 挂载阶段

```jsx

useEffect(() => {
  // 相当于 componentDidMount
  console.log('组件挂载')
  return () => {
    // 相当于 componentWillUnmount
    console.log('组件卸载')
  };
}, []); // 空依赖数组，表示组件只在挂载和卸载时执行

```

2. 更新阶段

```jsx

useEffect(() => {
  // 相当于 componentDidUpdate
  console.log('状态或props变化')
}) // 没有依赖数组，表示组件每次渲染都执行

// 特定状态变化时执行
useEffect(() => {
  console.log('count变化:', count)
}, [count]) // 表示只有组件 count 状态变化时，才执行

```

3. 其他生命周期功能

```jsx

// 相当于 getDerivedStateFromProps
function MyComponent({ propValue }) {
  const [state, setState] = useState(propValue);
  
  useEffect(() => {
    setState(propValue)
  }, [propValue])
}

// 相当于 shouldComponentUpdate
const MyComponent = React.memo(function MyComponent(props) {
  /* 只在props变化时重新渲染 */
})

```


## 组件通信方式


### Props 方式，父传到子

Props 是 React 中最基本的组件通信方式。通过 props 可以将父组件数据传递给子组件。

```jsx

// 函数组件的参数，就是组件调用时，设置的属性集合
function Welcome1(props) {
    return <div> {props.name} age is {props.age} </div>
}

// 函数组件的参数，就是组件调用时，设置的属性集合
function Welcome2({name}) {
    return <div>{name} age is {age}</div>
}

// 父级组件，使用子组件
<Welcome1 name="Hello World" age="20"/>

let user = {name: "Hello World", age: 20}
<Welcome2 name={user.name} age={user.age}/>

```

### 事件回调传递数据，子传到父

子组件可以通过事件回调，可以将数据传递回父组件。

```jsx

// 子组件
function ChildComponent({ onClick }) {
  const handleClick = () => {
    onClick("Hello from Child!");
  };

  return <button onClick={handleClick}>Click Me</button>;
}


// 父组件
function ParentComponent() {
  const handleChildClick = (data) => {
    console.log("Data from child:", data);
  };

  return <ChildComponent onClick={handleChildClick} />;
}

```

### 兄弟组件通信

兄弟组件通信，通过共同的父组件（状态提升）

```jsx

function Parent() {
  const [sharedData, setSharedData] = useState("");
  
  return (
    <>
      <SiblingA onUpdate={setSharedData} />
      <SiblingB data={sharedData} />
    </>
  );
}

```


### Context 跨层级通信

Context 跨层级通信，Context 可以在组件树中跨层级传递数据的方式，避免使用 Props 层层传递的。

1. 示例

```jsx
import React, {useContext} from 'react'

// 创建 Context，注意需要父和子组件都能同时访问到 MyContext 变量
const MyContext = React.createContext();

// 父级组件
function App() {
  return (
    <MyContext.Provider value={{ data: "全局数据" }}>
      <ChildComponent />
    </MyContext.Provider>
  );
}

// 子组件
function ChildComponent() {
  const context = useContext(MyContext);
  return <div>{context.data}</div>;
}


```

2. 示例

```jsx

import React, {useContext, useState} from 'react'

// 创建Context
const RootContext = React.createContext();

// 父组件
function RootElement({children}) {
    const [value, setValue] = useState('Hello World')
    return (
        <RootContext.Provider value={{value, setValue}}>
            {children}
        </RootContext.Provider>
    )
}

// 子组件
function childElement(){
    const {value, setValue} = useContext(RootContext)
    return (
        <>
            <h1> value </h1>
            <button onClick={() => setValue('Hello React')}> Update Value </button> 
        </>
    )
}

// 组件
function App() {
    return (
        <RootElement> 
            <childElement/>
        </RootElement>
    )
}

```

### Refs 访问子组件

```jsx

// 父组件
function Parent() {
  const childRef = useRef()
  const handleClick = () => {
    childRef.current.doSomething();
  }
  return (
    <>
      <button onClick={handleClick}>调用子组件方法</button>
      <Child ref={childRef} />
    </>
  )
}
// 子组件 (需要使用 forwardRef)
const Child = forwardRef((props, ref) => {
  useImperativeHandle(ref, () => ({
    doSomething: () => {
      console.log("子组件方法被调用")
    }
  }));
  
  return <div>子组件</div>
})

```

### 任意组件通信

通过事件发布订阅的方式实现任意组件通信


```jsx

// eventBus.js
const events = {}

export default {
  on(event, callback) {
    if (!events[event]) events[event] = []
    events[event].push(callback);
  },
  emit(event, data) {
    if (events[event]) {
      events[event].forEach(callback => callback(data))
    }
  }
};

// 组件A - 发布
eventBus.emit("someEvent", { data: "任意数据" })

// 组件B - 订阅
useEffect(() => {
  eventBus.on("someEvent", handleEvent)
  return () => eventBus.off("someEvent", handleEvent)
}, [])

```


## React 实现插槽

1. 第一种方式，通过组件属性的方式实现

```jsx

// 可以使用 props 也可以使用解构写法
function Welcome({render}) {
        const [user, setUser] useState({name: 'React'})
    return <div>{render(user)}</div>
}

<Welcome render={({name})=>{<h1>{name}</h1>}}/>


function Welcome(props) {
    return props.hello
  }

<div><Welcome hello={<h1>Hello World</h1>}></Welcome></div>
// 返回的是文本，文本也是组件
<div><Welcome hello="<h1>Hello World</h1>"></Welcome></div>


```

2. 第二种方式，子元素的方式实现

在 React 中，每一个组件都有一个特殊的 children 属性，通过 children 属性，我们可以获取当前组件下的所有子元素。

```jsx

function Greeting({children}) {
    const [user, setUser] useState({name: 'React'})
     // 子元素可以是函数，调用后，返回组件元素
    return children(user)
}

<Greeting>
    {({name}) => <h1>{name}</h1> }
</Greeting>

function Greeting({children}) {
    // 子元素传递 map
    return (<div> 
        {children.header("React")}
        {children.footer}
    </div>)
}

<Greeting> 
    {{
        header: (name) => <h1> Greeting {name} </h1>
        footer: <h2> Footer </h2>
    }}
</Greeting>


function Welcome(props) {
    // 子元素是组件时，直接返回组件元素
    return props.children
}

<Welcome><h1>Hello World</h1></Welcome>
// 文本，也是子元素
<Welcome>Hello World</Welcome>


function Welcome(props) {

    let children = props.children
    let type =  typeof children
 
    if(type == 'function') { // 判断是否函数

    }else if(React.createElement(children)) {  // 判断是否 jsx
       
    }else {  // 否则当作文本组件处理
       

    }

}


```

## 组件懒加载

React 通过内置的 Suspense 组件和 React.lazy 函数来实现组件的懒加载

Suspense 组件提供的 fallback 属性，是一个插槽属性，用于展示加载过程。

```jsx

// 使用 React.lazy 动态导入组件
const LazyComponent = React.lazy(() => import('./LazyComponent'));

export default function App() {
    return (
        <div>
            <h1> 懒加载示例 </h1>
            <Suspense fallback={<div> 组件加载中... </div>}>
                <LazyComponent />
            </Suspense>
    </div>
    )
}

```


## React.memo

当组件的 props 或 state 发生变化时，React 会重新渲染该组件及其子组件。但是有时候父级组件重新渲染时，我们并不希望也重新渲染子组件，
则可以通过 React.memo 防止子组件自行不必要的重新渲染。

如果一个组件是  React.memo 类型的组件， React.memo 会对组件的 props 进行浅比较（shallow compare），只有当 props 发生变化时才会重新渲染组件。

#### 基本语法

```jsx
// 通过普通函数定义 React.memo 组件
const MyComponent1 = React.memo(function MyComponent(props) {
  /* 使用 props 渲染 */
})

// 箭头函数定义  React.memo 组件
const MyComponent2 = React.memo((props) => {
  /* 使用 props 渲染 */
})

// 通过自定义函数，判断是否渲染组件
const MyComponent3 = React.memo(
  function MyComponent(props) {
    /* 使用 props 渲染 */
  },
  (prevProps, nextProps) => {
    /* 返回 true 表示跳过渲染，false 表示需要渲染 */
    return prevProps.value === nextProps.value;
  }
)

```

#### 使用列子

```jsx
import React, { useState } from 'react'

// 只要父组件重新渲染，子组件就也会重新渲染
const Child1 =({ value }) => {
  console.log('Child1 渲染了')
  return <div>Child1 =  {value}</div>
}
// value 改变时，子组件才会重新渲染
const Child2 = React.memo(({ value }) => {
  console.log('Child2 渲染了')
  return <div>Child2 = {value}</div>
})

export default function Parent() {
  const [count, setCount] = useState(0)
  const [text, setText] = useState('hello')
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>计数: {count}</button>
      <button onClick={()=> setText(preText => preText.indexOf('H') != -1 ? 'hello' : 'Hello' )}>渲染</button>
      <Child1 value={text} />
      <Child2 value={text} />
    </div>
  )
}

```


## Portals

在 React 中，组件通常按父子组件的层级结构渲染成 DOM 树解构的，当我们想将某个组件渲染到 DOM 树中的其他位置时，可以通过 React 提供的 Portals
功能来实现。

```jsx

import React from 'react'
import ReactDOM from 'react-dom'

function Modal({ children }) {
  return ReactDOM.createPortal(
    <div className="modal">
      {children}
    </div>,
    document.body // 将模态框渲染到 body 中
  );
}

export default function Parent() {

  const handleClick = () => {
    console.log('父组件捕获到点击事件')
  }
  // 注意：Portal 中的事件会按照 React 树而非 DOM 树冒泡
  return (
    <!-- Modal 最终不是渲染在 Parent 的 DOM 子树中，而是在 Body 的 DOM 树中 -->
    <div className="Parent">
      <p>点击按钮时，事件会冒泡到这里</p>
      <Modal>
        <button>点击我</button>
      </Modal>
    </div>
  )
}

```


## 常用 Hooks 说明

### useState 

useState 是一个函数，它接收一个初始状态值作为参数，并返回一个包含两个元素的数组：一个元素是当前状态值，另一个元素是用于更新该状态的函数。

#### 基本语法

```jsx
//  setState 是异步的，通过 setState 更新状态时，立即获取 state 
const [state, setState] = useState(initialState)
// 通过 useEffect 监听状态变化
useEffect(()=> console.log(state), [state])
```
- initialState: 状态的初始值
- state: 当前的状态值
- setState: 用于更新状态的函数，注意如果 state 是一个对象，通过 setState 更新时，必须返回新增的对象实例

#### 使用例子

```jsx

export default function UserForm() {
  const [user, setUser] = useState({
    name: '',
    age: 0,
    email: ''
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setUser(prevUser => ({
      ...prevUser,
      [name]: value
    }));
  };
 // 
 const [count, setCount] = useState(0);
 const increment = () => setCount(prevCount => prevCount + 1);
 // 数组状态
 const [todos, setTodos] = useState([]);

  return (
    <form>
      <input name="name" value={user.name} onChange={handleChange} />
      <input name="age" value={user.age} onChange={handleChange} />
      <input name="email" value={user.email} onChange={handleChange} />
    </form>
    <button onClick={increment}>{count}</button>;
  );
}

```

### useEffect 

useEffect 用于在函数组件中执行副作用操作（副作用指：如数据获取、订阅、手动修改 DOM 等）它可以看作是 componentDidMount、componentDidUpdate 和 componentWillUnmount 的组合。

#### 基本语法

```jsx
import React, { useEffect } from 'react';

useEffect(() => {
  // 副作用代码
  return () => {
    // 清理函数 (可选)
  };
}, [dependencies]); // 依赖数组 (可选)
```
- 第一个参数：是一个函数，用于执行副作用代码
- 第二个参数：是一个依赖数组，用于控制副作用的执行时机
- 返回值（可选的）：返回值是一个函数，组件卸载时，该函数会被运行。

#### 使用例子

```jsx

// 省略依赖组时，表示只要组件重新渲染，都会执行
useEffect(() => {
  console.log('组件每次渲染都会执行')
})


// 类似 componentDidMount 和 componentWillUnmount
useEffect(() => {
  // 组件挂载时执行代码
  console.log('组件已挂载')
  // 组件卸载时，执行 return 的函数
  return ()=> {
    console.log('组件卸载了')
  }
}, []); // 空依赖数组，表示组件挂载和卸载时各执行一次

// 依赖更新时执行 (类似 componentDidUpdate)
const [count, setCount] = useState(0);
useEffect(() => {
  console.log(`count 已更新为: ${count}`);
  // 每次count变化时都会执行
}, [count]); // 指定依赖项

const [count, setCount] = useState(0);
useEffect(() => {
  console.log(`count 已更新为: ${count}，我是第二个 count 的 useEffect`);
  // 每次count变化时都会执行
}, [count]); // 指定依赖项

```

### useContext

useContext 是 React Hooks 中用于跨组件层级共享数据的 Hook，它可以让你在组件树中轻松访问上下文(Context)值，避免了繁琐的 props 逐层传递。

注意：Provider 的 value 变化会导致所有消费者组件重新渲染

#### 使用例子

```jsx

import React, { createContext, useContext,  useState} from 'react'

// 创建全局的 Context 对象，需要父组件和子组件都能访问到
const FirstContext = createContext()
const SecondContext = createContext()

// 子组件
function ChildComponent() {
  // 通过 useContext 获取 Context 对象的 value 值
  const context1 = useContext(FirstContext)
  const context2 = useContext(SecondContext)
  return <div>{context1}{context2.name}</div>
}

// 父级组件
export default function App() {
  // 状态
  const [value, setValue] = useState('Hello ')
  const value2 = { name: "React", version: "18" }
  // 同时使用多个 Context
  return (
    <FirstContext.Provider value={value}>
      <SecondContext.Provider value={value2}>
        <ChildComponent />
      </SecondContext.Provider>
    </FirstContext.Provider>
  )
}

```

### useReducer  

useReducer 类似 useState，它也是一个状态函数，主要用于管理组件中的复杂状态逻辑

#### 基本语法

```jsx
const [state, dispatch] = useReducer(reducer, initialState, initFunction)
```

- state：当前的状态
- initialState：初始值状态
- initFunction: 初始化函数，如果存在初始化函数，则使用 initialState 作为 initFunction 函数的第一个参数，并且将 initFunction 函数返回结果作为初始值，不存在 initFunction 函数，则默认使用 initialState 作为初始值
- dispatch：是一个函数，用于触发 reducer  状态更新函数
- reducer： 状态更新逻辑处理函数，该函数有两个参数，第一个参数是当前状态，第二个参数用于接收 dispatch 函数的参数 


#### 使用例子

```jsx

function init(initialCount) {
  return { count: initialCount };
}

function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      // 注意不能直接修改状态对象(state.count = state.count + 1)，修改对象无法触发重新渲染，必须返回新的状态对象
      return { count: state.count + 1 };
    case 'decrement':
      // 注意不能直接修改状态对象(state.count = state.count + 1)，修改对象无法触发重新渲染，必须返回新的状态对象
      return { count: state.count - 1 };
    case 'reset':
      return init(action.payload);
    default:
      throw new Error();
  }
}

export default function Counter({ initialCount = 0 }) {
  const [state, dispatch] = useReducer(counterReducer, initialCount, init);
  
  return (
    <>
      计数: {state.count}
      <button onClick={() => dispatch({ type: 'increment' })}> 增加 </button>
      <button onClick={() => dispatch({ type: 'reset', payload: initialCount })}> 重置 </button>
    </>
  )
}

```

### useCallback

useCallback 用于缓存函数引用，只有 useCallback 依赖项改变时，才会重新生成函数对象，主要作用是避免子组件因接收新函数而触发重渲染。

#### 基础语法

```jsx

const cacheCallback = useCallback(callback, dependencies)
// 等价于
const cacheCallback useMemo(() => callback, dependencies)
```
- callback：想要缓存的函数
- dependencies：依赖数组，当依赖项发生变化时，useCallback 才会返回一个新的函数实例

#### 使用例子

```jsx

import React, {useCallback, useState} from 'react'


let normalList = [], cacheList = []

/** 
 * 注意：React.memo 缓存组件的化，只有组件依赖状态改变时，才会重新渲染组件，
 * 如果不是 React.memo 缓存组件只要父级组件重新渲染，子组件也会跟着重新渲染
 * */ 
// 只有 onClick 引用变化时，children才会重渲染
const Children = React.memo(({ onClick }) => {
   console.log('Children 组件被重新渲染了')
  return <button onClick={onClick}>子按钮</button>
})

export default function Parent(props) {
  console.log('Parent 组件渲染了')
  const [count, setCount] = useState(0)
  const [value, preValue] = useState(0)
  // 普通函数 - Parent 组件每次渲染都会创建新函数实例
  const normalFn = () => {
    preValue(preValue => preValue + 1)
    if(value > 20) {
      setCount(preCount=>preCount + 1)
    }
  }
  // useCallback 缓存函数 - 只有 count 改变时，才会返回新的函数对象实例
  const cacheFn = useCallback(() => {
    console.log('缓存函数输出 count = ' + count)
  }, [count]) // 当前使用 count 作为依赖时，缓存函数内必须要引用该依赖，不然会报错

  if(normalList.length <= 0) {
    normalList[0] = normalFn
  }
  if(cacheList.length <= 0) {
    cacheList[0] = cacheFn
  }
  console.log('Parent 组件每次渲染后 normalFn 函数都生成新的实例：'+(normalList[0] == normalFn))
  console.log('Parent 组件每次渲染后 cacheFn 函数都生成新的实例：'+(cacheList[0] == cacheFn))
  
  return <>
    <button onClick={normalFn}>按钮{value}</button>
    <Children onClick={cacheFn} />
  </>

}

```

### useRef 

在 React 函数组件中，可以通过 useRef 钩子函数，访问 DOM 元素或存储可变值(值改变时，不会触发组件重新渲染)。

#### 基础语法

调用 useRef 钩子函数，会默认返回一个对象，该对象默认有一个 curent 属性，其值默认是 useRef 函数的第一个参数值。

```jsx
// 返回的一个新的可变对象，通过 current 属性去引用初始化对象，如：{current: {name: 'React'}}
const objRef = useRef({name: 'React'}) 
```


#### 通过 useRef 访问 DOM 元素

useRef 最常见的用途是访问 DOM 元素。通过将 useRef 钩子函数返回的对象，传递给 JSX 的 ref 属性，React 会将对应的 DOM 元素赋值给 ref.current。

##### 使用例子

```jsx
import React, { useRef } from 'react'

export default function App() {
  const inputRef = useRef(null)
  const handleClick = () => {
    console.log(inputRef.current)
    console.log(inputRef.current.value)
  }
  return (
    <div>
      <!-- 将输入框 DOM 元素对象赋值给 inputRef 的 current 属性 -->
      <input type="text" ref={inputRef} />
      <button onClick={handleClick}>查看</button>
    </div>
  )
}

```

#### 存储可变值

可变 ref 对象，在组件整个生命周期内保持不变。当 current 属性指向值改变时，组件不会重新渲染。

```jsx
import React, { useRef } from 'react'

export default function Counter() {
  let ref = useRef(0)
  function handleClick() {
    ref.current = ref.current + 1
    alert('当前 current = ' + ref.current)
  }
  return (
    <>
      <span>初始引用值：{ref.current}</span>
      <button onClick={handleClick}>查看当前值</button>
    </>
  )
}

```

### useMemo  

useMemo 是 React Hooks 中用于性能优化的 Hook，它通过缓存计算结果来避免组件在每次渲染时都进行昂贵的计算。

#### 基本语法

```jsx
const memoizedValue = useMemo(() => fn(args), dependencies);
```
- 第一个参数：返回计算结果的函数
- 第二个参数：当 dependencies 发生变化时，会重新计算

#### 使用例子

```jsx

function computeExpensiveValue(a, b) {
  // todo 此处逻辑比较耗时
  return a + b
}
// 组件重新渲染时，不会重新调用 computeExpensiveValue 函数计算，当依赖改变时，才会重新计算
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);

```


### 自定义 Hook

自定义 Hook 本质是一个以 use 开头的普通 JavaScript 函数, 它让开发者能够将组件逻辑提取到可重用的函数中，实现逻辑复用和代码组织优化。

#### 使用例子-1

```jsx
import React, { useState } from 'react'

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue)
  // Counter 中的 count 状态后，Counter 就会组件重新渲染，然后 useCounter 会重复调用
  console.log('Hello UseCounter = ' + count)
  const increment = () => setCount(c => c + 1)
  const decrement = () => setCount(c => c - 1)
  const reset = () => setCount(initialValue)

  return { count, increment, decrement, reset }
}

export default function Counter() {
  const { count, increment } = useCounter(0)
  return (
    <div>
      <span>计算器: {count}</span>
      <button onClick={increment}>增加</button>
    </div>
  );
}
```

#### 使用例子-2

```jsx
// 必须引入 React
import React, { useState, useEffect } from 'react'

function useWindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  })

  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      })
    }
    window.addEventListener('resize', handleResize)
    // return 一个清理函数，组件卸载时，会自动调用
    return () => window.removeEventListener('resize', handleResize)
  }, [])

  return windowSize
}

// 使用示例
export default function ResponsiveComponent() {
  const { width } = useWindowSize()
  const isMobile = width < 768
  
  return <div>{isMobile ? '移动端' : '桌面端'}</div>
}
```



## 参考链接
1. https://www.echo.cool/docs/framework/react/react-basics/react-elements-and-components
2. https://reactplayground.vercel.app/