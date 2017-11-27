---
layout: post
title:  "Hello Node.js"
date:   2015-12-25 14:00:00
categories: Development
---

# Node.js
Node.js采用了Google Chrome浏览器的V8引擎，性能很好，同时还提供了很多系统级的API，如文件操作、网络编程等

#### Node.js采用事件驱动、异步编程，为网络服务而设计
#### Node.js以单进程、单线程模式运行

## [Node.js Documentation](https://nodejs.org/api/)

## Node.js异步编程风格

## MEAN
#### MongoDB
#### Express
#### Angular.js

## Debug

## logging
#### console + [forever](https://www.npmjs.com/package/forever) | [pm2](https://www.npmjs.com/package/pm2)
````sh
forever start app.js
pm2 start app.js
````

#### [Winston](https://www.npmjs.com/package/winston)
````js
const winston = require('winston');
 
winston.log('info', 'Hello distributed log files!');
winston.info('Hello again distributed logs');
winston.info('Hello world!', {timestamp: Date.now(), pid: process.pid});
winston.log('info', 'tHello number: %d', 123);
````
````js
const winston = require('winston');
const logger = new winston.Logger();
 
logger.log('info', 'Hello distributed log files!');
logger.info('Hello again distributed logs');
````
````js
winston.loggers.add('category1', {console: { ... }, file: { ... }});
winston.loggers.add('category2', {irc: { ... }, file: { ... }});
````

#### [Bunyan](https://www.npmjs.com/package/bunyan)
````js
var bunyan = require('bunyan');
var log = bunyan.createLogger({name: 'myapp'});
log.info('hi');
log.warn({lang: 'fr'}, 'au revoir');
````
