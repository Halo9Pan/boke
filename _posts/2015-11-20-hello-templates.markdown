---
layout: post
title:  "Hello Templates!"
date:   2015-11-20 10:00:00
categories: Development
---

### Velocity

#### 基本语法
* "#"用来标识Velocity的脚本语句，包括#set、#if 、#else、#end、#foreach、#end、#iinclude、#parse、#macro等，如:  
{% highlight html %}
#if($info.images)
    <img src="$info.images">
#else
    <img src="noPhoto.jpg">
#end
{% endhighlight %}

* "$"用来标识一个变量，如
{% highlight html %}
$i
$msg.errorNum
{% endhighlight %}

* "!"用来强制把不存在的变量显示为空白
{% highlight html %}
$!msg
{% endhighlight %}

* 注释，如：
{% highlight html %}
## 这是一行注释，不会输出
#* xxxx
多行注释
xxxx *#
{% endhighlight %}

具体语法可参考[官网] (http://velocity.apache.org/engine/devel/user-guide.html)

#### 语法详解
* 变量赋值输出
{% highlight html %}
Welcome $name to halo9pan.cn!
today is $date.
tdday is $mydae.//未被定义的变量将当成字符串
{% endhighlight %}

* 设置变量值,所有变量都以$开头
{% highlight html %}
#set( $iAmVariable = "good!" )
Welcome $name to halo9pan.cn!
today is $date.
$iAmVariable
{% endhighlight %}

* if,else判断
{% highlight html %}
#set ($admin = "admin")
#set ($user = "user")
#if ($admin == $user)
    Welcome admin! 
#else
    Welcome user!
#end
{% endhighlight %}

* 迭代数据List ($velocityCount为列举序号，默认从1开始)
{% highlight html %}
#foreach( $product in $allProducts ) 
    <li>$velocityCount $product.title</li>
#end
{% endhighlight %}

* 迭代数据get key
{% highlight html %}
#foreach($key in $myProducts.keySet() )
    $key `s value: $myProducts.get($key)
#end
{% endhighlight %}

* 导入其它文件,可输入多个
{% highlight html %}
#parse("vm_a.vm")
#parse("vm_b.vm")
{% endhighlight %}

* 简单遍历多个div
{% highlight html %}
#foreach( $i in [1,2,3,4] )
<div>$i</div>
#end
{% endhighlight %}

* 双引号与单引号
{% highlight html %}
#set ($var="helo")
test"$var" 返回testhello
test'$var' 返回test'$var'
{% endhighlight %}
可以通过设置 stringliterals.interpolate=false改变默认处理方式

* Range
{% highlight html %}
#foreach( $foo in [1..5] )
{% endhighlight %}

* 定义宏macros，相当于函数
{% highlight html %}
#macro( d )
  <tr><td></td></tr>
#end
调用
#d() 
{% endhighlight %}

* 带参数的宏
{% highlight html %}
#macro( tablerows $color $somelist )
  #foreach( $something in $somelist )
  <tr><td bgcolor=$color>$something</td></tr>
  #end
#end
{% endhighlight %}

### Freemarker

#### 基础语法

* 变量
{% highlight json %}
data = {
	"user": "halo9pan"	
}
{% endhighlight %}
* 模板
{% highlight html %}
<h1>Hello ${user}</h1>
{% endhighlight %}

* 字符串链接
{% highlight html %}
${"Hello,${user}!"}
${"Hello,"+user+"!"};
{% endhighlight %}
${..}只能用于文本部分，下面的代码是错误的：
{% highlight html %}
<#if ${isBig}>Wow!</#if>
<#if "${isBig}">Wow!</#if>
{% endhighlight %}
应该写成：
{% highlight html %}
<#if isBig>Wow!</#if>
{% endhighlight %}

具体语法可参考[官网] (http://freemarker.incubator.apache.org/docs/index.html)

#### 内建函数
1. html：对字符串进行HTML编码
2. cap_first:将字符串第一个字母成大写
3. lower_case:将字符串转换成小写
4. upper_case:将字符串转换成大写
5. trim: 去掉字符串前后的空白字符
6. size： 获得序列中元素的数目
7. int 取得数字的整数部分

#### 指令

* if
{% highlight json %}
data = {
	"user": "halo9pan",
	"isNewGuys": true
}
{% endhighlight %}
{% highlight html %}
<h1><#if user == "halo9pan">Your are hero.</#if></h1>
<p>
	<#if isNewGuys>
		new guys!
	<#else>
		old
	</#if>
</p>
{% endhighlight %}

* list
{% highlight json %}
data = [
	{
		"user": "halo9pan",
		"gender": "male"
	},
	{
		"user": "panhao",
		"gender": "male"
	}
]
{% endhighlight %}
{% highlight html %}
<ul>
	<#list data as item
	<li>name:${item.user};Gender:${item.gender}</li>
	</#list>
</ul>
{% endhighlight %}

* include
{% highlight html %}
<#include "/xxx.html">
{% endhighlight %}


* 空值的处理
{% highlight html %}
<h1>Welcome ${user!"default"}!</h1>
{% endhighlight %}
如果user**不存在**或者为**null**,最后模板会输出 **default**这个值   
反之，输出**user**的真实值！

{% highlight html %}
<#if user??><h1>hello ${user}</h1></#if>
{% endhighlight %}
如果，user 存在则输出包括的内容！   
反之，则忽略整条模板内容！