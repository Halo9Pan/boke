---
layout: post
title:  "Hello Templates!"
date:   2015-11-20 10:00:00
categories: Development
---

### Velocity
#### 基本语法

* 1."#"用来标识Velocity的脚本语句，包括#set、#if 、#else、#end、#foreach、#end、#iinclude、#parse、#macro等，如:
		#if($info.images)
		<img src="$info.images">
		#else
		<img src="noPhoto.jpg">
		#end

* 2."$"用来标识一个变量，如
		$i、$msg.errorNum

* 3."!"用来强制把不存在的变量显示为空白
		$!msg

* 4.注释，如：
		## 这是一行注释，不会输出
    #* xxxx
    多行注释
    xxxx *#

#### 语法详解
具体语法可参考[官网] (http://velocity.apache.org/engine/devel/user-guide.html)

* 1.变量赋值输出
		Welcome $name to halo9pan.cn!
		today is $date.
		tdday is $mydae.//未被定义的变量将当成字符串

* 2.设置变量值,所有变量都以$开头
		#set( $iAmVariable = "good!" )
		Welcome $name to halo9pan.cn!
		today is $date.
		$iAmVariable

* 3.if,else判断
		#set ($admin = "admin")
		#set ($user = "user")
		#if ($admin == $user)
		Welcome admin!
		#else
		Welcome user!
		#end

* 4.迭代数据List ($velocityCount为列举序号，默认从1开始) 
		#foreach( $product in $allProducts )
			<li>$velocityCount $product.title</li>
		#end

* 5.迭代数据get key
		#foreach($key in $myProducts.keySet() )  
			$key `s value: $myProducts.get($key)
		#end

* 6.导入其它文件,可输入多个
		#parse("vm_a.vm")
		#parse("vm_b.vm")

* 7.简单遍历多个div
    #foreach( $i in [1,2,3,4] )
        <div>$i</div>
    #end

* 8.双引号 与 引号 
    #set ($var="helo")
    test"$var" 返回testhello
    test'$var' 返回test'$var'
    可以通过设置 stringliterals.interpolate=false改变默认处理方式

* 9.Range
    #foreach( $foo in [1..5] )

* 10.定义宏macros，相当于函数
    #macro( d )
      <tr><td></td></tr>
    #end
    调用 
    #d()

* 11.带参数的宏
    #macro( tablerows $color $somelist )
      #foreach( $something in $somelist )
      <tr><td bgcolor=$color>$something</td></tr>
      #end
    #end

#### Collection

#### Event

#### View

#### Router
