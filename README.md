## Javascript



### 作用域和闭包

#### 引擎查询

LHS/RHS：变量在赋值操作左边(LHS)、在赋值操作右边(RHS)

```js
a = 2 //LHS
b = a // ..=a RHS
```

嵌套作用域：作用域发生嵌套，引擎会在外层作用域中继续查找，直到找到或者抵达全局作用域。

#### 异常

RHS中若找不到，则抛出**ReferenceError**

LHS找不到，则会在全局作用域中创建该变量，**注意！**若采用**'use strict'** (ES5中引入) 则不会创建，同样抛出**ReferenceError**

RHS若查找到但进行了不合理操作，则抛出**TypeError**

#### 查找

**”遮蔽效应“**：作用域查找会在找到第一个匹配的标识符时停止。逐级向上查找。

全局变量会自动成为全局对象，例如window.a

#### 欺骗词法

1. eval

```js
function foo(str,a) {
  eval( str )
  console.log( a, b );
}
var b = 2;
foo( "var b = 3;" 1 ); // 1, 3 遮蔽了外部同名变量b=2
```

通过代码欺骗和假装**词法期**来修改**词法作用域环境**。

在严格模式中，eval有其自己的词法作用域，无法修改所在的作用域：

```js
function foo(str) {
  "use strict";
  eval( str );
  console.log( a ); //ReferenceError: a is not defined
}
foo( "var a = 2" )
```

2. with

重复引用同一个对象中多个属性的快捷方式

```js
var obj = {
  a: 1,
  b: 2,
  c: 3
}
//可通过with(...)
with(obj) {
  a = 3;
  b = 4;
  c = 5
}
```

同时还可以用于赋值

```js
function foo(obj) {
  with(obj) {
    a = 2;
  }
}

var o1 = {
  a: 3
};

var o2 = {
  b: 3
};

foo( o1 );
console.log( o1.a ); //2

foo( o2 );
console.log( o2.a ); //undefined
console.log( a ); //a被泄露到了全局作用域
```

上述两种方式会存在运行效率问题，同时不推荐使用，在严格模式下会被限制。



#### 函数作用域

通过最小暴露原则进行设计，可以实现内容私有化，规避作用域冲突。

```js
function doSomething(a) {
  function doSomethingElse(a) { //内部函数
    return a - 1;
  }
  var b; //内部变量
  b = a + doSomethingElse( a * 2);
  console.log( b * 3);
  
}
doSomething( 2 )
```

通过内部定义可以隐藏变量和函数定义，但是会导致额外的**作用域污染**

```js
//...全局作用域
function foo() {
}
foo(); //具名函数foo()污染了全局作用域

//可通过包装函数来解决作用域污染
(function foo(){ //函数会当做函数表达式来处理
  ...
})(); //第二个()表示立即执行 IIFE(Immediately Invoked Function Expression)
```



#### 匿名和具名

匿名函数会导致

1. 调试困难
2. 引用自身时需要使用已过期的arguments.callee
3. 可读性差

```js
setTimeout( function() { //匿名函数
  ...
},1000)

setTimeout( function timeoutHandler() { //行内函数表达式
  ...
},1000)
```



#### 立即执行函数表达式

两种方式：

```js
(function foo(){ .. })()
(function(){ .. }())
```

IIFE可以当做函数并传递参数进去

```js
(function IIFE( global ){ //将window参数名global传入
  var a = 3;
  console.log( a ); //3
  console.log( global.a ); //2
})( window );
console.log( a ); //2
```

可以解决undefined默认标识符被错误覆盖导致的异常

```js
undefined = true; //错误赋值

(function IIFE( undefined ){
  var a;
  if(a === undefined){
    console.log("Undefined is safe here!");
  }
})(); //undefined
```

可以倒置代码运行顺序，UMD中广泛使用：

```js
var a = 2;
(function IIFE( def ){
  def( window );
})(function def( global ){ //将window作为global在def中使用
  var a = 3;
  console.log( a ); //3
  console.log( global.a ); //2
});
```



#### 块作用域

在for中，定义的变量i会被绑定在**外部作用于中**！！！

```js
for (var i=0; i<10; i++) {
  console.log( i );
}
```

##### 1.with

##### **2.try/catch**

从ES3规范中catch分句中会创建一个块作用域，仅在catch中有效

