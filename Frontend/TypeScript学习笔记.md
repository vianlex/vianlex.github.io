# TypeScript 学习笔记

## TypeScript 简介

TypeScript 是带了 “类型检查” 的 JavaScript，其最大的特点就是给**变量、函数、对象**加上类型约束。


## 环境安装与运行

1、环境安装

```bash
npm install -g typescript
```

2、编译运行

```bash
# 将文件由 ts 编译 js
tsc 文件名.tsc 

# 然后使用 node 命令运行 js
node 文件名.js
```

3、快速编译运行

```bash
# 安装 ts-node 命令
npm install -g ts-node

# 运行 ts-node 命令直接编译执行 ts 文件
ts-node 文件名.ts
```

## 变量声明

在 TypeScript 中，可以通过 var、let、const 关键字类声明变量。
- let 用于声明块级作用域的变量，其值可以改变
- const 用于声明块级作用域的变量，变量声明后，其值不能改变
- var 是 JavaScript 中传统的变量声明方式，它没有块级作用域，只有函数作用域和全局作用域。

### 类型约束

```ts
// 
let name: string = "Hello ts"
```

### 类型自动推断

当我们在声明变量时，没有显式指定类型，TypeScript 会根据变量的初始值自动推断出类型。

```ts
// 会自动推断出 name 变量的类型为 string 类型
let name = "Hello World" 
name = 12 // 无法编译通过，因为 name 的类型为 string 类型

// 会自动推断出函数的类型为：(name: string)=>string 
function hello(name :string) {
  return "Hello " + name
}
// 会自动推断出函数类型为：()=>void
function func() {
  console.log("Hello World")
}

// 自动推断出 user 变量的类型为：{name : string; age: number } 
let user = {
  name: "ts"
  age: 20
}

```


## 基础类型

### 基本类型

TypeScript 支持的基本类型，如下：

- string: 字符串类型
- number: 数字类型，包括整数和浮点数
- boolean: 布尔类型
- null 和 undefined: 空值和未定义类型
- any: 任意类型（不推荐使用，因为对 any 类型的变量，TS 不做任何类型检查，any 可以赋值给任何类型，也可以被任何类型赋值，可以访问任意属性、调用任意方法，编译都不会报错）
- void: 表示没有返回值的函数

```ts
// string 字符串类型
let name: string = "TypeScript"

// 数字类型
let age: number = 20

// 布尔类型
let boolean isDone = true

// null 类型
let nl: null = null

// undefined 类型
let ud: undefined = undefined

// any 类型，表示任何类型
let anyVar:any = "Hello"
anyVar = 20
```


### object 对象类型

object 类型表示非原始类型，即非 number、string、boolean、null 和 undefined 等原始类型，但它非常宽泛，不包含任何具体属性信息。

```ts
let obj: object = {  name: "ts"}
// 无法编译通过，因为 object 类型不包含任何具体属性，故 ts 无法访问 name 属性，编译不通过
console.log(obj.name) 
```

### 数组类型

```ts
let nums: number[] = [1,2,3,4,5]
let nums2: Array<number> = [1,2,3,4,5]
  
let strs: string[] = ["Hello", "World"]
```

### 元组类型

```ts
let tuple: [string, number] = ["TypeScript", 20]
```

### 枚举类型

```ts
enum Color {
  Red = "RED"
  Green = "GREEN"
  Blue = "BLUE"
}
```

## 类型断言、类型转换和类型推断以及类型兼容性

### 类型断言 

类型断言表示不改变数据类型，只是告诉编译器，变量是什么类型，让 TS 能编译通过，作用时机是编译时，无运行成本。

```ts
let someValue: any = "this is a string "
// 通过 as 关键字断言类型
let stringValue: string = someValue as string

// 通过 <type> 断言类型
let someValue2: any = "Hello World"
let stringValue2: string = (<string>someValue2)
```

### 类型转换

