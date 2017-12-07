---
layout:     post
title:      Thrift 简介
date:       2016-01-15
categories: Development
---

# Thrift  
Thrift是一个跨语言的服务部署框架，最初由Facebook于2007年开发，2008年进入Apache开源项目。Thrift通过一个中间语言(IDL, 接口定义语言)来定义RPC的接口和数据类型，然后通过一个编译器生成不同语言的代码

## IDL
### 类型

#### 基本类型
* bool: 布尔类型
* byte: 有符号字节
* i16: 16位有符号整型
* i32: 32位有符号整型
* i64: 64位有符号整型
* double: 64位浮点数
* string: 未知编码或者二进制的字符串
  
thrift不支持无符号整型

#### 容器类型
* List<T>：一系列T类型的元素组成的有序表，元素可以重复
* Set<T>：一系列T类型的元素组成的无序表，元素唯一
* Map<K,V>：key/value对,key的类型是K且key唯一，value类型是V
  
容器中的元素类型可以是除了service外的任何合法thrift类型（包括结构体和异常）

#### 结构体
Thrift结构体在概念上同C语言结构体类型。在面向对象语言中，thrift结构体被转换成类。
{% highlight c %}
struct Tweet {
  1: required i32 userId;                  //每个域有一个唯一的，正整数标识符
  2: required string userName;             //每个域可以标识为required或者optional，也可以不注明
  3: required string text;
  4: optional Location loc;                //结构体可以包含其他结构体
  16: optional string language = "english" //域可以有缺省值
}
 
struct Location {                          //一个thrift中可定义多个结构体，并存在引用关系
  1: required double latitude;
  2: required double longitude;
}
{% endhighlight %}
规范的struct定义中的每个域均会使用required或者optional关键字进行标识。如果required标识的域没有赋值，thrift将给予提示。如果optional标识的域没有赋值，该域将不会被序列化传输。如果某个optional标识域有缺省值而用户没有重新赋值，则该域的值一直为缺省值。
与service不同，结构体不支持继承。

#### 异常
异常在语法和功能上类似于结构体，只不过异常使用关键字exception而不是struct关键字声明。当定义一个RPC服务时，开发者可能需要声明一个远程方法抛出一个异常。

#### 服务
服务的定义方法在语法上等同于面向对象语言中定义接口。Thrift编译器会产生实现这些接口的client和server桩。
在流行的序列化/反序列化框架（如protocol buffer）中，thrift是少有的提供多语言间RPC服务的框架。
Thrift编译器会根据选择的目标语言为server产生服务接口代码，为client产生桩代码。
{% highlight c %}
service Twitter {                               //"Twitter"与"{"之间需要有空格
//方法定义方式类似于C语言中的方式，它有一个返回值，一系列参数和可选的异常列表  
//参数列表和异常列表定义方式与结构体中域定义方式一致  
  void ping(),                                    //函数定义可以使用逗号或者分号标识结束
  bool postTweet(1:Tweet tweet);                  //参数可以是基本类型或者结构体，参数是只读的
  TweetSearchResult searchTweets(1:string query); //返回值可以是基本类型或者结构体
  //"oneway"标识符表示client发出请求后不必等待回复（非阻塞）直接进行下面的操作，
  //"oneway"方法的返回值必须是void
  oneway void zip()                               //返回值可以是void
 
}
{% endhighlight %}
函数中参数列表的定义方式与struct完全一样
Service支持继承，一个service可使用extends关键字继承另一个service

### 类型定义
Thrift支持C/C++风格的typedef  
{% highlight c %}
typedef i32 MyInteger   \\末尾没有逗号  
typedef Tweet ReTweet   \\struct可以使用typedef
{% endhighlight %}

### 枚举类型
可以像C/C++那样定义枚举类型，如：
{% highlight c %}
enum TweetType {
  TWEET,       //编译器默认从0开始赋值
  RETWEET = 2, //可以赋予某个常量某个整数
  DM = 0xa,    //允许常量是十六进制整数
  REPLY        //末尾没有逗号
}
 
struct Tweet {
  1: required i32 userId;
  2: required string userName;
  3: required string text;
  4: optional Location loc;
  5: optional TweetType tweetType = TweetType.TWEET //给常量赋缺省值时，使用常量的全称
  16: optional string language = "english"
}

