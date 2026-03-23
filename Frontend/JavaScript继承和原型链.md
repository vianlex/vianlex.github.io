# JavaScript 继承和原型链

JavaScript 对象继承是通过原型链的方式实现的，每一个对象都有一个私有原型属性，除了 null 之外。原型属性也是对象也有自己的一个原型属性，层层往上直到原型属性为 null 继承才终止，因为 null 没有原型属性。

在 ECMAScript 标准规范中，用 `someObject.[[Prototype]]` 符号标识 `someObject` 的原型，但是现代浏览器不并支持 ES 标准规范属性  `someObject.[[Prototype]]`  方式访问对象原型，而是通过非标准的 `someObject.__proto__` 属性访问器来实现，即在浏览器只能使用 `someObject.__proto__` 属性访问器或者使用 ` Object.getPrototypeOf(someObject) ` 和 ` Object.setPrototypeOf(someObject) `  函数，修改查看对象的原型属性。

JavaScript 对象的属性和方法是动态的，当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。

在 JavaScript 中除了 null 和 undefined(表示变量未赋值) 的一切对象，都是通过显式或者隐式的构造函数方法实例化创建的。并且除了 null 和 undefined 类型外，所有的基本数据类型和引用类型，在 JavaScript 中表现为函数，如 Number、String、Boolean、BigInt、Object 等，所以在 JavaScript 中任何函数都是引用类型，故都可以通过 new 关键字方式调用创建实例对象。

在浏览器中可以通过 `__proto__ ` 属性或者构造函数的 prototype 属性去指定或者修改原型属性值。

```javaScript

// 字面量对象，通过 __proto__ 属性，显式指定和修改对象的原型
var obj = {
    'a': "a1",
    __proto__ : {
        'a' : 'a2',
        'b' : 'b2'
        __proto__ : {
          "c" : "c3"
        }
    }
}
/**
 * 对象访问属性时，先查找对象本身是否有该属性，有则直接返回，没有则继续查找对象的原型对象是否有该属性，有则返回，
 * 没有则，继续查找原型的原型是否有该属性，一直查找到原型链的末端。
 */
console.log(obj.a) // 输出 'a1'，
console.log(obj.b) // 输出 'b2'
console.log(obj.c) // 输出 'c3'

// 字面量函数对象
var obj2 = {
  "a" : "aa",
   __proto__: {
    "b": "bb",
    __proto__: {
      "c": "cc",
      __proto__: null
    }
   }
}
/** 显示指定对象的末端原型时，无法通过 __proto__ 访问原型，只能通过 Object.getPrototypeOf 函数访问 */
console.log(obj2.__proto__) // 输出 undefined
console.log(Object.getPrototypeOf(obj2)) // 输出 {b: 'bb'}
console.log(Object.getPrototypeOf(Object.getPrototypeOf(obj2))) // 输出 {c: 'cc'}

// 变量对象
var num = 12
// num 的构造方方法为 JavaScript 内置的 Number 对象
console.log(num.constructor == Number)
/** 输出 true，num 对象获取属性时，先查找自身是否由该属性，有则直接返回，没有则继续查找它的原型是否有该属性，一直查找到原型链的末尾，
  故，num.constructor 返回的是 num 对象原型的 constructor 属性值。
*/
console.log(num.constructor == num.__proto__.constructor)

// 在内置 Number 对象的 prototype 属性对象上新增 hello 属性
Number.prototype.hello = 'hello Number'
// nn 实际上是隐式调用 Number 内置对象（构造方法）生成的实例对象
var nn = 12
// 输出 'hello Number'，通过构造函数创建对象，会将构造函数的 prototype 属性设置为新生成实例对象的原型属性。
console.log(nn.hello)


```


## 构造函数

在 JavaScript 中，能用 new 关键字来调用的函数，称为构造函数（构造函数首字母一般建议使用大写）。

注意：并不是所有的函数都是构造函数，如箭头函数 和 `Function.prototype` 等 JS 内置的函数并非构造函数。

在 JavaScript 中使用 new 关键字调用构造函数，创建对象时，实际是执行以下操作：

- 分配内存并创建一个空对象
- 设置新对象原型，即将新创建对象的 [[Prototype]] 原型属性对象指向构造函数的 prototype 属性对象
- 绑定 this 关键字，将构造函数内部的 this 绑定到新创建的对象上
- 执行构造函数中的代码（通常是用于初始化新对象的属性）
- 返回初始化创建好的新对象


```JavaScript
// 普通函数
function func() {}
// 当通过 new 关键字调用 func 函数，则称 func 为构造函数
var obj = new func()

// 输出 'object' 表示 f1 是一个对象
typeof obj 
// 输出 true 表示 obj 是 func 类型的实例对象
obj instanceof func

```

在 JavaScript 中，所有的函数都有一个名为 prototype 的属性，当函数作为构造函数创建实例对象时，JavaScript 会将函数的 prototype 属性设置为新对象的 [[prototype]] 属性。

注意，函数的 prototype 属性对象默认包含有一个 Constructor 属性，Constructor 属性的值指向构造函数本身。