```js
try {
  undefined(); //创造异常
}
catch (err) {
  console.log( err );
}
console.log( err ) //ReferenceError
```

###### ES6之前的块作用域替代方案

```js
// 在ES6中可以利用块作用域
{
  let a = 2;
  console.log( a ); // 2
}
console.log( a ); // ReferenceError

// ES6之前，try/catch中的catch是块作用域，从而实现。
try{throw 2;}catch(a){
  console.log( a ); // 2
}
console.log( a ); // ReferenceError
```



##### **3.let**

let形成了一个块作用域

```js
var foo = true;
if(foo) {
  { // <--显式的块
    let bar = foo * 2;
    bar = something( bar );
    console.log( bar );
  }
}
console.log( bar ); //ReferenceError
```

(1) 垃圾收集

```js
function process(data) {
}
var someReallyBigData = { .. }
process( someReallyBigData );
var btn = document.getElementById( "my_button" );
btn.addEventListener( "click", function click(evt){
  console.log("button clicked");
}, /*capturingPhase*/false );
```

click形成了一个覆盖整个作用域的闭包，会导致someReallyBigData无法回收。可通过块作用域解决：

```js
{ //块中定义内容可以销毁
  let someReallyBigData = { .. };
  process( someReallyBigData )
}
```

(2) let循环

let不仅将i绑定到了for循坏的块中，事实上将其重新绑定到了循环的每一个迭代中重新赋值。

```js
{
  let j;
  for (j=0; j<10; j++){
    let i = j //每个迭代重新绑定
    console.log( i );
  }
}
```

###### let和var的区别

1. var存在变量提升，let不存在

2. let不允许在相同作用域内，重复声明同一个变量

3. 暂时性死区

   ```js
   // 在代码块内，使用let声明前该变量均不可用
   //暂时性死区 temporal dead zone TDZ
   var tmp = 123;
   if(true){
     tmp = 'abc'; //ReferenceError
     let tmp;
   }
   ```

   

4. 213

5. 



##### 4.const

const为固定常量，也可以用来创建块作用域变量。



#### 提升

##### 1.变量提升

变量和函数的所有声明会在任何代码被执行前首先被处理。

范例1：

```js
// 代码
a = 2;
var a;
console.log( a );
// 执行顺序
var a;
a = 2;
console.log( a ); //2
```

范例2：

```js
// 代码
console.log( a );
var a = 2;
// 执行顺序
var a;
console.log( a ); //undefined
a = 2;
```

范例3：

```js
foo(); //不是ReferenceError，而是TypeError
var foo = functon bar() {
  // ...
};
```

foo变量定义被提升，但并未赋值，对于undefined进行函数调用抛出TypeError异常。

范例4：

```js
foo(); //TypeError
bar(); //ReferenceError
var foo = function bar() {
  // ...
};

// 实际提升后
var foo;
foo(); //TypeError
bar(); //ReferenceError
foo = function(){
  var bar = ...self...
  // ...
}
```

##### 2.函数优先

函数声明和变量声明都被提升，但是函数优先级高

````js
// var foo和function foo声明重复而被忽略，函数声明被提到普通变量之前。同时出现在后面的函数会覆盖前面的,最终输出3
foo(); //3
var foo;
function foo(){
  console.log( 1 );
}
foo = function(){ //2
  console.log( 2 );
};
function foo(){
  console.log( 3 );
};

//引擎理解形式如下：
function foo() {
  console.log( 3 );
}
foo(); //3
foo = function(){
  console.log( 2 );
};
````



#### 闭包

当函数可以记住并访问所在的词法作用域，即使函数实在当前词法作用域之外执行，这时就产生了闭包。

范例1：

```js
//bar()在自己定义的词法作用域以外执行，它拥有涵盖foo()内部作用域的闭包，使该作用于一直存活。
function foo() {
  var a = 2;
  function bar(){
    console.log( a );
  }
  return bar;
}
bar baz = foo();
baz(); //闭包
```

范例2：

```js
function foo(){
  var a = 2;
  function baz(){
    console.log( a ); //2
  }
  bar( baz )
}
function bar(fn){
  fn(); //闭包。 fn即baz拥有foo()的整个作用域，访问变量a
}
foo();
```

范例3：

