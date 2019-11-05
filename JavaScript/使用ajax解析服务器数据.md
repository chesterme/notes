1. 新建一个*XMLHttpRequest*对象
```js
var xhr = false;
if(window.XMLHttpRequest){
    xhr = new XMLHttpRequest();
}
else{
    if(window.ActiveXObject){
        try{
            xhr = new ActiveXObject("Michosoft.XMLHTTP");
        }catch(e){}
    }
}
```

2. 发起一个http请求
```js
if(xhr){
    xhr.onreadystatechange = showPicture;
    xhr.open("GET", "getPictures.do?filename=pictures.xml", true);
    xhr.send();
}
else{
    alert("Sorry, but I couldn't create an XMLHttpRequest");
}
```

3. 处理服务器传过来的xml数据
```js
function showPicture(){
    var tempDiv = document.createElement("div");
    var tempText = document.createElement("div");

    if(xhr.readyState == 4){
        if(xhr.status == 200){

            if(xhr.responseXML){
                // 可以使用访问html元素的方式来访问xml内的元素
                // 获取xml文件中所有picture标签内容
                var allImages = xhr.responseXML.getElementsByTagName("picture");
                // 遍历所有的pitcure标签
                for(var i = 0; i < allImages.length; i++){
                    var tempNode = document.createElement("img");
                    var testLink = allImages[i].getElementsByTagName("link")[0];
                    console.log(testLink.innerHTML);
                    var testTitle = allImages[i].getElementsByTagName("title")[0];
                    console.log(testTitle.innerHTML);
                    tempNode.setAttribute("src", allImages[i].getElementsByTagName("link")[0].innerHTML);
                    tempNode.setAttribute("title", allImages[i].getElementsByTagName("title")[0].innerHTML);
                    document.getElementById("pictureBar").appendChild(tempNode);
                }
            }
            else{
                alert("There was a problem with the request " + xhr.status);
            }
        }
    }
}

function getPixVal(inVal){
    return (inVal.textContent) ? inVal.textContent : inVal.text;
}
```