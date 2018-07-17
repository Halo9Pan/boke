---
layout:     slide
title:      Python asyncio Part A
date:       2018-07-14
categories: Programming
theme:      beige
mermaid:    true
---

<section data-markdown data-separator="^\n---\n$" data-separator-vertical="^\n----\n$">
<script type="text/template">

# Python asyncio

----

## Part A

---

### 概念介绍

----

<!-- .slide: style="text-align: left;"> -->
**异步 IO**：就是发起一个 IO 操作，却不用等它结束，可以继续做其他事情。当它结束时，会得到通知。

----

<!-- .slide: style="text-align: left;"> -->
**asyncio**，并发（concurrency）的一种方式。对 Python 来说，并发还可以通过线程（threading）和多进程（multiprocessing）来实现。

----

<!-- .slide: style="text-align: left;"> -->
**asyncio**，并不能带来真正的并行（parallelism），因为 GIL（全局解释器锁）的存在，Python 的多线程也不能带来真正的并行。

----

<!-- .slide: style="text-align: left;"> -->
**协程**：可交给 asyncio 执行的任务，称为协程（coroutine）。一个协程可以放弃执行，把机会让给其它协程（即 yield from 或 await）

----

### 定义协程

----

````python
async def do_some_work(x):
    print("Waiting " + str(x))
    await asyncio.sleep(x)
````

----

````python
import asyncio

@asyncio.coroutine
def decorated(x):
    yield from x
````
````python
async def native(x):
    await x
````

