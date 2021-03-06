---
layout:     post
title:      Grunt 简介
date:       2015-09-25
categories: Development
---

### Node包管理器
一个自动任务运行器

### 安装Grunt
{% highlight sh %}
npm install -g grunt-cli
npm install grunt
{% endhighlight %}

### NPM简单命令
{% highlight sh %}
npm search <word>
npm list
npm install -g <package>
npm install <package>
npm update -g
npm update
npm update <package>
npm uninstall <package>
{% endhighlight %}

### 安装
{% highlight sh %}
npm --registry https://registry.npm.taobao.org install <package>
npm config set registry https://registry.npm.taobao.org
{% endhighlight %}
{% highlight sh %}
npm init
npm install grunt --save-dev
npm install \
grunt-contrib-clean\
grunt-contrib-jshint \
grunt-contrib-concat \
grunt-contrib-uglify \
grunt-contrib-copy \
grunt-contrib-watch \
grunt-contrib-qunit \
grunt-contrib-cssmin \
--save-dev
{% endhighlight %}
{% highlight js %}
//.npmrc
# proxy=http://127.0.0.1:8087/
# https-proxy=http://127.0.0.1:8087/
registry=https://registry.npm.taobao.org
{% endhighlight %}

### 一键安装
{% highlight sh %}
npm install
{% endhighlight %}
{% highlight json %}
//package.json
{
  "name": "hello",
  "version": "0.1.1",
  "description": "Hello Grunt",
  "main": "index.js",
  "scripts": {
    "test": "grunt qunit"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/Halo9Pan/boke.git"
  },
  "keywords": [
    "grunt",
    "example"
  ],
  "author": "Halo9Pan <halo9pan@gmail.com>",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/Halo9Pan/boke/issues"
  },
  "homepage": "https://github.com/Halo9Pan/boke#readme",
  "devDependencies": {
    "grunt": "^0.4.5",
    "grunt-contrib-clean": "^0.6.0",
    "grunt-contrib-concat": "^0.5.1",
    "grunt-contrib-copy": "^0.8.1",
    "grunt-contrib-cssmin": "^0.14.0",
    "grunt-contrib-jshint": "^0.11.3",
    "grunt-contrib-qunit": "^0.7.0",
    "grunt-contrib-uglify": "^0.9.2",
    "grunt-contrib-watch": "^0.6.1"
  }
}
{% endhighlight %}

### 命令脚本文件Gruntfile.js
{% highlight js %}
module.exports = function(grunt) {

  // 配置Grunt各种模块的参数
  grunt.initConfig({
    clean: [
      'dist'
    ],
    jshint: {},
    concat: {
      options: {
        separator: ';',
        sourceMap: true
      },
      dist: {
        src: [
          'public/components/jquery/dist/jquery.js',
          'public/components/underscore/underscore.js',
          'public/components/backbone/backbone.js'
        ],
        dest: 'dist/components.min.js'
      },
      min: {
        src: [
          'public/components/jquery/dist/jquery.min.js',
          'public/components/underscore/underscore-min.js',
          'public/components/backbone/backbone-min.js'
        ],
        dest: 'dist/components.min.js'
      }
    },
    uglify: {
      options: {
        mangle: true,
        beautify: true
      },
      target: {
        expand: true,
        cwd: 'boilerplate/js/',
        src: ['*.js', '!*.min.js'],
        dest: 'dist/js/',
        ext: '.min.js'
      }
    },
    cssmin: {
      minify: {
        expand: true,
        cwd: 'boilerplate/css/',
        src: ['*.css', '!*.min.css'],
        dest: 'dist/css/',
        ext: '.min.css'
      },
      combine: {
        files: {
          'dist/css/public.min.css': [
            'public/components/bootstrap/dist/css/bootstrap.min.css',
            'public/components/bootstrap/dist/css/bootstrap-theme.min.css'
          ]
        }
      }
    },
    watch:  { /* watch的参数 */ }
  });

  // 从node_modules目录加载模块文件
  grunt.loadNpmTasks('grunt-contrib-clean');
  grunt.loadNpmTasks('grunt-contrib-jshint');
  grunt.loadNpmTasks('grunt-contrib-copy');
  grunt.loadNpmTasks('grunt-contrib-concat');
  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-contrib-cssmin');
  grunt.loadNpmTasks('grunt-contrib-watch');
  grunt.loadNpmTasks('grunt-contrib-qunit');

  // 每行registerTask定义一个任务
  grunt.registerTask('default', ['concat', 'uglify']);
  grunt.registerTask('check', ['jshint']);
  grunt.registerTask('css', ['cssmin:minify','cssmin:combine']);

};
{% endhighlight %}