```js
//timer涵盖wait(..)的作用域闭包，传递给setTimeout(..),同时还保留对变量message的引用
function wait(message){
  setTimeout( function timer(){
    console.log( message )
  }, 1000);
}
wait( "Hello,closure!" )
```

##### 循环

```js
//延迟函数的回调会在循环结束时才执行。
for(var i=1;i<=5;i++){
  setTimeout(function timer(){
    console.log(i); //6 6 6 6 6
  },i*1000);
}

//将i作为参数传递给IIFE解决问题。
for(var i=1;i<=5;i++){
  (function(j)
  	setTimeout(function timer(){
      console.log( j ); //1 2 3 4 5
    }, j*1000 );
  )( i );
}

//进一步的
for(let i=1; i<=5; i++){
  setTimeout( function timer(){
    console.log( i );
  }, i*1000 );
}
```

##### 模块

以下模式称为**模块**，实现模块方式的方法称为模块暴露

模块模式的必要条件：

1. 必须有外部的封闭函数，该函数被至少调用一次
2. 封闭函数返回一个内部函数，这样内部函数才能在私有作用域中形成闭包，并可以访问或者修改私有状态。

```js
function CoolModule(){
  var something = "cool";
  var another = [1, 2, 3];
  function doSomething(){
    console.log( something );
  }
  function doAnother(){
    console.log( another.join( " ! ") );
  }
  return{
    doSomething: doSomething,
    doAnother: doAnother
  };
}
var foo = CoolModule();

foo.doSomething(); //cool
foo.doAnother(); // 1 ! 2 ! 3
```

单例模式：

```js
var foo = (function CoolModule(){
  var something = "cool";
  var another = [1, 2, 3];
  function doSomething(){
    console.log( something );
  }
  function doAnother(){
    console.log( another.join( " ! ") );
  }
  return {
    doSomething: doSomething,
    doAnother: doAnother
  }
})(); //立即调用函数IIFE

foo.doSomething();
foo.doAnother();
```

###### 现代模块机制

通过封装进API，实现模块依赖加载器/管理器

```js
var MyModules = (function Manager(){
  var modules = {};
  function define(name, deps, impl){
    for(var i=0; i<deps.length; i++){
      deps[i] = module[deps[i]];
    }
    modules[name] = impl.apply( impl, deps );
  }
  function get(name){
    return modules[name];
  }
  return {
    define: define,
    get: get
  }
})();
```

使用方法：

```js
MyModules.define("bar",[],function(){
  function hello(who){
    return "Let me introduce: " + who;
  }
  return {
    hello: hello
  };
});

MyModules.define("foo",["bar"],function(bar){
  var hungry = "hippo";
  function awesome(){
    console.log( bar.hello( hungry ).toUpperCase());
  }
  return {
    awesome: awesome
  };
});

var bar = MyModules.get( "bar" );
var foo = MyMudules.get( "foo" );

console.log( 
  bar.hello("hippo")
); //Let me introduce: hippo
foo.awesome(); // LET ME INTRODUCE: HIPPO
```

###### 未来的模块机制

ES6增加一级语法支持，导入或导出其他模块或特定的API成员，模块需要定义在**独立文件中**，没有**“行内”**格式。

```js
//bar.js
function hello(who){
  return "Let me introduce: " + who;
}
export hello;
//foo.js
import hello from "bar";
var hungry = "hippo";
function awesome(){
  console.log(
    hello( hungry ).toUpperCase()
  );
}
export awesome;
//baz.js
module foo from "foo";
module bar from "bar";
console.log(
	bar.hello("rhino")
); //let me introduce: rhino
foo.awesome(); //LET ME INTRODUCE: HIPPO
```



### this解析

调用栈中的调用位置，决定了this的绑定

```js
function baz() {
  // 当前调用栈是：baz
  // 因此，当前调用位置是全局作用域
  console.log( "baz" );
  bar(); // <-- bar 的调用位置
}

function bar() {
  // 当前调用栈是 baz -> bar
  // 因此，当前调用位置在 baz 中
  console.log( "bar" );
  foo(); // <-- foo 的调用位置
}

function foo() {
  // 当前调用栈是 baz -> bar -> foo
  // 因此，当前调用位置在 bar 中
  console.log( "foo" );
}

baz(); // <-- baz 的调用位置
```

#### 绑定规则

##### 默认绑定

