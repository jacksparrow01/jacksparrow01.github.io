---
layout: default
title: 简单jQuery的draggable功能实现
description: 简单jQuery的draggable功能实现
---
> 很多时候，我们都需要拖拽的功能，本文实现简单的jQuery的拖拽功能，直接上代码：
<pre><code>
(function($) {
    $.dragHelper = {
        position: null,
        eventPosition: null,
        dragEventNameSpace: "draggable",
        handle: null,//响应鼠标事件的元素
        element: null, //需要移动的container
        options: null,
        offsetParent: null, // TODO 通过这个参数控制范围
        initOptions: function(opts) {
            this.element = opts.element;
            this.handle = opts.handle;
            this.options = opts;
        },
        drag: function(event) {
            // 判断是否在当前元素上发生mousedown事件
            if (!$(event.target).closest(this.handle).length) {
                return;
            }

            // 计算position
            var self = this;
            this.options.cursor && this.handle.css("cursor", this.options.cursor);
            this.position = this.element.offset();
            this.eventPosition = { x: event.pageX, y: event.pageY };
            this._dragStarted = true;

            // 测试发现，注册在document上面会比注册在元素本身上面性能好
            // 原因未明，需要继续观察
            $(document).bind("mousemove." + this.dragEventNameSpace, function(e) {
                self.dragMove(e);
            }).bind("mouseup." + this.dragEventNameSpace, function(e) {
                self.dragStop(e);
            });
            event.preventDefault(); // disable selection
        },
        dragMove: function(event) {
            if (!this._dragStarted)
                return;

            var self = this, newPos = {
                left: self.position.left + event.pageX - self.eventPosition.x,
                top: self.position.top + event.pageY - self.eventPosition.y
            };
            self.element.css(newPos);
            this.eventPosition = { x: event.pageX, y: event.pageY };
            this.position = newPos;

        },
        dragStop: function(event) {
            if (!this._dragStarted)
                return;

            this._dragStarted = false;
            this.options.cursor = '';
            this.handle.css("cursor", '');
            $(document).unbind("." + this.dragEventNameSpace);
        }
    }; //初始化draggable

    $.fn.draggable || ( $.fn.draggable = function(opts) {

        //opt = $.extend({handle:"",cursor:"move"}, opt);
        
        return this.each(function() {

            var opt = $.extend({handle:"",cursor:"move"}, opts), 
                $this = $(this),
                dragHelper = $.dragHelper;

            opt.handle = opt.handle === "" ? $this : $this.find(opt.handle);

            if (!opt.handle.length)
                opt.handle = $this;

            opt.element = $this;

            opt.handle.on("mousedown." + dragHelper.dragEventNameSpace, function(e) {
                
                dragHelper.initOptions(opt);
                dragHelper.drag(e);

            });   
        });

    } ); //初始化draggable对象

})(jQuery);
</code></pre>
>拖拽功能原理很简单，通过mousedown、mousemove、mouseup三个事件，可以得到鼠标拖拽DOM元素移动的距离，这个通过jQuery的Event对象的pageX和pageY对象可以计算得到，然后通过jQuery的offset函数，计算对象相对于文档的偏移，不断在mousemove中设置DOM元素的offset属性。上面代码中的$.dragHelper是全局的拖拽辅助对象，存储了当前拖拽对象的offset属性和上次的mousemove或者mousedown事件的pageX和pageY，方便下次的计算。另外，mousedown事件注册在DOM元素上，mousemove和mouseup注册在document上，实践证明，这样得到的拖拽效果比注册在DOM元素上更顺畅，大家可以尝试，原因未明。最后，将拖拽功能封装为jQuery的draggable函数，通过下面的代码使用，相当方便：
<pre><code>
$("#drag").draggable({
	handle: ".title", //selector类型参数，拖拽的手柄，鼠标拖拽该元素的时候，对应的jQuery对象#drag会移动
	cursor: "pointer" //鼠标拖拽的时候的样式
});
// 另外，如果需要使用全局的dragHelper，可以通过下面的代码实现
// 在mousedown中使用，具体参考draggable的实现
$.dragHelper.initOptions(opt);
$.dragHelper.drag(event);
</code></pre>
