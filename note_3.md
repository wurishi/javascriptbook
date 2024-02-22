# 第三部分：类型和语法

# 1. 类型

## 1.1 类型

## 1.2 内置类型

- 空值（null）
- 未定义（undefined）
- 布尔值（boolean）
- 数字（number）
- 字符串（string）
- 对象（object）
- 符号（symbol，ES6 新增）

```js
function a(b, c) {}
a.length // 2 表示该函数声明了两个命名参数 b,c
```

## 1.3 值和类型

JavaScript 中的变量是没有类型的，只有值才有。

- undefined 和 undeclared

  ```js
  var a
  typeof a === 'undefined'
  var b = 42
  var c
  b = c
  typeof b === 'undefined'
  typeof c === 'undefined'
  
  a; // undefined
  aa; // ReferenceError: aa is not defined
  typeof aa === 'undefined'
  ```

  在 javascript 中 `undefined` 表示在作用域中声明但还未赋值的变量。`undeclared` 则是还没有在作用域中声明过的变量。二者在使用时会有区别（前者的值是`undefined`，后者会抛出一个 `ReferenceError`），但在 `typeof` 时二者都是 `undefined`，这是因为 `typeof` 本身有一个特殊的安全防范机制。

- typeof Undeclared

  因为 `typeof` 的安全防范机制，在判断一些用户常量或内建 API 时就会非常有用：

  ```js
  if (DEBUG) {
      // 如果 DEBUG 没有定义就会报错
  }
  if (typeof DEBUG !== 'undefined') {
      // 这样是安全的
  }
  // 当然也可以通过检查全局变量是否在全局对象上
  if (window.DEBUG) {
      // 访问不存在的对象属性并不会报错
  }
  ```

# 总结

JavaScript 中有七种内置类型：`null, undefined, boolean, number, string, object, symbol`，可以通过 `typeof` 来查看，其中 `typeof null` 由于各种原因，结果是 `object`

JavaScript 中变量没有类型，值有类型。类型定义了值的行为特征。

`undefined` 和 `undeclared` 二者是有区别的，`typeof` 的安全防范机制导致它们的返回结果都是 `undefined`。`typeof` 的安全防范机制在某些情况下（判断用户常量，API 是否定义）还是不错的。

# 2. 值

## 2.1 数组

数组可以容纳任何类型的值，甚至是其他数组（多维数组）。数组声明后即可向其中加入值，不需要预先设定大小。

使用 `delete` 可以将单元从数组中删除，但数组的 `length` 不会发生改变。

数组通过数字进行索引，但因为数组是对象的子类型，所以也可以包含字符串键值的属性，但这种属性不会计算在数组长度内。特别注意的是如果字符串能够被强制类型转换为十进制数字的话，它就会作为数字索引进行处理。

一些 DOM 查询操作返回的 DOM 元素列表它们并非真正意义上的数组。`arguments` 对象将函数参数作为列表来访问。它们都是类数组，需要一些操作才能转换成数组：

```js
function foo() {
    var arr = Array.prototype.slice.call(arguments)
    arr // 数组
    // ES6
	var arr = Array.from(arguments)
}
```

## 2.2 字符串

## 2.3 数字

JavaScript 中的数字类型是基于 IEEE 754 标准来实现的浮点数，采用的是双精度格式即 64 位二进制。

- 数字的语法

```js
42.toFixed(3) // 是无效语法，因为 42. 会作为数字常量的一部分，所以没有.来调用 toFixed 方法
(42).toFixed(3) // 有效
0.42.toFixed(3) // 有效
42..toFixed(3) // 有效
42 .toFixed(3) // 有效，注意42和.之间的空格，不建议使用
```

```js
0xf3 // 243的十六进制
0Xf3 // 同上

// 非严格模式下
0363 // 243 的八进制
// 从ES6开始的严格模式下
0o363 // 243 的八进制
0O363 // 同上，注意第一位是数字0，第二位是字母（大写O/小写o）
```

