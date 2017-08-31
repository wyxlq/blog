---
title: 'JavaScript中的new运算符'
date: 2017-08-31 15:31:31
tags:
---
    
一般来说，js中new这个关键字应该是不会陌生的，通过new我们创建一个对象。最近比较好奇new操作符到底做了些什么事情。
```
function Car() {}
Car.prototype.wheel = 4

car1 = new Car()
console.log(car1.wheel) //4
```
在上面的例子中，我们通过new创建了一个Car的实例。实际上面的代码也可以这么写：
```
function Car() {}
Car.prototype.wheel = 4

car1 = new Car
console.log(car1.wheel) //4
```
这个结果是一样的。然后再看下面的例子
```
function Car() {
    this.wheel = 4
    return null
}

car1 = new Car()
console.log(car1) //{wheel : 4}

function Shop() {
    this.wheel = 0
    return 1
}

shop1 = new Shop()
console.log(shop1) //{wheel : 0}

function Bike() {
    this.wheel = 2
    return {}
}

bike1 = new Bike()
console.log(bike1) //{}
```
这里，我们给构造函数增加了一个返回操作（JavaScript里面的函数即使你不写返回也会默认返回一个undefined）。结果发现，但构造函数返回类型不一样的时候new操作的结果发生了变化。然后我查阅了相关的文档，原来new操作实际上相当于做了一下事情：

```
var a = new Foo()
```

* 创建一个空对象。var o = {}
* 将新对象继承自构造函数的prototype。 o.__proto__ = Foo.prototye
* 将构造函数的this指向这个对象，并执行它。 Foo.call(o)
* 返回对象o。注意，如果构造函数返回了一个非null的对象（自然包括数组），那么这里将返回这个构造函数返回的对象，而不是刚刚创建的o。

下面用代码简单地模拟一下这个过程：
```
function factory(){
    var Constructor = [].shift.call(arguments)
    var o = {}
    o.__proto__ = Constructor.prototype
    var res = Constructor.apply(o, arguments)
    return res && typeof res === 'object' ? res : o
}
```
我们重新执行一下上面的例子，结果是不是一样的呢？
```
function factory(){
    var Constructor = [].shift.call(arguments)
    var o = {}
    o.__proto__ = Constructor.prototype
    var res = Constructor.apply(o, arguments)
    return res && typeof res === 'object' ? res : o
}

function Car() {
    this.wheel = 4
    return null
}

car1 = factory(Car) 
console.log(car1) //{wheel : 4}

function Shop() {
    this.wheel = 0
    return 1
}

shop1 = factory(Shop) 
console.log(shop1) //{wheel : 0}

function Bike() {
    this.wheel = 2
    return {}
}

bike1 = factory(Bike) 
console.log(bike1) //{}
```