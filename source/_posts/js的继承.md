---
title: js的继承
date: 2020-07-21 16:44:03
tags: 
- 继承
- Javasript
- 面向对象
---

继承是面向对象编程的一个基本特征，在ES6之前，Javascript并没有提供接口去实现继承，于是很多人就使用Javascript的特效`原型链`来实现继承，继承方式主要分成6种类型。在ES6之后，可以使用`extends`关键字直接实现继承。

### 原型链继承
原型链继承的思路主要是利用原型链的思路，使子类的原型对象`prototype`指向父类实例的原型指针对象`__proto__`。
```javascript
function SuperType() {
	this.property = true;
}

SuperType.prototype.getSuperValue = function() {
	return this.property;
}

function SubType() {
	// this.property = false;
}

// 实现继承
SubType.prototype = new SuperType()

var instance = new SubType()
console.log(instance.getSuperValue()); // 输出true
```
#### 缺点
- 导致引用对象实例被共享，即修改一个引用对象会导致另一个对象也跟着修改

### 借用构造函数继承
借用构造函数主要是利用的是`this`指向的思想，在子类的构造函数中，使父类显示绑定子类的`this`指针，因此子类的`this`指针就添加了父类的实例。借用构造函数解决了原型链继承引用对象共享的问题。
```javascript
function SuperType() {
	this.color = ['yello', 'blue', 'green'];
}

function SubType() {
	 // 父类显示绑定子类的this指针实现继承
	SuperType.call(this);
}

var instance1 = new SubType();
var instance2 = new SubType();
instance1.color.push('red');
console.log(instance1.color); // ["yello", "blue", "green", "red"]
// 引用对象不会被共享
console.log(instance2.color); // ["yello", "blue", "green"]
```
#### 缺点
- 没有办法继承父类的原型属性和方法

### 组合继承
组合继承的思想就是把`原型链继承`与`借用构造函数`结合起来，然后综合使用它们的优点。实现方式是继承父类的属性时，使用借用构造函数；继承原型属性时就使用原型链方式，但是使用完后需要把子类的`constructor`对象指回子类的构造函数；
```javascript
function SuperType(name) {
	this.name = name;
	this.color = ['red', 'green', 'blue'];
}

SuperType.prototype.sayName = function() {
	console.log(this.name);
}

function SubType(name, age) {
	// 继承属性
	SuperType.call(this, name);
	this.age = age;
}

// 原型链方式继承原型属性
SubType.prototype = new SuperType();
// 使构造函数重新指回子类
SubType.constructor = SubType;
SubType.prototype.sayAge = function() {
	console.log(this.age);
}

var ins1 = new SubType('peter', 28);
var ins2 = new SubType('mary', 29);
ins1.color.push('yellow');
console.log(ins1.color); // ["red", "green", "blue", "yellow"]
ins1.sayAge(); // 28
ins1.sayName(); // 'peter'
console.log(ins2.color); // ["red", "green", "blue"]
ins2.sayAge(); // 29
ins2.sayName(); // 'mary'
```
#### 缺点
- 需要调用两次父类构造函数
- 实例对象和原型对象属性name、color(ins1.name,ins1.__proto__.name)

### 原型式继承
先创建一个空白的构造函数，然后传入对象作为这个构造函数的原型，最后这个构造函数的实例。本质就是对传入的对象进行一次浅拷贝。ES5的Object.create()就是原型式继承的一种实现。
```javascript
function object(obj) {
	function F() {}
	F.prototype = obj
	return new F();
}

var person = {
	name: 'peter',
	friends: ['mary', 'jhon', 'leo']
}

var anotherPerson = object(person);
anotherPerson.name = 'greg';
anotherPerson.friends.push('rob');

var yetAnotherPerson = object(person);
yetAnotherPerson.name = 'linda';
yetAnotherPerson.friends.push('barbie');

console.log(person.friends); // ["mary", "jhon", "leo", "rob", "barbie"]
console.log(person.name); // peter
console.log(anotherPerson.friends); // ["mary", "jhon", "leo", "rob", "barbie"]
console.log(anotherPerson.name); // greg
console.log(yetAnotherPerson.friends); // ["mary", "jhon", "leo", "rob", "barbie"]
console.log(yetAnotherPerson.name); // linda
```
#### 缺点
- 缺点与原型链继承一样，引用类型会被共享

### 寄生式继承
寄生式继承的思想主要是在`原型式继承`的基础上，使用`工厂模式`进行封装，最后返回新的实例。
```javascript
function createAnother(original) {
	var clone = object(original);
	clone.sayHi = function() {
		console.log('hi');
	}
}
```
#### 缺点
- 与原型式继承一样

### 寄生组合式继承
这个继承的思想和`组合继承`类似，但是该模式是组合`寄生继承`和`借用构造函数继承`组合起来，避免调用2次父类的构造函数，从而达到性能优化。
```javascript
function inheritPrototype(subType, superType) {
	const prototype = Object.create(superType.prototype);
	prototype.constructor = subType;
	subType.prototype = prototype;
}

function SuperType(name) {
	this.name = name;
	this.colors = ['red', 'blue', 'green'];
}

SuperType.prototype.sayName = function() {
	console.log(this.name);
}

function SubType(name, age) {
	SuperType.call(this, name);
	this.age = age;
}

inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function() {
	console.log(this.age);
}

const a = new SubType('peter', 2);
a.sayName();  // peter
a.sayAge(); // 2
```

### ES6的extends
ES6继承的核心思想其实和`寄生组合继承`的思想是一致的，但是ES6的继承是先创建父类的实例对象`this`，然后用子类的构造函数修改`this`，因为子类没有自己的`this`对象，所以必须先调用父类的super方法（ES5的是可以先调用`this`的），否则就会报错。