- 较小的数值

  二进制浮点数最大的问题就是精度不够：

  ```js
  0.1 + 0.2 === 0.3 // false
  // 可以通过设置一个误差范围值，通常称为“机器精度”（machine epsilon），对于 JavaScript 来讲这个值通常是 2^-52（2.220446049250313e-16）
  // 从 ES6 开始该值定义在了 Number.EPSILON，ES6 之前的版本写 polyfill：
  if (!Number.EPSILON) {
      Number.EPSILON = Math.pow(2, -52)
  }
  // 然后通过这个误差值来比较两个数字是否相等
  function numbersCloseEnoughToEqual(n1, n2) {
      return Math.abs(n1 - n2) < Number.EPSILON
  }
  numbersCloseEnoughToEqual(0.1 + 0.2, 0.3) // true
  ```

  最大浮点数：`1.798e+308` 定义在 `Number.MAX_VALUE` 中

  最小浮点数：`5e-324` 它不是负数但无限接近于 0，定义在 `Number.MIN_VALUE` 中

- 整数的安全范围

  最大整数：`2^53-1 即 9007199254740991` 在 ES6 中被定义为 `Number.MAX_SAFE_INTEGER`

  最小整数：`-9007199254740991`  在 ES6 中被定义为 `Number.MIN_SAFE_INTEGER`

- 整数检测

  要检测一个值是否是整数：

  ```js
  // ES6
  Number.isInteger(42)
  // ES6 之前的 polyfill
  if (!Number.isInteger) {
      Number.isInteger = function(num) {
          return typeof num === 'number' && num % 1 == 0
      }
  }
  ```

  要检测一个值是否是安全的整数：

  ```js
  // ES6
  Number.isSafeInteger(Math.pow(2, 53) - 1)
  // ES6 之前的 polyfill
  if (!Number.isSafeInteger) {
      Number.isSafeInteger = function(num) {
          return Number.isInteger(num) && Math.abs(num) <= Number.MAX_SAFE_INTEGER
      }
  }
  ```

- 32 位有符号整数

  虽然整数最大能够达到 53 位，但是有些数字操作（如数位操作）只适用于 32 位数字，它的安全范围为：`Math.pow(-2, 31) (-2147483648 约-21亿) 到 Math.pow(2, 31) - 1 (2147483647 约21亿)`

  `a | 0` 可以将变量 a 中的数值转换为 32 位有符号整数。因为数位运算符 | 只适用于 32 位整数。

## 2.4 特殊数值

- 不是值的值

  `null / undefined`

- undefined

  在非严格模式下，可以为全局标识符 undefined 赋值。

  通过 `void` 运算符可以得到真正的 undefined。

  ```js
  var a = 42
  console.log(void a, a) // undefined 42
  ```

  按惯例可以用 `void 0` 获得 undefined，当然也可以使用其他的 `void true / void 1`

- 特殊的数字

  1. 不是数字的数字

     如果数学运算的操作数不是数字类型（或者无法解析为常规的十进制或十六进制）就会返回 `NaN`

     `NaN` 是一个特殊值，它和自身不相等，是唯一一个非自反（自反，reflexive，即 x===x 不成立）的值。而 `NaN != NaN` 为 true。

     如何判断值是 `NaN`：

     ```js
     // ES6
     Number.isNaN(n)
     // ES6 之前的 polyfill
     if (!Number.isNaN) {
         Number.isNaN = function(n) {
             return typeof n === 'number' && window.isNaN(n)
         }
     }
     ```

  2. 无穷数

     如果数学运算的结果超出处理范围（两数相加大于MAX_VALUE，或除以零）就会返回 `Infinity` （Number.POSITIVE_INfiNITY）。如果操作数中有一个负数，则结果为 `-Infinity` （Number.NEGATIVE_INfiNITY）

     ```js
     var a = Number.MAX_VALUE
     a + a // Infinity
     a + Math.pow(2, 970) // Infinity
     a + Math.pow(2, 969) // MAX_VALUE
     // 因为 IEEE 754 规范中的就近取整（round-to-nearest）模式，Number.MAX_VALUE + Math.pow(2, 969) 与 MAX_VALUE 更接近，所以它被向下取整（round down）。而 Number.MAX_VALUE + Math.pow(2, 970) 与 Infinity 更接近，所以它被向上取整（round up）
     Infinity / Infinity // 在 JavaScript 中是一个未定义的操作，结果为 NaN
     ```

  3. 零值

     在 JavaScript 中有一个常规的 0 （也叫作+0）和一个 -0。

     ```js
     var a = 0 / -3 // -0
     var b = 0 * -3 // -0
     // 加减法不会得到负零
     -0 == 0 // true
     -0 === 0 // true
     0 > -0 // false
     ```

     对 -0 进行字符串化会返回 "0"

     数字的符号位有时会被用来表示方向，所以即使变量的值到 0 了，也应该保留符号位以免方向信息的丢失。

