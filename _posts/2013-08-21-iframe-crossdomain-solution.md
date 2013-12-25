---
layout: default
title: iframe跨域解决方案
description: iframe跨域解决方案
---

### iframe跨域解决方案
文章来源：http://www.alloyteam.com/2012/08/lightweight-solution-for-an-iframe-cross-domain-communication/

> 前端的JS是无法跨域的，新的HTML5的postMessage支持跨域发送数据，目前，postMessage的浏览器支持已经比较好了，如下图所示：

&nbsp;<br/><img style="width:100%;" src="http://jacksparrow01.github.io/images/postMessage-support.png" /><br/>

> 因此，只需要支持IE6/7即可，只要能找到解决方案在一些低端的浏览器上实现iframe跨域，就完美了。兴庆的是，如果想访问一个iframe的window.name时，只要将其location改为‘about:blank’即可，这个在IE6/7下是成功的，因此，可以在iframe页面中新建两个iframe用于发送和接受，IE6/7下parent和iframe窗口都可以访问到这两个iframe，并通过设置window.name属性进行跨域，原理参考下图（来自原文）。

&nbsp;<img width="100%" src="http://www.alloyteam.com/wp-content/uploads/2012/08/two_messenger.png" />

> 最后，本人耐不住寂寞，对https和http两种协议的跨域情况进行了一番折腾测试，发现这个方案同样支持https和http的跨域，下面是测试的结果，相当牛X。

&nbsp;<br /><img width="100%" src="http://jacksparrow01.github.io/images/iframe-crossdomain-test.png" />
