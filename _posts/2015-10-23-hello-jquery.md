---
layout: post
title:  "Hello jQuery!"
date:   2015-10-23 14:00:00
categories: Development
---

[jQuery API](http://api.jquery.com/)  

### jQuery 使用场景

#### DOM Traversal and Manipulation
Get the *\<button\>* element with the class 'continue' and change its HTML to 'Next Step...'
{% highlight js %}
$( "button.continue" ).html( "Next Step..." )
{% endhighlight %}

#### Event Handling
Show the *#banner-message* element that is hidden with *display:none* in its CSS when any button in *#button-container* is clicked.
{% highlight js %}
var hiddenBox = $( "#banner-message" );
$( "#button-container button" ).on( "click", function( event ) {
  hiddenBox.show();
});
{% endhighlight %}

#### Ajax
Call a local script on the server /api/getWeather with the query parameter *zipcode=97201* and replace the element #weather-temp's html with the returned text.
{% highlight js %}
$.ajax({
  url: "/api/getWeather",
  data: {
    zipcode: 97201
  },
  success: function( data ) {
    $( "#weather-temp" ).html( "<strong>" + data + "</strong> degrees" );
  }
});
{% endhighlight %}

### CSS 选择器

| 选择器               | 例子                  | 例子描述                                            | CSS |
| -------------------- | --------------------- | -------------------------------------------------- | --- |
| .class               | .intro                | 选择 class="intro" 的所有元素                      | 1   |
| #id                  | #firstname            | 选择 id="firstname" 的所有元素                     | 1   |
| *                    | *                     | 选择所有元素                                       | 2   |
| element              | p                     | 选择所有 \<p\> 元素                                | 1   |
| element,element      | div,p                 | 选择所有 \<div\> 元素和所有 \<p\> 元素             | 1   |
| element element      | div p                 | 选择 \<div\> 元素内部的所有 \<p\> 元素             | 1   |
| element>element      | div>p                 | 选择父元素为 \<div\> 元素的所有 \<p\> 元素         | 2   |
| element+element      | div+p                 | 选择紧接在 \<div\> 元素之后的所有 \<p\> 元素       | 2   |
| [attribute]          | [target]              | 选择带有 target 属性所有元素                       | 2   |
| [attribute=value]    | [target=_blank]       | 选择 target="_blank" 的所有元素                    | 2   |
| [attribute~=value]   | [title~=flower]       | 选择 title 属性包含单词 "flower" 的所有元素        | 2   |
| [attr&#124;=value]   | [lang&#124;=en]       | 选择 lang 属性值以 "en" 开头的所有元素             | 2   |
| :link                | a:link                | 选择所有未被访问的链接                             | 1   |
| :visited             | a:visited             | 选择所有已被访问的链接                             | 1   |
| :active              | a:active              | 选择活动链接                                       | 1   |
| :hover               | a:hover               | 选择鼠标指针位于其上的链接                         | 1   |
| :focus               | input:focus           | 选择获得焦点的 input 元素                          | 2   |
| :first-letter        | p:first-letter        | 选择每个 \<p\> 元素的首字母                        | 1   |
| :first-line          | p:first-line          | 选择每个 \<p\> 元素的首行                          | 1   |
| :first-child         | p:first-child         | 选择属于父元素的第一个子元素的每个 \<p\> 元素      | 2   |
| :before              | p:before              | 在每个 \<p\> 元素的内容之前插入内容                | 2   |
| :after               | p:after               | 在每个 \<p\> 元素的内容之后插入内容                | 2   |
| :lang(language)      | p:lang(it)            | 选择带有以 it 开头的 lang 属性值的每个 \<p\> 元素  | 2   |
| element1~element2    | p~ul                  | 选择前面有 \<p\> 元素的每个 \<ul\> 元素            | 3   |
| [attribute^=value]   | a[src^="https"]       | 选择其 src 属性值以 "https" 开头的每个 \<a\> 元素  | 3   |
| [attribute$=value]   | a[src$=".pdf"]        | 选择其 src 属性以 ".pdf" 结尾的所有 \<a\> 元素     | 3   |
| [attribute*=value]   | a[src*="abc"]         | 选择其 src 属性中包含 "abc" 子串的每个 \<a\> 元素  | 3   |
| :first-of-type       | p:first-of-type       | 选择属于其父元素的首个 \<p\> 元素的每个 \<p\> 元素 | 3   |
| :last-of-type        | p:last-of-type        | 选择属于其父元素的最后 \<p\> 元素的每个 \<p\> 元素 | 3   |
| :only-of-type        | p:only-of-type        | 选择属于其父元素唯一的 \<p\> 元素的每个 \<p\> 元素 | 3   |
| :only-child          | p:only-child          | 选择属于其父元素的唯一子元素的每个 \<p\> 元素      | 3   |
| :nth-child(n)        | p:nth-child(2)        | 选择属于其父元素的第二个子元素的每个 \<p\> 元素    | 3   |
| :nth-last-child(n)   | p:nth-last-child(2)   | 同上，从最后一个子元素开始计数                     | 3   |
| :nth-of-type(n)      | p:nth-of-type(2)      | 选择属于其父元素第二个 \<p\> 元素的每个 \<p\> 元素 | 3   |
| :nth-last-of-type(n) | p:nth-last-of-type(2) | 同上，但是从最后一个子元素开始计数                 | 3   |
| :last-child          | p:last-child          | 选择属于其父元素最后一个子元素每个 \<p\> 元素      | 3   |
| :root                | :root                 | 选择文档的根元素                                   | 3   |
| :empty               | p:empty               | 选择没有子元素的每个 \<p\> 元素（包括文本节点）    | 3   |
| :target              | #news:target          | 选择当前活动的 #news 元素                          | 3   |
| :enabled             | input:enabled         | 选择每个启用的 \<input\> 元素                      | 3   |
| :disabled            | input:disabled        | 选择每个禁用的 \<input\> 元素                      | 3   |
| :checked             | input:checked         | 选择每个被选中的 \<input\> 元素                    | 3   |
| :not(selector)       | :not(p)               | 选择非 \<p\> 元素的每个元素                        | 3   |
| ::selection          | ::selection           | 选择被用户选取的元素部分                           | 3   |


{% highlight js %}
console.group("jQuery");
var allArticles = $('article[id^="post"]')
allArticles.each(function(index, article){
    var content = $('header>h1>a', article).text().trim();
    var href = $('header>h1>a', article).attr("href");
    console.log("[%s](%s)  ", content, href);
});
console.groupEnd();
{% endhighlight %}

### jQuery 选择器
[Category: Selectors](http://api.jquery.com/category/selectors/)
[All Selector (“*”)](//api.jquery.com/all-selector/)  
[:animated Selector](//api.jquery.com/animated-selector/)  
[Attribute Contains Prefix Selector [name|=”value”]](//api.jquery.com/attribute-contains-prefix-selector/)  
[Attribute Contains Selector [name*=”value”]](//api.jquery.com/attribute-contains-selector/)  
[Attribute Contains Word Selector [name~=”value”]](//api.jquery.com/attribute-contains-word-selector/)  
[Attribute Ends With Selector [name$=”value”]](//api.jquery.com/attribute-ends-with-selector/)  
[Attribute Equals Selector [name=”value”]](//api.jquery.com/attribute-equals-selector/)  
[Attribute Not Equal Selector [name!=”value”]](//api.jquery.com/attribute-not-equal-selector/)  
[Attribute Starts With Selector [name^=”value”]](//api.jquery.com/attribute-starts-with-selector/)  
[:button Selector](//api.jquery.com/button-selector/)  
[:checkbox Selector](//api.jquery.com/checkbox-selector/)  
[:checked Selector](//api.jquery.com/checked-selector/)  
[Child Selector (“parent > child”)](//api.jquery.com/child-selector/)  
[Class Selector (“.class”)](//api.jquery.com/class-selector/)  
[:contains() Selector](//api.jquery.com/contains-selector/)  
[Descendant Selector (“ancestor descendant”)](//api.jquery.com/descendant-selector/)  
[:disabled Selector](//api.jquery.com/disabled-selector/)  
[Element Selector (“element”)](//api.jquery.com/element-selector/)  
[:empty Selector](//api.jquery.com/empty-selector/)  
[:enabled Selector](//api.jquery.com/enabled-selector/)  
[:eq() Selector](//api.jquery.com/eq-selector/)  
[:even Selector](//api.jquery.com/even-selector/)  
[:file Selector](//api.jquery.com/file-selector/)  
[:first-child Selector](//api.jquery.com/first-child-selector/)  
[:first-of-type Selector](//api.jquery.com/first-of-type-selector/)  
[:first Selector](//api.jquery.com/first-selector/)  
[:focus Selector](//api.jquery.com/focus-selector/)  
[:gt() Selector](//api.jquery.com/gt-selector/)  
[Has Attribute Selector [name]](//api.jquery.com/has-attribute-selector/)  
[:has() Selector](//api.jquery.com/has-selector/)  
[:header Selector](//api.jquery.com/header-selector/)  
[:hidden Selector](//api.jquery.com/hidden-selector/)  
[ID Selector (“#id”)](//api.jquery.com/id-selector/)  
[:image Selector](//api.jquery.com/image-selector/)  
[:input Selector](//api.jquery.com/input-selector/)  
[:lang() Selector](//api.jquery.com/lang-selector/)  
[:last-child Selector](//api.jquery.com/last-child-selector/)  
[:last-of-type Selector](//api.jquery.com/last-of-type-selector/)  
[:last Selector](//api.jquery.com/last-selector/)  
[:lt() Selector](//api.jquery.com/lt-selector/)  
[Multiple Attribute Selector [name=”value”][name2=”value2″]](//api.jquery.com/multiple-attribute-selector/)  
[Multiple Selector (“selector1, selector2, selectorN”)](//api.jquery.com/multiple-selector/)  
[Next Adjacent Selector (“prev + next”)](//api.jquery.com/next-adjacent-selector/)  
[Next Siblings Selector (“prev ~ siblings”)](//api.jquery.com/next-siblings-selector/)  
[:not() Selector](//api.jquery.com/not-selector/)  
[:nth-child() Selector](//api.jquery.com/nth-child-selector/)  
[:nth-last-child() Selector](//api.jquery.com/nth-last-child-selector/)  
[:nth-last-of-type() Selector](//api.jquery.com/nth-last-of-type-selector/)  
[:nth-of-type() Selector](//api.jquery.com/nth-of-type-selector/)  
[:odd Selector](//api.jquery.com/odd-selector/)  
[:only-child Selector](//api.jquery.com/only-child-selector/)  
[:only-of-type Selector](//api.jquery.com/only-of-type-selector/)  
[:parent Selector](//api.jquery.com/parent-selector/)  
[:password Selector](//api.jquery.com/password-selector/)  
[:radio Selector](//api.jquery.com/radio-selector/)  
[:reset Selector](//api.jquery.com/reset-selector/)  
[:root Selector](//api.jquery.com/root-selector/)  
[:selected Selector](//api.jquery.com/selected-selector/)  
[:submit Selector](//api.jquery.com/submit-selector/)  
[:target Selector](//api.jquery.com/target-selector/)  
[:text Selector](//api.jquery.com/text-selector/)  
[:visible Selector](//api.jquery.com/visible-selector/)  

### jQuery 事件
[.bind()](//api.jquery.com/bind/)  
[.blur()](//api.jquery.com/blur/)  
[.change()](//api.jquery.com/change/)  
[.click()](//api.jquery.com/click/)  
[.dblclick()](//api.jquery.com/dblclick/)  
[.delegate()](//api.jquery.com/delegate/)  
[.die()](//api.jquery.com/die/)  
[.error()](//api.jquery.com/error/)  
[event.currentTarget](//api.jquery.com/event.currentTarget/)  
[event.data](//api.jquery.com/event.data/)  
[event.delegateTarget](//api.jquery.com/event.delegateTarget/)  
[event.isDefaultPrevented()](//api.jquery.com/event.isDefaultPrevented/)  
[event.isImmediatePropagationStopped()](//api.jquery.com/event.isImmediatePropagationStopped/)  
[event.isPropagationStopped()](//api.jquery.com/event.isPropagationStopped/)  
[event.metaKey](//api.jquery.com/event.metaKey/)  
[event.namespace](//api.jquery.com/event.namespace/)  
[event.pageX](//api.jquery.com/event.pageX/)  
[event.pageY](//api.jquery.com/event.pageY/)  
[event.preventDefault()](//api.jquery.com/event.preventDefault/)  
[event.relatedTarget](//api.jquery.com/event.relatedTarget/)  
[event.result](//api.jquery.com/event.result/)  
[event.stopImmediatePropagation()](//api.jquery.com/event.stopImmediatePropagation/)  
[event.stopPropagation()](//api.jquery.com/event.stopPropagation/)  
[event.target](//api.jquery.com/event.target/)  
[event.timeStamp](//api.jquery.com/event.timeStamp/)  
[event.type](//api.jquery.com/event.type/)  
[event.which](//api.jquery.com/event.which/)  
[.focus()](//api.jquery.com/focus/)  
[.focusin()](//api.jquery.com/focusin/)  
[.focusout()](//api.jquery.com/focusout/)  
[.hover()](//api.jquery.com/hover/)  
[jQuery.proxy()](//api.jquery.com/jQuery.proxy/)  
[.keydown()](//api.jquery.com/keydown/)  
[.keypress()](//api.jquery.com/keypress/)  
[.keyup()](//api.jquery.com/keyup/)  
[.live()](//api.jquery.com/live/)  
[.load()](//api.jquery.com/load-event/)  
[.mousedown()](//api.jquery.com/mousedown/)  
[.mouseenter()](//api.jquery.com/mouseenter/)  
[.mouseleave()](//api.jquery.com/mouseleave/)  
[.mousemove()](//api.jquery.com/mousemove/)  
[.mouseout()](//api.jquery.com/mouseout/)  
[.mouseover()](//api.jquery.com/mouseover/)  
[.mouseup()](//api.jquery.com/mouseup/)  
[.off()](//api.jquery.com/off/)  
[.on()](//api.jquery.com/on/)  
[.one()](//api.jquery.com/one/)  
[.ready()](//api.jquery.com/ready/)  
[.resize()](//api.jquery.com/resize/)  
[.scroll()](//api.jquery.com/scroll/)  
[.select()](//api.jquery.com/select/)  
[.submit()](//api.jquery.com/submit/)  
[.toggle()](//api.jquery.com/toggle-event/)  
[.trigger()](//api.jquery.com/trigger/)  
[.triggerHandler()](//api.jquery.com/triggerHandler/)  
[.unbind()](//api.jquery.com/unbind/)  
[.undelegate()](//api.jquery.com/undelegate/)  
[.unload()](//api.jquery.com/unload/)  

### Ajax 请求
[.ajaxComplete()](//api.jquery.com/ajaxComplete/)  
[.ajaxError()](//api.jquery.com/ajaxError/)  
[.ajaxSend()](//api.jquery.com/ajaxSend/)  
[.ajaxStart()](//api.jquery.com/ajaxStart/)  
[.ajaxStop()](//api.jquery.com/ajaxStop/)  
[.ajaxSuccess()](//api.jquery.com/ajaxSuccess/)  
[jQuery.ajax()](//api.jquery.com/jQuery.ajax/)  
[jQuery.ajaxPrefilter()](//api.jquery.com/jQuery.ajaxPrefilter/)  
[jQuery.ajaxSetup()](//api.jquery.com/jQuery.ajaxSetup/)  
[jQuery.ajaxTransport()](//api.jquery.com/jQuery.ajaxTransport/)  
[jQuery.get()](//api.jquery.com/jQuery.get/)  
[jQuery.getJSON()](//api.jquery.com/jQuery.getJSON/)  
[jQuery.getScript()](//api.jquery.com/jQuery.getScript/)  
[jQuery.param()](//api.jquery.com/jQuery.param/)  
[jQuery.post()](//api.jquery.com/jQuery.post/)  
[.load()](//api.jquery.com/load/)  
[.serialize()](//api.jquery.com/serialize/)  
[.serializeArray()](//api.jquery.com/serializeArray/)  
