## cookie的格式

cookie是一个具有 **特定文本格式** 的文本字符串

```javascript
// cookie的键值对
cookieName=cookieValue; 
// cookie的过期日期，当过期时，浏览器会自动删除这个cookie
expires= expirationDateGMT; 
// 可以在url中存储一个url
path=URLpath; 
// 可以保存一个域值
domain=siteDomain;
```

## cookie技术

1. 在`http`响应报文中有一个cookie的首部行
2. 在`http`请求报文中有一个cookie的首部行
3. 在用户端系统中保留一个cookie文件，并由用户的浏览器进行管理
4. 在位于web站点的后端数据库中有一个与该cookie关联的表项

## cookie的工作过程

![用cookie跟踪用户状态](\images\用cookie跟踪用户状态.png)

## 设置cookie

![1562578244087](images\1562578244087.png)

```javascript
window.addEventListener("load",nameFieldInit,false);

function nameFieldInit() {
	var userName = "";
	if (document.cookie != "") {
		userName = document.cookie.split("=")[1];
	}

	document.getElementById("nameField").value = userName;
    // 当该区域失去焦点时，就设置cookie
	document.getElementById("nameField").onblur = setCookie;
	document.getElementById("cookieForm").onsubmit = setCookie;
}

function setCookie() {
    // 设置有效时间为6个月
	var expireDate = new Date();
	expireDate.setMonth(expireDate.getMonth()+6);
	// 设置一个键值对
	var userName = document.getElementById("nameField").value;
	document.cookie = "userName=" + userName + ";expires=" + expireDate.toGMTString();
	// 使得该区域失去焦点
	document.getElementById("nameField").blur();
	return false;
}
```

> ps: `EventTaget.addEventListener(type, listener[, options])`
>
> 将指定监听器注册到EventTarget上，当该对象触发指定的事件时，指定的回调函数就会被执行。 事件目标可以是一个文档上的元素 [`Element`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element),[`Document`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document)和[`Window`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window)或者任何其他支持事件的对象 (比如 `XMLHttpRequest`)。
>
> `addEventListener()`的工作原理是将实现[`EventListener`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventListener)的函数或对象添加到调用它的[`EventTarget`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget)上的指定事件类型的事件侦听器列表中。

## 设置多个cookie

可以在一个页面上设置多个cookie，格式如下：
```javascript
document.cookie = "pageHit=" + hitCt + ";expires=" + expireDate.toGMTString();
document.cookie = "pageVisit=" + now + ";expires=" + expireDate.toGMTString();
```

![1562580802772](D:\notes\JavaScript\images\1562580802772.png)

同样，只有名称和值对是必须有的字段。

`split("; ")`命令将多个`cookie `记录分隔为数组，各个`cookie` 从零开始编号。注意，这个命令中分号后面有一个空格。`cookieArray[0]`是多个cookie 记录中的第一个cookie，`cookieArray[1]`是下一个`cookie`，以此类推。

## 读取cookie

```javascript
window.addEventListener("load",nameFieldInit,false);

function nameFieldInit() {
	if (document.cookie != "") {
		document.getElementById("nameField").innerHTML = "Hello, " + document.cookie.split(";")[1].split("=")[1];
	}
}

```

![1562579421171](\images\1562579421171.png)

cookie的信息保存在document.cookie对象中

## 显示cookie

```javascript
window.addEventListener("load",showCookies,false);

function showCookies() {
	var outMsg = "";

	if (document.cookie == "") {
		outMsg = "There are no cookies here";
	}
	else {
        // 得到cookie所有的键值对
		var thisCookie = document.cookie.split("; ");
		
        // 显示所有的键值对
		for (var i=0; i<thisCookie.length; i++) {
			outMsg += "Cookie name is '" + thisCookie[i].split("=")[0];
			outMsg += "', and the value is '" + thisCookie[i].split("=")[1] + "'<br>";
		}
	}
	document.getElementById("cookieData").innerHTML = outMsg;
}

```

## 使用cookie作为计数器

因为cookie 是持久性的，也就是说，它们可以跨Web 服务器和浏览器之间的多次会话持久地存在，所以可以使用cookie 存储特定用户访问某个页面的次数。但是，这并不是在许多网页上看到的页面计数器。因为cookie 是与一个用户相关联的，所以只能记录这个用户的访问次数，不能使用cookie存储所有用户访问这个页面的次数。

```javascript
window.addEventListener("load",initPage,false);

function initPage() {
    // 设置有效时间为6个月
	var expireDate = new Date();
	expireDate.setMonth(expireDate.getMonth()+6);
	
    // 获取cookie中键为pageHit的值，它用来保存当前页面的访问次数，最初的值为0
	var hitCt = parseInt(cookieVal("pageHit"));
	hitCt++;
	
    // 设置cookie
	document.cookie = "pageHit=" + hitCt + ";expires=" + expireDate.toGMTString();
    
    // 在页面中显示
	document.getElementById("pageHits").innerHTML = "You have visited this page " + hitCt + " times.";
}

function cookieVal(cookieName) {
	var thisCookie = document.cookie.split("; ");
	
	for (var i=0; i<thisCookie.length; i++) {
		if (cookieName == thisCookie[i].split("=")[0]) {
			return thisCookie[i].split("=")[1];
		}
	}
	return 0;
}

```

