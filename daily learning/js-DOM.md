# DOM docunment object model   
文档对象模型 (DOM) 是HTML和XML文档的编程接口。它提供了对文档的结构化的表述，并定义了一种方式可以使从程序中对该结构进行访问，从而改变文档的结构，样式和内容。    
DOM 将文档解析为一个由节点和对象（包含属性和方法的对象）组成的结构集合。简言之，它会将web页面和脚本或程序语言连接起来。   

一个web页面是一个文档。这个文档可以在浏览器窗口或作为HTML源码显示出来。  
但上述两个情况中都是同一份文档。文档对象模型（DOM）提供了对同一份文档的另一种表现，存储和操作的方式。 DOM是web页面的完全的面向对象表述，它能够使用如 JavaScript等脚本语言进行修改。   

DOM 并不是一个编程语言，但如果没有DOM， JavaScript 语言也不会有任何网页，XML页面以及涉及到的元素的概念或模型。   



文档可能会在多种浏览器上使用不同的DOM来访问。  

# DOM 和 JavaScript   
在文档中的每个元素— 包括整个文档，文档头部， 文档中的表格，表头，表格中的文本 — 都是文档所属于的文档对象模型（DOM）的一部分，因此它们可以使用DOM和一个脚本语言如 JavaScript，来访问和处理。  

API (web 或 XML 页面) = DOM + JS (脚本语言)  

尽管我们在本参考文档中会专注于使用JavaScript， 但DOM 也可以使用其他的语言来实现， 如Python   

# 如何访问 DOM?  
当您在创建一个脚本时-无论是使用内嵌 ```<script>```元素或者使用在web页面脚本加载的方法— 您都可以使用 document或 window 元素的API来操作文档本身或获取文档的子类（web页面中的各种元素）。


除了定义 JavaScript的 ```<script>``` 元素外， 当文档被装载（以及当整个DOM可以被有效使用时），JavaScript可以设定一个函数来运行。   
```
 window.onload = function() {}
```


在API参考文档中的语法实例通常会使用element(s) 指代节点，使用nodeList（s）或 element(s)来指代节点数组，使用 attribute(s)来指代属性节点。

# DOM 接口  
接口和对象   
许多对象都实现了多个接口。例如：  table对象   
1. 实现了 HTML Table Element Interface, 其中包括 createCaption 和 insertRow 方法。   
2. 但由于table对象也是一个HTML元素， table 也实现了 Element 接口（在  DOM element Reference 一章有对应描述）。   
3. 最后，由于HTML元素对DOM来说也是组成web页面或XML页面节点树中的一个节点， table元素也实现更基本的  Node 接口， Element 对象也继承这个接口。

```
var table = document.getElementById("table");
var tableAttrs = table.attributes; // Node/Element interface
for (var i = 0; i < tableAttrs.length; i++) {
  // HTMLTableElement interface: border attribute
  if(tableAttrs[i].nodeName.toLowerCase() == "border")
    table.border = "1";
}
// HTMLTableElement interface: summary attribute
table.summary = "note: increased border";

```

# DOM中核心接口  
在DOM编程时，通常使用的最多的就是 Document 和 window 对象。简单的说， window 对象表示浏览器中的内容，而 document 对象是文档本身的根节点。Element 继承了通用的 Node 接口,  将这两个接口结合后就提供了许多方法和属性可以供单个元素使用。在处理这些元素所对应的不同类型的数据时，这些元素可能会有专用的接口，如上节中的  table  对象的例子。    


More specifically, "Node" is an interface that is implemented by multiple other objects, including "document" and "element". All objects implementing the "Node" interface can be treated similarly. The term "node" therefore (in the DOM context) means any object that implements the "Node" interface. Most commonly that is an element object representing an HTML element.





# HTMLElement  表示所有的 HTML 元素
HTMLElement 接口表示所有的 HTML 元素。一些HTML元素直接实现了HTMLElement接口，其它的间接实现HTMLElement接口。   
如HTMLTableElement，HTMLTableElement 接口在常用的 HTMLElement 接口的基础上，提供了专门的属性和方法来处理 HTML 文档中表格的布局与展示。通过继承，它也可以访问父接口 HTMLElement 中的成员。   

继承自父接口Element和 GlobalEventHandlers的属性   
HTMLElement.hidden  获取/设置元素是否隐藏 等等 
从父元素继承的方法, Element：  
HTMLElement.blur()     元素失去焦点
HTMLElement.focus()   元素获得焦点
HTMLElement.click()   出发元素点击事件



node 是DOM的接口  
element对象也继承自这个接口   
document和window是对象