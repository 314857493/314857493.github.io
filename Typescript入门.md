---
title: Typescript入门
---

文字描述借鉴了网上各处大佬的文字描述，代码部分基本为本人编写，各位大佬可以批评指导。

- TOC
  {:toc}

# TS 简介

- ts 是微软开源的一种编程语言
- ts 是 js 的超集，完全支持 js 并对 js 进行了拓展
- ts 不能被常见的 js 运行环境（如浏览器，nodejs）直接执行，需要将 ts 编译为 js 执行
- ts 有很多的配置项，可以根据不同配置编译成各种版本的 js，并提供相应 polyfill

# 正文

## 类型系统

### 标注语法

冒号语法 `let n:number = 1`

### 基础类型

基础类型包括：`string` , `number` , `boolean`

```typescript
let title: string = "这是一个title";
let n: number = 123;
let isOk: boolean = true;
```

### 空和未定义类型

`null` 和 `undefined` 两种类型，表示了空和未定义类型，使用类型标注后类似于定义成了一个常量。默认情况下，这两种类型是所有类型的子类型，也就是说可以赋值给其他所有类型。因为 `null` 和 `undefined` 是其他类型的子类型，所以在默认情况下会有一些问题。

```typescript
let n: number = 123;
a = null;
// 在默认情况下，即未开启 "strictNullChecks": true 时，不能检查出错误
// 开启 "strictNullChecks": true 时，ts静态检查报错 不能将类型“null”分配给类型“number”。
a.toFixed(2);
// 默认情况下运行时错误 TypeError: Cannot read property 'toFixed' of null
```

### void 类型

表示没有任何数据的类型，与其他强类型语言相同，通常用于标注无返回值函数的类型标注，未标注类型的函数默认标注类型为 `void`

```typescript
function fun(): void {
  // do something
  // 没有 return 或者 return undefind
}
```

在未开启 "strictNullChecks": true 时，`null` 和 `undefined` 都可以赋值给 `void`，开启之后，只有 `undefined` 可以。

### 对象类型

在 js 里有很多内置对象，我们可以通过这些对象的构造函数进行标注。

```typescript
let obj: Object = {};
let arr: Array<number> = [1, 2, 3, 4];
let date: Date = new Date();
```

### 自定义对象类型

- 字面量标注：方便使用，但是不利于复用和后期维护

```typescript
let person: { name: string; age: number };
person = {
  name: "张三",
  age: 18,
  sex: "male", //err “sex”不在类型“{ name: string; age: number; }”中
};
person.nickName = "小张"; //err “类型“{ name: string; age: number; }”上不存在属性“nickName”
```

- 类型别名

```typescript
type MyString = string;
let myStr: MyString = "abcabc";

type Person = {
  name: string;
  age: number;
};
let person: Person = {
  name: "张三",
  age: 18,
};
```

- 接口  
  接口定义了一个结构，这个结构为你的代码提供契约。

```typescript
interface Person {
  name: string;
  age: number;
}
let person: Person = {
  name: "张三",
  age: 18,
};
```

- 类和构造函数

```typescript
class Person {
  name: string;
  age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}
let person: Person = {
  name: "张三",
  age: 18,
};
```

这种方式与上文中的使用内置构造函数进行标注意义相同

### 包装对象

指 js 中的`String`、`Number`、`Boolean`，我们知道 js 中的`string`与`String`不一样，在 ts 中，也不一样。

```typescript
let str: string;
str = "aa";
str = new String("bbb"); // err 不能将类型“String”分配给类型“string”。

let str2 = new String("ccc");
str2 = "ddd"; // is ok
```

包装对象的包含范围大于基础类型

### 数组

TS 中的数组所有元素必须符合类型标注，所以在标注某一变量的类型为数组时也需要标注数组中存储元素的类型。

```typescript
let arr: Array<number> = [1, 2, 3, 4, 5, 6];
arr.push(100);
arr.push("aaa"); // err 类型“string”的参数不能赋给类型“number”的参数。
```

简单标注：

```typescript
let arr: number[] = [1, 2, 3, 4, 5, 6];
arr.push(100);
arr.push("aaa"); // err 类型“string”的参数不能赋给类型“number”的参数。
```

