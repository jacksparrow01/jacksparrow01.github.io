---
layout: default
title: JavaScript的日期处理总结
description: JavaScript的日期处理总结
---
>最近项目过程中使用到了JavaScript的Date对象，也碰到了一些问题，其中有时区的问题和日期和字符串之间的格式转换问题，下面一一分析。

## 时区问题<br/>
>给定时间戳（从UTC时间1970年1月1日到现在的毫秒数），输出对应的时间字符串，使用了下面的代码进行转换：
<pre><code>new Date(millsecond).toString();</code></pre> 
>得到的结果比实际的时间多了8个小时。这是因为给的millsecond是UTC时区的时间戳，而new Date使用的是当前的时区的时间，所以出现了误差，这个误差可以通过getTimezoneOffset函数进行调整，该函数返回同一时间当前时区的时间戳和UTC时间戳的差值的分钟数，比如中国时间调用该函数得到-480（8个小时）。纠正后应该是下面的代码：
<pre><code>new Date(millsecond + new Date().getTimezoneOffset() * 60000).toString();</code></pre>
>下面列出常用的时间函数使用的时区：
<pre><code>Date.parse("2012-02-01"); // UTC时间，输出1328054400000
Date.UTC(2012, 1, 1); // UTC时间，输出1328054400000
new Date(2012, 1, 1).getTime(); //当前时区时间，输出1328025600000
// Date的构造函数都使用当前时区
</code></pre>

## Date.parse跨浏览器<br/>
>Date.parse在Chrome浏览器下对格式"yyyy-mm-dd"可以转换成Date对象，而在IE下这是失败的，IE下认格式"mm/dd/yyyy"，所以，应该构造兼容的字符串转化为日期的函数：
<pre><code>parseDate = function(dateStr) {
var arr = dateStr.split(/[^0-9]/);
var curTime = null;
if (arr.length == 3)
{
    curTime = new Date(arr[0], arr[1] - 1, arr[2]);
}
else if (arr.length >= 6)
{
    curTime = new Date(arr[0], arr[1] - 1, arr[2], arr[3], arr[4], arr[5]);
}
return curTime;
}</code></pre>

## 日期的toString函数输出格式
>日期的字符串格式在不同的地区和浏览器不一致，所以应该构造函数输出统一的时间格式：
<pre><code>
dateFormat = function (format, timestamp) {
if (!defined(timestamp) || isNaN(timestamp)) {
	return 'Invalid date';
}
// pick取到参数列表中第一个非空的参数返回
format = pick(format, '%Y-%m-%d %H:%M:%S');

var date = new Date(timestamp),
    key, // used in for constuct below
    hours = date[getHours](),
    day = date[getDay](),
    dayOfMonth = date[getDate](),
    month = date[getMonth](),
    fullYear = date[getFullYear](),
    lang = defaultOptions.lang,
    langWeekdays = lang.weekdays,
    // List all format keys. Custom formats can be added from the outside. 
    replacements = ({
    'a': langWeekdays[day].substr(0, 3), // Short weekday, like 'Mon'
    'A': langWeekdays[day], // Long weekday, like 'Monday'
    'd': pad(dayOfMonth), // pad函数将一位的月份添加前置0变成两位
    'e': dayOfMonth, // Day of the month, 1 through 31

    // Month
    'b': lang.shortMonths[month], // Short month, like 'Jan'
    'B': lang.months[month], // Long month, like 'January'
    'm': pad(month + 1), // Two digit month number, 01 through 12

    // Year
    'y': fullYear.toString().substr(2, 2), // Two digits year, like 09 for 2009
    'Y': fullYear, // Four digits year, like 2009

    // Time
    'H': pad(hours), // Two digits hours in 24h format, 00 through 23
    'I': pad((hours % 12) || 12), // Two digits hours in 12h format, 00 through 11
    'l': (hours % 12) || 12, // Hours in 12h format, 1 through 12
    'M': pad(date[getMinutes]()), // Two digits minutes, 00 through 59
    'p': hours &lt; 12 ? 'AM' : 'PM', // Upper case AM or PM
    'P': hours &lt; 12 ? 'am' : 'pm', // Lower case AM or PM
    'S': pad(date.getSeconds()), // Two digits seconds, 00 through  59
    'L': pad(mathRound(timestamp % 1000), 3) // Milliseconds (naming from Ruby)
});


// do the replaces
for (key in replacements) {
    while (format.indexOf('%' + key) !== -1) { // regex would do it in one line, but this is faster
    	format = format.replace('%' + key, typeof replacements[key] === 'function' ? replacements[key](timestamp) : replacements[key]);
    }
}

};</code></pre>
