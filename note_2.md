# 第二部分：this和对象原型

# 1. 关于 this

## 1.1 为什么要用 this

this 提供了一种更优雅的方式来隐式传递一个对象的引用，这样可以将 API 设计得更加简洁并且易于复用。

## 1.2 误解

- 指向自身
- 它的作用域

## 1.3 this 到底是什么

this 是在运行时绑定的，而非在编写时绑定。它的上下文取决于函数调用时的各种条件。this 的绑定和函数的位置没有任何关系，只取决于函数的调用方式。

当一个函数被调用时，会创建一个执行上下文。它会包含函数是在哪里被调用的（调用栈），函数的调用方式，传入的参数信息等。this 就是这个上下文中的一个属性，在函数执行过程中会被用到。

# 总结

this 实际上是在函数被调用时发生绑定的，它指向什么完全取决于函数在哪里被调用。

# 2.  this 全面解析

## 2.1 调用位置

寻找调用位置就是寻找“函数被调用的位置”，最重要的就是分析调用栈。

## 2.2 绑定规则

- 默认绑定
  最常用的函数调用类型：独立函数调用。可以把这条规则看作是无法应用其他规则时的默认规则。
  非严格模式下，函数独立调用时会应用 this 的默认绑定，因此 this 指向全局对象。
  在严格模式（strict mode）下，不能将全局对象用于默认绑定，因此 this 会绑定到 undefined。
- 隐式绑定
  当函数作为一个对象的属性时，通过"对象.函数"调用时，隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象。

  隐式丢失（隐式丢失常见于回调函数丢失）：
  ```js
  function foo() {
      console.log(this.a)
  }
  var obj = { a:2, foo: foo }
  obj.foo() // 2
  var bar = obj.foo
  var a = 0
  bar() // 0 发生了隐式丢失
  ```

- 显式绑定
  通过函数的 `call()` 或 `apply()` 调用时，第一个参数就可以指定 this。
  硬绑定：通过函数的 `bind()` 会返回一个硬绑定后的新函数。
- new 绑定
  使用 new 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作：
  1. 创建（或者说构造）一个全新的对象。
  2. 这个新对象会被执行 [[Prototype]] 连接。
  3. 这个新对象会绑定到函数调用的 this。
  4. 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象。

## 2.3 优先级

	1. 函数是否在 new 中调用（new 绑定）？如果是的话 this 绑定的是新创建的对象。
 	2. 函数是否通过 call, apply 显式绑定或硬绑定调用（显式绑定）？如果是的话，this 绑定的是指定对象。
 	3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this 绑定的是那个上下文对象。
 	4. 如果都不是，则使用默认绑定。如果在严格模式下，就绑定到 undefined，否则绑定到全局对象。

## 2.4 绑定例外

- 被忽略的 this
  如果把 null, undefined 作为 this 绑定对象传入 call, apply 或 bind。那么这个值将被忽略，实际应用的是默认绑定规则。
  但这样做可能会带来一些副作用，如果某个函数确实使用了 this，但使用默认绑定规则将 this 绑定到了全局对象，可能会导致许多难以分析和追踪的 bug。
  更安全的作法是传入一个特殊的不会对程序产生任何副作用的对象，称之为 DMZ（demilitarized zone）。

  ```js
  function foo(a, b) { console.log("a:" + a + ", b:" + b) }
  var ø = Object.create(null)
  // 把数组展开成参数
  foo.apply(ø, [2, 3])
  // 使用 bind 进行柯里化
  var bar = foo.bind(ø, 2)
  bar(3)
  ```

  使用变量名 ø 不仅让函数更安全，而且可以提高代码的可读性，因为 ø 表示“我希望 this 是空”，这比 null 的含义更清楚。
- 间接引用
  间接引用最容易在赋值时发生：

  ```js
  function foo() { console.log(this.a) }
  var a = 2
  var o = { a: 3, foo: foo }
  var p = { a: 4 }
  o.foo() // 3
  (p.foo = o.foo)() // 2
  ```

  赋值表达式 `p.foo = o.foo` 的返回值是目标函数的引用，因此在这个位置调用函数将是 `foo()` 而不是 `p.foo() / o.foo()`，所以这里会触发默认绑定。
  要注意的是，对于默认绑定而言，决定 this 绑定对象的并不是调用位置是否处于严格模式，而是函数体是否处于严格模式。

