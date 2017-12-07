---
layout:     post
title:      Javascript Module
date:       2016-02-26
categories: Development
---

## 模块化的发展
````js
function m1(){
  //...
}
function m2(){
  //...
}
````
````js
var module1 = new Object({
  _count: 0,
　m1: function (){
    //...
  },
  m2: function (){
    //...
  }
});
````
````js
(function () {
　var m1 = function (){
    //...
  };
  var m2 = function (){
    //...
  };
}());
````
````js
var module1 = (function(){
  var _count = 0;
　var _m1 = function (){
    //...
  };
  var _m2 = function (){
    //...
  };
  return {
    m1: _m1,
    m2: _m2
  };
});
````
````js
var module1 = (function (mod){
  mod.m3 = function () {
    //...
  };
  return mod;
})(module1);
````
````js
var module1 = (function ($) {
  //...
})(jQuery);
````

## CommonJS
CommonJS定义的模块分为:{模块引用(require)} {模块定义(exports)} {模块标识(module)}  
require()用来引入外部模块；exports对象用于导出当前模块的方法或变量，唯一的导出口；module对象就代表模块本身。  

````js
const http = require('http');

http.createServer( (request, response) => {
  response.writeHead(200, {'Content-Type': 'text/plain'});
  response.end('Hello World\n');
}).listen(12880);

console.log('Server running at http://127.0.0.1:12880/');
````

## AMD
Asynchronous Module Definition  

````js
require([module], callback);
````
[require.js](http://requirejs.org/)  
[curl.js](https://github.com/cujojs/curl)  

````js
// main.js
require(['moduleA', 'moduleB', 'moduleC'], function (moduleA, moduleB, moduleC){
  // some code here
});
````
````js
require(['jquery', 'underscore', 'backbone'], function ($, _, bb){
  // some code here
});
````
````js
require.config({
  paths: {
    "jquery": "libs/jquery.min",
    "underscore": "libs/underscore.min",
    "backbone": "libs/backbone.min"
  }
});
````
````js
// math.js
define(function (){
  var add = function (x,y){
    return x+y;
  };
  return {
    add: add
  };
});
````
````js
require.config({
  shim: {
    'underscore':{
      exports: '_'
    },
    'backbone': {
      deps: ['underscore', 'jquery'],
      exports: 'Backbone'
    }
  }
});
````