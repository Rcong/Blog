---
title: 高性能JavaScript学习手记-执行与加载
date: 2016-12-02 15:48:41
tags:
    - js
clearReading: true
metaAlignment: center
categories: 技术
---

浏览器在处理HTML页面渲染和JavaScript脚本执行的时候是单一进程的,所以在当浏览器在渲染HTML遇到了`<script>`标签会先去执行标签内的代码(如果是使用src属性加载的外链文件,则先下载再执行),在这个过程中,页面渲染和交互都会被阻塞。

<!-- excerpt -->

浏览器在处理HTML页面渲染和JavaScript脚本执行的时候是单一进程的,所以在当浏览器在渲染HTML遇到了`<script>`标签会先去执行标签内的代码(如果是使用src属性加载的外链文件,则先下载再执行),在这个过程中,页面渲染和交互都会被阻塞。

...虽然会有阻塞,但还是有几招可以减少JavaScript对性能的影响的。

---
## **1.script标签的位置**

当`<script>`出现在`<head>`中的时候,比如:

``` javascript
<head>
    <script type="text/javascript" src="js1.js"></script>
    <script type="text/javascript" src="js2.js"></script>
    <script type="text/javascript" src="js3.js"></script>
</head>
```

这种加载多个js文件的时候,浏览器就会因先下载执行js代码而阻塞页面渲染从而出现白屏页面(浏览器解析到`<body>`标签之前,不会渲染页面任何内容),没法预览也没法交互,很差劲的用户体验。

**注意:**
- **现代浏览器支持资源并行下载,只限于`<script>`下载外部资源的时候不会阻塞其他`<script>`标签,但会阻塞其他资源的下载。**
- **下载JavaScript资源是异步的,但是执行JavaScript代码的时候仍是同步的,同样会造成阻塞。**

所以把`<script>`后置到`<body>`标签的底部,保证执行脚本之前已完成页面渲染,是一种比较常用的JavaScript优化手段。

---
## **2.合并多个script标签**

浏览器解析HTML时遇到`<script>`都会因为执行脚本而有一定的延迟,对于有src属性的外链则`<script>`更加,多HTTP请求则会带来更多的性能开销,尽量减少这种延迟,也是一种优化手段,可以合并多个js文件来减少HTTP请求的次数,减少三次握手的次数和多余的HTTP头传输,降低响应时间提高用户体验。网上有许多合并js的方案以及工具,在这不叙述了。

---
## **3.使用无阻塞下载JavaScript的方法**
1. 使用script标签的defer和async属性
2. 使用动态创建的script标签来下载执行JavaScript代码
3. 使用XHR对象下载JavaScript代码并注入页面
### **3.1.使用script标签的defer和async属性**

async和defer属性都是用于异步加载js文件,期间不会才生阻塞浏览器其他进程,区别在于async是加载完之后自动执行,而defer需要等到页面加载之后再执行,需要注意的一点是这两个属性必须在有src属性的`<script>`标签中(外链脚本)才有效。下面是demo:

``` javascript
<!DOCTYPE html>
<html>
<head>
    <title>defer example</title>
</head>
<body>
    <script type="text/javascript" src="defer.js" defer></script>
    <script>
        alert("script");
    </script>
    <script>
        window.onload= function(){
            alert("load");
        }
    </script>
    <div class="demo">defer demo</div>
    <div class="demo">defer demo</div>
    <div class="demo">defer demo</div>
    <div class="demo">defer demo</div>
    <div class="demo">defer demo</div>
    <div class="demo">defer demo</div>
    <div class="demo">defer demo</div>
    <div class="demo">defer demo</div>
    <div class="demo">defer demo</div>
    <div class="demo">defer demo</div>
    <div class="demo">defer demo</div>
    <div class="demo">defer demo</div>
</body>
</html>

//defer.js的文件下只有alert("defer");一行代码
```

