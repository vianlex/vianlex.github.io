# Promise 知识点

## 一、Promise 是什么

**Promise 是 ES6 引入的异步编程方案**，用来处理异步操作（网络请求、定时器、文件读写等），解决回调地狱（Callback Hell），让异步代码像 “同步” 一样链式书写、清晰易读。
一句话：**它是一个 “承诺”，承诺未来某个时刻会给出成功结果或失败原因。**

## 二、Promise 的三种状态

一个 Promise 只能处于以下互斥三态之一，且状态一旦改变就不可逆：
- pending（等待中）：初始态，异步未完成
- fulfilled（已成功）：异步成功，有结果
- rejected（已失败）：异步失败，有错误

状态流转：
- pending → fulfilled（只能一次）
- pending → rejected（只能一次）
- fulfilled/rejected 不能再变

## 三、基础用法

### 3.1 基础语法

```bash
const p = new Promise((resolve, reject) => {
  setTimeout(() => {
    // 模拟成功、失败
    const ok = true; 
    if (ok) {
      resolve("数据加载成功");
    } else {
      reject(new Error("网络错误"));
    }
  }, 1000);
});
```

Promise 构造函数的参数必须是一个函数，该参数函数必须有两个参数分别是 resolve、reject 形参，初始化 Promise 对象时，会将 Promise 类内部定义的 resolve 函数
和 reject 函数，分别赋值给参数函数中的 resolve 和 reject 形参。


### 3.2 核心实例方法

#### then(onFulfilled, onRejected)

- onFulfilled 成功回调函数（必选），onRejected 失败回调函数（可选）
- 返回新 Promise，支持链式调用

```js
new Promise((resolve,reject)=>{
  // todo
}).then(
  (res) => { console.log("成功：", res); },
  (err) => { console.error("失败：", err); }
);
```

#### catch(onRejected)

- 专门注册失败回调，等价于 .then(null, onRejected)
- 链式中任何环节出错，都会跳过后续 then，直接进入 catch

```js
new Promise((resolve, reject)=>{
  // todo 
})
.then(res => { throw new Error("中间出错");  })
.then(res=>{console.log("第一个 then 出错后，不会再执行该 then, 直接进入 catch。")})
.catch(err => { console.error("统一错误处理：", err); });
```

#### finally(onFinally)

- 无论成功 / 失败都会执行（如隐藏 loading、清理状态）
- 不接收参数，返回新 Promise

```js
new Promise((resolve, reject)=>{
  //todo 
}).then(res => { console.log("成功：", res) })
 .catch(err => { console.log("出错：", err) })
 .finally(() => { console.log("操作结束"); });
```

### 3.3 Promise 静态方法

#### Promise.resolve(value)

- 创建一个已成功状态的 Promise 对象。

```js
Promise.resolve("成功").then(res => console.log(res)); 
```

#### Promise.reject(reason) 

- 快速创建一个已失败的 Promise

```js
Promise.reject(new Error("失败")).catch(err => console.error(err)); // 失败
```

### Promise.all(promises[])

- 并行执行所有 Promise，全部成功才成功，返回结果数组（顺序对应）
- 任意一个失败 → 立即 reject（短路）
- 注意和 `Promise.allSettled` 的区别

```js
const p1 = Promise.resolve(1);
const p2 = new Promise(resolve => setTimeout(() => resolve(2), 1000));
const p3 = Promise.resolve(3);

Promise.all([p1, p2, p3]).then(results => {
  console.log(results); // 1 秒后，输出 [1,2,3]
}).catch(err => console.error(err));
```

#### Promise.allSettled(promises[])

- 并行执行，等待所有完成（无论成功 / 失败），返回结果对象数组
- 每个对象：{ status: 'fulfilled', value } 或 { status: 'rejected', reason }
- 与 `Promise.all` 的区别是 `Promise.all` 中所有 promises 成功才执行 then，只有一个 promise 执行失败立即执行 catch

```js
const p1 = Promise.resolve(1);
const p2 = Promise.reject(new Error("错误"));

Promise.allSettled([p1, p2])
.then(results => {
  console.log(results);
  // [
  //   { status: 'fulfilled', value: 1 },
  //   { status: 'rejected', reason: Error: 错误 }
  // ]
});
```


#### Promise.race(promises[])

- 并行执行，谁先完成就用谁的结果（无论成功 / 失败）

```js
const p1 = new Promise(resolve => setTimeout(() => resolve("快"), 500));
const p2 = new Promise(resolve => setTimeout(() => resolve("慢"), 1000));

Promise.race([p1, p2]).then(res => console.log(res)); // 快（500ms后）
```

#### Promise.any(promises[])（ES2021）