[PEP 492](https://www.python.org/dev/peps/pep-0492/#differences-from-generators)

----

````python
asyncio.iscoroutinefunction(decorated)
asyncio.iscoroutinefunction(native)
````
````python
inspect.isgeneratorfunction(decorated)
inspect.isgeneratorfunction(native)
````

----

* result = await future or result = yield from future
* result = await coroutine or result = yield from coroutine
* return expression
* raise exception

----

### 运行协程

----

````python
loop = asyncio.get_event_loop() # blocking
loop.run_until_complete(do_some_work(4))
````

----

### 协程回调

----

````python
def done_callback(futu):
    print('Done')

futu = asyncio.ensure_future(do_some_work(4))
futu.add_done_callback(done_callback)

loop.run_until_complete(futu)
````

----

### 多个协程

----

#### asyncio.gather

````python
coros = [do_some_work(4), do_some_work(8)]
loop.run_until_complete(asyncio.gather(*coros))
````

````python
futures = [asyncio.ensure_future(do_some_work(4)),
             asyncio.ensure_future(do_some_work(8))]

loop.run_until_complete(asyncio.gather(*futures))
````

---

### API 简介

----

#### Event loops

* [get_event_loop()](https://docs.python.org/3.6/library/asyncio-eventloops.html#asyncio.get_event_loop)
* [set_event_loop(loop)](https://docs.python.org/3.6/library/asyncio-eventloops.html#asyncio.set_event_loop)
* [new_event_loop()](https://docs.python.org/3.6/library/asyncio-eventloops.html#asyncio.new_event_loop)

----

![Base Event Loop](../assets/python-asyncio_classes1.png)

----

* [get_event_loop_policy()](https://docs.python.org/3.6/library/asyncio-eventloops.html#asyncio.get_event_loop_policy)
* [set_event_loop_policy(policy)](https://docs.python.org/3.6/library/asyncio-eventloops.html#asyncio.set_event_loop_policy)

----

![Base Event Loop](../assets/python-asyncio_classes2.png)

----

#### Tasks and coroutines

* [@asyncio.coroutine](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.coroutine)

----

* [as_completed(fs, *, loop=None, timeout=None)](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.as_completed)
* [ensure_future(coro_or_future, *, loop=None)](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.ensure_future)
* [wrap_future(future, *, loop=None)](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.wrap_future)
* [gather(*coros_or_futures, loop=None, return_exceptions=False)](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.gather)
* [wait(futures, *, loop=None, timeout=None, return_when=ALL_COMPLETED)](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.wait)
* [wait_for(fut, timeout, *, loop=None)](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.wait_for)

----

* [iscoroutine(obj)](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.iscoroutine)
* [iscoroutinefunction(func)](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.iscoroutinefunction)
* [run_coroutine_threadsafe(coro, loop)](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.run_coroutine_threadsafe)
* [sleep(delay, result=None, *, loop=None)](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.sleep)
* [shield(arg, *, loop=None)](https://docs.python.org/3.6/library/asyncio-task.html#asyncio.shield)

----

![Base Event Loop](../assets/python-asyncio_classes3.png)

----

#### Transports

----

![Base Event Loop](../assets/python-asyncio_classes4.png)

----

#### Protocols

----

![Base Event Loop](../assets/python-asyncio_classes5.png)

----

#### Streams

* [open_connection(host=None, port=None, *, loop=None, limit=None, **kwds)](https://docs.python.org/3.6/library/asyncio-stream.html#asyncio.open_connection)
* [start_server(client_connected_cb, host=None, port=None, *, loop=None, limit=None, **kwds)](https://docs.python.org/3.6/library/asyncio-stream.html#asyncio.start_server)
* [open_unix_connection(path=None, *, loop=None, limit=None, **kwds)](https://docs.python.org/3.6/library/asyncio-stream.html#asyncio.open_unix_connection)
* [start_unix_server(client_connected_cb, path=None, *, loop=None, limit=None, **kwds)](https://docs.python.org/3.6/library/asyncio-stream.html#asyncio.start_unix_server)

----

![Base Event Loop](../assets/python-asyncio_classes6.png)

----

#### Subprocess

* [create_subprocess_exec(*args, stdin=None, stdout=None, stderr=None, loop=None, limit=None, **kwds)](https://docs.python.org/3.6/library/asyncio-subprocess.html#asyncio.create_subprocess_exec)
* [create_subprocess_shell(cmd, stdin=None, stdout=None, stderr=None, loop=None, limit=None, **kwds)](https://docs.python.org/3.6/library/asyncio-subprocess.html#asyncio.create_subprocess_shell)

----

#### Synchronization

----

![Base Event Loop](../assets/python-asyncio_classes7.png)

----

#### Queues

----

![Base Event Loop](../assets/python-asyncio_classes8.png)


---

### 示例

----

#### Hello World
````python
import asyncio

async def hello_world():
    print("Hello World!")

loop = asyncio.get_event_loop()
# Blocking call which returns when the hello_world() coroutine is done
loop.run_until_complete(hello_world())
loop.close()
````

----

#### 打印时间
````python
import asyncio
import datetime

async def display_date(loop):
    end_time = loop.time() + 5.0
    while True:
        print(datetime.datetime.now())
        if (loop.time() + 1.0) >= end_time:
            break
        await asyncio.sleep(1)

loop = asyncio.get_event_loop()
# Blocking call which returns when the display_date() coroutine is done
loop.run_until_complete(display_date(loop))
loop.close()
````

----

#### 链式协程
````python
import asyncio

async def compute(x, y):
    print("Compute %s + %s ..." % (x, y))
    await asyncio.sleep(1.0)
    return x + y

async def print_sum(x, y):
    result = await compute(x, y)
    print("%s + %s = %s" % (x, y, result))

loop = asyncio.get_event_loop()
loop.run_until_complete(print_sum(1, 2))
loop.close()
````

----

![链式协程](https://docs.python.org/3.6/_images/tulip_coro.png)

----

#### Future
````python
import asyncio

async def slow_operation(future):
    await asyncio.sleep(1)
    future.set_result('Future is done!')

loop = asyncio.get_event_loop()
future = asyncio.Future()
asyncio.ensure_future(slow_operation(future))
loop.run_until_complete(future)
print(future.result())
loop.close()
````

----

#### 服务运行
````python
import asyncio

async def slow_operation(future):
    await asyncio.sleep(1)
    future.set_result('Future is done!')

def got_result(future):
    print(future.result())
    loop.stop()

loop = asyncio.get_event_loop()
future = asyncio.Future()
asyncio.ensure_future(slow_operation(future))
future.add_done_callback(got_result)
try:
    loop.run_forever()
finally:
    loop.close()
````

----

#### 并行任务
````python
import asyncio

async def factorial(name, number):
    f = 1
    for i in range(2, number+1):
        print("Task %s: Compute factorial(%s)..." % (name, i))
        await asyncio.sleep(1)
        f *= i
    print("Task %s: factorial(%s) = %s" % (name, number, f))

loop = asyncio.get_event_loop()
loop.run_until_complete(asyncio.gather(
    factorial("A", 2),
    factorial("B", 3),
    factorial("C", 4),
))
loop.close()
````

----

</script>
</section>