```javaScript

// 字面量对象，隐式构造函数为 Object 
var obj = {"name" : "Object"}
// 输出 true 说明 obj 对象是由 Object 构造函数创建的实例对象，并且 obj 的原型属性指向 Object 构造函数的 prototype 属性
console.log(obj.__proto__ == Object.prototype)


// 字面量函数对象，隐式构造函数为 Function 
function func() {
  console.log("Hello World Func")
}
// 输出 true，说明 func 对象是 Function 构造函数创建的实例对象，并且 func 对象的原型指向 Function 对象的 prototype  
console.log(func.__proto__ == Function.prototype)
// 输出 true，说明 Function.prototype 函数对象的构造函数是 Object 
console.log(Function.prototype.__proto__ == Object.prototype)
/** 获取 Function.prototype 对象的 constructor 属性，如果 Function.prototype 当前对象没有 constructor 属性，则继续查找它的原型以及原型的原型属性是否有该属性  */
console.log(Function.prototype.constructor  == Object.constructor)  // 输出 true
console.log(Function.prototype.__proto__.constructor  == Object ) // 输出 true
console.log(Function.prototype.constructor  == Function) // 输出 true


// 使用 func 作为构造函数去创建实例对象，注意函数的 prototype 属性对象是可以编辑修改，通过该特性，则可以实现个性化继承
func.prototype.hello = "Hello World"
func.prototype.sayHello = function() {console.log("Hello World!")}
// 通过 func 构建函数创建对象
var obj2 = new func()
var obj3 = new func()
// 两个对象，都会输出 'Hello World'
console.log(obj2.hello)
console.log(obj3.hello)

/** 函数的 prototype 属性对象的 constructor 属性指向构造函数本身，
    所以查看通过构造函数创建的实例对象原型时，可以查看到它的构建函数是那个函数（即那个引用数据类型）
**/
console.log(func.prototype.constructor == func) // 输出 true
console.log(obj2.constructor == func) // 输出 true
console.log(obj2.__proto__.constructor == func) // 输出 true


```

从上面的例子可以看出，函数在 JavaScript 也是一种引用数据类型，类似其他语言的类。像 JavaScript 的内置对象 Function , Object，Number 本质上也都是构造函数。


## 原型链说明

Object 构造函数 prototype 属性的 [[prototype]](原型属性) 指向 null，Object 构造函数本身的原型属性指向的是 Function.prototype

```javaScript

// 输出 null，故说明 Object.prototype 对象的 [[prototype]] 属性即 __proto__ 属性指向的是 null 
console.log(Object.prototype.__proto__)

// 普通字面量对象，也是隐式调用 Object 构造函数创建的
var obj = {'Hello' : 'World'}
// 输出 true 表示 obj 对象是隐式调用 Object 构造函数创建的对象 
console.log(obj.__proto__.constructor == Object)

// 输出 null, 可以看出 obj 对象的原型指向 Object.prototype 对象,  Object.prototype 对象的原型指向 null
// 故通过 Object 构造函数创建的实例对象原型继承链为: obj -> Object.prototype -> null 
console.log(obj.__proto__.__proto__)

/** 由以下输出可看出 Object 构造函数是 Function 构造函数的实例对象，即 Object 对象的原型对象指向的 Function 构造函数的 prototype 属性，
  Function.prototype 函数对象的原型指向 Object.prototype, Object.prototype 的原型指向 null，
  故 Object 对象的原型继承链为：Object -> Function.prototype -> Object.prototype -> null
*/
console.log(Object.__proto__ ==  Function.prototype ) // 输出 true
console.log(Function.prototype.__proto__ == Object.prototype) // 输出 true
console.log(Object.prototype.__proto__ == null) // 输出 true

```

Function 构造函数 prototype 属性和 [[prototype]] 原型属性，指向的是同一个对象。

```javaScript

// 输出 true，表示 Function 构造函数的 prototype 属性和 [[prototype]] 属性指向的都是同一个对象
console.log(Function.prototype == Function.__proto__)
// 字面量 func 函数对象，实际上也是隐式通过 Function 构造函数创建的
function func(){ console.log("Hello World") }
// 输出 true，说明 func 函数对象的构造函数是 Function，则 func 对象的原型指向的 Function 构造函数的 prototype 属性
console.log(func.__proto__.constructor == Function )
/** 输出 true，则说明 Function.prototype 属性对象的构造方法是 Object，
  那么 Function.prototype 对象的 [[prototype]] 原型属性指向的 Object 构造函数的 prototype 属性
**/
console.log(Function.prototype.__proto__.constructor == Object)
/** 输出 true，可以看出 func 对象是由 Function 构造函数创建的实例对象，
  则 func 的原型继承链为：func 的原型指向 Function.prototype， Function.prototype 的原型指向 Object.prototype，
  Object.prototype 的原型指向 null，即 func -> Function.prototype -> Object.prototype -> null 
**/
console.log(func.__proto__ == Function.prototype )
// 输出 null
console.log(func.__proto__.__proto__.__proto__)

/** 当使用 func 作为构造函数创建对象实例时，创建出来的对象的原型继承链为：
  ff -> func.prototype -> Object.prototype -> null
 **/
var ff = new func()
// 输出 true，func.prototype 对象的构造函数为 Object，故它的原型指向的 Object.prototype 
console.log(func.prototype.__proto__.constructor == Object)

```

## [[prototype]] 和 __proto__ 区别

[[prototype]] 和 __proto__ 标识符表示的都是对象的原型属性，[[prototype]] 是 ECMAScript 标准规范中定义的原型标识，__proto__ 是现代浏览器实现的访问对象原型属性的访问器。注意函数中的 prototype 属性不是函数的原型属性。


## 参考文档
1. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain