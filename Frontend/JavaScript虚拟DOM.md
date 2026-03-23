# 虚拟 DOM 

所谓的虚拟 DOM，其实就是通过 JS 对象去描述 DOM 节点，如以下示例：

```js
// <div id="hello">Hello <b>World</b></div>
{
    tag: 'div',
    attrs:{
        id: "hello"
    },
    children: [
        "Hello ",
        {
            tag: "b",
            children:[
                "World"
            ]
        }
    ]
}
```

## 虚拟 DOM 的作用

1. 性能优化：通过在内存中操作虚拟 DOM，而不是直接操作真实 DOM 减少浏览器的重绘（repaint）和回流（reflow），减少直接 DOM 操作，通过虚拟 DOM 只对必要的变更进行真实 DOM 更新。

2. 快速比较：对于一些框架，当状态变化时，框架会生成一个新的虚拟 DOM 树，并与旧的虚拟DOM树进行比较（也称为Diffing算法），只在内存中进行，不需要与浏览器交互，从而达到快速且高效。

3. 异步更新：虚拟 DOM 使得批量和异步的 DOM 更新成为可能，框架可以收集多个状态变更并在单个重渲染中一次性更新 DOM，而不是每个状态变更都触发 DOM 更新。

4. 状态管理：虚拟DOM使得状态管理更加直观，因为状态的变化可以直接映射到UI的变化。


## 虚拟 DOM 的简单示例

1. 创建虚拟 DOM 

```js
// 定义 VNode 虚拟节点对象类型
function VNode(tag, text , attrs={}, children=[]){
    // 节点的标签
    this.tag = tag
    // 节点的属性
    this.attrs = attrs
    // 当前节点的子节点，是一个数组
    this.children = children
    // 节点的文本
    this.text = text
}

// 创建文本节点
function createTextVNode(text){
    return  new VNode("string", text)
}

// 创建元素节点
function createElementVNode(tag, attrs={}, children=[]){
    return new VNode(tag, null, attrs, children)
}

```

2. 虚拟 DOM 渲染成真实 DOM 

```js
// 将虚拟 DOM 渲染成真实 DOM
function render(vNode){
    if("string" === vNode.tag){
        return document.createTextNode(vNode.text)
    }
    const element = document.createElement(vNode.tag)
    const children = vNode.children
    if(!children || children.length == 0){
        return element
    }
    children.forEach(child => {
        if (child.tag === 'string') {
            element.appendChild(document.createTextNode(child.text));
        } else {
            element.appendChild(render(child));
        }
    })
    return element
}
```

3. 挂载虚拟 DOM 

```js

// 挂载点
const container = document.getElementById('app');
// 创建虚拟节点
const appNode = createElementVNode("div")
const childNode1 = createTextVNode("Hello ")
const childNode2 = createElementVNode("b",{},[createTextVNode("World")])
appNode.children.push(childNode1, childNode2)
// 将虚拟节点渲染真实 DOM
const realDom =  render(appNode)
container.appendChild(realDOM);

```




## 在线 playground

1. https://astexplorer.net/#/1CHlCXc4n4