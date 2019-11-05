## 什么是ajax

Ajax 是Asynchronous JavaScript and XML（异步JavaScript 和XML）的缩写

Ajax 本身并不是一种新技术，它是由几种长期存在的Web 技术组合而成的：

1. 使用HTML 和CSS 控制页面结构和表示方式；
2. 使用DOM 显示和操纵页面；
3. 使用浏览器的XMLHttpRequest 对象在客户机和服务器之间传输数据；
4. 使用XML 作为在客户机和服务器之间传输的数据的格式；
5. 使用JavaScript 动态地显示所有内容并且提供交互功能；

> 如果用户的操作并不要求向服务器发出请求（例如，显示已经存储在本地的数据），那么Ajax 引擎会进行响应。这使浏览器能够对许多用户操作立刻作出反应，使网页的反应像桌面程序那样迅速。

> 如果用户操作需要服务器调用，Ajax 引擎就 **异步**地执行它，因此用户不需要等待服务器的响应。用户可以继续与应用程序进行交互，当请求的数据到达时，引擎会更新页面。这里的重点是，**用户的操作不会由于等待服务器而暂停**。
>
> 使用XMLHttpRequest (XHR)对象可以与服务器交互。您可以从URL获取数据，而无需让整个的页面刷新。这使得Web页面可以只更新页面的局部，而不影响用户的操作。

## 读取服务器数据

```javascript
window.addEventListener("load",initAll,false);
// 表示一个XMLHttpRequest对象
var xhr = false;

function initAll() {
	document.getElementById("makeTextRequest").addEventListener("click",getNewFile,false);
	document.getElementById("makeXMLRequest").addEventListener("click",getNewFile,false);
}

function getNewFile(evt) {
    // 传递当前对象中的href给makeRequest函数
	makeRequest(this.href);
    // 不执行默认的操作
	evt.preventDefault();
}

function makeRequest(url) {
    // 检查浏览器是否支持XMLHttpRequest对象
	if (window.XMLHttpRequest) {
		xhr = new XMLHttpRequest();
	}
	else {
        // 对于不支持XMLHttpRequest对象的浏览器，检查是否支持ActiveXObject对象
		if (window.ActiveXObject) {
			try {
                // 若支持则根据ActiveX创建XMLHttpRequest对象
				xhr = new ActiveXObject("Microsoft.XMLHTTP");
			}
			catch (e) {
			}
		}
	}
	// 如果成功创建XMLHttpRequest对象
	if (xhr) {
        // 设置XMLHttpRequest对象的onreadystatechange事件处理程序
        // 每当xhr.readyState属性值发生变化时，就会触发这个处理程序。
		xhr.addEventListener("readystatechange",showContents,false);
        // 设置http请求体的内容，http请求方法，请求url和是否使用异步请求方式
		xhr.open("GET", url, true);
		// 发送http请求
        xhr.send(null);
	}
    // 不支持创建XMLHttpRequest对象
	else {
		document.getElementById("updateArea").innerHTML = "Sorry, but I couldn't create an XMLHttpRequest";
	}
}

function showContents() {
    // 检查请求的状态，即检查服务器返回的状态码
	if (xhr.readyState == 4) {
        // 如果服务器返回200状态码
		if (xhr.status == 200) {
			if (xhr.responseXML && xhr.responseXML.childNodes.length > 0) {
				var outMsg = getText(xhr.responseXML.getElementsByTagName("choices")[0]);
			}
			else {
				var outMsg = xhr.responseText;
			}
		}
		else {
			var outMsg = "There was a problem with the request " + xhr.status;
		}
        // 内容嵌入html页面
		document.getElementById("updateArea").innerHTML = outMsg;
	}
	
	function getText(inVal) {
		if (inVal.textContent) {
            // 返回该节点对象包含的所有文本
			return inVal.textContent;
		}
		return inVal.text;
	}
}
```

> XMLHttpRequest.readyState属性：
>
> | 0    | 代理被创建，但尚未调用 open() 方法。                | UNSENT           |
> | ---- | --------------------------------------------------- | ---------------- |
> | 1    | `open()` 方法已经被调用。                           | OPENED           |
> | 2    | `send()` 方法已经被调用，并且头部和状态已经可获得。 | HEADERS_RECEIVED |
> | 3    | `下载中； `responseText 属性已经包含部分数据。      | LOADING          |
> | 4    | 操作已经完成                                        | DONE             |