- 软绑定

  ```js
  if (Function.prototype.softBind) {
      Function.prototype.softBind = function(obj) {
          var fn = this
          var curried = [].slice.call(arguments, 1)
          var bound = function() {
              return fn.apply((!this || this === (window || global)) ? obj : this, curried.concat.apply(curried, arguments))
          }
          bound.prototype = Object.create(fn.prototype)
          return bound
      }
  }
  
  function foo() { console.log("name: " + this.name) }
  var obj = { name: 'obj' }, obj2 = { name: 'obj2' }, obj3 = { name: 'obj3' }
  var fooOBJ = foo.softBind(obj)
  fooOBJ() // obj
  obj2.foo = foo.softBind(obj)
  obj2.foo() // obj2
  fooOBJ.call(obj3) // obj3
  setTimeout(obj2.foo, 10) // obj
  ```

## 2.5 this 词法

  在 ES6 中有一种无法使用以上规则的特殊函数类型：箭头函数

# 总结

  如果要判断一个运行中函数的 this 绑定，就需要先找到这个函数直接调用的位置，然后按顺序应用下面这四条规则来判断 this 的绑定对象

  1. 由 new 调用？绑定到新创建的对象。
  2. 由 call / apply / bind 调用？绑定到指定的对象。
  3. 由上下文对象调用？绑定到这个上下文对象。
  4. 默认：在严格模式下绑定到 undefined，否则绑定到全局对象。

  另外在 ES6 中的箭头函数并不会使用以上四条绑定规则，而是根据当前的词法作用域来决定 this的。具体来说，箭头函数会继承外层函数调用的 this 绑定。

# 3. 对象

## 3.1 语法

对象可以通过两种形式定义：声明形式和构造形式

```js
// 声明（文字）形式
var myObj = {
    key: value,
}
// 构造形式
var myObj2 = new Object()
myObj2.key = value
```

## 3.2 类型

JavaScript 中一共有六种主要类型：

- string
- number
- boolean
- null
- undefined
- object

其中简单基本类型（string, number, boolean, null, undefined）本身并不是对象（`typeof null === 'object'` 但这是因为对于 JavaScript 而言二进制前三位都为 0 就会被判断为 object 类型，null 的二进制形式全是 0，自然也会被 `typeof` 判断为是对象）

JavaScript 中有许多特殊的对象子类型，可以称之为复杂基本类型。比如说函数，数组等。

通常称之为内置对象，有些内置对象的名字看起来和简单基础类型相似：

- String
- Number
- Boolean
- Object
- Function
- Array
- Date
- RegExp
- Error

这些内置对象实际上只是 JavaScript 中的一些内置函数，通过对它们使用 new 调用来实现构造函数调用，以创建一个对应的子类型的新对象。

## 3.3 内容

对象的内容是由一些存储在特定命名位置的任意类型的值组成的，一般称之为属性。

- `.`操作符
    通常被称作“属性访问”，只能接受满足标识符命名规范的属性名。