类型转换的作用时机是运行时，会生成转换逻辑代码,改变变量的数据类型，可能会转换类型失败导致执行异常，有运行时成本。

```ts
// 类型转换
let value2: unknown = "123";
// 将 unknown 类型的值，转换成数字类型
let num2 = Number(value2);     
// 编译后的 JavaScript：let num2 = Number(value2); 
let str2 = String(value2);     
// 编译后的 JavaScript：let str2 = String(value2);
```


### 类型推断

```ts
// 声明变量并初始化时，typescript 会自动推断比变量类型
let value = 3  
value = "Hello World" // 会编译报错，因为 value 的类型是 number 类型

// 声明变量，未赋值时，TypeScript 会自动推断变量类型为 any
let hello;
hello = "Hello World";
hello = 123; // 不会报错

// 自动推断出 user 变量的类型为：{name: string ; age: number }
let user = {name: "Hello", age: 18}
```

### 类型兼容性

在 TypeScript 中，类型兼容性是指一个类型的值是否可以赋值给另一个类型的变量，因为 TypeScript 的类型系统是基于结构子类型的，只要两个类型的结构相同或兼容，它们就可以相互赋值，而不需要显式地声明继承关系。TypeScript 不会强制要求两个类型在名义上完全一致，而是检查它们的结构是否兼容。

#### 基础类型兼容

```ts

type numberOrString = number | string 
let v1: numberOrString = "Hello World"
// 可以编译通过，因为 v1 变量可以是 string 或者 number, 所以它们是兼容的能编译通过
let v2: string = v1

let hello1 : string = "Hello World"
// 无法编译通过，类型不兼容
let hello2 : number = hello1
```

#### 对象类型兼容

对于对象类型的变量，TypeScript 进行类型检查时，只会检查它们的属性结构是否兼容，比如只要 A 类型包含 B 类型的所有成员，TS 就认为 A 兼容 B。

```ts 
interface Person {
  name : string 
  age: number 
}

type User = {
  name: string 
  age : number 
}

let user1: Person = {name : "ts", age: 18}
// 变量 user1 的值可以赋值给变量 user2，因为因为它们的属性相同，则它们是兼容的类型 
let user2: {name: string; age: number } = user1 
// 以下也是类型兼容的，能编译通过
let user3: User = user2 
```

#### 类兼容

类也是看结构，只要结构一样，就算类名不同，也兼容。 

```ts 
class User { name: string }
class Person { name: string }
// 兼容，能编译通过
let u: User = new Person(); 
```

#### 函数兼容

函数兼容主要看参数个数 + 返回值类型是否兼容。需要注意的一点是参数少的兼容参数多的，但是返回值一定要兼容。

```ts
type Fn = (a: number, b: number)=> number 

// 参数少的兼容参数多的，能编译通过
let fn: Fn = function () {
  let num: number = 12;
  // 函数可以不显示指定返回值类型，会自动根据返回值推断出返回值类型
  return num;
}
// 编译不通过，参数类型不兼容
let fn2: Fn = function(a: string) {
  return 12
}
```

#### TS 类型兼容的规则

- 多的兼容少的（对象属性多 → 兼容少的）
- 少的兼容多的（函数参数少 → 兼容多的）
- 子类型兼容父类型
- 只看结构，不看名字


## 常用关键字

### typeof 关键字

typeof 关键字用于获取变量的类型。可以起到断言的作用。

```ts

function hello1(value: string: number): voud {
  // 无法编译通过，ts 无法判断 value 是 string 还是 number 类型
  console.log(value + 2)
}

function hello2(value: string | number): void {
  // 起到断言的作用，以下代码能编译通过
  if (typeof value === "string") {
    console.log("hello" + value);
  } else {
    console.log(value + 2);
  }
}
```

### keyof 关键字

`keyof` 关键字，用于获取对象类型的键的常量联合类型。