- 并行执行，第一个成功的结果，全部失败才 reject（AggregateError）

```js
const p1 = Promise.reject(new Error("错1"));
const p2 = Promise.resolve("成功");
const p3 = Promise.reject(new Error("错2"));

Promise.any([p1, p2, p3]).then(res => console.log(res)); // 成功
```

## 四、手写 Promise（理解原理）

```js
class SimplePromise {
    // 构造函数的 executor 参数必须是一个函数 
    constructor(executor) {
      // Promise 默认状态是待处理
      this.state = "pending"
      // 定义 Promise 的默认值为 undefined
      this.value = undefined;
      // 成功的回调队列，存放 then 链式函数中回调函数
      this.onFulfilledCallbacks = []  
      // 失败的回调队列，存放 catch 链式函数中的回调函数
      this.onRejectedCallbacks = []   
      // 定义 resolve 函数, 该函数接收一个 value 值
      const resolve = (value)=>{
        // 将 Promise 状态改成功
        this.state = "fulfilled"
        // Promise 的成功值 
        this.value = value 
        // 然后执行 then 链式回调，默认放入微任务队列中，这里使用 setTimeout 实现
        this.onFulfilledCallbacks.foreach(cb => setTimeout(()=>cb(value),0))
      }
      // 定义内部的 reject 函数
      const reject = (error)=>{
        this.state = "rejected"
        this.value = error
        this.onRejectedCallbacks.foreach(cb => setTimeout(()=> cb(error),0))
      }
      // 构造函数，真正的初始化逻辑
      try {
        executor(resolve, reject)
      } catch (err) {
        reject(err)
      }
    }
    // 定义 then 链式函数，注意要返回 Promise 对象，才能实现链式操作 .then(res=>{}).then(res=>{})
    then(onFulfilled, onRejected) {
      // 防止传空参数
      onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
      onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason }
      return new SimplePromise((resolve, reject)=>{
        try{
          // 如果当前的 Promise 是待处理状态
          if(this.state === "pending"){
            this.onFulfilledCallbacks.push(onFulfilled)
            this.onRejectedCallbacks.push(onRejected)
          }else if(this.state === "fulfilled"){
            const res = onFulfilled(this.value);
            resolve(res)
          }else if(this.state === "rejected") {
            const res = onRejected(this.value)
            reject(res)
          }
        }catch(error){
          reject(error)
        }
      })
    }
    catch(onRejected) {
       return this.then(null, onRejected);
    }
    // 定义静态 resolve 方法，直接返回一个 fulfilled 状态的 Promise
    static resolve(value) {
      return new SimplePromise((resolve) => {
        resolve(value)
      })
    }
} 

```

## Promise + async/await

async/await 是 Promise 的语法糖，本质还是 Promise，只是把链式写法改成同步写法。

### async 关键字

使用 async 关键字，可以将函数声明为异步函数，自动返回 Promise。

1. aysnc 修饰没有返回值的函数

```js 
// async 修饰的普通函数，没有显示指定返回值，则默认返回一个已成功值为 undefined 的 Promise 对象 
async function hello() {
  console.log("Hello World")
}
// 执行函数后，会默认返回一个 fulfilled 已成功的 Promise 对象，Promise 的 value 默认为 undefined 
hello()
```

2. async 修饰有返回值的函数

```js
async function p1() {
  return "HelloWorld"
}

// 执行函数后，返回一个已成功的值为 helloWorld 的 Promise 对象。
p1()


async function p2() { 
  return setTimeout(()=> console.log("HelloWorld"), 5000)
}
// 执行后函数后，返回一个已成功的值 Promise 对象
timer()

async function p3() { 
  return new Promise((resolve, reject)=>{
    setTimeout(()=> resolve("HelloWorld"), 5000)
  })
}
// 注意 p3 和 p4 是等价的
function p4() { 
  return new Promise((resolve, reject)=>{
    setTimeout(()=> resolve("HelloWorld"), 5000)
  })
}
// 执行函数后，会一个待处理状态的 Promise 对象。
p3()
```

### await 关键字

await 关键字，用于等待 Promise 完成，只能在 async 函数内使用，注意 await 不能等待普通函数。

```js



```











## 六、常见面试点总结

1. 三态 + 不可逆：pending → fulfilled/rejected，一旦改变不能回退。
2. 链式调用原理：.then 返回新 Promise，实现链式。
3. 错误冒泡：链式错误会传到最近的 .catch。
4. Promise.all/race/allSettled/any 区别：
  - all：全成才成，一败即败
  - race：谁先完用谁
  - allSettled：全完成，返回所有结果
  - any：第一个成功，全败才败
  5. async/await 是语法糖：基于 Promise，用 try/catch 处理错误。
