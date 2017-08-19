# CSS3选择器

标签（空格分隔）： 前端技术

------

|选择器|介绍|例子|
|:--|:--|:---|
|**_属性选择器_**||
|[attr=val]|attr属性值等于val|input[type=number]{...}
|[attr*=val]|attr属性值中包含val|
|[attr^=val]|attr属性值以val开头|
|[attr$=val]|attr属性值以val结尾|
|**_结构性伪类选择器_**||
|root|将样式绑定到页面的根元素中，在html页面中就是``<html>``部分|:root{...}|
|not|排除某个结构元素下的子结构元素|body *:not(h1){...}|
|empty|指定当元素中内容为空白时使用的样式|:empty{...}|
|target|对页面的某个target元素指定样式，元素的id被当做页面的超链接来使用，该样式只在用户点击了页面中的超链接并且跳转到target元素后起作用|:target{background:yellow;}
|first-child|指定父元素的第一个子元素的样式|li:first-child{...}
|last-child|指定父元素的最后一个子元素的样式|li:last-child{...}
|nth-child|指定父元素中某个指定序号的子元素使用样式|li:nth-child(2){...} li:nth-child(odd){...} li:nth-child(even){...}
|nth-last-child|指定父元素中倒数第n个指定序号的子元素使用样式|li:nth-last-child(2){...}
|nth-of-type|指定父元素中某个指定序号的子元素使用样式，只针对同类型元素进行计算
|nth-last-of-type|指定父元素中倒数第n个指定序号的子元素使用样式，只针对同类型元素进行计算
|only-child|父元素中只有一个子元素时使用样式|li:only-child{}|
|first-line|向某个元素中的第一行文字使用样式|p:first-line {...}|
|first-letter|向某个元素中的文字的首字母或第一个字使用样式
|before|在某个元素之前插入一些内容|li:before{content: ·}|
|after|在某个元素之后插入一些内容|li:after{content: url(test.wav)}|
|**_UI元素状态伪类选择器_**||
|E:hover|鼠标移动到元素上时的样式||
|E:active|指定元素被激活时（鼠标在元素上按下还没有松开）的样式|
|E:focus|指定元素获得光标焦点时的样式|
|E:enabled|指定元素处于可用状态时的样式|
|E:disabled|指定元素处于不可用状态时的样式|
|E:read-only|指定元素处于只读状态时的样式|
|E:read-write|指定元素处于非只读状态时的样式|
|E:checked|指定表单中的单选框或复选框处于选取状态时的样式|
|E:default|指定当页面打开时默认处于选取状态的单选框或复选框控件的样式|input[type="checkbox"]:default{...}
|E:indeterminate|指定当页面打开时，一组单选框中没有任何一个单选框被设定为选取状态时整组单选框的样式，_暂时iOS Safari不支持_
|E::selection|指定元素处于选中状态时的样式，主要使用两个冒号|p::selection{...}
|E:invalid|指定元素不能通过H5中诸如required、pattern等属性指定的检查或元素内容不符合规定格式时的样式|
|E:valid|指定元素通过H5中诸如required、pattern等属性指定的检查或元素内容符合规定格式时的样式|
|E:required|指定input、select、textarea元素设定了required属性时的样式
|E:optional|指定input、select、textarea元素未设定required属性时的样式
|E:in-range|指定元素的输入值在指定的范围内时的样式
|E:out-of-range|指定元素的输入值不在指定的范围内时的样式
