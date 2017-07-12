###1、void 0 究竟是什么
最近看了些源码，看到了`void 0`，zepto 里有这一个文件里用到了 `void 0`，另外`void 0`是不是的确的确比 undefined 快一点，这些都是待检测的，现在的疑问是`void 0`究竟是什么。

<pre><code>
typeof void 0 //得到"undefined"
console.log(void 0) //输出undefined
console.log(void 0 === "undefined") //true
</code></pre>
这样就可以说`void 0`就是'undefined'吗？
###2、规范
<blockquote>
void操作符产生式 UnaryExpression : void UnaryExpression 按如下流程解释:令 expr 为解释执行UnaryExpression的结果。调用GetValue(expr).
返回 undefined.


注意：GetValue一定要调用，即使它的值不会被用到，但是这个表达式可能会有副作用(side-effects)。
</blockquote>


[参看](https://segmentfault.com/a/1190000000474941)
[参看2](http://www.cnblogs.com/jams742003/archive/2010/01/13/1646631.html)