- `[]`操作符
    通常被称作“键访问”，可以接受任意 UTF-8/Unicode 字符串作为属性名。
    另外因为接受的是字符串，所以这个键可以在程序中构造如：`var idx = 'a'; obj[idx] === obj['a'];`
    1. 可计算属性名
        ```js
        var prefix = 'foo'
        var obj = {
            [prefix + 'bar']: 'hello',
            [prefix + 'baz']: 'world'
        }
        obj['foobar'] // hello
        obj['foobaz'] // world
        ```
    2. 属性与方法
        从技术角度来说，函数永远不会属于一个对象，因为函数的 this 绑定存在自己的一整套规则。
    3. 数组
        数组也支持 `[]` 访问，不过期望的是数值下标，也称作为值存储的位置/索引（整数）。
    4. 复制对象
    5. 属性描述符
        从 ES5 开始，所有属性都具备了属性描述符。
        ```js
        var myObject = { a: 2 }
        Object.getOwnPropertyDescriptor(myObject, 'a')
        // value: 2 （数据值）
        // writable: true （可写）
        // enumerable: true （可枚举）
        // configurable: true （可配置）
        
        // 可以通过 defineProperty 来修改

        // writable: false 时，myObject.a = 3 会报错

        // configurable: false 时，再次调用 defineProperty 会报错，也就是说 configurable 修改成 false 是单向操作。
        // 例外：configurable: false 时，还是可以将 writable 由 true 改为 false，但是无法由 false 改为 true。
        // 另外 configurable: false 时，调用 delete myObject.a 也会报错

        // enumerable: false 时，属性仍然可以正常访问，但是无法通过 for...in 等遍历对象的属性枚举时访问到。
        ```
    6. 不变性
        创建的都是浅不变性，即只会影响目标对象和它的直接属性。
        - 对象常量
            结合 `writable: false` 和 `configurable: false` 就可以创建一个真正的常量属性（不可修改，重定义或删除）
        - 禁止扩展
            如果想禁止一个对象添加新的属性并且保留已有属性，可以使用 `Object.preventExtensions()`
        - 密封
            `Object.seal()` 会创建一个密封的对象，这个方法实际上会在对象上调用 `Object.preventExtensions()`，并把所有现有属性标记为 `configurable: false`
            密封之后对象不能添加新属性，也不能重新置或删除任何现有属性，但是可以修改属性的值。
        - 冻结
            `Object.freeze()` 会创建一个冻结对象，这个方法实际上会在对象上调用 `Object.seal()` 并把所有“数据访问”属性标记为 `writable: false`。
            这个方法可以禁止对对象及其任何直接属性进行修改。
    7. [[Get]]
        属性访问并不仅仅是从对象上查找属性，其实是在对象上执行了 [[Get]]() 调用。
    8. [[Put]]
        和 [[Get]] 对应的，对属性值进行操作会执行 [[Put]]() 调用，它会大致进行如下的检查：
        - 属性是否是访问描述符？如果是并且存在 setter 就调用 setter。
        - 属性的数据描述符中 wriable 是否为 false？如果是，在非严格模式下静默失败，在严格模式下抛出 TypeError 异常。
        - 如果都不是，将该值设置为属性的值。
    9. Getter 和 Setter
        在 ES5 中可以使用 getter 和 setter 部分改写默认操作，但只能应用在单个属性上。
    10. 存在性
        使用 `hasOwnProperty()` 或 `in` 可以在不访问属性值的情况下判断对象中是否存在这个属性。
        
        - 枚举
    
## 3.4 遍历

`for...in` 可以用来遍历对象的可枚举属性列表，那么如何遍历属性的值呢？

for 循环严格意义上来说并不是在遍历值，它只是遍历下标然后指向值。
ES5 中增加了一些数组的辅助迭代器，包括：`forEach / every / some`
ES6 增加了一种用来遍历数组的 `for...of` 循环语法，数组有内置的 `@@iterator`，`for...of` 其实就是遍历它。

# 总结

JavaScript 中的对象有字面形式（`var a = {}`）和构造形式（`var a = new Object()`）。

JavaScript 中对象是6个基础类型之一，内置的子对象类型其实是提供了内置构造函数。

对象就是键/值对的集合。可以通过`.`操作或`[]`语法来获取属性值。

属性的特性由属性描述符来控制，比如 `writable / configurable`。可以使用 `Object.preventExtensions() / Object.seal() / Object.freeze()` 来设置对象的不可变性级别。

属性的可枚举/不可枚举性决定了属性是否会出现在 `for...in` 循环中。

可以使用 ES6 的 `for...of` 语法来遍历数据结构。它会寻找内置或自定义的 `@@iterator` 对象并调用它的 `next()` 方法来遍历数据值。

# 4. 混合对象“类”

## 4.1 类理论

- “类”设计模式
    迭代器模式，观察者模式，工厂模式，单例模式等是面向对象中常见的设计模式。对于函数式编程而言，类本身也是一种设计模式。
- JavaScript 中的“类”
    JavaScript 中只有一些近似的语法元素（比如 new 和 instanceOf），ES6 中还添加了关键字 `class`。但是它们都只是 JavaScript 提供的设计类的语法。

## 4.2 类的机制

- 建造
    从类创建实例
- 构造函数
    构造函数属于类，并且通常和类同名。大多都需要和 new 一起使用。

## 4.3 类的继承

在面向类的语言中，你可以先定义一个类，然后定义一个继承前者的类。