- 特殊等式

  0 和 -0 如何作比较：

  ```js
  // ES6
  Object.is(-3 * 0, -0) // true
  Object.is(-3 * 0, 0) // false
  // ES6 之前的 polyfill
  if (!Object.is) {
      Object.is = function(v1, v2) {
          // 判断是否是 -0
          if (v1 === 0 && v2 === 0) {
              return 1 / v1 === 1 / v2
          }
          // 判断是否是 NaN
          if (v1 !== v1) {
              return v2 !== v2
          }
          // 其他情况
          return v1 === v2
      }
  }
  ```

## 2.5 值和引用

在 JavaScript 中没有语法来切换赋值和参数传递是通过值复制（value-copy）或是引用复制（reference-copy）来完成的。在 JavaScript 中值和引用赋值/传递只由值的类型来决定。

简单值（即标量基本类型值 scalar primitive） `null, undefined, string, number, boolean, symbol` 总是通过值复制的方式来赋值/传递。

复合值（compound value）对象（包括数组等）和函数则总是通过引用复制的方式来赋值/传递。

# 总结

JavaScript 中的数组是通过数字索引的一组任意类型的值。数字包括了“整数”和“浮点型”两种。

`undefined` 不是关键字，会被修改。`void` 运算符可以返回安全的 `undefined`。

数字类型有几个特殊值：`NaN`（意为 “not a number”，更确切地说是“invalid number”），`+Infinity, -Infinity, -0`

JavaScript 中变量赋值和传递都是根据值类型来决定是值复制的方式还是引用复制的方式。

# 3. 原生函数

常用的原生函数：

- String()
- Number()
- Boolean()
- Array()
- Object()
- Function()
- RegExp()
- Date()
- Error()
- Symbol()

要注意的是通过这些原生函数构造出来的对象，它们的类型都是对象类型的子类型，而非基本类型。

```js
var a = new String("abc")
typeof a // 是 object 而不是 String
a instanceOf String // true
Object.prototype.toString.call(a) // [object String]
```

## 3.1 内部属性[[Class]]

所有 typeof 返回值为 "object" 的对象（如数组）等都包含一个内部属性 [[Class]]。该属性无法直接访问，一般通过 `Object.prototype.toString()` 来查看。

```js
Object.prototype.toString.call([1, 2, 3]) // [object Array]
Object.prototype.toString.call(/regex-literal/i) // [object RegExp]

Object.prototype.toString.call(null) // [object Null]
Object.prototype.toString.call(undefined) // [object Undefined]
// 虽然 Null() 和 Undefined() 这样的原生构造函数不存在，但它们内部[[Class]]属性值仍然是 Null 和 Undefined

Object.prototype.toString.call('abc') // [object String]
Object.prototype.toString.call(42) // [object Number]
Object.prototype.toString.call(true) // [object Boolean]
// 基本类型会返回装箱（boxing）后的结果
```

## 3.2 封装对象包装（装箱）

基本类型值并没有 `.length` 或 `.toString()` 这样的属性和方法，JavaScript 会自动为基本类型值包装（box/wrap）成一个封装对象。

注意事项：

```js
var a = new Boolean(false)
if (!a) { // a 永远是真值
}
// 想要自动封装基本类型，可以使用 Object() 函数（不带 new 关键字）
var a = 'abc'
var b = new String(a)
var c = Object(a)

typeof a // string
typeof b // object
typeof c // object

b instanceof String // true
c instanceof String // true

Object.prototype.toString.call(b) // [object String]
Object.prototype.toString.call(c) // [object String]
```

