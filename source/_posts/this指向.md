---
title: 重学JavaScript - this 指向
date: 2021-03-25 23:55:44
categories:
  - 重学JavaScript
---

## this 机制

this 是执行上下文的一部分，this 的指向取决与函数的调用方式，它的指向是当函数调用的时候才决定的，**JS 中 this 的指向机制其实是函数内部有一个私有的属性 [[thisMode]]**

[[thisMode]] 包含：

- lexical：表示从上下文中找 this，这对应了箭头函数。
- global：表示当 this 为 undefined 时，取全局对象，对应了普通函数。
- strict：当严格模式时使用，this 严格按照调用时传入的值，可能为 null 或者 undefined。

## this 指向规则

### 默认规则

```javascript
var a = 1;
function foo() {
  console.log(this.a);
}
foo(); // >> 1
```

函数独立调用是, 在非严格模式下 this 指向 Global/window，在严格模式下 this 指向 undefined

### 方法调用

所谓方法就是一个函数作为一个对象上的属性

```javascript
var a = 3
function foo(){
  var a = 2
	console.log(this.a)
}

var o = {
  a: 1
  foo: foo
}
o.foo() // >> 1
```

如果是该函数是一个对象的方法，则它的 this 指针指向这个对象

### 构造函数

```javascript
class A{
	this.name = 'hello'
	foo(){
    console.log(this.name)
	}
}
const a = new A()
a.foo() // >> hello
```

如果是该函数是一个构造函数，this 指针指向一个新的对象

### 显示指向

JavaScript 内置对象`Function`的三个原型方法`call()`、`apply()`和`bind()`，它们的第一个参数是一个对象，它们会把这个对象绑定到`this`，接着在调用函数时让`this`指向这个对象。

```javascript
function foo() {
  var a = 2;
  console.log(this.a);
}

var o = {
  a: 1,
};

foo.call(o);

foo(); // >> 2
```

如果参数传入 null 或者 undefined ，那么 this 就是指向全局

## 特殊情况

### 隐式丢失

```javascript
var a = 3
function foo(){
  var a = 2
	console.log(this.a)
}

var o = {
  a: 1
  foo: foo
}
var bar = o.foo

bar() // 3
```

虽然 bar 是 o.foo 的引用，但是在这里，他相当于被独立调用，所以应该使用默认规则，this 指向全局

### 箭头函数

this 指向继承于他的外面第一个不是箭头函数的函数

```javascript
function foo() {
  return () => {
    console.log(this);
  };
}
foo()(); // >> Global
```