> `XMLHttpRequest.responseXML`属性：
>
> 返回一个[`Document`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document)，其中包含该请求的响应，如果请求未成功、尚未发送或不能解析为XML或HTML，则返回null。
>
> Document接口表示任何在浏览器中已经加载好的网页，并作为一个入口去操作网页内容（也就是[DOM tree](https://developer.mozilla.org/en-US/docs/Using_the_W3C_DOM_Level_1_Core)）。DOM tree包括像 [`<body>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/body) 、[`<table>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/table)这样的还有其他的元素。它提供了全局操作document的功能，像获取网页的URL和在document里创建一个新的元素。

> `Document.ChildNode`属性：
>
> `ChildNode`接口包含特定于[`Node`](https://developer.mozilla.org/zh-CN/docs/Web/API/Node) 对象的方法，这些对象可以有一个父对象。
>
> `ChildNode`是一个原始接口，并且不能创建此类型的对象；它通过[`Element`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element)、[`DocumentType`](https://developer.mozilla.org/zh-CN/docs/Web/API/DocumentType)和 [`CharacterData`](https://developer.mozilla.org/zh-CN/docs/Web/API/CharacterData) 对象实现。

> `Node.textContent`属性：
>
> The `textContent` property of the [`Node`](https://developer.mozilla.org/en-US/docs/Web/API/Node) interface represents the text content of a node and its descendants.返回该节点包含的所有的文本

![1562832283073](D:\notes\JavaScript\images\1562832283073.png)

> `XMLHttpRequest.responseText`
>
> 返回一个[`DOMString`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMString)，该[`DOMString`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMString)包含对请求的响应，如果请求未成功或尚未发送，则返回null。

> `XMLHttpReqeust.send()`
>
> `XMLHttpRequest.send()` 方法用于发送 HTTP 请求。如果是异步请求（默认为异步请求），则此方法会在请求发送后立即返回；如果是同步请求，则此方法直到响应到达后才会返回。**XMLHttpRequest.send() 方法接受一个可选的参数，其作为请求主体；如果请求方法是 GET 或者 HEAD，则应将请求主体设置为 null。**

```javascript
var xhr = new XMLHttpRequest();
xhr.open("POST", '/server', true);
//发送合适的请求头信息
xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
xhr.onload = function () { 
    // 请求结束后,在此处写处理代码 
};
xhr.send("foo=bar&lorem=ipsum"); 
```

## 解析服务器数据

```javascript
window.addEventListener("load",initAll,false);
var xhr = false;

function initAll() {
	if (window.XMLHttpRequest) {
		xhr = new XMLHttpRequest();
	}
	else {
		if (window.ActiveXObject) {
			try {
				xhr = new ActiveXObject("Microsoft.XMLHTTP");
			}
			catch (e) {
			}
		}
	}

	if (xhr) {
		xhr.addEventListener("readystatechange",showPictures,false);
		xhr.open("GET", "flickrfeed.xml", true);
		xhr.send(null);
	}
	else {
		alert("Sorry, but I couldn't create an XMLHttpRequest");
	}
}

function showPictures() {
    // 获取展示区域
	var tempText = document.createElement("div");
    // 一个临时对象，用来保存一个p元素
	var theText;
	// 	请求已经被处理完毕	
	if (xhr.readyState == 4) {
        // 检测返回的状态码
		if (xhr.status == 200) {
            // 从返回的xml文件中获取所有的content元素
			var allImages = xhr.responseXML.getElementsByTagName("content");
			// 遍历这些content元素，获取并格式化content元素内容
			for (var i=0; i<allImages.length; i++) {
				tempText.innerHTML = getPixVal(allImages[i]);

				theText = tempText.getElementsByTagName("p")[1].innerHTML;
				theText = theText.replace(/240/g,"75");
				theText = theText.replace(/180/g,"75");
				theText = theText.replace(/_m/g,"_s");
				document.getElementById("pictureBar").innerHTML += theText;
			}
		}
		else {
			alert("There was a problem with the request " + xhr.status);
		}
	}
	
	function getPixVal(inVal) {
		return (inVal.textContent) ? inVal.textContent : inVal.text;
	}
}

```

> RegExp Syntax:
>
> ```javascript
> /pattern/flags
> new RegExp(pattern[, flags])
> RegExp(pattern[, flags])
> ```
>
> `pattern` 正则表达式
>
> `flags` 匹配模式
>
> | g    | global match; find all matches rather than stopping after the first match |
> | ---- | ------------------------------------------------------------ |
> | i    | ignore case; if `u` flag is also enabled, use Unicode case folding |
> | m    | multiline; treat beginning and end characters (^ and $) as working over multiple lines (i.e., match the beginning or end of *each* line (delimited by \n or \r), not only the very beginning or end of the whole input string) |
> | s    | "dotAll"; allows `.` to match newlines                       |
> | u    | Unicode; treat pattern as a sequence of Unicode code points  |
> | y    | sticky; matches only from the index indicated by the `lastIndex` property of this regular expression in the target string (and does not attempt to match from any later indexes). |
>
> 

## 刷新服务器数据

```javascript
window.addEventListener("load",initAll,false);
var xhr = false;

function initAll() {
	if (window.XMLHttpRequest) {
		xhr = new XMLHttpRequest();
	}
	else {
		if (window.ActiveXObject) {
			try {
				xhr = new ActiveXObject("Microsoft.XMLHTTP");
			}
			catch (e) {
			}
		}
	}

	if (xhr) {
		getPix();
	}
	else {
		alert("Sorry, but I couldn't create an XMLHttpRequest");
	}
}

function getPix() {
	xhr.open("GET", "flickrfeed.xml", true);
	xhr.addEventListener("readystatechange",showPictures,false);
	xhr.send(null);
	
	setTimeout(getPix, 5 * 1000);
}

function showPictures() {
    // 创建一个div元素
	var tempText = document.createElement("div");
	// 检测请求是否已经处理完毕	
	if (xhr.readyState == 4) {
        // 检测返回的状态码
		if (xhr.status == 200) {
            // 获取xml文件中的所有content元素
			var allImages = xhr.responseXML.getElementsByTagName("content");
            // 生成一个随机的索引
			var randomImg = Math.floor(Math.random() * allImages.length);
			// 在div中嵌入一张图片
			tempText.innerHTML = getPixVal(allImages[randomImg]);
            // 获取图片的html表示
			var thisImg = tempText.getElementsByTagName("p")[1];
            // 在页面中嵌入该图片
			document.getElementById("pictureBar").innerHTML = thisImg.innerHTML;
		}
		else {
			alert("There was a problem with the request " + xhr.status);
		}
	}
	
	function getPixVal(inVal) {
		return (inVal.textContent) ? inVal.textContent : inVal.text;
	}
}
```


