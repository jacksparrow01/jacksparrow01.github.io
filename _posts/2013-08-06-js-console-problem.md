---
layout: default
title: Chrome的Console.log中的奇怪问题
description: Chrome的Console.log中的奇怪问题
---
> 这个问题非常奇怪。在Chrome 12版本的浏览器中，执行下面的代码：
<pre><code>
var opt = {
  "a" : "b"
};
console.log(opt);
opt.a = "aaaaa";
console.log(opt);
</code></pre>
>两次输出的结果是一样的：
![](/images/js-console-problem-1.jpg)
> 对于值类型的输出结果是正确的，初步估计是console.log是在一次的CPU周期中，只会输出引用对象的最终结果。
<pre><code>Object {a: "b"}
Object {a: "b"} </code></pre>
> 这应该是一个bug，在版本 25.0.1364.97 m版本的浏览器中输出结果正常。
<pre><code>Object {a: "b"}
Object {a: "aaaaa"} </code></pre>