```ts
interface User {
  name: string
  age: number 
}

// 获取对象类型的键的常量联合类型
type keyType = keyof User // 返回的是: "name" | "age" 常量联合类型
// 等价于
type keyType2 = "name" | "age" 

// 变量的值，只能是 name 和 age 字符串
let str1: keyType  = "name"
let str2: keyType = "age"

// 限定 key 的类型，只能是 User 类型的 key 的联合类型
function getProperty(user: User, key: keyof User) {
  return user[key]
}
```

## 接口类型

在 TypeScript 中，接口是用来定义「结构 / 契约」的，仅做**类型检查**，编译后消失。与类的不同点是，类是用来创建「实例」的，包含实现代码，编译后会转为 JS 真实代码。

### 定义接口

#### 定义普通对象类型 

```ts
interface User {
  id: number 
  name: string 
  age?: number // 表示可选属性，实例化时，可以不赋值，非可选属性，实例化时，必须初始化赋值
  readonly sex: string // 表示只读属性，不能修改
}
```

#### 定义索引对象类型（数组类型、字典类型）

```ts
interface NumIndex {
  // 定义索引的类型是数字类型，然后值的类型是字符串类型
  [index: number]: string // 注意 index 变量可以随便字段，可以改成 key 等等
}

// 定义有固定 key 的索引类型
interface FixKeyIndex {
  name: string,
  age: number,
  [key: string]: string | number 
}
```

#### 定义函数类型

```ts
// 普通函数接口
interface AddFunc {
  (a: number, b: number): number;
}

function add: AddFunc = function(a: number, b: number): number {
  return a + b 
}

// 重载函数接口
interface OverloadFunc {
  (value: string): string;
  (value: number): number;
  (value: boolean): boolean;
}
// 可以不用声明返回值，会自动推断
let overlaodFunc = function(value : string | number | boolean) {
  if(typeof value == "string") {
    return "hello " + value 
  }else if(typeof value == "number") {
    return value + 1
  }else {
    return value
  }
}

interface OverloadFunc2 {
  (value: string): string;
  (value: number): number;
  (value: boolean): boolean;
  (value: any): any;
}

let overloadFunc2 = function(value: any) {
  return any
}

```

### 同名接口会合并

在一个模块中，如果存在多个同名的 interface，那么 TypeScript 会自动把它们合并成一个接口，但是多个同名接口，如果存在相同属性，那么类型必须一致，否则不能编译通过。

使用场景：扩展第三方库的类型，或者拆分大型接口，让代码更清晰，或者对全局类型扩展

```ts

interface Person {
  name: string 
}

interface Person {
  age: number 
}

let p: Person = {name: "ts001", age: 20}
console.log(p.name, p.age)

// 同名接口，并且属性相同时，如果类型不相同，则无法编译通过
interface User {
  name: string 
}
interface User {
  // ❌ 报错！类型冲突，无法合并
  name: number 
}

```

### 接口继承

TypeScript 接口支持单继承和多继承

```ts
interface A {
  value: number: string 
}
interface B extends A {
  // 子类属性可以缩小类型
  value: number 
  hello(): void
}

// 多继承
interface A {
  value: number 
}
interface B {
  value: string
}
interface C extends A, B {
  // 当多个父接口有同名属性时，子接口必须通过联合类型来兼容：
  value: string: number 
}

```

## 联合类型

联合类型使用 | 符号将多个类型组合在一起，形成新的类型约束。

注意：对于一个联合类型的变量，TS 只允许我们访问联合类型“共有的”方法和属性，非共有属性和方法需要断言后访问。

### 基本类型的联合类型

```ts 
// 表示变量可以是数字类型或者字符类型
let value: number: string = "Hello World"
// 可以编译通过
value = 20
// 不能直接相加，非共有属性和方法，即 ts 无法判断 value 变量是数字还是字符串类型，需要断言后才能相加
value = (value as number) + 2

function hello(id: string : number) {
  // 需要使用类型守卫来，缩小类型范围，才能进行相应的操作
  if(id typeof number) {
    console.log(id + 1)
  }else {
    console.log(id + "Hello World")
  }
}
```

