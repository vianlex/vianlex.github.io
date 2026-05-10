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

3.1 基础语法

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
