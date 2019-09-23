## 添加节点

```javascript
window.addEventListener("load",initAll,false);

function initAll() {	    document.getElementById("theForm").addEventListener("submit",addNode,false);
}

function addNode(evt) {
	var inText = document.getElementById("textArea").value;
    // 创建一个新的文本节点newText，它将包含textArea中的文本
	var newText = document.createTextNode(inText);
    // 创建一个p元素
	var newGraf = document.createElement("p");
    // 将文本添加到p元素中
	newGraf.appendChild(newText);
	
    // 这个getElementsByTagName()方法会给出页面上的每个body 标签。如果页面符合标准，那么应该只有一个body 标签。[0]属性是第一个body 标签，我们将它存储在docBody 中。
	var docBody = document.getElementsByTagName("body")[0];
    // 将newGraf节点添加在body节点下
	docBody.appendChild(newGraf);
	
	evt.preventDefault();
}
```

>`ps: Node.appendChild()`
>
>The **`Node.appendChild()` **method adds a node to the end of the list of children of a specified parent node. If the given child is a reference to an existing node in the document, `appendChild()` moves it from its current position to the new position (there is no requirement to remove the node from its parent node before appending it to some other node).
>
>This means that a node can't be in two points of the document simultaneously. So if the node already has a parent, the node is first removed, then appended at the new position. The [`Node.cloneNode()`](https://developer.mozilla.org/en-US/docs/Web/API/Node/cloneNode) method can be used to make a copy of the node before appending it under the new parent. Note that the copies made with `cloneNode` will not be automatically kept in sync.
>
>If the given child is a [`DocumentFragment`](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment), the entire contents of the [`DocumentFragment`](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment) are moved into the child list of the specified parent node.

> 既然对`innerHTML`进行简单的赋值就可以实现同样的效果，那么为什么要这么费事儿（创建文本节点、创建元素节点并且追加子节点）呢？有一个原因：利用这种方式，就不可能导致页面无效。例如，添加的每个<p>或<div>标签会自动结束。另一方面，如果使用`innerHTML`，就非常容易形成无效的标签（简直太容易了）。一旦发生这种情况，页面的DOM就很难处理了。例如，如果一个元素有开始标签，而没有结束标签，那么就无法读取这个元素的内容。

> 段落本身不能再包含段落。如果你试图传入多个用空行分隔的句子，这段代码将把它们合并成一个大段，而不是分别解析每一段。

## 删除节点

```javascript
function delNode(evt) {
	var allGrafs = document.getElementsByTagName("p");
	
    // 如果包含多个段落
	if (allGrafs.length > 1) {
        // 删除在body节点下的最后一个p子节点
		var lastGraf = allGrafs[allGrafs.length-1];
        // 通常情况下，一个页面只有一个body元素
		var docBody = document.getElementsByTagName("body")[0];
		docBody.removeChild(lastGraf);
	}
	else {
		alert("Nothing to remove!");
	}

	evt.preventDefault();
}
```

> `ps: Node.removeChild()`
>
> The `Node.removeChild()` method removes a child node from the DOM and returns the removed node.

## 删除特定的节点

```javascript
window.addEventListener("load",initAll,false);
var nodeChgArea;

function initAll() {
	document.getElementById("theForm").addEventListener("submit",nodeChanger,false);
    // 新增段落的父节点，新增的所有段落都添加在该节点下
	nodeChgArea = document.getElementById("modifiable");
}

function addNode() {
	var inText = document.getElementById("textArea").value;
	var newText = document.createTextNode(inText);

	var newGraf = document.createElement("p");
	newGraf.appendChild(newText);

	nodeChgArea.appendChild(newGraf);
}

function delNode() {
    // 从下拉列表中读取段落号，selectedIndex
	var grafChoice = document.getElementById("grafCount").selectedIndex;
	// 从新增段落的父节点中获取所有的段落
    var allGrafs = nodeChgArea.getElementsByTagName("p");
    // 在段落列表中获取下标为grafChoice的段落
    // 即等同于 var oldGraf = allGrafs[grafChoice]
	var oldGraf = allGrafs.item(grafChoice);

	nodeChgArea.removeChild(oldGraf);
}

function nodeChanger(evt)  {
	var actionType = -1;
    // 获取段落列表
	var pGrafCt = nodeChgArea.getElementsByTagName("p").length;
    // 获取操作选项，根据选择来判断是添加段落操作还是删除特定段落的操作
	var radioButtonSet = document.getElementById("theForm").nodeAction;
	
	for (var i=0; i<radioButtonSet.length; i++) {
		if (radioButtonSet[i].checked) {
			actionType = i;
		}
	}
	
	switch(actionType) {
		case 0:
			addNode();
			break;
		case 1:
			if (pGrafCt > 0) {
				delNode();
				break;
			}
		default:
			alert("No valid action was chosen");
	}
	// 重置下拉列表的选项大小为0
	document.getElementById("grafCount").options.length = 0;
	// 为下拉列表添加选项
	for (i=0; i<nodeChgArea.getElementsByTagName("p").length; i++) {
		document.getElementById("grafCount").options[i] = new Option(i+1);
	}

	evt.preventDefault();
}

```