- 多态
    多态并不表示子类和父类有关联，子类得到的只是父类的副本，其实就是复制。
- 多重继承
    多重继承的一个问题是“钻石问题”，即子类 D 继承自两个父类（B，C），这两个类又继承自 A。如果 A 中有 `drive()` 方法并且 B 和 C 都重写了这个方法（多态），那么 D 应该选择哪个版本呢？
    JavaScript 本身并不提供“多重继承”功能。

## 4.4 混入

在继承或实例化时，JavaScript 的对象机制并不会自动执行复制行为，它们只是会被关联起来。由于其他语言中类表现出来的都是复制行为，因此 JavaScript 开发者想出了一个方法模拟类的复制行为，这个方法就是混入。

- 显式混入
    1. 多态
    ```js
    function mixin(sourceObj, targetObj) {
        for(var key in sourceObj) {
            if (!(key in targetObj)) {
                targetObj[key] = sourceObj[key]
            }
        }
        return targetObj
    }
    var A = {
        drive: function() {}
    }
    var B = mixin(A, {
        drive: function() {
            A.drive.call(this) // 直接调用A.drive()，会让函数中的 this 绑定到 A上。
        }
    })
    ```
    2. 混合复制
    ```js
    function mixin(sourceObj, targetObj) {
        for (var key in sourceObj) {
            // 去掉存在性检查
            targetObj[key] = sourceObj[key]
        }
        return targetObj
    }
    ```
    3. 寄生继承
    ```js
    function A() {}
    A.prototype.drive = function() {}
    function B() {
        var a = new A();
        // 进行定制
        a.w = 4;
        // 保存 A 的方法
        var aDrive = a.drive;
        // 重写方法
        a.drive = function() {
            aDrive.call(this)
        };
        return a;
    }
    ```
- 隐式混入
    ```js
    var A = {
        cool: function() {}
    }
    var B = {
        cool: function() {
            A.cool.call(this)
        }
    }
    ```

# 总结

类是一种设计模式，许多语言提供了面向类软件设计的原生语法。JavaScript 也有类似的语法，但是和其他语言中的类完全不同。

类意味着复制。

传统的类被实例化时，它的行为会被复制到实例中。类被继承时，行为也会被复制到子类中。

多态看起来似乎是从子类引用父类，但是本质上引用的其实是复制的结果。

JavaScript 并不会自动创建对象的副本。

混入模式可以用来模拟类的复制行为，但较难维护。

# 5. 原型

## 5.1 [[Prototype]]

JavaScript 中的对象有一个特殊的 [[Prototype]] 内置属性，其实就是对其他对象的引用。

- Object.prototype
    所有普通的 [[Prototype]] 链最终都会指向内置的 `Object.prototype`。它包含 JavaScript 中许多通用的功能，比如：`.toString() .valueOf() .hasOwnProperty()` 等
- 属性设置和屏蔽
    1. 如果在链上层存在属性，并且没有被标记为只读，那就会在当前对象添加一个同名的新属性，它是屏敝属性。
    2. 如果在链上层存在只读属性，那将无法修改或在当前对象创建屏敝属性。如果运行在严格模式下，代码会抛出一个错误。否则赋值语句将会被忽略。
    3. 如果在链上层存在一个对应属性的 setter，那就一定会调用这个 setter。不会创建屏敝属性。
    有些情况下会隐式产生屏蔽
    ```js
    var foo = { a: 2 }
    var bar = Object.create(foo)
    foo.a // 2
    bar.a // 2
    foo.hasOwnProperty('a') // true
    bar.hasOwnProperty('a') // false
    bar.a++ // 隐式屏蔽！
    // 尽管 bar.a++ 看起来应该是查找到增加了 foo.a，但实际上这个操作等于 bar.a = bar.a + 1。因此 ++ 操作会先通过 [[Prototype]] 查找属性 a，并从 foo.a 获取当前属性值2，然后加1，接着用 [[Put]] 将3赋值给 bar 中新创建的屏敝属性 a。
    foo.a // 2
    bar.a // 3
    bar.hasOwnProperty('a') // true
    ```

## 5.2 类

- 类函数

- 构造函数

  在 JavaScript 中对于构造函数最准确的解释是：所有带 new 的函数调用。

- 技术

## 5.3 原型继承

错误方法：

