---
layout: post
title:  "Hello Bower!"
date:   2015-09-25 14:00:00
categories: Development
---

### 前端包管理器
一个可以帮助你管理你的应用的前端依赖库的包管理器

### 安装Bower
{% highlight sh %}
npm install -g bower
bower -v
{% endhighlight %}

### 寻找包
{% highlight sh %}
bower search <word>
{% endhighlight %}

### 安装包
{% highlight sh %}
bower install <package>
bower install <package>#<version>
{% endhighlight %}
{% highlight json %}
//.bowerrc
{
  "directory" : "public/components"
  "proxy": "http://127.0.0.1:8087",
  "https-proxy": "http://127.0.0.1:8087"
}
{% endhighlight %}

### 一键安装
{% highlight sh %}
bower install
{% endhighlight %}
{% highlight json %}
//bower.json
{
  "name": "hello",
  "version": "0.1.1",
  "homepage": "http://blog.halo9pan.cn",
  "authors": [
    "Halo9Pan <halo9pan@gmail.com>"
  ],
  "description": "Hello Bower!",
  "main": "main.js",
  "moduleType": [
    "amd",
    "es6"
  ],
  "keywords": [
    "bower",
    "example"
  ],
  "license": "MIT",
  "private": true,
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "public/components",
    "test",
  ],
  "dependencies": {
    "jquery": "~2.1.4",
    "bootstrap": "~3.3.5",
    "backbone": "~1.2.3",
    "jquery-ui": "~1.11.4"
  }
}
{% endhighlight %}

### 列出安装的包
{% highlight sh %}
bower list
{% endhighlight %}

### 更新包
{% highlight sh %}
bower update
bower update <package>
{% endhighlight %}

### 卸载包
{% highlight sh %}
bower uninstall <package>
{% endhighlight %}