### 元组

元组在 ts 中的定义与其他语言有所区别，如 python 中的元组表示一个不可变的序列，在数据库中代表一行数据，在 ts 中只要求在初始化时必须与标注类型相同，其他时候类似于联合类型的数组。

```typescript
let tuple: [string, number];
tuple = ["a", 1]; // 初始化时元素类型必须与类型标注一一对应
tuple.push("aa"); // push时只能push元组中存在类型的元
```

### 枚举

枚举的作用是定义一组有关联的数据，通过枚举我们可以将一些有特定意义的值赋予一些有意义的名字。枚举的值只能是数字或者字符串。

- 枚举值是常量，不可以通过重新赋值的方式修改
- 数字类型枚举第一个值默认为 0，后面的值依次+1
- 字符串枚举值后面不能省略枚举值

```typescript
enum HTTP_CODE {
  OK = 200,
  NOT_FOUND = 404,
  METHOD_NOT_ALLOW, //404+1 = 405
}

HTTP_CODE.OK; // 200
HTTP_CODE.NOT_FOUND; // 404

enum ErrorType {
  TYPE_ERROR = "type error",
  REFERENCE_ERROR = "reference error",
  OTHER_ERROR, // err 枚举成员必须具有初始化表达式。
}
```

### never 类型

当一个函数永远不可能执行到 `return` 的时候，返回值使用 `never`，常见为**抛出错误**

```typescript
function getError(): never {
  throw new Error("Error");
}
```

### any 类型

有些情况下，我们实在不知道这个值到底是什么类型或者不需要对这个值进行类型检测的时候，就可以将其标注为 `any` 类型，同时这也意味着放弃了对该值的类型检查，也放弃了 IDE 对其的类型提示。

- 一个变量未声明且未标注的情况下，默认类型为 `any`
- 任何类型都可以赋值给 `any`
- `any` 可以赋值给任何类型
- `any` 类型有任意属性和方法
- 当 `"noImplicitAny" : true` 时，当无法推断一个变量的时候就会报错（或者只能推断为 any 类型），你需要通过显示的类型注解来消除这个错误

```typescript
let a: any = 123;
let b: string = "aa";
b = a;
a = b; // all ok
```

### unknow 类型

`unknown` 类型在 ts3.0 版本的时候引入，是一种安全的类型。

- 和 `any` 一样，所有类型都可以被分配给 `unknown`
- 与 `any` 不同，`unknown` 只能被分配给 `any` 和 `unknown`
- 如果不限定类型，`unknown` 不可以做任何操作，如果需要操作，则需要使用断言来缩小范围

```typescript
let a: unknown = "hello";
a = 123; // is ok
a = "hello world";
let str: string = a; // err 不能将类型“unknown”分配给类型“string”。

a.toString(); // err 对象的类型为 "unknown"。
a.toFixed(2); // err 对象的类型为 "unknown"。

if (typeof a === "string") {
  a.toLocaleLowerCase(); // ok
}

(a as string).toLocaleLowerCase(); // ok
```

### 函数

#### 函数重点是参数类型和返回值类型

```typescript
// 直接标注
function fn(str: string): string {
  return str;
}
// 箭头函数标注
const fn1: (str: string) => string = (str: string) => {
  return str;
};
// 类型别名标注
type fnType = (str: string) => string;

const fn2: fnType = (str: string) => {
  return str;
};
// 接口标注
interface IFunc {
  (str: string): string;
}

const fn3: IFunc = (str: string) => {
  return str;
};

const fn4: IFunc = function (str) {
  return str;
};
```

#### 可选参数和默认参数

通过参数名后加 `?` 来标注参数是可选的

```typescript
type value = "100px" | "200px" | "300px"; // 值只能为这三个
function style(element: Element, attr: string, value?: value) {}

let box = document.querySelector("#root");
if (box) {
  style(box, "height");
  style(box, "height", "100px");
}
```

#### 默认参数

默认参数也会进行自动类型推导

