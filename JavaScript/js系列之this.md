##JavaScript中的this
### 1、在全局环境或是普通函数中直接调用

当函数独立调用的时候，在严格模式下它的this指向undefined，在非严格模式下，当this指向undefined的时候，自动指向全局对象(浏览器中就是window)

<pre>
<code>
var a = 1;
var obj  =  {
    a: 2,
      b: function () {
        function fun() {
          return this.a
        }
       console.log(fun());
    }
} 
obj.b();//1
</code>
</pre>

### 2、作为对象的方法

obj对象的b属性存储的是对该匿名函数的一个引用，可以理解为一个指针。当赋值给t的时候，并没有单独开辟内存空间存储新的函数，而是让t存储了一个指针，该指针指向这个函数,相当于执行了这么一段伪代码。
<pre>
<code>
var a = 1;
var obj = {
  a: 2,
  b: function() {
    return this.a;
  }
}
console.log(obj.b())//2
//******************************************
var a = 1;
var obj = {
  a: 2,
  b: function() {
    return this.a;
  }
}
var t = obj.b;
console.log(t());//1
//******************************************
var a = 1;
function fun() {//此函数存储在堆中
    return this.a;
}
var obj = {
  a: 2,
  b: fun //b指向fun函数
}
var t = fun;//变量t指向fun函数
console.log(t());//1

</code>
</pre>
### 3、使用apply和call



<pre>
<code>
function fun() {
      return this.a;
}
fun();//1
//严格模式
fun.call(undefined)
//非严格模式
fun.call(window)

//******************************************
//为啥说是万能公式呢？再看函数作为对象的方法调用:
var a = 1;
var obj = {
  a: 2,
  b: function() {
    return this.a;
  }
}
obj.b()//2
obj.b.call(obj)//2
</code>
</pre>

如上，是不是很强大，可以理解为其它两种都是这个方法的语法糖罢了，那么apply和call是不是真的万能的呢？并不是，ES6的箭头函数就是特例，因为箭头函数的this不是在调用时候确定的，这也就是为啥说箭头函数好用的原因之一，因为它的this固定不会变来变去的了。关于箭头函数的this我们稍后再说。
### 4、作为构造函数

何为构造函数？所谓构造函数就是用来new对象的函数，像Function、Object、Array、Date等都是全局定义的构造函数。其实每一个函数都可以new对象，那些批量生产我们需要的对象的函数就叫它构造函数罢了。注意，构造函数首字母记得大写。

<pre>
<code>
function Fun() {
  this.name = 'Damonre';
  this.age = 21;
  this.sex = 'man';
  this.run = function () {
    return this.name + '正在跑步';
  }
}
Fun.prototype = {
  contructor: Fun,
  say: function () {
    return this.name + '正在说话';
  }
}
var f = new Fun();
f.run();//Damonare正在跑步
f.say();//Damonare正在说话
如上，如果函数作为构造函数用，那么其中的this就代表它即将new出来的对象。为啥呢？new做了啥呢？

伪代码如下：

function Fun() {
  //new做的事情
  var obj = {};
  obj.__proto__ = Fun.prototype;//Base为构造函数
  obj.name = 'Damonare';
  ...//一系列赋值以及更多的事
  return obj
}
也就是说new做了下面这些事:

创建一个临时对象
给临时对象绑定原型
给临时对象对应属性赋值
将临时对象return
也就是说new其实就是个语法糖，this之所以指向临时对象还是没逃脱上面说的几种情况。

当然如果直接调用Fun(),如下：

function Fun() {
  this.name = 'Damonre';
  this.age = 21;
  this.sex = 'man';
  this.run = function () {
    return this.name + '正在跑步';
  }
}
Fun();
console.log(window)
其实就是直接调用一个函数，this在非严格模式下指向window，你可以在window对象找到所有的变量。

另外还有一点，prototype对象的方法的this指向实例对象，因为实例对象的__proto__已经指向了原型函数的prototype。这就涉及原型链的知识了，即方法会沿着对象的原型链进行查找。
</code>
</pre>

### 5、箭头函数

刚刚提到了箭头函数是一个不可以用call和apply改变this的典型。
<pre>
<code>


我们看下面这个：

var a = 1;
var obj = {
  a: 2
};
var fun = () => console.log(this.a);
fun();//1
fun.call(obj)//1
以上，两次调用都是1。

那么箭头函数的this是怎么确定的呢？箭头函数会捕获其所在上下文的 this 值，作为自己的 this 值，也就是说箭头函数的this在词法层面就完成了绑定。apply，call方法只是传入参数，却改不了this。

var a = 1;
var obj = {
  a: 2
};
function fun() {
    var a = 3;
    let f = () => console.log(this.a);
      f();
};
fun();//1
fun.call(obj);//2
如上，fun直接调用，fun的上下文中的this值为window，注意，这个地方有点绕。fun的上下文就是此箭头函数所在的上下文，因此此时f的this为fun的this也就是window。当fun.call(obj)再次调用的时候，新的上下文创建，fun此时的this为obj，也就是箭头函数的this值。

再来一个�：

function Fun() {
    this.name = 'Damonare';
}
Fun.prototype.say = () => {
    console.log(this);
}
var f = new Fun();
f.say();//window
有的同学看到这个🌰会很懵逼，感觉上this应该指向f这个实例对象啊。不是的，此时的箭头函数所在的上下文是__proto__所在的上下文也就是Object函数的上下文，而Object的this值就是全局对象。

那么再来一个:

function Fun() {
    this.name = 'Damonare';
      this.say = () => {
        console.log(this);
    }
}
var f = new Fun();
f.say();//Fun的实例对象
如上，this.say所在的上下文，此时箭头函数所在的上下文就变成了Fun的上下文环境，而因为上面说过当函数作为构造函数调用的时候(也就是new的作用)上下文环境的this指向实例对象。
</code>
</pre>

###6、总结
https://juejin.im/post/59748cbb6fb9a06bb21ae36d