### 对象联合类型

```ts
type User = { name: string; age: number } | { name: string; sex: string };
// 能编译通过
let user: User = { name: "Hello", age: 20, sex: "man" };
// 但是无法直接访问 age, 因为不是共有属性
console.log(user.age);
// 能编译通过
console.log((user as any).age);
// 能编译通过
console.log((user as {sex: string}).sex)
```

### 数组联合类型

```ts
let arr: ( string| number )[] = [20, "hello"]
```


## 交叉类型

交叉类型通过 & 符号将多个类型的属性和方法组合在一起，形成一个新的类型。表示变量必须同时拥有所有类型的属性。

```ts
// 两个独立类型
type Name = { name: string };
type Age = { age: number };

// 交叉类型：必须同时有 name + age
type Person = Name & Age;

// 必须两个属性都要有，少一个编译就不能通过！
const p: Person = {
  name: "Hello Ts",
  age: 18 
};
```

## 字面量类型

字面量类型主要用于限制变量只能取固定值。

### 字符串字面量类型

示例1： 

```ts 
// 定义 num 变量只能固定值 12
let num: 12 = 12 

// 定义 value 变量只能取固定值 hello 
let value: "hello" = "hello"
```

示例2：

```ts
// 定义 method 变量的值，只能是 “GET” 或者 “POST”
let method : "GET" | "POST" = "GET"
// 能编译通过
method = "POST"
// 无法编译通过
method = "PUT"
```

### 数字字面量类型

```ts
type Size = 10 | 20 | 30;
let size: Size = 10
```

### 布尔字面量类型

```ts
type isEnabled = true 
// 能编译通过
let enable: isEnabled = true
// 无法编译通过
enable = false
```

### 对象字面量类型

```ts
// 定义对象中的 name 属性只能取固定值 "start" 和 "closed"
type TimingEvent = { name: "start"; userStarted: boolean } | { name: "closed"; duration: number };
// 能编译通过
let t1: TimingEvent = {name: "start", userStarted: false }

// 定义 value 变量的 name 和 age 属性都只能取固定值
let value: {name: "Hello", age: 20} = {name: "Hello", age: 20}
```



## 可空类型

在 JavaScript 中，变量可以被随意赋值为 null 或 undefined，容易引发潜在错误。
TypeScript 通过类型系统提供了可空类型支持，允许我们明确声明一个变量可能为 null 或 undefined，并在编译阶段提前检查出空值相关的问题。

```ts 
let name: string | null = null;
// 能编译通过
name = "Ts";
// 能编译通过
name = null; 

function hello(name: string | null) {
  // 通过类型守卫，我们可以在代码中明确检查变量是否为 null 或 undefined
  if(name == null) {
    console.log( "I am null")
  }
  console.log(`Hello ${name}`)
}

```



## 类型别名

类型别名是给类型起个新的名字，本身不创建新类型，只是引入一个同义名称，能让代码更清晰、简洁或便于维护。

### 基本类型别名

```ts 
// 给 string 类型定义一个新的别名
type Str = string 
let s1: string = "Hello World"
// 能编译通过
let s2: Str = s1

type numberOrString = number | string 
// 限制 value 变量的类型只能是 number 或者 string 
let value: numberOrString = "hello World"
value = 90 // 能编译通过
value = true // 编译不通过报错 
```

### 对象类型别名

```ts
// 不使用别名：写起来又长又乱
let user: { name: string; age: number; id: number } = { ... }

// 使用别名：清爽简洁
type User = { name: string; age: number; id: number };
let user: User = { ... };
```

### 函数类型别名

```ts 
// 给函数类型定义别名
type Callback = (code: number, msg: string) => boolean;
```

### 联合和交叉类型别名

```ts
type NumOrStr = number | string 

type User = {name: string} & {age: number}
```