```typescript
function sort(arr: Array<number>, orderBt = "desc") {}

sort([2, 3]);

sort([5, 4, 6], "acs"); // 用"acs"替换"desc"

sout([2, 3], 1); // err 类型“1”的参数不能赋给类型“string | undefined”的参数。
```

#### 剩余参数

剩余参数是一个数组

```typescript
interface IArgs {
  [key: string]: any;
}

function func(target: IArgs, ...args: IArgs[]) {}
```

#### this 的类型

```typescript
type T = {
  a: number;
  fn: (x: number) => number;
};

let obj: T = {
  a: 1,
  fn: function (this: T, x) {
    console.log(this.a);
    return x;
  },
};

obj.fn(123); // 123
```

箭头函数：`this` 指向固定的外部函数的上下文

```typescript
interface T {
  a: number;
  fn: (x: number) => any;
}

// let obj2: T = {
//   a: 1,
//   fn: (this: T, x: number) => {
//     // err 箭头函数不能包含 "this" 参数。
//     return x;
//   },
// };

let obj2: T = {
  a: 1,
  fn(this: T, x: number) {
    return () => {
      console.log(this.a);
      return x;
    };
  },
};
```

#### 函数重载

对于不同类型的参数，返回不同类型的内容

```typescript
function getSomething(target: Object): Object;
function getSomething(target: number): number;
function getSomething(target: string): string;

function getSomething(target: Object | number | string) {
  return target;
}
```

## 类型保护

类型保护允许你使用更小范围下的对象类型。  
使用 `instanceof` 和 `typeof`，对类型进行限定，即可在更小的类型范围内使用类型。

### typeof

```typescript
function doSomething(x: number | string) {
  if (typeof x === "string") {
    x.split("");
    x.toFixed(1); // err 属性“toFixed”在类型“string”上不存在。
  }

  x.split(""); // err 不能保证x的类型是string
}
```

### instanceof

```typescript
class Success {
  data = "success";
  code = 200;
}

class Fail {
  message = "failed";
  code = 500;
}

function dealResponse(response: Success | Fail) {
  if (response instanceof Success) {
    console.log(response.data);
    console.log(response.message); // err 类型“Success”上不存在属性“message”
  }
  if (response instanceof Fail) {
    console.log(response.data); // err 类型”Fail”上不存在属性“data”
    console.log(response.message);
  }
}
```

TS 甚至可以理解 else。当使用 if 进行类型缩小时，ts 知道 else 中的类型不是 if 中的类型

```typescript
class Success {
  data = "success";
  code = 200;
}

class Fail {
  message = "failed";
  code = 500;
}

function dealResponse2(response: Success | Fail) {
  if (response instanceof Success) {
    console.log(response.data);
    console.log(response.message); // err 类型“Success”上不存在属性“message”
  } else {
    console.log(response.data); // err 类型“Fail”上不存在属性“data”
    console.log(response.message);
  }
}
```

### in

`in` 操作符可以安全的检查一个对象上是否存在一个属性，它通常也被作为类型保护使用

```typescript
function dealResponse3(response: Success | Fail) {
  if ("data" in response) {
    console.log(response.data);
    console.log(response.message); // err 类型“Success”上不存在属性“message”
  } else {
    console.log(response.data); // err 类型“Fail”上不存在属性“data”
    console.log(response.message);
  }
}
```

### 字面量类型保护

当你在类型中使用字面量类型时，可以检查字面量来进行类型保护

```typescript
type Foo = {
  kind: "foo"; // 字面量类型
  foo: number;
};

type Bar = {
  kind: "bar"; // 字面量类型
  bar: number;
};

function doSomething(arg: Foo | Bar) {
  if (arg.kind === "foo") {
    // arg:Foo
    console.log(arg.foo);
  } else {
    // arg:Bar
    console.log(arg.bar);
  }
}
```

### 自定义类型保护

当使用普通的 js 对象时，我们甚至无法使用 `typeof` 和 `instanceof` 进行类型判断。这种情况下，我们需要自定义类型保护函数，这仅仅是一个返回值类似 `somethin is SomeType` 的函数，如下：