async的例子也是相同的页面结构,这里就不摆例子了,可以戳下面的链接。
[defer example的链接戳这里!](http://book.jirengu.com/Rcong/my-practical-code/defer-async-demo/defer.html)
[async example的链接戳这里!](http://book.jirengu.com/Rcong/my-practical-code/defer-async-demo/async.html)
虽然页面结构一样,但不一样的是
- 打开defer.html依次看到是: 弹出"script"的alert框=>页面渲染出文字=>弹出"defer"的alert框=>弹出"load"的alert框
- 打开async.html依次看到是: 弹出"script"的alert框=>弹出"async"的alert框=>页面渲染出文字=>弹出"load"的alert框
### **3.2.使用动态创建的script标签来下载执行JavaScript代码**

``` javascript
var script = document.createElement("script");
script.type = "text/javascript";
script.src = "file.js";
document.getElementByTagName("head")[0].appendChild(script);
```

file.js在script元素添加到页面时就启动下载,使用这种方式的优势在于**file.js的下载和执行不会阻塞页面其他进程。**
下面是普通方式和动态添加脚本方式的两个demo,file.js中仅仅是一个10000次的for循环和之后弹出一个alert框的几句代码。
[动态添加script元素的Demo链接戳这里!](http://book.jirengu.com/Rcong/my-practical-code/dynamic-script-element/dynamic.html)
[普通的引入script脚本Demo链接戳这里!](http://book.jirengu.com/Rcong/my-practical-code/dynamic-script-element/normal.html)
从demo上可以明显的看出动态加载方式可以在alert框弹出之前先看到页面上的文字,但是普通的方式只有在alert框弹出之后才可以看到页面上的文字。

我们可以封装一个跨浏览器的读取script脚本并动态创建script标签的函数:

``` javascript
function loadScript(url,callback){
    var script = document.createElement("script");
    script.type = "text/javascript";
    //检测客户端类型
    if(script.readyState){//IE
        script.onreadystatechange = function(){
            if(script.readyState==="loaded"||script.readyState==="complete"){
                script.onreadystatechange = null;
                callback();
            }
        }
    }else{//其他浏览器
        script.onload = function(){
            callback();
        }    
    }

    script.src = url;
    document.getElementsByTagName("head")[0].appendChild(script);
}
```

**这类动态加载脚本的方法兼容性好,也比较简单,是一种常用的无阻塞解决方案。**
### **3.3.使用XHR对象下载JavaScript代码并注入页面**

另一种无阻塞加载脚本的方式是使用XMLHttpRequest(XHR)对象获取脚本并注入页面中。
**这种技术会先创建一个XHR对象,然后用他下载JavaScript文件,最后通过常见动态`<script>`元素将代码注入页面中。**

``` javascript
var xhr = new XMLHttpRequest();
xhr.open("get","file.js",true);
xhr.onreadystatechange = function(){
    if(xhr.readyState===4){
        if(xhr.status>=200&&xhr.status<300||xhr.status==304){
            var script = document.createElement("script");
            script.type = "text/javascript";
            script.text = xhr.responseText;
            document.body.appendChild(script);
        }
    }
}
```

[Demo在这儿!快让我到碗里去!](http://book.jirengu.com/Rcong/my-practical-code/XMLHttpRequest-script-injection/XMLHttpRequest.html)
以上代码发送GET请求file.js文件,onReadyStateChange检测readyState是否为4(4表示请求完成)和HTTP状态吗是否有效(200表示有效响应,304表示读取缓存)。判断响应有效之后,就动态创建一个`<script>`标签,内容就是服务器接收到的responseText。

这种方法的优点以及缺点:
- 优点:下载JavaScript代码可以不立即执行,且兼容性好适合所有主流浏览器。
- 缺点:JavaScript文件必须与所请求页面处于同一个域,这种情况下JavaScript文件不能从CDN下载,不适合大型的Web应用。

---
## **4.一种推荐的无阻塞方案**

如果页面有大量的JavaScript代码需要添加,可以先在页面中去外链之前我们封装好的动态读取script脚本的函数loadScript,然后再使用它去加载其他所需脚本,例如:

``` javascript
<script type="text/javascript" src="loader.js"></script>
<script type="text/javascript">
    loadScript("file.js",function(){
        //do something
    });
</script>
```

这样只需在第一个`<script>`下载比较精简的loader.js文件时对页面有些许影响,之后的`<script>`并不会有太多影响。