## 3.3 拆封

想要得到封装对象中的基本类型值，可以使用 `valueOf()` 函数。

```js
var a = new String('abc')
var b = new Number(42)
var c = new Boolean(true)

a.valueOf() // 'abc'
b.valueOf() // 42
c.valueOf() // true

// 在需要用到封装对象中的基本类型值的地方会自动隐式拆封
var a = new String('abc')
var b = a + '' // 'abc'

typeof a // object
typeof b // string
```

## 3.4 原生函数作为构造函数

- Array()

  构造 Array() 不要求带 new 关键字。Array(1, 2, 3) 和 new Array(1, 2, 3) 的效果是一样的。

  ```js
  var a = new Array(3)
  var b = [undefined, undefined, undefined]
  var c = []
  c.length = 3
  // a, b 是不同的
  a.join('-') // '--'
  b.join('-') // '--'
  a.map((v, i) => i) // [undefined x3]
  b.map((v, i) => i) // [0, 1, 2]
  // 所以如果要创建包含 undefined 的数组，最好这样写：
  var a = Array.apply(null, { length: 3 })
  // Array() 第二个参数必须是一个数组，或者类似数组的值。{ length: 3 } 就是个类似数组的值，Array() 内部会遍历这个值的[0][1][2]...直到 length，但因为这个对象里面没有[0][1][2]，所以返回的就是 undefined
  ```

- Object() Function() RegExp()

  ```js
  var c = new Object()
  c.foo = 'bar'
  // 等效
  var d = { foo: 'bar' }
  ```

  ```js
  var e = new Function('a', 'return a * 2')
  // 等效
  var f = function(a) { return a * 2 }
  function g(a) { return a * 2 }
  ```

  ```js
  var h = new RegExp('^a*b+', 'g')
  // 等效
  var i = /^a*b+/g
  ```

- Date() Error()

  相较于其他原生构造函数，`Date() / Error()` 的用处要大的多，因为它们没有对应的常量形式来替代。

  Date() 主要用来获得当前的 Unix 时间戳（从 1970年1月1日 开始计算，以秒为单位），可以通过 `getTime()` 获得。

  ES5 开始可以通过静态函数 `Date.now()` 直接获得时间戳。

- Symbol()

  ES6 加入了一个新的基本数据类型：符号（Symbol）。符号是具有唯一性的特殊值。

  ES6 中有一些预定义符号，以 Symbol 的静态属性形式出现，如：`Symbol.create, Symbol.iterator`

  我们可以用 Symbol() 来自定义符号，比较特殊的是，它不能带 new 关键字。

- 原生原型

  原生构造函数有自己的 .prototype 对象，如 Array.prototype，String.prototype 等

  它们会提供一些方法给各自类型的值使用，如：`String#indexOf() Array#concat() Function#apply()` 等。但是有些原生原型会有一些特殊：

  ```js
  typeof Function.prototype // 'function'
  Function.prototype() // 空函数
  
  RegExp.prototype.toString() // '/(?:)/' 空正则表达式
  'abc'.match(RegExp.prototype) // ['']
  
  Array.isArray(Array.prototype) // true
  Array.prototype.push(1, 2, 3)
  Array.prototype // [1, 2, 3]
  ```

  这些未赋值的原型是很好的默认值，好处是 .prototype 已经被创建并且仅会创建一次。相反如果用 [] / function() {} / /(?:)/ 作为默认值 ，每次调用时都会创建一次，可能会造成垃圾回收。这样会造成内存和 CPU 资源的浪费。

# 总结

JavaScript 为基本数据类型提供了封装对象，称为原生函数。基本类型值在访问它的 .length 属性或 .prototype 上的方法时，引擎会自动对该值进行封装。

# 4. 强制类型转换

## 4.1 值类型转换

将值从一种类型转换为另一种类型通常称为类型转换（type casting），这是显式的情况；隐式的情况称为强制类型转换（coercion）

JavaScript 中的强制类型转换总是返回标量基本类型值，不会返回对象或函数。

也可以这样区分：类型转换发生在静态类型语言的编译阶段，而强制类型转换则发生在动态类型语言的运行时。

