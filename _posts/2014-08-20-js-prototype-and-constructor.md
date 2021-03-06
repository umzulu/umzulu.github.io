---
layout: post
title: "javascript的prototype与constructor"
description: "prototype constructor"
keywords: ""
category: javascript
tags: [prototype, constructor]
---

***1.原型对象的constructor***

上一篇 [javascript的\_\_proto\_\_与prototype]({{ page.previous.url }}) 讲到了`__proto__` 与 `prototype`的概念，这一篇接着讲原型对象里的一个重要属性`constructor`。

我们可以用对象的`constructor`方法返回创建该对象的构造函数，需要注意的是：<span class="warning">constructor 并不是该对象自身的属性，而是其构造函数的原型对象的属性<span>。

{% highlight javascript %}
function Person (name) {this.name = name};
var p = new Person("zhangsan");

> p.constructor  // function Person (name) {this.name = name}
> p.hasOwnProperty("constructor")  // false
> Person.prototype.constructor  // function Person (name) {this.name = name}
> Person.prototype.hasOwnProperty("constructor")  // true
{% endhighlight %}

图例：

![原型对象的constructor]({{site.cdn}}/constructor.jpg)

首先`Person`的实例对象`p`的原型属性`__proto__`指向`Person`的原型对象`Person.prototype`；

然后`Person`的原型对象`Person.prototype`有一个`constructor`属性，而这个`constructor`又指回`Person`；

于是`p`很自然地通过原型链取到了属性`constructor`，即是其构造函数`Person`。

{% highlight javascript %}
> p.__proto__ === Person.prototype  // true
> Person.prototype.constructor === Person  // true
> p.constructor === Person  // true
{% endhighlight %}

上面的例子只是为了说明`constructor`在原型链中的作用，更实际一点的意义在于通过`prototype`和`constructor`来实现继承。

***3.原型继承*** 

{% highlight javascript %}
// 父类
function Person(name) {
    this.name = name;
}
Person.prototype.getName = function () {
    console.log("my name is " + this.name);
}
// 子类
function Programmer(name) {
    this.name = name;
}
// 将子类的原型指向父类的一个实例
Programmer.prototype = new Person();

var p = new Programmer("zhangsan");

> p.getName() // "my name is zhangsan" 
{% endhighlight %}

图例：

![原型继承]({{site.cdn}}/prototype-inherit.png)

很容易看出`Programmer.prototype.__proto__ === Person.prototype`，进而`p.__proto__.__proto__ === Person.prototype`。

有人会问既然都是通过改变原型的指针来实现继承，那我直接把子类的原型指向父类的原型岂不更省力？即`Programmer.prototype = Person.prototype`。
我来反驳一下，如果这样二者之间会形成强耦合，那么任何对`Programmer.prototype`的修改，都会反映到`Person.prototype`，这肯定是不合适的。

上面的原型继承还存在一些问题：

(1) 在定义子类时就先实例化了父类，这是不合适的。

(2) 子类原型对象的`constructor`指针发生改变。

{% highlight javascript %}
> p.constructor  // function Person(name) {this.name = name;}
// p是Programmer的实例，但是p.constructor不是指向Programmmer，而是Person
{% endhighlight %}

***3.改进的原型继承***

{% highlight javascript %}
// 父类
function Person(name) {
    this.name = name;
}
Person.prototype.getName = function () {
    console.log("my name is " + this.name);
}
// 子类
function Programmer(name) {
    this.name = name;
}
// 空函数作为中介
function F(){}
F.prototype = Person.prototype;
// 将子类的原型指向空函数的实例
Programmer.prototype = new F();
// 修正子类construtor指针
Programmer.prototype.constructor = Programmer;

var p = new Programmer("zhangsan");

> p.getName() // "my name is zhangsan" 
{% endhighlight %}

图例：

![改进的原型继承]({{site.cdn}}/improved-prototype-inherit.png)

可以看出`Programmer.prototype.__proto__ === F.prototype === Person.prototype`，进而`p.__proto__.__proto__ === Person.prototype`。