```typescript
class Success {
  data = "success";
  code = 200;
}

class Fail {
  message = "failed";
  code = 500;
}

function isSuccess(response: Success | Fail): response is Success {
  return (response as Success).data !== undefined;
}

function dealResponse4(response: Success | Fail) {
  if (isSuccess(response)) {
    console.log(response.data);
  } else {
    console.log(response.message);
  }
}
```

## 接口和高级类型

### 接口

对复杂的对象类型进行标注的一种方式，或者给其他代码定义的一种契约

```typescript
interface Point {
  x: number;
  y: number;
}

let point1: Point = { x: 1, y: 2 };
```

接口是一种类型，不可以当做值来用。

```typescript
interface Point {
  x: number;
  y: number;
}

console.log(Point); // err “Point”仅表示类型，但在此处却作为值使用
```

#### 可选属性

```typescript
interface Point {
  x: number;
  y: number;
  color?: string;
}

let point1: Point = { x: 1, y: 2 };
let point2: Point = { x: 1, y: 2, color: "red" };
```

#### 只读属性

```typescript
interface Point {
  readonly x: number;
  y: number;
  color?: string;
}

let point1: Point = { x: 1, y: 2, color: "red" };
point1.x = 123; // err 无法分配到 "x" ，因为它是只读属性。
```

#### 任意属性

```typescript
interface ISomething {
  [key: string]: number;
}

let something: ISomething = {
  Alpha: 0,
  Beta: 1,
  Gamma: 2,
};
something.Delta = 3;
something.Epsilon = 4;
something.Zeta = 5;
```

#### 标注函数

```typescript
interface IFunc {
  (arg: string): string;
}

const doSomething: IFunc = (arg: string) => arg;
```

#### 接口合并

多个同名接口会被合并，不会和 js 变量一样被覆盖。

```typescript
interface Box {
  height: number;
  width: number;
}

interface Box {
  length: number;
}

let box: Box = {
  height: 123,
  width: 123,
  length: 123,
};
```

#### 接口继承

接口可以被继承

```typescript
interface NamedBox extends Box {
  name: string;
}

let box1: NamedBox = {
  height: 123,
  width: 123,
  length: 123,
  name: "小黑",
};
```

#### 接口实现

接口可以被类实现

```typescript
class BoxClass implements Box {
  height: number;
  width: number;
  length: number;
  constructor(height: number, width: number, length: number) {
    this.height = height;
    this.width = width;
    this.length = length;
  }
}
```

### 高级类型

#### 联合类型

联合类型也可以称为多选类型，当我们希望标注一个变量为多个类型之一的时候可以使用，各个标注类型之间是 **或** 的关系。

```typescript
type SomeType = number | string;
let a: SomeType = 123;
a = "str";
```

#### 交叉类型

交叉类型也可以称为合并类型，把多个类型合并成一个新类型，结果类似于前文接口的合并，各个标注类之间是 **且** 的关系

```typescript
interface IObj1 {
  x: number;
  y: number;
}

interface IObj2 {
  z: number;
}

type TObj3 = IObj1 & IObj2;

let obj: IObj1 & IObj2 = { x: 1, y: 2, z: 3 };
let obj2: TObj3 = { x: 2, y: 3, z: 4 };
```

#### 字面量类型

有些时候，我们希望标注的不是一个类型，是一个具体的值，这种情况就可以使用字面量类型，配合联合类型，可以让我们的变量在固定的几个值之间选择。

```typescript
type Code = 200 | 201;
function getCode(code: Code) {
  // do something
  return code;
}
const code = getCode(200);
const code2 = getCode(201);
const code3 = getCode(301); // err 类型“301”的参数不能赋给类型“Code”的参数。
```

#### 类型别名

有时候类型名称很复杂时，我们可以采用类型别名简化它
比如说在 React 中

```typescript
interface FunctionComponent<P = {}> {
  (props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}
type FC<P = {}> = FunctionComponent<P>;
```

`interface`和`type`的区别

`interface`:

- 只能描述`object`/`class`/`function`的类型
- 同名的`interface`自动合并，利于拓展
- 可以继承

`type`:

- 可以描述所有类型
- 不能重名
- 可以被 interface 继承