## 4.2 抽象值操作

- ToString

  ```js
  null -> "null"
  undefined -> "undefined"
  true -> "true"
  // 对于对象来说，除非自定义 toString()，否则返回内部属性 [[Class]] 的值，如 "[object Object]"
  ```

  JSON 字符串化并非严格意义上的强制类型转换

  ```js
  JSON.stringify(42) // "42"
  JSON.stringify("42") // ""42""
  JSON.stringify(null) // "null"
  JSON.stringify(true) // "true"
  // 在对象中遇到 undefined, function, symbol 时会自动忽略，在数组中则会返回 null 以保证单元位置不变。
  JSON.stringify(undefined) // undefined
  JSON.stringify(function() {}) // undefined
  JSON.stringify([1, undefined, function() {}, 4]) // "[1,null,null,4]"
  JSON.stringify({ a: 2, b: function() {} }) // "{"a":2}"
  ```

  如果对象定义了 `toJSON()` 方法，JSON 字符串化会首先调用该方法，然后用它的返回值进行序列化。

  如果要对含有非法 JSON 值的对象做字符串化，就需要定义 `toJSON()` 方法来返回一个安全的 JSON 值。

  ```js
  var o = {}
  var a = {
      b: 42, c: o, d: function() {}
  }
  o.e = a // 循环引用
  JSON.stringify(a) // 错误
  a.toJSON = function() {
      return { b: this.b } // 仅序列化 b
  }
  JSON.stringify(a) // "{"b":42}"
  ```

  `.toJSON()` 返回的是一个能够被字符串化安全的 JSON 值，而不是返回一个 JSON 字符串。

  ```js
  // 可以向 JSON.stringify() 传递一个可选参数 replacer，它可以是数组或函数，用来指定对象序列化过程骂她些属性应该被处理，哪些应该排除。
  var a = {
      b: 42,
      c: '42',
      d: [1, 2, 3]
  }
  JSON.stringify(a, ['b', 'c']) // "{"b":42,"c":"42"}"
  
  JSON.stringify(a, function(k, v) {
      if (k !== 'c') return v
  }) // "{"b":42,"c":"42"}"
  
  // 还有一个可选参数 space，用来指定输出的缩进格式。space 为正整数时是指定每一级缩进的字符数。它还可以是字符串，最前面的十个字符将被用于每一级的缩进。
  ```

- ToNumber

  ES5 规范在 9.3 节定义了抽象操作 ToNumber。

  其中 true 转换为 1，false 转换为 0。undefined 转换为 NaN，null 转换为 0。处理失败是返回 NaN，要注意的是对以 0 开头的十六进制数仍然会以十进制处理。

  对象（包括数组）会首先转换为相应的基本类型值，如果返回的是非数字的基本类型值，再遵循以上规则将其强制转换为数字：

  首先检查该值是否有 `valueOf()` 方法，如果有并且返回基本类型值，就使用该值强制类型转换。如果没有就使用 `toString()` 的返回值来进行强制类型转换。如果均不返回基本类型值，会产生 `TypeError` 错误。

  从 ES5 开始，使用 `Object.create(null)` 创建的对象 [[Prototype]] 属性为 null，并且没有 `valueOf()` 和 `toString()` 方法，因此无法进行强制类型转换。

- ToBoolean

  1. 假值（falsy value）

     JavaScript 中的值可以分为两类：

     A: 可以被强制类型转换为 false 的值

     ​	`undefined, null, false, +0, -0, NaN, ""`

     B: 其他（被强制类型转换为 true 的值）

  2. 假值对象（falsy object）

     ```js
     var a = new Boolean(false)
     var b = new Number(0)
     var c = new String('')
     // 它们都是封装了假值的对象，但它们本身其实都是 true
     var d = Boolean(a & b & c) // true
     ```
  3. 真值（truthy value）

## 4.3 显式强制类型转换

