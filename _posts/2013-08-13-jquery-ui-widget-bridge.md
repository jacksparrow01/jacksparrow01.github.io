---
layout: default
title: 简单jQuery的widget实现
description: 简单jQuery的widget实现
---
> jQuery提供了插件的功能，可以让开发者在原来的jQuery的基础上扩展函数，可以通过在$.fn对象上添加函数进行扩展，比如$.fn.dialog = function() {}，但是插件的扩展方式存在下面的问题：
1. 如果一个功能存在这多个函数，比如对话框有close、open等，就必须都在$.fn上定义，这样插件多了之后容易产生冲突，就是没有名字空间的概念。
2. 插件如果需要存储私有变量，会比较困难。
> 因此，就产生了jQuery的ui的组件功能，开发者可以通过$.widget函数进行扩展，ui组件支持私有变量、名字空间、统一方法调用、组件继承、自定义事件等功能，相当强大。但是，很多时候，我们在项目中并不需要这么强大的功能，因此，本文对jQuery的widget函数进行简化，得到简化版的widget函数，主要达到下面的目标：
1. 支持私有变量，统一方法调用，支持名字空间。
2. 简单，容易维护
3. 和原生的widget兼容，即使用原生的widget替换简化版的，组件功能正常。
> 不多说，直接上代码：
<pre><code>
$.widgetBridge = function( name, object ) {
	var fullName = name;
	$.fn[ name ] = function( options ) {
		var isMethodCall = typeof options === "string",
			args = [].slice.call( arguments, 1 ),
			returnValue = this;
		
		options = options || {};
		// 以$().widgetName("method", args);
		// 这是组件的方法调用
		if ( isMethodCall ) {
			var methodValue;
			// 以下划线开头的函数是私有函数，不能通过这种方法调用
			if (options.indexOf("_") !== 0) {
				this.each(function() {
					var	instance = $.data( this, fullName ); //挂在DOM元素上
	
					if ( !instance ) {
						return $.error( "cannot call methods on " + name + " prior to initialization; " +
							"attempted to call method '" + options + "'" );
					}
					// 返回第一个匹配的实例
					if (options === "instance") {
						methodValue = instance;
						return false;
					}
					// 返回第一个options的参数
					if (options === "option") {
						methodValue = instance.options;
						return false;
					}
	
					if (!methodValue)
						methodValue = ( instance[ options ] || jQuery.noop ).apply( instance, args );
					if (options === "destroy") {
						// 执行默认的销毁函数的操作
						//$.removeData(this, fullName); // 移除挂在DOM上的数据
					}
				});
			}
			return methodValue; //返回第一个值
	
		} else {
			this.each(function() {
				var self = this, instance = $.data( this, fullName );
				
				// 首次创建，第二次之后不再做任何事情
				if ( !instance ) {
					instance = $.extend(true, {}, object);
					// 添加destroy函数
					instance.destroy || ( instance.destroy = function() {
						$.removeData(self, fullName); // 移除挂在DOM上的数据
					} );
					instance.options = $.extend(true, instance.options || {}, options);
					$.data( this, fullName, instance );
					instance.element = $(this);
					// 默认_create函数是构造函数
					instance._create && instance._create.call(instance, options);
				}
			});
		}
	
		return returnValue;
	};
};
</code></pre>
>之后，可以通过$.widgetBridge("dialog", options)进行创建jQuery插件，和原生的widget使用方法一样。原理很简单，widget首先约定，如果第一个参数是字符串，则是方法调用，否则是创建widget对象，创建的时候默认调用_create函数，创建完毕之后，会把所有的属性通过$.data挂在DOM元素下面，已达到存储私有属性的目的，下面是简单的使用例子：
<pre><code>
//创建名字为popup的widget
$.widgetBridge("popup", {
	options: {
	},
	_create: function() {
	},
	show: function() {
		
	},
	hide: function() {
	},
	remove: function() {
	}
});
// 通过下面的代码初始化和调用show函数代码
var popup = $("#selector").popup({
	
});
popup.popup("show"); //调用show方法
// ......
</code></pre>