#### 类型推导

每次声明变量都显式标注类型的话很麻烦，TS 提供了一种更加方便的特性：类型推导。TS 会根据当前上下文自动地推导出对应的类型标注，这个过程发生在一下几个过程中：

- 初始化变量
- 设置函数默认值参数
- 返回函数值

```typescript
let x = 1; // x:number

// function fn(a?: number): number
function fn(a = 1) {
  // a:number
  return a;
}
```

#### 类型断言

有时候我们需要标注一个更精确的类型（缩小类型的范围），比如

```typescript
let img = document.querySelector("#img");
// img:Element
img && img.src; // err 类型“Element”上不存在属性“src”。
img && (<HTMLImageElement>img).src;
img && (img as HTMLImageElement).src; // ok
```

## 泛型

**在定义函数或者类的时候，遇到不一定明确的类型就可以使用泛型**

### 标注方法

```typescript
function getSomething<T>(arg: T): T {
  return arg;
}

let getSomething1: <T>(arg: T) => T = <T>(arg: T): T => {
  return arg;
};

interface IFunc {
  <T>(arg: T): T;
}

let getSomething2: IFunc = <T>(arg: T): T => {
  return arg;
};

interface IFunc1<T> {
  (arg: T): T;
}

let getSomething3: IFunc1<number> = getSomething;
```

### 泛型变量可以有多个

```typescript
function someFunc<T, F>(arg: T, m: F): T {
  console.log(m);
  return arg;
}

const a = someFunc<string, number>("string", 3);
```

### 泛型类

```typescript
class SomeClass<T> {
  value: T;
  constructor(value: T) {
    this.value = value;
  }
}

let something = new SomeClass<number>(123);
```

### 在泛型中使用 class 类型

```typescript
const create = <T>(C: { new (): T }): T => {
  return new C();
};

class Person {
  name!: string;
}

const person = create(Person);
person.name = "aaa";
```

## OOP

### 类

在常规面向对象中最重要的核心就是“类”，当我们使用面向对象的方式进行编程的时候，通常会使去分析具体的功能，把特性相似的东西抽象成一个个的类，然后通过这些类实例化出来的具体对象来完成具体的业务需求。

#### 类的基础

在类的基础中，包含以下几个核心的内容，也是 `ts` 和 `es6+(ecmascript2015+)` 在类方面一些共有的特性。

- class 关键字
- 构造函数：`constructor`
- 成员属性
- 成员方法
- `this` 关键字

除了以上的共有特性以外，在 ts 中还有一些 es 中没有的或者还没有支持的特性，比如抽象。

#### class 关键字

通过 class 就可以描述和阻止一个类的数据结构

```typescript
// 命名规范一般为大驼峰
class Person {
  // 特征定义
}
```

#### 构造函数

通过 class 定义一个类后，我们就可以使用 `new` 关键字来调用这个类从而得到一个改类型的具体对象，这个过程也称为实例化。
为什么类可以像函数一样去调用呢，其实我们并不是执行这个类，而是执行其中的一个特殊的函数：构造函数 `constructor`

```typescript
class User {
  constructor() {
    console.log("这是实例化的时候执行的方法");
  }
}
let user = new User();
```

- 默认情况下，构造函数是一个空函数
- 构造函数会在类被实例化的时候调用
- 我们定义的构造函数会覆盖默认的构造函数
- 如果在实例化的时候不用传入参数，可以将`()`省略，如 `let user = new User;`
- 构造函数 `constructor` 不允许有 `return` 和返回值类型标注(因为默认返回实例对象)  
  通常情况下，我们会把一个类实例化时初始化的相关内容写在构造函数里，比如成员属性的初始化赋值等。

#### 成员属性和成员方法

```typescript
class User {
  id: number;
  username: string;
  constructor(id: number, username: string) {
    this.id = id;
    this.username = username;
    console.log("这是实例化的时候执行的方法");
  }
  doSomething() {
    console.log(`做了一些事`);
  }
}

let user = new User(1, "jxd");
user.doSomething(); // 做了一些事
```

#### this 关键字