- 字符串和数字之间的显式转换

  ```js
  // 通过 `String()` 和 `Number()` 这两个内建函数来实现字符串和数字之间的转换。
  var a = 42
  var b = String(a) // '42'
  var c = '3.14'
  var d = Number(c) // 3.14
  
  // 通过 toString() 显式转换
  var a = 42
  var b = a.toString() // "42" 42会先封装成对象，再调用 toString()。所以显式转换中含有隐式转换。
  
  // 通过 + 一元运算符转换成数字
  var c = '42'
  var d = +c // 42
  
  // 通过 + 日期显式转换为数字
  var timestamp = +new Date() // number
  // 但是仍然建议使用显式转换的方法
  var timestamp = new Date().getTimer()
  // 或
  var timestamp = Date.now()
  
  // |
  0 | -0 // 0
  0 | NaN // 0
  0 | Infinity // 0
  0 | -Infinity // 0
  
  // ~ ~x大致等同于-(x+1)
  ~42 // -(42+1) ==> -43
  // -(x+1) 中唯一能得到 0 的 x 值是 -1，当 -1 作为哨位值时，如 indexOf，则可以结合 ~ 一起使用
  var a = 'Hello World'
  a.indexOf('lo') >= 0 // true
  a.indexOf('lo') != -1 // true
  a.indexOf('ol') < 0 // true
  a.indexOf('ol') == -1 // true
  // 现在可以这样写
  ~a.indexOf('lo') // true
  !~a.indexOf('ol') // true
  
  // ~~ 字位截除 ~~ 和 Math.floor() 类似可以截除数字的小数部分。
  // ~~ 只适用于 32 位数字，并且对负数的处理与 Math.floor() 不同
  Math.floor(-49.6) // -50
  ~~-49.6 // -49
  ```

- 显式解析数字字符串

  解析字符串中的数字和字符串强制类型转换为数字返回的结果都是数字，但两者还是有区别的：

  ```js
  var a = '42'
  var b = '42px'
  Number(a) // 42
  parseInt(a) // 42
  Number(b) // NaN
  parseInt(b) // 42
  ```

  解析允许字符串中含有非数字字符，解析按从左到右的顺序，遇到非数字字符就会停止。而转换不允许出现非数字字符，否则就会失败并返回 NaN。

  ES5 之前的 `parseInt()` 如果没有第二个参数用来指定转换基本，那它就会根据字符串的第一个字符来自行决定基数，即当遇到 "08" "09" 时，因为它们并不是有效的八进制数，则会被转换为 0。

- 显式转换为布尔值

  建议使用 `Boolean(a)` 或 `!!a` 来进行显式强制类型转换。

## 4.4 隐式强制类型转换

- 隐式地简化

- 字符串和数字之间的隐式强制类型转换

  如果 + 的其中一个操作数是字符串，则执行字符串拼接，否则执行数字加法。

- 布尔值到数字的隐式强制类型转换

  ```js
  function onlyOne() {
      var sum = 0
      for(var i=0; i<arguments.length;i++) {
          if (arguments[i]) {
              sum += arguments[i]
          }
      }
      return sum === 1
  }
  var a = true
  var b = false
  onlyOne(b, a) // true
  onlyOne(b, a, b, b, b) // true
  ```

- 隐式强制类型转换为布尔值

- || 和 &&

- 符号的强制类型转换

  ES6 允许从符号到字符串的显式强制类型转换，但隐式强制类型转换会产生错误。

  ```js
  var s1 = Symbol('cool')
  String(s1) // "Symbol(cool)"
  
  var s2 = Symbol('not cool')
  s2 + '' // TypeError
  ```

## 4.5 宽松相等和严格相等

== 允许在相等比较中进行强制类型转换，而 === 不允许。

- 相等比较操作的性能

