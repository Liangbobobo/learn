# Element  元素所普遍具有的方法和属性
Element 是一个通用性非常强的基类，所有 Document 对象下的对象都继承自它。这个接口描述了所有相同种类的元素所普遍具有的方法和属性。一些接口继承自 Element 并且增加了一些额外功能的接口描述了具体的行为。  

例如， HTMLElement 接口是所有 HTML 元素的基本接口，而 SVGElement 接口是所有 SVG 元素的基础。大多数功能是在这个类的更深层级（hierarchy）的接口中被进一步制定的  

## 属性  
所有属性继承自它的祖先接口 Node，并且扩展了 Node 的父接口 EventTarget，并且从以下部分继承了属性：ParentNode、ChildNode、NonDocumentTypeChildNode，和 Animatable。：   
Element.id
    是一个DOMString 表示这个元素的id。  
Element.className
    一个 DOMString，表示这个元素的 class。  
Element.id
    是一个DOMString 表示这个元素的id。
Element.innerHTML
    是一个DOMString 表示这个元素的内容文本。   


等等  

## Properties included from Slotable

The Element interface includes the following property, defined on the Slotable mixin.

Slotable.assignedSlot (en-US)只读
    Returns a HTMLSlotElement representing the <slot> the node is inserted in.  


## 事件句柄  
Element.onfullscreenchange
    事件 fullscreenchange (en-US) 的回调方法, 在元素进入或退出全屏模式时触发. 不仅可用于观察（监听）可预期的过度变化，还可以观察（监听）未知的变化，如：当你的应用程序在后台运行。

Element.onfullscreenerror
    事件 fullscreenerror (en-US) 的回调方法, 在进入全屏模式过程中出现错误时触发.  

## 方法   
Inherits methods from its parents Node, and its own parent, EventTarget, and implements those of ParentNode, ChildNode, NonDocumentTypeChildNode, and Animatable.    

EventTarget.addEventListener()
    Registers an event handler to a specific event type on the element.

Element.attachShadow()
    Attatches a shadow DOM tree to the specified element and returns a reference to its ShadowRoot.

## 事件  
Listen to these events using addEventListener() or by assigning an event listener to the oneventname property of this interface.      
select
    Fired when some text has been selected.
    Also available via the onselect property.   

wheel
    Fired when the user rotates a wheel button on a pointing device (typically a mouse).
    Also available via the onwheel property.


## 剪贴板事件  


copy
    Fired when the user initiates a copy action through the browser's user interface.
    Also available via the oncopy property.

cut
    Fired when the user initiates a cut action through the browser's user interface.
    Also available via the oncut property.

paste
    Fired when the user initiates a paste action through the browser's user interface.
    Also available via the onpaste property. 

## 键盘事件  
keydown
    Fired when a key is pressed.
    Also available via the onkeydown property. 

keyup
    Fired when a key is released.
    Also available via the onkeyup property.

## 鼠标事件  

## Touch events