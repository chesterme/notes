1. 请求按钮：
```html
<p>
    <a id="makeTextRequest" href="getFile.do?filename=gAddress.txt">Request a text file</a>
    <a id="makeXMLRequest" href="getFile.do?filename=states.xml">Request an XML file</a>
</p>
```

2. 显示区域:
```html
<div id="updateArea"></div>
```

3. 为按钮添加处理函数:
```js
window.onload = initAll;
var xhr = false;
function initAll(){
    document.getElementById("makeTextRequest").onclick = getNewFile;
    document.getElementById("makeXMLRequest").onclick = getNewFile;
}

function getNewFile(){
    makeRequest(this.href);
    return false;
}
```

4. 获取*XMLHttpRequest*对象：
```js
if(window.XMLHttpRequest){
    xhr = new XMLHttpRequest();
}
else{
    if(window.ActiveXObject){
        try{
            xhr = new ActiveXObject("Microsoft.XMLHTTP");
        }catch(e){}
    }
}
```

5. 编写请求体和指定响应处理函数：
```js
if(xhr){
    xhr.onreadystatechange = showContents;
    xhr.open("GET", url, true);
    xhr.send(null);
}
else{
    document.getElementById("updateArea").innerHTML = "Sorry, but I couldn't create an XMLHttpRequest";
}
```

6. 编写响应处理函数：
```js
// readState的值指示请求是否已经处理完毕，值为4表示请求已经处理完毕
if(xhr.readyState == 4){
    // 指示服务器返回的状态码，200表示一切正常
    if(xhr.status == 200) {
        // 检查文件类型
        if (xhr.responseXML && xhr.responseXML.childNodes.length > 0) {
            var outMsg = getText(xhr.responseXML.getElementsByTagName("choices")[0]);
        } else {
            var outMsg = xhr.responseText;
        }
    }
    else{
        var outMsg = "There was a problem with the request " + xhr.status;
    }
    document.getElementById("updateArea").innerHTML = outMsg;
}
```