# node 接口，各种类型的 DOM API 对象会从这个接口继承  
继承自EventTarget   

node只是一个接口，其所有的方法和属性都需要一个对象来继承使用，如document、element。

Node (DOM)
    在 DOM 的上下文中，节点（Node）是节点树中的单个点。包括文档本身、元素、文本以及注释都属于是节点。 

Node.js  
Node.js 是一个跨平台 JavaScript 运行环境，使开发者可以搭建服务器端的JavaScript应用程序。


Node 是一个接口，各种类型的 DOM API 对象会从这个接口继承。它允许我们使用相似的方式对待这些不同类型的对象；比如, 继承同一组方法，或者用同样的方式测试。   

以下接口都从 Node 继承其方法和属性：   
Document, Element, Attr, CharacterData (which Text, Comment, and CDATASection inherit), ProcessingInstruction (en-US), DocumentFragment, DocumentType, Notation, Entity, EntityReference   

在方法和属性不相关的特定情况下，这些接口可能返回 null。它们可能会抛出异常 - 例如，当将子节点添加到不允许子节点存在的节点时。

## 属性  
从其父类型 EventTarget[1] 继承属性。  


Node.baseURI只读
    返回一个表示base URL的DOMString。不同语言中的base URL的概念都不一样。 在HTML中，base URL表示协议和域名，以及一直到最后一个'/'之前的文件目录。 

Node.childNodes只读
    返回一个包含了该节点所有子节点的实时的NodeList。NodeList 是动态变化的。如果该节点的子节点发生了变化，NodeList对象就会自动更新。 

Node.firstChild 只读
    返回该节点的第一个子节点Node，如果该节点没有子节点则返回null。

Node.isConnected只读
    返回一个布尔值用来检测该节点是否已连接(直接或者间接)到一个上下文对象上，比如通常DOM情况下的Document对象，或者在shadow DOM情况下的ShadowRoot对象。

Node.lastChild 只读
    返回该节点的最后一个子节点Node，如果该节点没有子节点则返回null。

Node.nextSibling 只读
    返回与该节点同级的下一个节点 Node，如果没有返回null。

Node.nodeName 只读
    返回一个包含该节点名字的DOMString。节点的名字的结构和节点类型不同。比如HTMLElement的名字跟它所关联的标签对应，就比如HTMLAudioElement的就是 'audio' ，Text节点对应的是 '#text' 还有Document节点对应的是 '#document'。


docunment是什么 ，继承自node，所有documen下的对象都继承自Element，是实例对象，表示任何在浏览器中载入的网页，并作为网页内容的入口，也就是DOM 树。 


Node.nodeType只读
    返回一个与该节点类型对应的无符号短整型的值

等等


# 方法 

从其父类型 EventTarget[1] 继承方法。

Node.appendChild()
    将指定的 childNode 参数作为最后一个子节点添加到当前节点。
    如果参数引用了 DOM 树上的现有节点，则节点将从当前位置分离，并附加到新位置。

Node.cloneNode()
    克隆一个 Node，并且可以选择是否克隆这个节点下的所有内容。默认情况下，节点下的内容会被克隆。

Node.compareDocumentPosition()
    比较当前节点与文档中的另一节点的位置。

Node.contains()
    返回一个 Boolean 布尔值，来表示传入的节点是否为该节点的后代节点。

Node.getRootNode()
    返回上下文对象的根节点。如果shadow root节点存在的话，也可以在返回的节点中包含它。

Node.hasChildNodes()
    返回一个Boolean 布尔值，来表示该元素是否包含有子节点。

Node.insertBefore()
    在当前节点下增加一个子节点 Node，并使该子节点位于参考节点的前面。

Node.isDefaultNamespace()
    返回一个 Boolean 类型值。接受一个命名空间 URI 作为参数，当参数所指代的命名空间是默认命名空间时返回 true，否则返回 false。

Node.isEqualNode()
    返回一个 Boolean 类型值。当两个 node 节点为相同类型的节点且定义的数据点匹配时（即属性和属性值相同，节点值相同）返回 true，否则返回 false。

Node.isSameNode()
    返回一个 Boolean 类型值。返回这两个节点的引用比较结果。

Node.lookupPrefix()
    返回包含参数 URI 所对应的命名空间前缀的 DOMString，若不存在则返回 null。如果存在多个可匹配的前缀，则返回结果和浏览器具体实现有关。

Node.lookupNamespaceURI()
    接受一个前缀，并返回前缀所对应节点命名空间 URI 。如果 URI 不存在则返回 null。传入 null 作为 prefix 参数将返回默认命名空间。

Node.normalize()
    对该元素下的所有文本子节点进行整理，合并相邻的文本节点并清除空文本节点。

Node.removeChild()
    移除当前节点的一个子节点。这个子节点必须存在于当前节点中。

Node.replaceChild()
    对选定的节点，替换一个子节点 Node 为另外一个节点。 



eachNode(rootNode, callback);   
描述

使用递归的方式对 rootNode 的所有后代节点执行回调函数（包括 rootNode 自身）

如果没有设定 callback，这函数会返回一个Array，包含 rootNode 和它的所有后代节点。

如果设定了 callback（回调函数），如果回调函数在调用中返回 Boolean false，则当前层级的遍历会中止，同时恢复上一层级的遍历。这个可以用来在找到需要的节点之后中断循环（比如用来查找包含指定文本的文本节点）

参数

rootNode
    需要进行后代节点遍历的 Node 对象。

callback
    一个可选的回调函数，接受一个节点作为唯一参数。如果没有设定， eachNode 返回一个包含了 rootNode 所有后代节点以及 rootNode 自身的Array 

```
<div id="box">
	<span>Foo</span>
	<span>Bar</span>
	<span>Baz</span>
</div>


var box = document.getElementById("box");
eachNode(box, function(node){
	if(null != node.textContent){
		console.log(node.textContent);
	}
});
```




element是什么   ，继承自node，是基类，描述的是相同元素的通用基本方法和属性，接口继承自 Element 并且增加了一些额外功能的接口描述了具体的行为，如：   
HTMLElement 接口是所有 HTML 元素的基本接口     
SVGElement 接口是所有 SVG 元素的基础。   

node是什么  ，描述的是所有节点的关系（属性和方法，包括查询、增加、删除），使用相似的方式对待这些不同类型的对象；比如, 继承同一组方法，或者用同样的方式测试。