决定this绑定对象的并不是调用位置是否处于严格模式，而是函数体是否处于严格模式，从而this绑定到undefined或者全局对象上

```js
//this的默认绑定指向全局对象
function foo(){
  console.log( this.a )
}
var a =2;
foo(); // 2

//在严格模式下，全局对象无法使用默认绑定，此处的this会绑定到undefined
function foo(){
  "use strict"
  console.log( this.a )
}
var a = 2;
foo(); // TypeError: this is undefined

//虽然this的绑定取决于调用位置，但是只有foo()运行在非strict mode下时，默认绑定才能绑定到全局对象；严格模式下与foo()的调用位置无关。
function foo() {
  console.log( this.a );
}
var a = 2;
(function(){
  "use strict";
  foo(); // 2
})();
```

##### 隐式绑定

对象属性引用链中只有最顶层或者说最后一层会影响调用位置。

```js
function foo() {
  console.log( this.a );
}
var obj2 = {
  a: 42,
  foo: foo
};
var obj1 = {
  a: 2,
  obj2: obj2
};
obj1.obj2.foo(); // 42
```

###### 隐式丢失

被隐式绑定的函数会丢失绑定对象，应用默认绑定，将this绑定到全局对象或者undefined上。 

```js
function foo() {
  console.log( this.a );
}
function doFoo(fn) {
  // fn 其实引用的是 foo
  fn(); // <-- 调用位置！
}
var obj = {
  a: 2,
  foo: foo
};
var a = "oops, global"; // a 是全局对象的属性
doFoo( obj.foo ); // "oops, global"

//调用语言内置函数没有区别
setTimeout( obj,foo, 100); // "oops, global"
```

参数传递也是一种隐式赋值

```js
function foo() {
  console.log( this.a );
}
function doFoo(fn) {
  // fn 其实引用的是
  foo fn(); // <-- 调用位置！
}
var obj = {
  a: 2,
  foo: foo
};
var a = "oops, global"; // a 是全局对象的属性
doFoo( obj.foo ); // "oops, global"
```

##### 显式绑定

可通过call(..)和apply(..)进行显式绑定，但显式绑定无法解决丢失绑定问题。

```js
function foo(){
  console.log( this.a );
}
var obj = {
  a:2
};
foo.call( obj ); //2
//通过.call(..)强制将this绑定到obj上
```

当传入一个原始值来当做this的绑定对象，这个原始值会被转换成它的对象形式( new String(..)、new Boolean(..)或者new Number(..))。这成为“装箱”。

###### 硬绑定

显式的强制绑定，无论如何调用函数bar，都会手动在obj上调用foo

```js
function foo() {
  console.log( this.a );
}
var obj = {
  a:2
};
var bar = function() {
  foo.call( obj );
};
bar(); // 2
setTimeout( bar, 100 ); // 2
// 硬绑定的 bar 不可能再修改它的 this
bar.call( window ); // 2
```

硬绑定的典型应用场景就是创建包裹函数，传入所有的参数并返回接收到的所有值

```js
var bar = function(){
  return foo.apply( obj, arguments );
}

//也可以创建一个可复用的辅助函数
function bind(fn, obj){
  return function(){
    return fn.apply( obj, arguments );
  }
}
...
var bar = bind( foo, obj );
var b = bar( 3 );
```

由于硬绑定的常用性，在ES5中提供了内置的方法Function.prototype.bind，使用.bind(..)会返回一个硬编码的新函数，并将参数设置为this的上下文并调用原始函数。

###### API调用的“上下文”

第三方库或者JavaScrpit语言和宿主环境中的许多新的内置函数，都提可选参数，被称为“上下文”(context)，确保回调函数使用指定的this

```js
[1, 2, 3].forEach( foo, obj ); //把this绑定到obj
```

##### new绑定

构建新对象并绑定到调用的this上

```js
function foo(a) {
this.a = a; }
var bar = new foo(2);
console.log( bar.a ); // 2
```

##### 优先级

new优先级 > 显式优先级

```js
function foo(p1,p2) {
this.val = p1 + p2; }
// 之所以使用 null 是因为在本例中我们并不关心硬绑定的 this 是什么
// 反正使用 new 时 this 会被修改
var bar = foo.bind( null, "p1" );
var baz = new bar( "p2" );
baz.val; // p1p2
```