- 抽象相等

  有几个非常规情况需要注意：

  1. NaN 不等于 NaN
  2. +0 等于 -0

  - 字符串和数字之间的相等比较

    ES5 规范 11.9.3.4-5 这样定义：

    （1） 如果 Type(x) 是数字，Type(y) 是字符串，则返回 x == ToNumber(y) 的结果。

    （2） 如果 Type(x) 是字符串，Type(y) 是数字，则返回 ToNumber(x) == y 的结果。

    ```js
    var a = 42
    var b = "42"
    
    a === b // false
    a == b // true
    ```

  - 其他类型和布尔类型之间的相等比较

    ES5 规范 11.9.3.6-7 定义：

    (1) 如果 Type(x) 是布尔类型，则返回 ToNumber(x) == y

    (2) 如果 Type(y) 是布尔类型，则返回 x == ToNumber(y)

    ```js
    true == '42' // false
    '42' == false // false
    // 要注意的是 '42' 即不等于 true 也不等于 false。但其实非空字符串是真值。只是在这里，会进行多次隐式转换：
    // true == '42' ==> 1 == '42' ==> 1 == 42 ==> false
    // '42' == false ==> '42' == 0 ==> 42 == 0 ==> false
    ```

  - null 和 undefined 之间的相等比较

    ES5 规范 11.9.3.2-3 规定：

    (1) 如果 x 为 null，y 为 undefined，则结果为 true

    (2) 如果 x 为 undefined，y 为 null，则结果为 true

  - 对象和非对象之间的相等比较

    ES5 规范 11.9.3.8-9 定义：

    (1) 如果 Type(x) 是字符串或数字，Type(y) 是对象，则返回 x == ToPromitive(y) 的结果。

    (2) 如果 Type(x) 是对象，Type(y) 是字符串或数字，则返回 ToPromitive(x) == y 的结果。

    ```js
    var a = 42
    var b = [42]
    a == b // [42] 调用 ToPromitive 抽象操作转换为 "42"，42 == "42" 再转换为 42 == 42
    
    var a = "abc"
    var b = Object(a)
    a === b // false
    a == b // true b 通过拆箱返回基本类型 "abc"
    ```

    但有一些例外：

    ```js
    var a = null
    var b = Object(a)
    a == b // false null 和 undefined 不能封装，所以 Object(null) 和 Object() 均返回一个常规对象
    
    var c = undefined
    var d = Object(c)
    c == d // false
    
    var e = NaN
    var f = Object(e)
    e == f // false NaN 能够被封装为数字封装对象，但在拆封之后因为 NaN 不等于 NaN，所以 NaN == NaN 返回 false
    ```

- 比较少见的情况

  (1) 通过修改 `.valueOf()` 返回其它数字

  ```js
  Number.prototype.valueOf = function() {
      return 3
  }
  new Number(2) == 3 // true
  2 == 3 // false 注意都是基本类型时不会调用 valueOf()
  ```

  (2) 假值的相等比较

  ```js
  '0' == null			// false
  '0' == undefined	// false
  '0' == false		// true !!!
  '0' == NaN			// false
  '0' == 0			// true
  '0' == ''			// false
  
  false == null		// false
  false == undefined	// false
  false == NaN		// false
  false == 0			// true !
  false == ''			// true !
  false == []			// true !
  false == {}			// false
  
  '' == null		// false
  '' == undefined	// false
  '' == NaN		// false
  '' == 0			// true !
  '' == []		// true !
  '' == {}		// false
  
  0 == null		// false
  0 == undefined	// false
  0 == NaN		// false
  0 == []			// true !
  0 == {}			// false
  ```

## 4.6 抽象关系比较

ES5 规范 11.8.5 定义了“抽象关系比较”，分为两个部分：比较双方都是字符串和其他情况。

```js
// 如果出现非字符串，就根据 ToNumber 规则将双方强制类型转换为数字来进行比较
var a = [42]
var b = ['43']
a < b // true
b < a // false

// 如果双方都是字符串，则以字符串比较法比较，
var a = ['42']
var b = ['043']
a < b // false a转换为'4,2'，b转换为'0,4,3'，然后 '0' 在字母顺序上小于 '4'
```

```js
var a = {b:42}
var b = {b:43}

a < b // false [object Object] < [object Object] 
a == b // false
a > b // false

a <= b // true 在 JavaScript 中 <= 是不大于的意思，即 !(a > b)，由上可以得到 !(false) => true
a >= b // true
```

# 总结

显式强制类型转换明确告诉我们哪里发生了类型转换，有助于提高代码可读性和可维护性。

隐式强制类型转换是其他操作（+ =）的副作用。

# 5. 语法

## 5.1 语句和表达式

在 JavaScript 中语句相当于句子，表达式相当于短语，运算符相当于标点和连接词。

