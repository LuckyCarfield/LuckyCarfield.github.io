---
layout: post
title:  "JavaScript基础-let&&const"
date:   2021-07-14 11:29:08 +0800
categories: jekyll update
---
# let && const 
## 1. let
### 基本用法
- let声明的变量仅在所属的代码块内有效
```javascript
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```
变量i是let声明的，当前的i只在本轮循环有效，所以每一次循环的i其实都是一个新的变量，所以最后输出的是6。你可能会问，如果每一轮循环的变量i都是重新声明的，那它怎么知道上一轮循环的值，从而计算出本轮循环的值？这是因为 JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量i时，就在上一轮循环的基础上进行计算。

另外，for循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。
- 不存在变量提升

var命令会发生“变量提升”现象，即变量可以在声明之前使用，值为undefined。let命令改变了语法行为，它所声明的变量一定要在声明后使用，否则报错。


- 暂时性死区（temporal dead zone，简称 TDZ）
```javascript
console.log(aVar) // undefined
console.log(aLet) // causes ReferenceError: aLet is notdefined
var aVar = 1
let aLet = 2
```
为什么出现*暂时性死区*？

ES6 标准中对 let/const 声明中的解释 第13章，有如下一段文字：

The variables are created when their containing Lexical Environment is instantiated but may not be accessed inany way until the variable’s LexicalBinding is evaluated.
  当程序的控制流程在新的作用域（module function 或 block 作用域）进行实例化时，在此作用域中用let/const声明的变量会先在作用域中被创建出来，但因此时还未进行词法绑定，所以是不能被访问的，如果访问就会抛出错误。因此，在这运行流程进入作用域创建变量，到变量可以被访问之间的这一段时间，就称之为暂时死区。

例如:

```javascript
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```
这是因为只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响。而在实例化作用域时，let声明的变量已经创建，只是未进行lexicalbindin evaluate
- 不允许重复声明
```javascript
// 报错
function func() {
  let a = 10;
  var a = 1;
}
```
因此，不能在函数内部重新声明参数。
```javascript
// 报错
function func(arg) {
  let arg;
}
func() // 报错

function func(arg) {
  {
    let arg;
  }
}
func() // 不报错
```
## 2. 块级作用域
- 为什么需要块级作用域？

ES5 只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景。

第一种场景，内层变量可能会覆盖外层变量。var声明的变量自动提升到函数作用域。
```javascript
var tmp = new Date();

function f() {
  console.log(tmp);
  if (false) {
    var tmp = 'hello world';//
  }
}

f(); // undefined
```
第二种场景，用来计数的循环变量泄露为全局变量。
```javascript
var s = 'hello';

for (var i = 0; i < s.length; i++) {
  console.log(s[i]);
}

console.log(i); // 5
```
- ES6 的块级作用域
块级作用域的出现，实际上使得获得广泛应用的匿名立即执行函数表达式（匿名 IIFE）不再必要了。
```
// IIFE 写法
(function () {
  var tmp = ...;
  ...
}());

// 块级作用域写法
{
  let tmp = ...;
  ...
}
```
- 块级作用域与函数声明
ES5 规定，函数只能在顶层作用域和函数作用域之中声明，不能在块级作用域声明；ES6 引入了块级作用域，明确允许在块级作用域之中声明函数。但是为了减轻因此产生的不兼容问题，ES6 在附录 B里面规定，浏览器的实现可以不遵守上面的规定，有自己的行为方式。

    - 允许在块级作用域内声明函数。

    - 函数声明类似于var，即会提升到全局作用域或函数作用域的头部。
    
    - 同时，函数声明还会提升到所在的块级作用域的头部。

    - 函数声明和变量声明都会被提升。但是有一个需要注意的细节是函数会首先被提升，然后才是变量。
    - 函数表达式并不会提升
```javascript
console.log(say); //输出：[Function: say]

function say(){
  console.log('1');
};

var say = '2';

console.log(say); //输出'2'
```
预编译阶段javascript引擎会先执行一遍扫描，将函数和变量var变量声提升到全局作用域或函数作用域顶部，上述代码执行效果等同于：
```javascript
function (){ //函数声明（包括定义）提升
  console.log('1');
};

错误版本 var say; //只是声明，并不会覆盖say的值

console.log(say); //故输出：[Function: say]


正确版本及位置var say = '2'; //因为函数会首先被提升，当提升var say时，因为重复声明被编译器忽略，直到执行位置后面的声明覆盖了前面的。

console.log(say); //输出'2'
```
- 块级作用域内部默认的变量（非var，let和const修饰）
```javascript
{
    console.log(a); // undefined
    var a = 10;
    console.log(b); //Uncaught ReferenceError: b is not defined
    b = 10;
    console.log(c); //Uncaught ReferenceError: Cannot access 'c' before initialization
    let c = 10;
}
```
在块级作用域内部声明的默认变量(不适用let,var,const修饰),只有等到执行过你定义那个变量的那行代码后才可以访问，才给window赋值这个属性，在那行代码之前访问会报错;块内的默认变量依旧是全局变量;在块内的默认变量没执行之前不可以访问这个变量
- ES6 的块级作用域必须有大括号

如果没有大括号，JavaScript 引擎就认为不存在块级作用域。
```javascript
// 第一种写法，报错
if (true) let x = 1;

// 第二种写法，不报错
if (true) {
  let x = 1;
}
```
## 3. const
**const本质：** const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，const只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。因此，将一个对象声明为常量必须非常小心。
```javascript
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only
```
如果真的想将对象冻结，应该使用Object.freeze方法。
```javascript
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```
除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数。
```javascript
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, i) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] ); //或者arguments.callee正在执行函数的指针(arguments.callee已经弃用)
    }
  });
};
```
- ES6 声明变量的六种方法

ES5 只有两种声明变量的方法：var命令和function命令。ES6 除了添加let和const命令，后面章节还会提到，另外两种声明变量的方法：import命令和class命令。所以，ES6 一共有 6 种声明变量的方法。