```typescript
class User {
  id: number;
  username: string;
  constructor(id: number, username: string) {
    this.id = id;
    this.username = username;
    console.log("这是实例化的时候执行的方法");
  }
  doSomething(thing: string) {
    // 通过this访问成员属性
    console.log(`${this.username} 做了 ${thing}`);
  }
}

let user = new User(1, "jxd");
user.doSomething("吃饭"); // jxd 做了 吃饭
```

#### 继承

在 ts 中，也是通过 `extends` 关键字来实现类的继承

```typescript
class Father {}

class Child extends Father {}
```

#### super 关键字

在子类中，我们可以通过 `super` 关键字来引用父类并调用父类中的构造函数

- 如果子类中没有定义构造函数，则会在默认的 `constructor` 中调用 `super()`
- 如果子类有自己的构造函数，则需要在子类的构造函数中显示地调用 `super` ，否则会报错
- 在子类构造函数调用 `super()` 之后才能访问 `this`
- 在子类中，可以通过 `super` 来访问父类的成员属性和方法
<!-- 3 -->

```typescript
class Father {
  age: number;
  constructor(age: number) {
    this.age = age;
  }
}

class Child extends Father {
  name: string;
  constructor(age: number, name: string) {
    this; // err 访问派生类的构造函数中的 "this" 前，必须调用 "super"。
    super(age);
    this.name = name;
  }
}
```

<!-- 4 -->

```typescript
class Father {
  private age: number;
  constructor(age: number) {
    this.age = age;
  }
  logAge() {
    console.log(this.age);
  }
}

class Child extends Father {
  name: string;
  constructor(age: number, name: string) {
    super(age);
    this.name = name;
  }
  logInfo() {
    super.logAge(); // 调用父类的logAge方法，打印父类的私有变量age
    console.log(this.name);
  }
}

var child = new Child(23, "jxd");
child.logInfo();
```

#### 方法重写和重载

默认情况下，子类的成员方法继承自父类，但是子类也可以对他们进行重写或重载

```typescript
class Father {
  private age: number;
  constructor(age: number) {
    this.age = age;
  }
  logInfo() {
    console.log(this.age);
  }
}

class Child extends Father {
  name: string;
  constructor(age: number, name: string) {
    super(age);
    this.name = name;
  }
  logInfo(): void;
  logInfo(someInfo: string): void;
  logInfo(someInfo: string, someInfo2: string): void;
  logInfo(someInfo?: string, someInfo2?: string) {
    super.logInfo(); // 这里调用的仍然是父类的logInfo方法
    console.log(this.name);
    if (someInfo) {
      console.log(someInfo);
    }
    if (someInfo2) {
      console.log(someInfo2);
    }
  }
}

var child = new Child(23, "jxd");
child.logInfo();
child.logInfo("info1");
child.logInfo("info1", "info2");
```

#### 访问修饰符

有的时候，我们希望对类的成员（属性，方法）进行一定的访问控制，来保证数据的安全，通过访问修饰符可以做到这一点，目前提供了四种修饰符：

- public 公开的，默认。访问级别：自身，子类，类外
- protected 受保护的。访问级别：自身，子类
- private 私有。访问级别：自身
- readonly 只读。访问级别：同 public，但是不可以进行修改，且必须在声明或者构造函数内被初始化

```typescript
class User {
  protected username: string;
  private password: string;
  readonly id: number;
  constructor(username: string, password: string, id: number) {
    this.username = username;
    this.password = password;
    this.id = id;
  }
  log() {
    console.log(this.username, this.password, this.id);
  }
}

class ChildUser extends User {
  log() {
    console.log(this.id);
    console.log(this.username);
    // err 属性“password”为私有属性，只能在类“User”中访问。
    console.log(this.password);
  }
}

let childUser = new ChildUser("jxd", "123456", 1);
console.log(childUser.id);

childUser.id = 1; // err 无法分配到 "id" ，因为它是只读属性。
console.log(childUser.username); // err 属性“username”受保护，只能在类“User”及其子类中访问。
console.log(childUser.password); // err 属性“password”为私有属性，只能在类“User”中访问。
```

#### get set

