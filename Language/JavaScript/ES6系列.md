## 一、ES6和JS的关系

ECMAScript 是JavaScript的一个标准规范
JS 语法的实现
TS js的超集

babel转码器 把ES6转为浏览器可识别的ES5

## 二、let与const

- 使用var声明的变量会被提升到作用域的顶部，let、const因为暂时性死区的原因，不能在声明前使用
- var在全局作用域下声明变量会导致变量挂载在window上，其他两者不会
- let和const作用基本一致，但是const声明的变量不能再次赋值
- ES6之前JS只有函数作用域和全局作用域，ES6新增的let和const可以定义块级作用域变量

函数提示优先于变量提升，函数提升会把整个函数挪到作用域顶部，变量提升只会把声明挪到作用域顶部

const一旦定义不能只声明不赋值，值一定要初始化，声明的常量不可改变(引用类型的值可修改，不能再次声明)

## 三、箭头函数

箭头函数表达式的语法比函数表达式更简洁，并且没有自己的this，arguments，super或new.target。箭头函数表达式更适用于那些本来需要匿名函数的地方，并且它不能用作构造函数。

如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。

由于大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上括号，否则会报错。
```
let getUser = id => ({ id: id, name: "levin" })
```

### 关于this

- 默认绑定外层this: 箭头函数没有 this，所以需要通过查找作用域链来确定 this 的值
- 不能用call()、apply()、bind()方法修改里面的this
- 没有 arguments: 可以通过命名参数或者 rest 参数的形式访问参数
- 不能通过 new 关键字调用
- 没有原型

JavaScript 函数有两个内部方法：[[Call]] 和 [[Construct]]。

当通过new调用函数时，执行[Construct]]方法，创建一个实例对象，然后再执行函数体，将 this 绑定到实例上。

当直接调用的时候，执行[[Call]]方法，直接执行函数体。

箭头函数并没有[[Construct]]方法，不能被用作构造函数，如果通过 new 的方式调用，会报错。

```
(function() {
  return [
    (() => this.x).bind({ x: 'inner' })() // 无效的bind,最终this还是指向外层
  ];
}).call({ x: 'outer' });
// ['outer']


let nums = (...nums) => nums
```

### ES5实现箭头函数

```
setTimeout(function () {
  return this;
}.bind(Window), 100);
```

## 四、模板字符串

模板字符串是增强版的字符串，用反引号（`）标识，可以当作普通字符串使用，也可以用来定义多行字符串

## 五、set、map数据结构

Set 对象允许你存储任何类型的唯一值，无论是原始值或者是对象引用。它是值的集合，你可以按照插入的顺序迭代它的元素。 Set中的元素只会出现一次，即**Set 中的元素是唯一的**。

*类似数组*

```
let mySet = new Set();

mySet.add(1); // Set [ 1 ]
mySet.add(5); // Set [ 1, 5 ]
mySet.add(5); // Set [ 1, 5 ]
mySet.add("some text"); // Set [ 1, 5, "some text" ]
mySet.has(1); // true
mySet.has(3); // false
```

### 数组去重
```
const numbers = [2,3,4,4,2,3,3,4,4,5,5,6,6,7,5,32,3,4,5]
console.log([...new Set(numbers)])
```

Map 对象保存键值对，并且能够记住键的原始插入顺序。任何值(对象或者原始值) 都可以作为一个键或一个值。一个Map对象在迭代时会根据对象中元素的插入顺序来进行 — 一个  for...of 循环在每次迭代后会返回一个形式为[key，value]的数组。

*类似对象*，**Map的键可以是任何类型**

```
let myMap = new Map();

let keyObj = {};
let keyFunc = function() {};
let keyString = 'a string';

// 添加键
myMap.set(keyString, "和键'a string'关联的值");
myMap.set(keyObj, "和键keyObj关联的值");
myMap.set(keyFunc, "和键keyFunc关联的值");

let m2 = new Map([['name', 'levin'], ['age', 10]])
```

### 使用 for..of 方法迭代 Map

```
let myMap = new Map();
myMap.set(0, "zero");
myMap.set(1, "one");
for (let [key, value] of myMap) {
  console.log(key + " = " + value);
}
```

## 六、WeakSet、mWeakMap数据结构

WeakSet 对象是一些对象值的集合, 并且其中的每个对象值都只能出现一次。在WeakSet的集合中是唯一的

它和 Set 对象的区别有两点:

- 与Set相比，WeakSet 只能是对象的集合，而不能是任何类型的任意值。
- WeakSet持弱引用：集合中对象的引用为弱引用。 如果没有其他的对WeakSet中对象的引用，那么这些对象会被当成垃圾回收掉。 这也意味着WeakSet中没有存储当前对象的列表。 正因为这样，WeakSet 是不可枚举的。

```
var ws = new WeakSet();
var foo = {};
var bar = {};

ws.add(foo);
ws.add(bar);
ws.has(foo);    // true
ws.has(bar);   // true
ws.delete(foo); // 从set中删除 foo 对象
```

WeakMap 对象是一组键/值对的集合，其中的键是弱引用的。其键必须是对象，而值可以是任意的。

WeakMap 的 key 只能是 Object 类型。 原始数据类型 是不能作为 key 的（比如 Symbol）

```
const wm1 = new WeakMap(),
      wm2 = new WeakMap(),
      wm3 = new WeakMap();