### `html` option element

> option:
>
> The **HTML <option> element** is used to define an item contained in a `<select>`, an `<optgroup>`, or a `<datalist>` element. As such, `<option>` can represent menu items in  pop ups (弹出窗口) and other lists of items in an HTML document.

```html
<label for="pet-select">Choose a pet:</label>

<select id="pet-select">
    <option value="">--Please choose an option--</option>
    <option value="dog">Dog</option>
    <option value="cat">Cat</option>
    <option value="hamster">Hamster</option>
    <option value="parrot">Parrot</option>
    <option value="spider">Spider</option>
    <option value="goldfish">Goldfish</option>
</select>
```

> `HTMLOptionElement`
>
> The `HTMLOptionElement` interface represents [`<option>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option) elements and inherits all classes and methods of the [`HTMLElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement) interface.
>
> `Option()`
>
> Is a constructor creating an `HTMLOptionElement` object. It has four values: 
>
> the text to display, `text`, 
>
> the value associated, `value`,
>
>  the value of `defaultSelected`, 
>
> and the value of `selected`. The last three values are optional.

```javascript
/* assuming we have the following HTML 
<select id='s'> 

</select> 
*/  

var s = document.getElementById('s');
var options = [Four, Five, Six];

options.forEach(function(element,key) {
    s[key] = new Option(element,key);
});
```

> `HTMLSelectElement.SelectedIndex`
>
> The `HTMLSelectElement.selectedIndex` is a `long` that reflects the index of the first or last selected [`<option>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/option) element, depending on the value of `multiple`. The value `-1` indicates that no element is selected.
>
> syntax
>
> ```javascript
> var index = selectElem.selectedIndex;
> selectElem.selectedIndex = index;
> ```

```html
<p id="p">selectedIndex: 0</p>

<select id="select">
  <option selected>Option A</option>
  <option>Option B</option>
  <option>Option C</option>
  <option>Option D</option>
  <option>Option E</option>
</select>
```

```javascript
ar selectElem = document.getElementById('select');
var pElem = document.getElementById('p');

// When a new <option> is selected
selectElem.addEventListener('change', function() {
  var index = selectElem.selectedIndex;
  // Add that data to the <p>
  pElem.innerHTML = 'selectedIndex: ' + index;
})
```

### `html dom item()` 

>`HTML DOM item() Method`
>
>example:
>
>```javascript
>// get the html content of the first p element(index 0) inside the document
>var nodeList = document.getElementsByTagName("P").item(0).innerHTML;
>```
>
>The item() method returns a node at the specified index in a `NodeList` object.
>
>The nodes are sorted as they appear in the source code, and the index starts at 0.
>
>A Node object's collection of child nodes is an example of a `NodeList`object.
>
>**Note:** There are two ways to access a node at the specified index in a node list:
>
>```javascript
>document.body.childNodes.item(0);  // The first child node of <body>
>document.body.childNodes[0];       // The first child node of <body>
>```
>
>

### `Event.preventDefault()`

> The [`Event`](https://developer.mozilla.org/en-US/docs/Web/API/Event) interface's `preventDefault()` method tells the [user agent](https://developer.mozilla.org/en-US/docs/Glossary/user_agent) that if the event does not get explicitly handled, its default action should not be taken as it normally would be. The event continues to propagate(传播) as usual, unless one of its event listeners calls [`stopPropagation()`](https://developer.mozilla.org/en-US/docs/Web/API/Event/stopPropagation) or [`stopImmediatePropagation()`](https://developer.mozilla.org/en-US/docs/Web/API/Event/stopImmediatePropagation), either of which terminates propagation at once.
>
> As noted below, calling `preventDefault()` for a non-cancelable event, such as one dispatched via [`EventTarget.dispatchEvent()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent), without specifying `cancelable: true`  has no effect.  