```js
Bar.prototype = Foo.prototype
// 这样并不会创建一个关联到 Bar.prototype 的新对象，它只是让 Bar.prototype 直接引用 Foo.prototype 对象。因此当你执行诸如 Bar.prototype.fn = ... 时，实际上修改的是 Foo.prototype 本身。

Bar.prototype = new Foo()
// 虽然确实会创建一个关联到 Bar.prototype 的新对象。但是因为使用了 Foo() 构造函数调用，如果函数 Foo 中有一些副作用（写日志，修改状态，注册其他对象，给 this 添加属性时），就会影响到 Bar() 的后代。
```

推荐方法：

```js
// ES6 之前抛弃默认的 prototype
Bar.prototype = Object.create(Foo.prototype)
// ES6 开始可以直接修改现有的 Bar.prototype
Object.setPrototypeOf(Bar.prototype, Foo.prototype)
```

```js
function Foo() {}
Foo.prototype.blah = {}

var a = new Foo()
// 怎样检查类关系？

// 1.
a instanceOf Foo; // true 只能处理对象(a)和函数(带 .prototype 引用的 Foo) 之间的关系
// 2.
Foo.prototype.isPrototypeOf(a) // true 在 a 的整条[[Prototype]]链中是否出现过 Foo.prototype
// 3. ES5 中的标准方法：
Object.getPrototypeOf(a) === Foo.prototype // true
// 4. 绝大多数浏览器支持的一种非标准的方法来访问内部[[Prototype]]属性
a.__proto__ === Foo.prototype // true __proto__ 在ES6之前并不是标准
```

## 5.4 对象关联

如果在对象上没有找到需要的属性或方法引用，引擎就继续在[[Prototype]]关联的对象上进行查找。同理，如果在后者中也没有找到就会继续查找它的[[Prototype]]，以此类推。这一系列对象的链接被称为“原型链”。

- 创建关联

  `Object.create()` 会创建一个新对象并把它关联到指定的对象。

  `Object.create(null)` 会创建一个拥有空[[Prototype]]链接的对象，这个对象无法进行委托。由于这个对象没有原型链，所以 `instanceOf` 总是返回 false。这种对象通常被称作为“字典”，它们完全不受原型链的干扰，适合用来存储数据。

  `Object.create()` 是 ES5 中新增的函数，在 ES5 之前的环境中，如果要支持这个功能就需要使用一段简单的 polyfill 代码：

  ```js
  if (!Object.create) {
      Object.create = function(o) {
          function F() {}
          F.prototype = o
          return new F()
      }
  }
  ```

  

- 关联关系是备用

  原型机制让对象上无法处理的属性和方法可以使用备用的，但如果出于 API 的考虑，使用内部委托，给出更明确的 API 定义会更加清晰：

  ```js
  var anotherObject = {
      cool: function() {}
  }
  var myObject = Object.create(anotherObject)
  myObject.doCool = function() {
      this.cool() // 内部委托
  }
  ```

# 总结

要访问对象中并不存在的一个属性，[[Get]]操作就会查找对象内部[[Prototype]]关联的对象。这个关联关系实际上定义了一条“原型链”，在查找属性时会对它进行遍历。

所有普通对象都有内置的 Object.prototype，指向原型链的顶端。`toString() valueOf()` 和其他一些通用的功能都存在于 Object.prototype 对象上，因此语言中所有的对象都可以使用这些方法。

关联两个对象最常用的方法是使用 new 关键词进行函数调用。

尽管 JavaScript 中的机制和传统面向类语言中的“类初始化”和“类继承”很相似。但在 JavaScript 中有一个核心区别，那就是不会进行复制，对象之间是通过内部的原型链关联的。所以比起”继承“，”委托“是一个更适合的术语。

# 6. 行为委托

## 6.1 面向委托的设计

- 类理论

  类设计模式鼓励你在继承时使用方法重写（和多态）。

- 委托理论

  委托意味着某些对象找不到的属性或方法时，会把这个请求委托给另一个对象。

  1. 禁止互相委托
  2. 调试

- 比较思维模型

## 6.2 类与对象