{% endhighlight %}
不同于protocol buffer，thrift不支持枚举类嵌套，枚举常量必须是32位的正整数

### 注释
Thrfit支持shell注释风格，C/C++语言中单行或者多行注释风格
{% highlight sh %}
# This is a valid comment.
{% endhighlight %}
{% highlight c %}
/*
* This is a multi-line comment.
* Just like in C.
*/
// C++/Java style single-line comments work just as well.
{% endhighlight %}

#### 命名空间
Thrift中的命名空间同C++中的namespace和java中的package类似，它们均提供了一种组织（隔离）代码的方式。因为每种语言均有自己的命名空间定义方式（如python中有module），thrift允许开发者针对特定语言定义namespace：
{% highlight text %}
namespace cpp com.example.project  //转化成namespace com { namespace example { namespace project {
namespace java com.example.project //转换成package com.example.project
{% endhighlight %}

#### 文件包含
Thrift允许thrift文件包含，用户需要使用thrift文件名作为前缀访问被包含的对象，如：
{% highlight text %}
include "tweet.thrift"          //thrift文件名要用双引号包含，末尾没有逗号或者分号
...
struct TweetSearchResult {
  1: list<tweet.Tweet> tweets;  //注意tweet前缀
}
{% endhighlight %}

#### 常量
Thrift允许用户定义常量，复杂的类型和结构体可使用JSON形式表示。
{% highlight text %}
const i32 INT_CONST = 1234;    //分号是可选的，可有可无；支持十六进制赋值
const map<string,string> MAP_CONST = {"hello": "world", "goodnight": "moon"}
{% endhighlight %}

## 支持语言
* [C GLib](https://thrift.apache.org/lib/c_glib)
* [C++](https://thrift.apache.org/lib/cpp)
* [C#](https://thrift.apache.org/lib/csharp)
* [Delphi](https://thrift.apache.org/lib/delphi)
* [Erlang](https://thrift.apache.org/lib/erl)
* [Go](https://thrift.apache.org/lib/go)
* [Haskell](https://thrift.apache.org/lib/hs)
* [Java](https://thrift.apache.org/lib/java)
* [JavaScript](https://thrift.apache.org/lib/js)
* [Node.js](https://thrift.apache.org/lib/nodejs)
* [OCaml](https://thrift.apache.org/lib/ocaml)
* [Perl](https://thrift.apache.org/lib/perl)
* [PHP](https://thrift.apache.org/lib/php)
* [Python](https://thrift.apache.org/lib/py)
* [Ruby](https://thrift.apache.org/lib/rb)

## 网络模型
````text
  +-------------------------------------------+
  | Server                                    |
  | (single-threaded, event-driven etc)       |
  +-------------------------------------------+
  | Processor                                 |
  | (compiler generated)                      |
  +-------------------------------------------+
  | Protocol                                  |
  | (JSON, compact etc)                       |
  +-------------------------------------------+
  | Transport                                 |
  | (raw TCP, HTTP etc)                       |
  +-------------------------------------------+
````
#### 传输格式
* TBinaryProtocol: 二进制格式
* TCompactProtocol: 压缩格式
* TJSONProtocol: JSON格式
* TSimpleJSONProtocol: 提供JSON只写协议, 生成的文件很容易通过脚本语言解析
* TDebugProtocol: 使用易懂的可读的文本格式，以便于debug
#### 数据传输方式
* TSocket: 阻塞式socker
* TFramedTransport: 以frame为单位进行传输，非阻塞式服务中使用
* TFileTransport: 以文件形式进行传输
* TMemoryTransport: 将内存用于I/O. java实现时内部实际使用了简单的ByteArrayOutputStream
* TZlibTransport: 使用zlib进行压缩， 与其他传输方式联合使用。当前无java实现
#### 服务模型
* TSimpleServer: 简单的单线程服务模型，常用于测试
* TThreadPoolServer: 多线程服务模型，使用标准的阻塞式IO
* TNonblockingServer: 多线程服务模型，使用非阻塞式IO，需使用TFramedTransport数据传输方式