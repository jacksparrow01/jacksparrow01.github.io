---
layout: default
title: JavaScript的日期处理总结
description: JavaScript的日期处理总结
---
## Date时间类<br/>
Date类表示时间，用于构造一个时间对象，下面是创建时间的几种方式：
<pre><code>new Date(); // 返回当前时区的当前时间
Date.parse("2/1/2012"); // 将2012年2月1日字符串转化为对应时间的毫秒数并返回，在IE和Chrome对字符串的格式要求不一致，返回的毫秒数是当前时区的时间
Date.UTC(2012, 1, 1); // 返回2012年2月1日的UTC时间的毫秒数，月份是0-11，所以1表示的是2月
new Date(2012, 1, 1); // 返回2012年2月1日的当前时区的时间
new Date("2/1/2012"); // 同Date.parse函数</code></pre>

## UTC和当前时区<br/>
UTC是标准时间，中国时区的时间比UTC时间快8小时，时间的毫秒数是当前时间相对于1970年1月1日计算得到的数值，所以0在UTC时间是1970-1-1 00:00:00，对于中国时区就是1970-1-1 08:00:00，所以，对于相同的毫秒数，UTC时间比中国时区时间慢8小时，对于同一时间，UTC时间的毫秒数比中国时区的毫秒数多了8小时，Date类中的函数getTimezoneOffset是返回了相同时间下两个时区的相差的分钟数。比如在1970年1月1日，UTC时间的8点毫秒数是28800000，而中国时区的8点的毫秒数是0，所以两者相差-480分钟，所以getTimezoneOffset返回-480。

## Date.parse跨浏览器<br/>
	Date.parse在Chrome浏览器下对格式"yyyy-mm-dd"可以转换成Date对象，而在IE下这是失败的，IE下认格式"mm/dd/yyyy"，所以，应该构造兼容的字符串转化为日期的函数：
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

## 接下来再贴出通用格式化输出日期的函数：
	<pre><code>dateFormat = function (format, timestamp, capitalize) {
if (!defined(timestamp) || isNaN(timestamp)) {
	return 'Invalid date';
}
// pick取到参数列表中第一个非空的参数返回
format = pick(format, '%Y-%m-%d %H:%M:%S');

var date = new Date(timestamp),
	key, // used in for constuct below
	// get the basic time values
	hours = date[getHours](),
	day = date[getDay](),
	dayOfMonth = date[getDate](),
	month = date[getMonth](),
	fullYear = date[getFullYear](),
	lang = defaultOptions.lang,
	langWeekdays = lang.weekdays,

	// List all format keys. Custom formats can be added from the outside. 
	replacements = extend({

	// Day
	'a': langWeekdays[day].substr(0, 3), // Short weekday, like 'Mon'
	'A': langWeekdays[day], // Long weekday, like 'Monday'
	'd': pad(dayOfMonth), // pad函数将一位的月份添加前置0变成两位
	'e': dayOfMonth, // Day of the month, 1 through 31

	// Week (none implemented)
	//'W': weekNumber(),

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
	'p': hours < 12 ? 'AM' : 'PM', // Upper case AM or PM
	'P': hours < 12 ? 'am' : 'pm', // Lower case AM or PM
	'S': pad(date.getSeconds()), // Two digits seconds, 00 through  59
	'L': pad(mathRound(timestamp % 1000), 3) // Milliseconds (naming from Ruby)
}, Highcharts.dateFormats);


// do the replaces
for (key in replacements) {
	while (format.indexOf('%' + key) !== -1) { // regex would do it in one line, but this is faster
		format = format.replace('%' + key, typeof replacements[key] === 'function' ? replacements[key](timestamp) : replacements[key]);
	}
}

// Optionally capitalize the string and return
return capitalize ? format.substr(0, 1).toUpperCase() + format.substr(1) : format;
};</code></pre>