```js
var a = 3 * 6; // 3 * 6 是一个表达式，整行是一个声明语句
var b = a; // a 是一个表达式，整行是一个声明语句
b; // b 也是一个表达式，同时整行也是一个语句，这种情况通常称为“表达式语句”
```

- 语句的结果值：

  语句都有一个结果值（statement completion value, undefined 也算）

  ```js
  var b;
  if (true) {
      b = 4 + 38;
  };
  // 上述代码块在调试工具中会显示42，即代码块的结果值如同一个隐式返回，会返回最后一个语句的结果值。
  
  // 但你无法主动的获取代码块的结果值，以下代码无法正常运行
  var a, b;
  a = if (true) {
      b = 4 + 38;
  };
  
  // 可以用 eval 获得结果值
  var a, b;
  a = eval("if (true) { b = 4 + 38; }");
  a; // 42
  
  // 或者使用 ES7 提案的 do 表达式
  var a, b;
  a = do {
      if (true) {
          b = 4 + 38;
      }
  };
  a; // 42
  ```

- 表达式的副作用

  大部分表达式是没有副作用的。但有一些例外：

  ```js
  // 1. 函数调用
  function foo() {
      a = a + 1;
  }
  var a = 1;
  foo(); // 结果值为 undefined，副作用为 a 的值被改变了。
  
  // 2. ++ --
  var a = 42;
  var b = a++; // 副作用为 a 的值为 43。
  // ++a++ 会产生 ReferenceError 错误的原因是，先执行 a++ 之后返回的是 42。即下一步会执行 ++42，此时就会产生错误。
  // 逗号运算符，即 b = (a++, a) 可以提供 ++a 的效果。
  
  // 3. delete
  var obj = {
      a: 42;
  }
  delete obj.a // true
  obj.a // undefined 副作用：属性被从对象中删除了。
  
  // 4. = += -=
  var a;
  a = 42; // 看起来 = 没有副作用，实际上它的结果值是 42，副作用：将 42 赋值给了 a。
  var a, b, c;
  a = b = c = 42; // 因为 = 有副作用，才能让这行代码正确运行。
  ```

- 上下文规则

  在 JavaScript 语法规则中，有时同样的语法在不同的情况下会有不同的解释：

  ```js
  // 1. 大括号
  var a = {
      foo: bar()
  }
  // {} 会被赋值给 a，因为此时大括号是用来定义对象常量的。
  {
      foo: bar()
  }
  // foo: 是标签语句（与goto配合使用）
  
  // 2. 代码块
  [] + {}; // "[object Object]", [] 强制转换成""，{} 这里被当作空对象强制转换成"[object Object]"
  {} + []; // 0, {} 被当作独立代码块
  
  // 3. 对象解构（ES6）
  var { a, b } = { a: 42, b: 'foo' };
  
  // 4. else if
  // JavaScript 中其实没有 else if
  if (a) {}
  else if (b) {}
  else {}
  // 会被转换为
  if (a) {}
  else {
      if (b) {}
      else {}
  }
  ```

## 5.2 运算符优先级

- 短路 `&& || `
- 更强的绑定
- 关联

## 5.3 自动分号

有时候 JavaScript 会自动为代码行补上缺失的分号，即自动分号插入（Automatic Semicolon Insertion, ASI）

## 5.4 错误

```js
var a = /+foo/; // 非法的正则表达式

var a;
42 = a; // 赋值对象必须是一个标识符

function foo(a, b, a) { "use strict"; } // ES5 严格模式下函数的参数不能重名

var a = {
    b: 42,
    b: 43
} // ES5 严格模式下不能包含多个同名属性
```

ES6 规范定义了一个新概念，叫作 TDZ（Temporal Dead Zone，暂时性死区）。

它指的是代码中的变量还没有初始化而不能被引用的情况。

```js
{
    a = 2; // ReferenceError
    let a;
}
// 对未声明的变量使用 typeof 不会生产错误，但在 TDZ 中却会出错
{
    typeof a; // undefined
    typeof b; // ReferenceError
    let b;
}
```

## 5.5 函数参数

## 5.6 try..finally

## 5.7 switch

# 总结