- 控件类

  ```js
  // 父类
  function Widget(width, height) {}
  Widget.prototype.render = function($where) {}
  
  // 子类
  function Button(width, height, label) {
      // 调用 super 构造函数
      Widget.call(this, width, height)
  }
  // Button 继承 Widget
  Button.prototype = Object.create(Widget.prototype)
  // 重写
  Button.prototype.render = function($where) {
      // super 调用
      Widget.prototype.render.call(this, $where)
  }
  Button.prototype.onClick = function(evt) {}
  ```

  ES6 class 语法糖

  ```js
  class Widget {
      constructor(width, height) {}
      render($where) {}
  }
  class Button extends Widget {
      constructor(width, height, label) {
          super(width, height)
      }
      render($where) {
          super($where)
      }
      onClick(evt) {}
  }
  ```

  要注意的是 ES6 的 class 只是语法糖，内部仍然使用的是原型链机制。

- 委托控件对象

  ```js
  var Widget = {
      init: function(width, height) {},
      insert: function($where) {}
  }
  var Button = Object.create(Widget)
  // 委托调用
  Button.setup = function(width, height, label) {
      this.init(width, height)
  }
  Button.build = function($where) {
      this.insert($where)
  }
  Button.onClick = function(evt) {}
  ```

  委托相比较类的设计在实例对象时会有一些区别：

  ```js
  var btn1 = new Button()
  
  var btn2 = Object.create(Button)
  btn2.setup()
  ```

  但是往往在类的设计时，也会被要求把构造和初始化分开。常见的是实例池功能。

## 6.3 更简洁的设计

## 6.4 更好的语法

## 6.5 内省

# 总结

行为委托认为对象之间是兄弟关系，而不是父类和子类的关系。

# 附录A: ES6 中的 Class

## A.1 class

ES6 中的 class 解决了以下问题：

- 基本上不再引用杂乱的 .prototype 了
- 继承直接使用关键字 `extends`，不再需要通过 `Object.create()` 来替换 .prototype 对象，也不需要设置 `__proto__` 或 `Object.setPrototypeOf()`
- 可以通过 `super()` 来实现相对多态
- `class` 字面语法不能声明属性，只能声明方法。
- 可以通过 `extends` 很自然地扩展内置对象类型，比如：Array 或 RegExp 等。

## A.2 class 陷阱

如果修改或替换了父类中的一个方法，那么子类和所有实例也都会影响，因为它们在定义时并没有进行复制。

```js
class C {
    constructor() {
        this.num = Math.random()
    }
    rand() {
        console.log(this.num)
    }
}
var c1 = new C()
c1.rand() // 0.432
C.prototype.rand = function() {
    console.log(this.num * 1000)
}
var c2 = new C()
c2.rand() // 867
c1.rand() // 432
```

class 语法无法定义类成员属性，如果要在实例之间共享状态，就必须使用 .prototype 语法：

```js
class C {
    constructor() {
        C.prototype.count++
        console.log(this.count)
    }
}
C.prototype.count = 0
var c1 = new C() // 1
var c2 = new C() // 2
c1.count === 2
c1.count === c2.count
```

class 仍然面临意外屏蔽的问题：

```js
class C {
    constructor(id) {
        // id 属性屏蔽了 id() 方法
        this.id = id
    }
    id() {
        console.log(this.id)
    }
}
var c1 = new C('c1')
c1.id() // TypeError -- c1.id 现在是字符串 'c1'
```

super 也存在一些非常细微的问题

出于性能考虑，super 并不是动态绑定的，它会在声明时静态绑定。所以可能（TC39）需要用 `toMethod()` 来手动绑定 super。

```js
class P {
    foo() { console.log('P') }
}
class C extends P {
    foo() { super() }
}
var c1 = new C()
c1.foo() // P

var D = {
    foo: function() { console.log('D') }
}
var E = {
    foo: C.prototype.foo
}
// 把 E 委托到 D
Object.setPrototypeOf(E, D)
E.foo() // P

// toMethod 手动委托
var E = Object.create(D)
E.foo = C.prototype.foo.toMethod(E, 'foo')
E.foo() // D
```

## A.3 静态大于动态吗

ES6 的 class 似乎是想告诉我们，动态太难实现了，这里有一种看起来像静态的语法，所以编写静态代码吧。

# 总结

class 很好地伪装成 JavaScript 中类和继承设计模式的解决方案，它隐藏了许多问题但也带来了更多更小但更危险的问题。