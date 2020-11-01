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

## 四、set、map数据结构

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

## 五、this的指向

[参考](./JS的this指向.md)