###### 判断this

1. new绑定，this绑定的是新创建的对象

   ```js
   var bar = new foo()
   ```

2. call、apply (显式绑定)，this绑定是指定的对象

   ```js
   var bar = foo.call(obj2)
   ```

3. 上下文对象调用(隐式绑定)，this绑定的是上下文对象。

   ```js
   var bar = obj1.foo()
   ```

4. 如果都不是，使用默认绑定。在严格模式下绑定undefined，否则绑定到全局对象

   ```js
   var bar = foo()
   ```

#### 绑定例外

##### 忽略this

把null或者undefined作为this的绑定对象传入call、apply或者bind，应用默认规则绑定。
**应用场景**：使用apply(..)来展开一个数组，并当做参数传入函数。bind(..)可以对参数进行柯里化(预先设置一些参数)。
但是默认规则会把**this绑定到全局对象**，会导致全局对象修改等问题、

```js
function foo(a,b){
  console.log( "a:" + a + ", b:" + b );
}
//把数组展开成参数
foo.apply( null, [2, 3] ); // a:2, b:3

//使用bind(..)进行柯里化
var bar = foo.bind( null, 2);
bar( 3 ); // a:2, b:3
```

###### **更安全的this**

可以传入特殊对象，把this绑定到DMZ(demilitarized zone，非军事区)

```js
//创建了一个空对象，但是相比{}，不会创建Object.prototype这个委托
var ø = Object.create( null ); //创建一个DMZ对象
foo.apply( ø, [..] );
```

##### 间接引用

```js
function foo() {
  console.log( this.a );
}
var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };
o.foo(); // 3
(p.foo = o.foo)(); // 2
//p.foo = o.foo 返回值是目标函数的引用，调用的是foo()而不是p.foo()或者o.foo()，因此这里应用的是默认绑定。
```

##### 软绑定

硬绑定后无法通过显式或者隐式来修改this，可以给默认绑定指定一个全局对象或者undefined以外的值来实现硬绑定相同效果，同时保留修改this的能力。

```js
if (!Function.prototype.softBind) {
  Function.prototype.softBind = function(obj) {
    var fn = this;
    // 捕获所有 curried 参数
    var curried = [].slice.call( arguments, 1 );
    var bound = function() {
      return fn.apply(
        (!this || this === (window || global)) ?
        	obj : this
        curried.concat.apply( curried, arguments )
      );
    };
    bound.prototype = Object.create( fn.prototype );
    return bound;
  };
}
```

上述代码会检查调用时的this，如果是全局对象或者undefined，就把指定的默认对象obj绑定到this，佛足额不会修改this

```js
function foo() {
  console.log("name: " + this.name);
}
var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };
var fooOBJ = foo.softBind( obj );
fooOBJ(); // name: obj
obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2 <---- 看！！！
fooOBJ.call( obj3 ); // name: obj3 <---- 看！
setTimeout( obj2.foo, 10 );
// name: obj <---- 应用了软绑定
```

#### this词法

箭头函数根据外层（函数或全局）作用域来决定this，箭头函数的绑定无法被修改。

```js
function foo(){
  return (a) => {
    //this继承自foo()
    console.log( this.a )
  };
}
var obj1 = { a:2 };
var obj2 = { a:3 };

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2
```



### 对象

#### 类型

JavaScript共有六种主要类型：string、number、boolean、null、undefined、object

前5种简单基本类型不是对象，null有时会被当做一种对象，但是其实这是语言本身的一个bug，即对null执行typeof null会返回字符串"object"。（JavaScript中二进制前三位为0会被判断为object，**null的二进制全是0**，所以执行typeof会返回"object"）

##### 内置对象

对象子类型被称为内置对象：

String、Number、Boolean、Object、Function、Array、Date、RegExp、Error

这些内置函数可以当做构造函数来使用（new产生的函数调用）

```js
//此处只是一个字面量不可变，若进行操作需要转换成String对象
//语言会在必要时自动把字符串转换成一个String对象
var strPrimitive = "I am a string";
typeof strPrimitive; // "string"
strPrimitive instanceof String; // false

var strObject = new String( "I am a string" );
typeof strObject; // "object"
strObject instanceof String; // true

// 检查 sub-type 对象
Object.prototype.toString.call( strObject ); // [object String]
```