const o1 = {},
      o2 = function(){},
      o3 = window;

wm1.set(o1, 37);
wm1.set(o2, "azerty");
wm2.set(o1, o2); // value可以是任意值,包括一个对象或一个函数
wm2.set(o3, undefined);
wm2.set(wm1, wm2); // 键和值可以是任意对象,甚至另外一个WeakMap对象

wm1.get(o2); // "azerty"
wm2.get(o2); // undefined,wm2中没有o2这个键
wm2.get(o3); // undefined,值就是undefined

wm1.has(o2); // true
wm2.has(o2); // false
wm2.has(o3); // true (即使值是undefined)

wm3.set(o1, 37);
wm3.get(o1); // 37

wm1.has(o1);   // true
wm1.delete(o1);
wm1.has(o1);   // false
```

## 七、this的指向

[参考](./JS的this指向.md)

## 八、Promise

Promise是异步编程的一种解决方案，比传统的解决方案（回调函数和事件）更合理、强大

promise.then里的回调函数会放到相应宏任务的微任务队列里，等宏任务里面的同步代码执行完再执行

## 九、async、await

使用 async/await, 搭配promise,可以通过编写形似同步的代码来处理异步流程, 提高代码的简洁性和可读性
async 用于申明一个 function 是异步的，而 await 用于等待一个异步方法执行完成

async函数表示函数里面可能会有异步方法，await后面跟一个表达式

async方法执行时，遇到await会立即执行表达式，然后把表达式后面的代码放到微任务队列里，让出执行栈让同步代码先执行

## 十、解构赋值

解构赋值语法是一种 Javascript 表达式。通过解构赋值, 可以将属性/值从对象/数组中取出,赋值给其他变量。

```
var a, b, rest;
[a, b] = [10, 20];
console.log(a); // 10
console.log(b); // 20

[a, b, ...rest] = [10, 20, 30, 40, 50];
console.log(a); // 10
console.log(b); // 20
console.log(rest); // [30, 40, 50]

({ a, b } = { a: 10, b: 20 });
console.log(a); // 10
console.log(b); // 20
```

## 十一、... 展开运算符

可以将数组或对象里面的值展开；还可以将多个值收集为一个变量

```
var array1 = [1, 2, 3];
var array2 = [4, 5, 6];
var array3 = [...array1, ...array2, 7, 8]; // [1,2,3,4,5,6,7,8]
array1.push(...array2 ) // 当然也可以使用concat等数组合并方法，但是扩展运算符提供了一种新的方式
```

## 十二、Symbol

symbol 是一种基本数据类型。Symbol()函数会返回symbol类型的值，该类型具有静态属性和静态方法。每个从Symbol()返回的symbol值都是唯一的。一个symbol值能作为对象属性的标识符；这是该数据类型仅有的目的。

## 十三、Proxy

Proxy对象用于定义基本操作的自定义行为（如属性查找、赋值、枚举、函数调用等）。

通俗的将，Proxy对象是目标对象的一个代理器，任何对目标对象的访问，都必须通过该代理器。因此我们可以对外界的访问进行过滤改写等操作。

```
let p = new Proxy(target, handler)
```

target-用Proxy包装的目标对象（可以是任何类型的对象，包括原生数组、函数，甚至另一个代理）。

handler-一个对象，其属性是当执行一个操作时定义代理的行为函数。

## 十四、class 类的继承

ES6 的class可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的class写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已

Class 可以通过extends关键字实现继承

```
class Point {}

class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 调用父类的constructor(x, y)
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的toString()
  }
}
```

## 十五、装饰器

decorator是一个函数，用来修改类甚至于是方法的行为。修饰器本质就是编译时执行的函数

装饰器是一种函数，写成@ + 函数名。它可以放在类和类方法的定义前面

```
@frozen class Foo {
  @configurable(false)
  @enumerable(true)
  method() {}

  @throttle(500)
  expensiveMethod() {}
}
```

```
@testable
class MyTestableClass {
  // ...
}

function testable(target) {
  target.isTestable = true;
}

MyTestableClass.isTestable // true
```

## 十六、import、export导入导出

ES6标准中，Js原生支持模块(module)。将JS代码分割成不同功能的小块进行模块化，将不同功能的代码分别写在不同文件中，各模块只需导出公共接口部分，然后通过模块的导入的方式可以在其他地方使用

```
// 只导入一个
import { sum } from "./example.js"

// 导入多个
import { sum, multiply, time } from "./exportExample.js"

// 导入一整个模块
import * as example from "./exportExample.js"

```

```
// 可以将export放在任何变量,函数或类声明的前面
export const Name = 'levin';
export const year = 2020;

// 也可以使用大括号指定所要输出的一组变量
const Name = 'levin';
const year = 1958;
export { Name, year };

// 使用export default时，对应的import语句不需要使用大括号
let addNum = function add() {}
export default addNum;
import moment from 'moment';

// 不使用export default时，对应的import语句需要使用大括号
let addNum = function add() {}
export addNum;
import { moment } from 'moment';
```
