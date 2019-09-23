

## `dom`上常用的元素

1. document，表示文档对象，可以通过它来获取文档中的元素，比如：

```javascript
document.getElementById
document.getElementsByName
document.getElementsByTagName等
```

2. 文档中所有的图像，链接保存在`document.images`、`document.links`数组中，可以通过遍历这些数组，找到特定的图像和链接

```javascript
for(int i = 0; i < document.images.length; i++){
    // do something with document.images[i]
}
```

3. 获取某个元素的父节点，可以通过它的**`parentNode`**对象来访问

```
// 检测它的父节点是不是a标签
if(document.getElementById("idName").parentNode.tagName == "A"){
	// do something
}
```

4. 如何获取`iframe`[**HTML `Inline` Frame element**]中的页面元素

```javascript
// 某个iframe的id名称为icontent
// contentWindow返回iframe元素的window对象
document.getElementById("icontent").contentWindow
// 通过该window对象访问document对象
document.getElementById("icontent").contentWindow.document
```

5. 跳转页面

```javascript
document.location.href = "anotherPage.html";
```

6. 打开一个新的窗口

```javascript
let windowObjectReference = window.open(strUrl, strWindowName, [strWindowFeatures]);
```

> `strUrl` === 要在新打开的窗口中加载的URL。
>
> `strWindowName` === 新窗口的名称。
>
> `strWindowFeatures` === 一个可选参数，列出新窗口的特征(大小，位置，滚动条等)作为一个`DOMString`

```javascript
window.open("images/photo.jpg", "photo window", "resizable=no, width=350, height=260");
```

7. `RegExp.exec(str)`，**在一个指定字符串中执行一个搜索匹配**。返回一个结果数组或 [`null`]；如果匹配成功，`exec`() 方法返回一个数组，并更新正则表达式对象的属性。返回的数组将`完全匹配成功的文本`作为第一项，将`正则括号里匹配成功`的作为数组填充到后面。

![1562631628687](D:\notes\JavaScript\images\1562631628687.png)

```javascript
// 这里的每一个括号，表示一个匹配模式，它对应以一个匹配结果，比如：(\S)它匹配一个非空
// 白的字符，当匹配成功时，将会更改RegExp对象的属性，由于它是第一个匹配模式，所以对应
// RegExp.$1，即可以通过RegExp.$1来进行访问
re = /^(\S)(\S+)\s(\S)(\S+)$/;
for (var k=0; k<nameList.length; k++) {
	if (nameList[k]) {
        // 如果匹配成功，`exec`() 方法返回一个数组，并更新正则表达式对象的属性
	    re.exec(nameList[k]);
		newNames[k] = RegExp.$1.toUpperCase() + RegExp.$2.toLowerCase() + " " + RegExp.$3.toUpperCase() + RegExp.$4.toLowerCase();
	}
}
```

![1562554685102](D:\notes\JavaScript\images\1562554685102.png)



8. 可以通过`style对象`来更改样式

```javascript
document.getElementById("pageBody").style.backgroundColor = "#00F";
document.getElementById("pageBody").style.color = "#F00";
```

9. `Element.classList`

`Element.classList` 是一个只读属性，返回一个元素的类属性的实时[`DOMTokenList`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMTokenList) 集合

​	**toggle** **( String [, force] )**

当只有一个参数时：切换 class value; 即如果类存在，则删除它并返回`false`，如果不存在，则添加它并返回`true`。



## 常用的方法

1. `HTMLElement.blur()`

`blur`方法用来**移除当前元素所获得的键盘焦点**.

```javascript
window.onfocus = moveBack;

function moveBack(){
    self.blur();
}
```

2. `confirm(string)`

创建一个确认窗口

```javascript
confirm("Do you want to delete the cookies?")
```