```typescript
class User {
  private password: string;
  constructor() {
    this.password = "123456";
  }
  get _password() {
    return this.password;
  }
  set _password(password: string) {
    this.password = password;
  }
}

let user = new User();
user._password = "password";
user._password; // password
```

#### 静态成员

前面我们说的成员属性和方法都是实例的，但是有时候，我们需要给类本身添加成员

```typescript
class User {
  private static readonly company: string = "JD";
  static get _company() {
    return this.company;
  }
}

console.log(User._company);
```

#### 抽象类

和其他语言的抽象类一样，一般用于被其他类继承，而不能被直接实例化。抽象类和类内部定义的抽象方法，使用 `abstract` 关键字修饰，在抽象类里定义的抽象方法，在子类中必须实现，否则会报错。

```typescript
abstract class User {
  constructor(name: string) {}
  abstract logName(): void;
}

class ChildUser extends User {
  name: string;
  age: number;
  constructor(name: string, age: number) {
    super(name);
    this.name = name;
    this.age = age;
  }
  // 若不实现，会抛出 非抽象类“ChildUser”不会实现继承自“User”类的抽象成员“logName”。 错误
  logName() {
    console.log(this.name);
  }
}
```

#### 类和接口

使用接口可以强制一个类定义必须包含某些内容：使用关键字 `implements`。 **`implements`** 关键字指定类要实现的接口。

```typescript
interface UserInterface {
  name: string;
  age: number;
}

class User implements UserInterface {
  name: string;
  age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}
```

## 命名空间

避免重复命名，如下所示，有两个名为 `FunctionComponent` 的 interface，但是两个 interface 的内容不一样，我们可以使用不同的 namespace 来隔离两个 `FunctionComponent` 来让他们都可以正常使用。

```typescript
// main.ts
// 使用export default {} 之后，main.ts是一个单独的模块，与全局命名空间相隔离，使用三斜线指令，可以将指定文件的命名空间引入
// 三斜线指令可以理解为导入命名空间的import
/// <reference path='./namespace.ts'/>
const Index: React.FunctionComponent = (props: any) => {};
const Index2: MyReact.FunctionComponent = (props: any) => {};

export default {};

// namespace.ts
// 也可以在namespace关键字之前使用declare关键字，让其存在于全局的命名空间，这样不用三斜线指令也可以使用这个命名空间
namespace React {
  export interface FunctionComponent {
    (props: any): any;
  }
}

namespace MyReact {
  export interface FunctionComponent {
    (props: number): any;
  }
}
```

## 模块

TS 的模块与 ES6+的模块系统基本一致，使用 `import` 关键字进行导入，使用 `export` 关键字进行导出

```typescript
// helloworld.ts
const helloworld = () => {
  console.log("helloworld");
};
export default helloworld;

// main.ts
import helloworld from "./helloworld";
helloworld(); // helloworld
```

## 装饰器

装饰器是一种特殊的类型声明，它能够被附加到**类、方法、属性、参数**上

- 语法：装饰器使用 `@expression` 这种形式，`expression` 是一个函数，它会在运行时被调用，传入参数是被装饰的内容。
- 若要使用装饰器，需要在`tsconfig.ts`里启用`experimentalDecorators`选项
- 装饰器相当于一个高阶函数，即 `@f @g x` 相当于 `f(g(x))`

基础写法：

```typescript
type Constructor = new (...args: any[]) => Object;

// 类装饰器
function addRun(target: Constructor) {
  return class extends target {
    run() {
      console.log("run run run");
    }
  };
}

@addRun
class Cat {}

const cat = new Cat();
// 这里需启用 @ts-ignore 因为 Cat本身并没有run属性，所以会报错
cat.run(); // log:run run run
```

装饰器工厂：

```typescript
type Constructor = new (...args: any[]) => Object;

function addRun(address: string) {
  return function (target: Constructor) {
    return class extends target {
      run() {
        console.log(`run to ${address}`);
      }
    };
  };
}

@addRun("不知道去哪了")
class Cat {}

const cat = new Cat();
// @ts-ignore
cat.run(); // log:run to 不知道去哪了
```
