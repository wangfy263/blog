---
title: javascript实现继承的几种方式
date: 2018-12-31 09:32:38
tags:
---

背景
---
近期的项目需要兼容IE8，很多ES5、ES6的属性无法使用；其中一部分用到了一些继承的思想，发现自己对js实现继承的几种方式理解还不够透彻，因此花了一晚上的时间，基于《javascript高级程序设计》和《javascript语言精粹》还有ES6的extends方法，重新梳理了所有的继承方式，以及他们的优缺点。

原型链继承
---
原型链继承，借助原型链，将父类的属性和方法共享给子类；
实现很简单：子类原型指向父类实例。
```js
function SuperType(){
	this.property = true;
}
SuperType.prototype.getSuperValue = function(){
	return this.property;
}
function SubType(){
	this.subproperty = false;
}
SubType.prototype = new SuperType();
var instance = new SubType();
```
instance可以访问到property属性和getSuperValue方法

### 缺点
1. 父类中的属性是所有子类实例共享的，如父类中有引用类型，例如数组，那么其中一个子类实例修改了数组元素，会影响到所有的子类实例；
2. 父类的属性是在父类实例化时已经确定，子类实例化时无法向父类够高函数传参；

借用构造函数
---
为了规避原型链继承的缺点，可以在子类构造函数中，借助父类的构造函数，把this指向子类构造函数
```js
function SubType(){
	SuperType.call(this)
}
```
父类属性会随着构造函数被调用，加在子类的实例上（这样就不会出父类属性共享的问题，因为属性是子类实例自己的属性了）
#### new
这里的实现依据是new操作符的行为，当对构造函数执行new操作符时，行为如下(以new SubType()为例)：
```js
var obj = {};
obj.__proto__ = SubType.prototype;
SubType.call(obj);
return obj;
```
### 缺点
只能继承属性，在原型链上的方法不可见。

组合继承
---
将原型链继承与借用构造函数继承组合在一起，既能共享原型链的属性，同时还能避免父类属性共享的问题。
```js
function SuperType(){
	this.property = true;
}
SuperType.prototype.getSuperValue = function(){
	return this.property;
}
function SubType(){
	SuperType.call(this);
	this.subproperty = false;
}
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;  // 这里需要重置构造器
var instance = new SubType();
```
以上代码把前面两种方式组合在一起，需要注意的是，重设子类构造函数的原型对象为父类实例后，子类原型对象丢失了constructor属性，这里必须补上（指向构造函数本身即可），原因是new的过程，需要从constructor中找到构造函数并调用执行；
### 缺点
父类构造函数被调用了两次，子类构造函数中调用一次，重置原型对象时又调用了一次。实际上，父类实例中仍然有属性，而子类实例也有同名的属性；父类中的属性实例化是多余的部分；

原型式继承
---
本质上与原型链继承是一样的，只是通过类似Object.create()的方法，不需要单独为子类定义构造函数。还有一个优点，可以把对象字面量直接加入到原型链中；
```js
var superType = {
	this.property = true;
}
var instance = Object.create(superType);
```
Object.create()
---
IE8以上（不包含）可以直接使用，IE8需要自行实现：
```js
function object(obj){
	function F(){};
	F.prototype = obj;
	return new F();
}
```
### 缺点
新对象与原对象保持一致，可以说是原对象的副本，只是对于新对象来说，属性都在原型链上；
与原型链继承存在的问题基本一样，父类中的引用类型仍然是共享的；

寄生式继承
---
在原型式继承上，做了一点扩展，生成新对象后，对新对象进行增强，再返回新对象；
```js
function createAnother(original){
	var clone = object(original);
	clone.sayHi = function(){
		alert("hi");
	}
	return clone;
}
```
### 缺点
sayHi方法不能复用


寄生组合式继承
---
寄生组合式继承，借鉴了寄生式继承的思路，解决了组合继承中父类实例两次实例化属性的问题。
先上代码：
```js
function object(obj){
	function F(){};
	F.prototype = obj;
	return new F();
}
function inheritPrototype(subType, superType){
	var prototype = object(superType.prototype);  // prototype替代了new SuperType(),__proto__都指向SuperType.prototype,但是不需要实例化SuperType构造函数中的属性；
	prototype.constructor = subType;   // 替换成子类的构造函数，当执行new SubType()时，运行的是SubType的构造函数；
	subType.prototype = prototype;
}
function SuperType(){
	this.property = true;
}
SuperType.prototype.getSuperValue = function(){
	return this.property;
}
function SubType(){
	SuperType.call(this);
	this.subproperty = false;
}
inheritPrototype(SubType, SuperType);
var instance = new SubType();
```
寄生组合方式，通过使用一个新对象，替换掉原先组合继承方式中，父类的实例，达到原型链不变，但是不需要实例化父类属性.
原型链如下:
组合继承：   子类实例 --- 父类实例  ---  父类原型；
寄生组合：   子类实例 --- 新对象    ---  父类原型；

完美解决了两次实例化父类属性的问题；
### 缺点
唯一的缺点就是实现相对复杂，IE8以上可以通过Object.create()简化部分流程