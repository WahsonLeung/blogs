# Web Components，了解一下？

标签（空格分隔）： 前端技术

[TOC]

---

## 1. Why Web Components?
### 1.1 曾经的困惑
> 
1. 在网页开发中，同一段html时常会多次出现在同一页面甚至多个页面中，这种复制粘贴的方式，到处散落的代码会导致日后难以维护。
2. 开发的组件样式非常容易受到外部的污染。
3. 时常的id/名称冲突。
> 
Web Components是多个新W3C标准（``HTML Templates``，``Custom Elements``，``Shadow DOM``，``HTML Imports``）的集合。他们让我们轻易地定义界面组件（元素），这些组件管理着自己的结构（``HTML``）和样式（``CSS``），并且确保这些组件外部的样式不会轻易穿透进来（新的标准依然提供外部修改内部样式的方式）。另外组件一旦定义了，就可以像使用原生HTML元素一样使用组件。

## 2. HTML Templates
> 新标准提供了一个新的标签  ``<template>``，自定义的组件结构（``HTML代码``)以及内联样式（``<style>``）需要放在它内部。

一个简短的例子：
```html
<template>
  <style>
    :host {
      display: block;
    }
    div {
      border: 1px solid seagreen;
    }
  </style>
  <div>test</div>
</template>

```
`template` 标签提供了 `content` 属性用以返回标签的所有子项。
```js
const tmpl = document.querySelector('template');
const content = tmpl.content;
```
### 2.1 浏览器兼容性
![此处输入图片的描述][1]
[Can I Use ?](https://caniuse.com/#search=templates)
> 在4个新的标准中，``Templates`` 是被浏览器支持的最好的一个。

## 3. Shadow DOM
> Shadow DOM标准提供了组件外部与内部结构与样式隔离的解决方案。组件的DOM是独立的，外部不能通过document.querySelector()返回组件shadow DOM中的节点，并且内部的样式规则不会泄露出去，外部的样式也不会渗入。
这跟 ``iframe`` 极其相似，他们都是一个独立的沙箱，iframe有自己的 ``window`` 和 ``document`` 对象，而shadow DOM并没有window，但它有一个叫 [document fragment](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment) 的轻量级document。
因此，id/类名称命名冲突的问题也不需要担心了。

### 3.1 概念
#### 3.1.1 Shadow Host
> 影子宿主，即shadow DOM依附的元素。

从 DevTools 看，元素 ``custom-element`` 就是影子树的宿主。
![此处输入图片的描述][4]
#### 3.1.2 Shadow Root
> 影子根，也就是shadow DOM 的 document fragment。

#### 3.1.3 Shadow Boundary
> 影子边界，shadow DOM 通过这一层不可见的堡垒让内部的HTML和CSS与外部隔离。

### 3.2 使用
#### 3.2.1 创建shadow DOM
```js
const shadowRoot = document.createElement('div').attachShadow({mode: 'open'});
```
> 参数 `{mode: 'open'}` 指创建一个开放的影子根，你可以传入 `{mode: 'closed'}` 创建闭合的影子根，但是并不建议这样做，闭合的影子根会导致完全封闭的组件，你甚至不能在组件内部通过 `querySelector` 等api 获取到内部的节点。
#### 4.2.2 组合和slot
> ``slot`` 翻译过来是插槽的意思，shadow DOM通过 ``<slot>``，给我们提供了不同组件间组合的能力，不同的组合，我们可以灵活的产生不同效果的组件。比如开发一个自定义按钮 ``<my-button>`` , 可以组合 ``<img>``，添加一张图片作为按钮背景。
更深入一点，slot是组件内部的占位符，组件内部可以定义多个占位符。通过slot引入的组件，称为分布式节点。

> 另外需要区分一下两个概念 `Light DOM` 和 `Shadow DOM`：
``Light DOM``：元素实际的子项。
``Shadow DOM``：组件的内部实现，定义了组件的内部结构，样式以及实现详情。
看图找真相：
![此处输入图片的描述][5]
> 
展开shadow-root：
![此处输入图片的描述][6]

接下来举个简单的栗子：
```
<slot></slot>

<!-- 如果没有提供 light DOM，则以默认内容显示 -->
<slot>Fancy button</slot> 

<!-- 如果没有提供 light DOM，则以默认内容显示 -->
<slot>
  <h2>Title</h2>
  <summary>Description text</summary>
</slot>
```

#### 3.2.3 创建命名的slot
用户可以通过名称指定Light DOM节点放置到那个插槽，如果没有指定名称，则放置到默认插槽。
看一下栗子：
```html
<template>
  <slot name="title"></slot>
  <div>---分割线---</div>
  <slot>default</slot>
</template>
```
```
<custom-element>
    <p>我是谁</p> 
    <p>我在哪</p>
    <p>我在干嘛</p> <!-- 所有p元素被放置到默认插槽-->
    <div slot="title">我是标题</div> <!-- 被放置到name是title的插槽-->
</custom-element>
```
#### 3.2.4 使用 `:host选择器` 设定样式
> 组件可通过 `:host(<selector>)` 选择器对自身进行样式设定，但是要注意 _用户可以从外部替换 `:host` 中设定的样式_。一个常见的实践是，你可以在 `:host` 选择器中设定组件的默认宽高，颜色，布局等样式，使用时用户根据需要自行在外部修改这些默认设定。

惯例，看个例子：
```
<style>
:host {
  opacity: 0.4;
}
:host(:hover) {
  opacity: 1;
}
:host([disabled]) { /* style when host has disabled attribute. */
  background: grey;
  pointer-events: none;
  opacity: 0.4;
}
:host(.blue) {
  color: blue; /* color host when it has class="blue" */
}
:host(.pink) > #tabs {
  color: pink; /* color internal #tabs node when host has class="pink". */
}
</style>
```
#### 3.2.5 `:host-context(<selector>)`
> `:host-context(<selector>)`常用于主题化组件，如果组件是 `selector` 的后代元素，此选择器中的样式将被应用。
```
<body class="darktheme">
  <div>
    <custom-element>
    ...
    </custom-element>
  </div>
</body>
```
```
:host-context(.darktheme) {
  color: white;
  background: black;
}
```
#### 4.2.6 `::slotted(<compound-selector>)`
> 可以通过 `::slotted` 选择器给分布式节点设定样式。

用法也很简单：
```html
<style>
/* 当分布式节点是p元素时，应用样式 */
::slotted(p) {
    color: blue;
}
/* 当分布式节点包含title类时，应用样式 */
::slotted(.title) {
    color: red;
}
</style>
```
```
<custom-element>
    <p>我是谁</p>  
    <p>我在哪</p>
    <p>我在干嘛</p> <!-- 前3个p元素的字体颜色被设置为blue色 -->
    <p class="title">我是标题</p> <!-- 字体颜色被设置为red色 -->
</custom-element>
```
> _敲黑板_：跟 `:host` 选择器一样，**外部样式总是优先于在 shadow DOM 中定义的样式**。

#### 3.2.7 `slotchange` 事件
> 当用户从light DOM中添加或删除子项时，`slotchange` 事件会触发。
但是组件实例首次初始化时，也就是组件内部定义了默认的slot内容，不会触发此事件。
`slot.assignedNodes()` 可以查看slot标签中渲染的元素。
`slot.assignedNodes({flatten: true})` 加上参数的调用，如果没有light DOM分配给此slot，则返回此 `slot` 在组件中定义的默认内容。

一个很微妙的例子：
```html
<slot>default</slot> 
```
```html
<custom-element>
</custom-element>
```
> slot.assignedNodes()的返回结果会是什么？这里看起来 `custom-element` 并没有子项，所以理应返回结果是一个空的数组（`[]`）。可是，我们来看
![此处输入图片的描述][7]
实际上返回了一个包含一个文本节点的数组(`[text]`)，而文本节点的内容是一个回车符，并且这里还触发了一次 `slotchange` 事件。似乎是手误多敲的一个回车引起了这一系列困惑，但是应该要注意的是，元素子项的任何内容都会称为 `slot.assignedNodes()` 返回结果的一部分。
> 
一个反向的api， `element.assignedSlot` 可以返回element分配到哪个 `slot`。

#### 3.2.8 Shadow DOM 事件模型
> 当事件从 shadow DOM 中触发时，其目标将会调整为维持 shadow DOM 提供的封装。 也就是说，事件的目标重新进行了设定，因此这些事件看起来像是来自组件，而不是来自 shadow DOM 中的内部元素。但是要注意事件不会从shadow DOM中传播出去，除了以下事件：
> 
- 聚焦事件：blur、focus、focusin、focusout
- 鼠标事件：click、dblclick、mousedown、mouseenter、mousemove，等等
- 滚轮事件：wheel
- 输入事件：beforeinput、input
- 键盘事件：keydown、keyup
- 组合事件：compositionstart、compositionupdate、compositionend
- 拖放事件：dragstart、drag、dragend、drop，等等

组件内部自定义的事件一般不会穿透到影子边界以外，除非事件是通过 `composed: true` 标记创建的。
例如：
```js
this.dispatchEvent(new CustomEvent("event-name", {bubbles: true, composed: true}));
```

### 3.3 浏览器兼容性
![此处输入图片的描述][8]

![此处输入图片的描述][9]
[Can I Use ?](https://caniuse.com/#search=shadow)

## 4. Custom Elements
> 使用 `自定义元素` 的API，我们可以创建新的HTML标记、扩展现有的HTML标记，或者扩展其他开发者编写的组件。
#### 4.1 定义新元素
> 核心的API是 `customElements.define(name, constructor, options)`，通过这个全局性的API定义能让浏览器识别的自定义元素。

来看个简短的栗子：
```js
class CustomElement extends HTMLElement {...}
window.customElements.define("custom-element", CustomElement);
```
请注意自定义元素需要扩展基础的 `HTMLElement`，以确保自定义的元素继承完整的DOM API。另外有几条规则：

- 自定义元素的名称必须包含 **短横线（-）**。如 `<x-tags>`、`<x-elem>` 等。
- 同一标记不能被多次注册。
- 自定义元素不能自我封闭，比如 `<custom-element/>` 这样的写法，浏览器会把`斜杠 /` 去掉，当做一个新的标签的开始，直到遇到父级元素的闭合，然后自动补全结束标签。这可能导致意料之外的结果，如你本来想创建一个空的标签，但由于浏览器的机制，导致兄弟元素被该标签包裹起来了。

### 4.2 扩展元素
#### 4.2.1 扩展自定义元素
比较简单，直接上代码：
```js
class CustomElementChild extends CustomElement {
    constructor() {
        super();  // 构造器中请记住总是首先调用父类的构造器。
        ...
    }
}
window.customElements.define('custom-element-child', CustomElementChild);
```
> 与创建自定义元素的区别是这里继承的是自定义元素 `CustomElement`。
#### 4.2.2 扩展原生HTML元素
```js
class CustomButton extends HTMLButtonElement {
    constructor() {
        super();
        this.addEventListener('click', () => console.log('click'));
    }
}
window.customElements.define('custom-button', CustomButton, {extends: 'button'});
```
> 这里 `CustomButton` 继承的是 `HTMLButtonElement` 而不是 `HTMLElement`，因为 `HTMLButtonElement` 是button更直接的父类接口，提供了按钮组件相关的API。另外你应该注意到 `customElements.define` 的第三个参数，该参数是为了告知浏览器要扩展的标记。

### 4.3 自定义元素生命周期钩子
> 在自定义元素不同的生命周期，可以定义特殊的钩子函数，执行特定的代码。
|名称|	调用时机|
|---|---|
|constructor	|创建或升级元素的一个实例。用于初始化状态、设置事件侦听器或创建 Shadow DOM。
|connectedCallback|	元素每次插入到 DOM 时都会调用。用于运行安装代码，例如获取资源或渲染。一般来说，您应将工作延迟至合适时机执行。
|disconnectedCallback|	元素每次从 DOM 中移除时都会调用。用于运行清理代码（例如移除事件侦听器等）。
|attributeChangedCallback(attrName, oldVal, newVal)|	属性添加、移除、更新或替换。解析器创建元素时，或者升级时，也会调用它来获取初始值。注：仅 `observedAttributes` 属性中列出的特性才会收到此回调。
|adoptedCallback()|	自定义元素被移入新的 document（例如，有人调用了 document.adoptNode(el)）。

注：这些钩子函数的调用都是同步的，比如你的元素被插入到 `DOM` 上时，`connectedCallback` 会立即调用。

### 4.4 attribute（特性）和 property（属性）
> 平时我们写html代码的时候定义在标签上的叫做特性（attribute），比如 `<input type="text" value="Name:">`， 这里input标签有2个attribute。而我们知道input是一个DOM节点，本质上它是一个对象（object），自然而然 property 就是对象上的属性。

- 一般情况下，HTML属性的值会自动映射回同名的特性上，例如下面修改了div的id和hidden属性：
```
div.id = 'my-id';
div.hidden = true;
```
- 最终这些值会以特性的形式映射到DOM上：
```
<div id="my-id" hidden>
```
对于HTML原生的属性，这个过程会自动发生，但是对于用户自定义的属性，我们需要自己实现这个过程。但是，但是我们为什么要这样做？为什么非要把属性映射为特性？-- 特性可以用来声明式配置元素，无障碍功能和 CSS 选择器等某些 API 依赖于特性工作。

- 看一个属性与特性同步的例子：
```js
get disabled() {
  return this.hasAttribute('disabled');
}

set disabled(val) {
  // Reflect the value of `disabled` as an attribute.
  if (val) {
    this.setAttribute('disabled', '');
  } else {
    this.removeAttribute('disabled');
  }
  // do something else
}

static get observedAttributes() {
  return ['disabled'];
}

attributeChangedCallback(name, oldValue, newValue) {
   if(name === "disabled" && oldVal !== newVal) {
     setTimeout(() => {
       this.disabled = newVal === '';
     }, 0);
   }
 }
```
```css
:host([disabled]) {
  opacity: 0.5;
  pointer-events: none;
}
```
此时组件的disabled属性变更时，特性和样式都会得到响应。

### 4.5 元素定义内容
> 前面讲了这么多，我们还没有涉及到给组件编写内部结构（HTML）的方式。这一章节我们将重点介绍。

- 先看一个朴素的方式，这里使用了匿名类的简写形式， `innerHTML` 的内容被添加到元素的 `Light DOM`，而且添加的内容会覆盖元素的子项。
```
window.customElements.define('custom-element', class extends HTMLElement {
    connectedCallback() {
        this.innerHTML = '<div>test</div>';
    }
});
```
- 以上的做法并不符合设想。我们应该使用 `Shadow DOM` 。接下来是一个结合shadow DOM的例子：
```javascript
window.customElements.define('custom-element', class extends HTML {
    constructor() {
        super();
        const shadowRoot = this.attachShadow({mode: 'open'});
        shadowRoot.innerHTML = '<div>test</div>';
    }
});
```

### 4.6 浏览器兼容性
[Can I Use ?](https://caniuse.com/#search=custom%20Elements)
![此处输入图片的描述][2]

![此处输入图片的描述][3]

## 5. HTML Imports
> `HTML Imports` 允许我们使用带有 `rel="import"` 属性的 `link` 标签加载HTML文件，这些HTML文件允许包含脚本（script）、样式（stylesheets），网络字体（web fonts）。
### 5.1 组件引入
看例子：
```html
<link rel="import" href="/path/to/some/custom-element.html">
```
> 这里 `custom-element.html` 由浏览器加载，并保存起来以备用户使用。你可以使用Javascript 读取文件的内容（HTML）并添加到你的页面。但是很重要的一点，如果 `custom-element.html` 除了HTML，还包含有CSS和Javascript代码，这些代码会在主文档（main  document）上自动执行。

通过以下例子来加深理解：
```html
<!-- custom-element.html -->
<template id="custom-ele-tmpl">
  <div>test</div>
</template>
<div>won't render into main document</div>

<style>
    body {
        border: 1px solid blue;
    }
</style>
<script>
  console.log(document.querySelector('#custom-ele-tmpl'));   // null
  console.log(document.currentScript.ownerDocument.querySelector('#custom-ele-tmpl'));  // <template id="custom-ele-tmpl">...</template>
</script>
```
```html
<!DOCTYPE html>
<!-- index.html -->
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>WebComponent</title>
  <link rel="import" href="custom-element.html">
</head>
<body class="darktheme">
  <custom-element></custom-element>
</body>
</html>
```
> 在 `index.html` 页面 `<link rel="import" href="custom-element.html">` 这一行代码会导致浏览器加载 `custom-element.html` 页面， `custom-element.html` 页面中 `<style>` 和 `<script>` 标签中的代码会被执行（index.html页面中的body标签添加了蓝色的边框，并且浏览器控制台有console.log的日志输出），但是其他的html元素并不会渲染到index.html页面，而是被保存起来，等待用户使用（这也是 `console.log(document.querySelector('#custom-ele-tmpl'))` 输出null的原因）。
### 5.2 读取 `template`
- 在 `custom-element.html` 文件内部读取模板元素：
```js
document.currentScript.ownerDocument.querySelector('#custom-ele-tmpl'); 
```
- 在`custom-element.html` 文件外部读取模板元素：
```js
document.querySelector('link').import.querySelector('#custom-ele-tmpl')
```
### 5.3 浏览器兼容性
![此处输入图片的描述][10]
[Can I Use ?](https://caniuse.com/#search=HTML%20Imports)

## 6. Hack for browsers
> 都看过了上面的浏览器兼容性，除了Chrome，还真没一个浏览器能把这一整套玩起来，怎么办？既然浏览器不争气我们就自己打补丁开外挂吧。
我们可以在页面初始化前先加载 [Polyfill](https://github.com/webcomponents/webcomponentsjs)，让不支持web components特性的浏览器先安装上我们需要的api。

可以先做一个兼容性检测来决定是否需要加载polyfill：
```js
<script>
if (!('customElements' in window)
  || !("import" in document.createElement("link"))
  || !HTMLElement.prototype.attachShadow) {
  const script = document.createElement('script');
  script.src = "webcomponents-lite.js";
  document.head.appendChild(script);
}
</script>
```
## 7. 4个标准的整合
```html
<!-- webcomponent.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>WebComponent</title>
  
  <!-- 在不支持webcomponent标准的浏览器中加载polyfill -->
  <script>
    if (!('customElements' in window)
      || !("import" in document.createElement("link"))
      || !HTMLElement.prototype.attachShadow) {
      const script = document.createElement('script');
      script.src = "webcomponents-lite.js";
      document.head.appendChild(script);
    }
  </script>
  
  <!--HTML Import-->
  <link rel="import" href="custom-element.html">
  <style>
    div {
      border: 1px solid #2288dd;
    }
  </style>
</head>
<body>
<custom-element></custom-element>
</body>
</html>
```
```html
<!-- custom-element.html -->
<!--Template-->
<template id="custom-ele-tmpl">
  <style>
    :host {
      all: initial;  /* 将可继承样式重置为初始值 */
      outline: 5px solid seagreen;
      display: block;   /* 如果没有设定，默认值是inline*/
    }
    :host([hidden]) {
      display: none;
    }
    :host(:hover) {
      outline: 1px solid seagreen;
    }
    :host([disabled]) {
      opacity: 0.5;
      pointer-events: none;
    }
    :host(.blue) {
      color: blue; /* color host when it has class="blue" */
    }
    /*如果 :host-context(<selector>)
     或其任意父级与 <selector> 匹配，它将与组件匹配。
    一个常见用途是根据组件的环境进行主题化。
    */
    :host-context(.darktheme) {
      color: #FFFFFF;
      background: #000;
    }
    ::slotted([slot=slot]) {
      background: saddlebrown;;
    }
    :host(:focus) {
      background: orchid;
    }
    ::slotted(p) {
      background: #2288dd;
    }
    ::slotted(.title) {
      background: slateblue;
    }
  </style>
  <slot>default</slot>
</template>
<script>
  // https://developers.google.com/web/fundamentals/web-components/customelements
  class CustomElement extends HTMLElement {
    static get is() {
      return "custom-element";
    }
    
    static get observedAttributes() {
      return ['test-attr'];
    }
    
    /**
     * 创建或升级元素的一个实例。用于初始化状态、设置事件侦听器或创建 Shadow DOM。
     * 参见规范，了解可在 constructor 中完成的操作的相关限制。
     */
    constructor() {
      super();
      console.log("created");
      const tmpl = document.currentScript.ownerDocument.querySelector('#custom-ele-tmpl');
      // Shadow DOM
      /** delegatesFocus: true， 将元素的焦点行为拓展到影子树内,
       * 如果您点击 shadow DOM 内的某个节点，且该节点不是一个可聚焦区域，那么第一个可聚焦区域将成为焦点。
       * 当 shadow DOM 内的节点获得焦点时，除了聚焦的元素外，:focus 还会应用到宿主。
       **/
      const shadowRoot = this.attachShadow({mode: 'open', delegatesFocus: true});
      // import node from other document, and append to shadowRoot
      shadowRoot.appendChild(document.importNode(tmpl.content, true));
      this.shadowRoot.querySelector("slot")
        .addEventListener("slotchange", console.log);
    }
    
    /**
     * 元素每次插入到 DOM 时都会调用。用于运行安装代码，例如获取资源或渲染。
     * 一般来说，您应将工作延迟至合适时机执行。
     */
    connectedCallback() {
      console.log("connectedCallback");
    }
    
    /**
     * 元素每次从 DOM 中移除时都会调用。用于运行清理代码（例如移除事件侦听器等）。
     */
    disconnectedCallback() {
      console.log("detachedCallback")
    }
    
    /**
     * 属性添加、移除、更新或替换。
     * 解析器创建元素时，或者升级时，也会调用它来获取初始值。
     * 注：仅 observedAttributes 属性中列出的特性才会收到此回调。
     */
    attributeChangedCallback(attrName, oldVal, newVal) {
      console.log(attrName, oldVal, newVal);
    }
    
    /**
     * 自定义元素被移入新的 document（例如，有人调用了 document.adoptNode(el)）。
     */
    adoptedCallback() {
      console.log("adoptedCallback")
    }
  }
  window.customElements.define(CustomElement.is, CustomElement);
</script>
```
## 8. Polymer
[Polymer](https://www.polymer-project.org) 是Google开发的Javascript开源框架。它是对Web Component标准的封装，提供了更方便易用的API，同时还扩展了模板语法，数据绑定（data-binding），样式注入（@apply）等特性。
其他的Web Components 框架还有 [X-Tag](http://x-tag.github.io/)，[Bosonic](http://bosonic.github.io/)。

## 9. 结语
> Web Components未来会改变Web开发，但是现阶段包括浏览器兼容性，社区等还有很长的路要走。

【参考文献】

- https://www.webcomponents.org/community/articles/introduction-to-html-imports
- http://blog.teamtreehouse.com/introduction-html-imports

- https://stackoverflow.com/questions/6003819/what-is-the-difference-between-properties-and-attributes-in-html#answer-6004028

  [1]: https://ws2.sinaimg.cn/large/006tKfTcgy1fo7uxfbbwaj30z40cptax.jpg
  [2]: https://ws3.sinaimg.cn/large/006tKfTcgy1fo7uwbd6jtj30zb0c3mz3.jpg
  [3]: https://ws1.sinaimg.cn/large/006tKfTcgy1fo7uvvkvh1j30z40c3ac6.jpg
  [4]: https://ws3.sinaimg.cn/large/006tKfTcgy1fo7z6duc55j30io04rmxh.jpg
  [5]: https://ws1.sinaimg.cn/large/006tKfTcgy1fo815ttrbmj30n4084aax.jpg
  [6]: https://ws2.sinaimg.cn/large/006tKfTcgy1fo813h0llqj30qt0de40k.jpg
  [7]: https://ws4.sinaimg.cn/large/006tKfTcgy1fo95hjwufyj30sf0jd410.jpg
  [8]: https://ws2.sinaimg.cn/large/006tKfTcgy1fo7uybt8whj30z40eiwh6.jpg
  [9]: https://ws4.sinaimg.cn/large/006tKfTcgy1fo7uyqt846j30z50e9dik.jpg
  [10]: https://ws2.sinaimg.cn/large/006tKfTcgy1fo7v0sugi2j30z70csdi2.jpg