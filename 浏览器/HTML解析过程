#### 三个问题

1. DOM 树是怎么生成的？
2. 在解析过程中遇到 JavaScript 脚本，DOM 解析器是如何处理的？
3. DOM 解析器是如何处理跨站点资源的？


首先当浏览器进程接收到网络进程的响应头数据之后，便向渲染进程发起“提交文档”的消息；
渲染进程接收到“提交文档”的消息后，会和网络进程建立传输数据的“管道”；
**等文档数据传输完成之后**，渲染进程会返回“确认提交”的消息给浏览器进程；


#### HTML 解析器是等整个 HTML 文档加载完成之后开始解析的，还是随着 HTML 文档边加载边解析的？

在这里我统一解答下，HTML 解析器并不是等整个文档加载完成之后再解析的，而是网络进程加载了多少数据，HTML 解析器便解析多少数据。



#### HTML解析过程

分词器将网络进程传递来的字节流转换为token。HTML解析器维护一个token栈结构，通过对不同token的处理和栈情况的分析，构建一棵DOM树。


#### HTML解析器如何维护token栈？

如果分词器分词的结果是

1. startTag，放入栈中，并在DOM树中以栈相邻节点为父节点构建新Node。
2. 文本，不放入栈中，并在DOM树中以栈相邻节点为父节点构建文本节点
3. endTag，不放入栈中，观察栈顶元素是否为对应标签的startTag，是则出栈该startTag。DOM树中该元素解析完毕。
   ![Alt](https://i.loli.net/2020/10/02/jWzKlfEse9VSCy4.jpg)

这里需要补充说明下，HTML 解析器开始工作时，会默认创建了一个根为 document 的空 DOM 结构，同时会将一个 StartTag document 的 Token 压入栈底。


#### XSSAuditor

渲染引擎还有一个安全检查模块叫 XSSAuditor，是用来检测词法安全的。在分词器解析出来 Token 之后，它会检测这些模块是否安全，比如是否引用了外部脚本，是否符合 CSP 规范，是否存在跨站点请求等。如果出现不符合规范的内容，XSSAuditor 会对该脚本或者下载任务进行拦截。

---


#### 含有js代码的HTML文件如何解析？

在解析过程中如果遇到

1. 内嵌 JavaScript 脚本

   ```html
   <html>
   <body>
       <div>1</div>
       <script>
       let div1 = document.getElementsByTagName('div')[0]
       div1.innerText = 'time.geekbang'
       </script>
       <div>test</div>
   </body>
   </html>
   ```

   js的执行进程会阻塞DOM解析进程。之所以这么设计是因为js可能含有对DOM的操作。

2. js文件链接

   ```html
   <html>
   <body>
       <div>1</div>
       <script type="text/javascript" src='foo.js'></script>
       <div>test</div>
   </body>
   </html>
   ```

   首先下载js代码，再执行。在此过程中DOM解析进程被阻塞。

---

#### chrome的优化：预解析操作

当渲染引擎收到字节流之后，会开启一个预解析线程，用来分析 HTML 文件中包含的 JavaScript、CSS 等相关文件，解析到相关文件之后，预解析线程会提前下载这些文件。

---


#### async 或 defer

如果 JavaScript 文件中没有操作 DOM 相关代码，就可以将该 JavaScript 脚本设置为异步加载，通过 async 或 defer 来标记代码，使用方式如下所示：

```html
 <script async type="text/javascript" src='foo.js'></script>
 <script defer type="text/javascript" src='foo.js'></script>
```

async 和 defer 虽然都是异步的，不过还有一些差异，使用 async 标志的脚本文件一旦加载完成，会立即执行；而使用了 defer 标记的脚本文件，需要在 DOMContentLoaded 事件之前执行。

---

#### js 对 cssDOM 的依赖

```css
//theme.css
div {color:blue}
```

```html
<html>

    <head>
        <style src='theme.css'></style>
    </head>

<body>

    <div>1</div>
    <script>
            let div1 = document.getElementsByTagName('div')[0]
            div1.innerText = 'time.geekbang' //需要DOM
            div1.style.color = 'red'  //需要CSSOM
        </script>
    <div>test</div>
    </body>
</html>
```

在上面的js代码中，涉及到对于CSSDOM的操作，因此在js执行前，必须保证css已经解析完毕，生成cssDOM。



**注意，html是“解析”，最后得到一棵DOM树。css也是解析，最后得到cssDOM对象。js是执行，执行代码以完成一系列操作。**

---

看下面这样一段代码，你认为打开这个 HTML 页面，页面显示的内容是什么？

```html
<html>
<body>
    <div>1</div>
    <script>
    let div1 = document.getElementsByTagName('div')[0]
    div1.innerText = 'time.geekbang'

    let div2 = document.getElementsByTagName('div')[1]
    div2.innerText = 'time.geekbang.com'
    </script>
    <div>test</div>

</body>
</html>
```


会显示`time.geekbang`和`test`，JavaScript代码执行的时候第二个div还没有生成DOM节点，所以是获取不到div2的，页面会报错Uncaught TypeError: Cannot set property 'innerText' of undefined。