```javascript
// Toggling a checkbox is the default action of clicking on a checkbox. // This example demonstrates how to prevent that from happening:
document.querySelector("#id-checkbox").addEventListener("click", function(event) {
         document.getElementById("output-box").innerHTML += "Sorry! <code>preventDefault()</code> won't let you check this!<br>";
         event.preventDefault();
}, false);
```

```html
<p>Please click on the checkbox control.</p>

<form>
  <label for="id-checkbox">Checkbox:</label>
  <input type="checkbox" id="id-checkbox"/>
</form>

<div id="output-box"></div>
```

![1562725267414](D:\notes\JavaScript\images\1562725267414.png)

## 插入节点

```javascript
function insertNode() {
    // 获取下拉列表项
	var grafChoice = document.getElementById("grafCount").selectedIndex;
	// 获取textArea中的值
    var inText = document.getElementById("textArea").value;
	// 使用textArea中的值创建一个textNode
	var newText = document.createTextNode(inText);
    // 创建一个p元素
	var newGraf = document.createElement("p");
    // 将textNode节点添加在p元素下
	newGraf.appendChild(newText);
	// 获取某个元素下的所有p元素
	var allGrafs = nodeChgArea.getElementsByTagName("p");
    // 根据下拉列表选择的内容获取特定的p元素
	var oldGraf = allGrafs.item(grafChoice);
	// 在某个节点前插入一个节点
	nodeChgArea.insertBefore(newGraf,oldGraf);
}
```

>Syntax
>
>```javascript
>var insertedNode = parentNode.insertBefore(newNode, referenceNode);
>```
>
>- `insertedNode` The node being inserted, that is `newNode`
>- `parentNode` The parent of the newly inserted node.
>- `newNode` The node to be inserted.
>- `referenceNode` The node before which `newNode` is inserted.
>- If `referenceNode` is `null`, the `newNode` is inserted at the end of the list of child nodes.

## 替换节点

```javascript
function replaceNode() {
	var grafChoice = document.getElementById("grafCount").selectedIndex;
	var inText = document.getElementById("textArea").value;
	
    // 创建一个p元素，并添加从textArea中获取的内容
	var newText = document.createTextNode(inText);
	var newGraf = document.createElement("p");
	newGraf.appendChild(newText);
	
    // 获取特定的p元素
	var allGrafs = nodeChgArea.getElementsByTagName("p");
	var oldGraf = allGrafs.item(grafChoice);
	// 替换某个p元素
	nodeChgArea.replaceChild(newGraf,oldGraf);
}
```

> Syntax
>
> replaces a child node within the given (parent) node.
>
> ```javascript
> var replacedNode = parentNode.replaceChild(newChild, oldChild);
> ```
>
> - `newChild` is the new node to replace `oldChild`. If it already exists in the DOM, it is first removed.
> - `oldChild` is the existing child to be replaced.
> - `replacedNode` is the replaced node. This is the same node as `oldChild`.

## 用对象字面值编写代码

过程式JavaScript：

```javascript
var myCat = new Object;
myCat.name = "Pixel";
myCat.breed = "Tuxedo";
myCat.website = "www.pixel.mu";
function allAboutMyCat() {
	alert("Can I tell you about my cat?");
	tellMeMore = true;
}
```

对象字面值JavaScript：

```javascript
var myCat = {
    name: "Pixel",
    breed: "Tuxedo",
    website: "www.pixel.mu",
    allAbout: function() {
    	alert("Can I tell you about my cat?");
   	 	tellMeMore = true;
	}
}
myCat.allAbout();
```

> 使用对象字面值方式编写代码的好处：
>
> 1. 
> 2. 因为每个对象（包括方法和属性）都包含在一个父对象中，所以不会遇到无意间覆盖别人的代码的问题。如果在你和你的同事各自负责的.js 文件中都有一个名为myText 的变量，一些页面会加载这两个文件，那么页面后加载的文件优先——它会直接覆盖另一个变量，就像根本没有加载过原来的代码一样。解决方案是确保不使用全局变量，而实现这一点最简单的方法就是把你的所有代码都放在一个对象字面值中。
> 3. 对象字面值的一个子集被称为JavaScript Object Notation，简称为JSON。JSON 是Ajax 中最常用的数据格式之一，因此在使用Ajax 时经常会遇到这种格式。



## 小结

![10_操作dom](D:\notes\JavaScript\images\10_操作dom.png)

