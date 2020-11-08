## typeof

typeof 操作符返回一个字符串，表示未经计算的操作数的类型。

typeof对于原始类型来说，除了null都可以显示正确的类型
```
// 7个基本数据类型

typeof 10; // "number"
typeof '10'; //"string"
typeof true; // "boolean"
typeof null; // "object"
typeof undefined; // "undefined"
typeof Symbol(); // "symbol"
typeof 42n; // "bigint"
```

typeof对于对象类型，除了函数都会显示object，所以说**typeof并不能准确判断变量到底是什么类型**
```
// object类型
typeof {}; // "object"
typeof []; // "object"
typeof /^$/; // "object"
typeof new Date(); // "object"
typeof function() {}; // 'function';
typeof class C {}; // 'function'
typeof Math.sin; // 'function';
```

*在 JavaScript 最初的实现中，JavaScript 中的值是由一个表示类型的标签和实际数据值表示的。对象的类型标签是 0。由于 null 代表的是空指针（大多数平台下值为 0x00），因此，null 的类型标签是 0，typeof null 也因此返回 "object"。*

typeof返回的类型都是字符串形式，需注意:
```
console.log(typeof a == "string"); // true
console.log(typeof a == "String"); // false
```

## instanceof

instanceof 运算符用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上。

instanceof 后面一定要是对象类型
```
a instanceof Array
d instanceof Date
f instanceof Function
```

对于原始类型，我们想直接通过instanceof来判断类型是不行的，当然我们可以转换一下:
```
class PrimitiveString {
    static [Symbol.hasInstance](x) {
        return typeof x === 'string'
    }
}

console.log('hello' instanceof PrimitiveString) // true
```

Symbol.hasInstance就是一个能让我们自定义instanceof行为的东西

instanceof 只能用来判断两个对象是否属于实例关系， 而不能准确判断一个对象实例具体属于哪种类型。

## Object.prototype.toString

toString() 是 Object 的原型方法，调用该方法，默认返回当前对象的 [[Class]] 。这是一个内部属性，其格式为 [object Xxx] ，其中 Xxx 就是对象的类型。

对于 Object 对象，直接调用 toString()  就能返回 [object Object] 。而对于其他对象，则需要通过 call / apply 来调用才能返回正确的类型信息。

```
Object.prototype.toString.call('') ;   // [object String]
Object.prototype.toString.call(1) ;    // [object Number]
Object.prototype.toString.call(true) ; // [object Boolean]
Object.prototype.toString.call(undefined) ; // [object Undefined]
Object.prototype.toString.call(null) ; // [object Null]
Object.prototype.toString.call(Symbol()); //[object Symbol]
Object.prototype.toString.call(42n); //[object BigInt]

Object.prototype.toString.call(new Function()) ; // [object Function]
Object.prototype.toString.call(new Date()) ; // [object Date]
Object.prototype.toString.call([]) ; // [object Array]
Object.prototype.toString.call(new RegExp()) ; // [object RegExp]
Object.prototype.toString.call(new Error()) ; // [object Error]
Object.prototype.toString.call(document) ; // [object HTMLDocument]
Object.prototype.toString.call(window) ; //[object global] window 是全局对象 global 的引用
```

## 根据对象的constructor判断

constructor是原型prototype的一个属性，当函数被定义时候，js引擎会为函数添加原型prototype，并且这个prototype中constructor属性指向函数引用， 因此重写prototype会丢失原来的constructor。

```
a.constructor === Array;  // true
d.constructor === Date;   // true
f..constructor === Function;  // true

// constructor 在类继承时会出错
function A(){}
function B(){}
A.prototype = new B(); //A继承自B
var aObj = new A();
aobj.constructor === B;  // true
aobj.constructor === A;  // false

// instanceof方法不会出现该问题，对象直接继承和间接继承的都会报true
aobj instanceof B;  // true
aobj instanceof A;  // true

// 解决construtor的问题通常是让对象的constructor手动指向自己 aobj.constructor = A
```

存在的问题：

- null 和 undefined 无constructor，这种方法判断不了
- 如果自定义对象，开发者重写prototype之后，原有的constructor会丢失