> ps: javascript Date实例
>
> `Date` 对象则基于 [Unix Time Stamp](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap04.html#tag_04_16)，即自1970年1月1日（UTC）起经过的毫秒数。

```javascript
var date1 = new Date('December 17, 1995 03:24:00');
// Sun Dec 17 1995 03:24:00 GMT...

var date2 = new Date('1995-12-17T03:24:00');
// Sun Dec 17 1995 03:24:00 GMT...

console.log(date1 === date2);
// expected output: false;

console.log(date1 - date2);
// expected output: 0
```



## 删除cookie

将cookie 的过期日期设置为过去的某个日期，这会让浏览器自动地删除它。

```javascript
window.addEventListener("load",cookieDelete,false);

function cookieDelete() {
	var cookieCt = 0;
	
    // 如果存在cookie，确认是否删除cookie
	if (document.cookie != "" && confirm("Do you want to delete the cookies?")) {
		var thisCookie = document.cookie.split("; ");
		cookieCt = thisCookie.length;
		
        // 将有效时间设置为过去时间
		var expireDate = new Date();
		expireDate.setDate(expireDate.getDate()-1);
						   
		for (var i=0; i<cookieCt; i++) {
			var cookieName = thisCookie[i].split("=")[0];
			document.cookie = cookieName + "=;expires=" + expireDate.toGMTString();
		}
	}
	document.getElementById("cookieData").innerHTML = "Number of cookies deleted: " + cookieCt;
}

```

![1562580277250](\images\1562580277250.png)

![1562580309083](\images\1562580309083.png)

## 处理多个cookie

```javascript
window.addEventListener("load",initPage,false);

function initPage() {
	var now = new Date();
	var expireDate = new Date();
	expireDate.setMonth(expireDate.getMonth()+6);
   
	var hitCt = parseInt(cookieVal("pageHit"));
	hitCt++;
	
	var lastVisit = cookieVal("pageVisit");
	if (lastVisit == 0) {
		lastVisit = "";
	}
	
    // 设置两个cookie
	document.cookie = "pageHit=" + hitCt + ";expires=" + expireDate.toGMTString();
	document.cookie = "pageVisit=" + now + ";expires=" + expireDate.toGMTString();
	
	var outMsg = "You have visited this page " + hitCt + " times.";
	if (lastVisit != "") {
		outMsg += "<br>Your last visit was " + lastVisit;
	}
	document.getElementById("cookieData").innerHTML = outMsg;
}

function cookieVal(cookieName) {
	var thisCookie = document.cookie.split("; ");
	
	for (var i=0; i<thisCookie.length; i++) {
		if (cookieName == thisCookie[i].split("=")[0]) {
			return thisCookie[i].split("=")[1];
		}
	}
	return 0;
}

```

![1562580913787](images\1562580913787.png)

## 显示新内容提醒信息

可以使用cookie 和JavaScript 提醒经常访问站点的用户注意他们没看到过的内容。

![1562581869655](D:\notes\JavaScript\images\1562581869655.png)

```html
<p>Negrino and Smith's most recent books:</p>
<p id="New-20200601"><a href="http://www.javascriptworld.com">JavaScript: Visual QuickStart Guide, 9<sup>th</sup> Edition</a></p>
<p id="New-20130812"><a href="http://www.dreamweaverbook.com">Dreamweaver CC: Visual QuickStart Guide</a></p>
```

```javascript
window.addEventListener("load",initPage,false);

function initPage() {
	var now = new Date();
    // 获取最近访问该页面的时间
	var lastVisit = new Date(cookieVal("pageVisit"));
    // 设置当前cookie的有效时间为6个人
	var expireDate = new Date();
	expireDate.setMonth(expireDate.getMonth()+6);
	
    // 在cookie中保存当前访问页面的时间和有效时间
	document.cookie = "pageVisit=" + now + ";expires=" + expireDate.toGMTString();
    
    // 获取所有的p元素
	var allGrafs = document.getElementsByTagName("p");
	// 遍历所有的p元素，找出带有New-的id的p元素
	for (var i=0; i<allGrafs.length; i++) {
		if (allGrafs[i].id.indexOf("New-") != -1) {
			newCheck(allGrafs[i],allGrafs[i].id.substring(4));
		}
	}	
	
    // p元素和与该p元素关联的日期信息
	function newCheck(grafElement,dtString) {
        // 获取与该p元素关联的日期信息，即页面最近更改时间
		var yyyy = parseInt(dtString.substring(0,4),10);
		var mm = parseInt(dtString.substring(4,6),10);
		var dd = parseInt(dtString.substring(6,8),10);
		var lastChgd = new Date(yyyy,mm-1,dd);
		
        // 比较页面更改时间和最近访问时间，若更改发生在访问之后，则更改该p元素的classname
		if (lastChgd.getTime() > lastVisit.getTime()) {
			grafElement.className += " newImg";
		}
	}
}

function cookieVal(cookieName) {
	var thisCookie = document.cookie.split("; ");

	for (var i=0; i<thisCookie.length; i++) {
		if (cookieName == thisCookie[i].split("=")[0]) {
			return thisCookie[i].split("=")[1];
		}
	}
	return "1 January 1970";
}

```

