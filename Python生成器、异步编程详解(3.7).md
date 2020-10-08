Note from：3.5版本以前[摘自](https://snarky.ca/how-the-heck-does-async-await-work-in-python-3-5/)，3.7版本的asyncio[摘自](https://realpython.com/async-io-python/)
#yield(2.2+)
生成器中不能使用return语句(只有生成器这一种不能带return)，要使用`yield`。`yield`代表执行到此处暂停。如果使用的是`next()`方法，则会返回`yield`后面这个index变量。
当再次执行`next()`时会继续从刚才的地方继续执行知道碰到下一个`yield`。代码：
```python
def lazy_range(up_to):
    """Generator to return the sequence of integers from 0 to up_to, exclusive."""
    index = 0
    while index < up_to:
        yield index
        index += 1

iterator = lazy_range(2) # 虽说是生成器，但是从表现来看像是迭代器
# 使用foreach iterator并输出等价于下面
print(next(iterator)) # 输出0
print(next(iterator)) # 输出1
```
使用Python2.5版本开始的`iterator.send(object)`可以向`yield`处发送对象，并赋值给`yield`左边的变量jump（左进右出）。代码：
```python
def jumping_range(up_to):
    """Generator for the sequence of integers from 0 to up_to, exclusive.

    Sending a value into the generator will shift the sequence by that amount.
    """
    index = 0
    while index < up_to:
        jump = yield index
        if jump is None:
            jump = 1
        index += jump

iterator = jumping_range(5)
print(next(iterator))  # 0
print(iterator.send(2))  # 2
print(next(iterator))  # 3
print(iterator.send(-1))  # 2
for x in iterator:
    print(x)  # 3, 4
```
#从yield到yield from(3.3+)
`yield from iterator`其实就等价于`for x in iterator:  yield x`，内部还会自动处理一些异常。使用上面的lazy_range为例，将`yield`改造为`yield from`：
```python
def lazy_range(up_to):
    """Generator to return the sequence of integers from 0 to up_to, exclusive."""
    index = 0
    def gratuitous_refactor():
        nonlocal index
        while index < up_to:
            yield index
            index += 1
    yield from gratuitous_refactor()
```
再看一个复杂一些的例子
```python
def bottom():
    # Returning the yield lets the value that goes up the call stack to come right back
    # down.
    return (yield 42)

def middle():
    return (yield from bottom())

def top():
    return (yield from middle())

# Get the generator.
gen = top()
value = next(gen)
print(value)  # Prints '42'.
try:
    value = gen.send(value * 2)
except StopIteration as exc: # 这里是因为send执行后直接return了，再也找不到下一个yield点，所以会报错。而exc.value就是返回来的东西
    value = exc.value
print(value)  # Prints '84'.
```
#从yield from到await(3.5+)
##事件循环的概念
事件可以是IO、鼠标点击、页面滚动等，只要具有回调函数就会进入任务队列。循环可理解为异步任务队列每次暂停都把队首挪入队尾，然后执行下一个异步任务。
##asyncio.coroutine(3.4+)
###3.4版本写法
倒计时程序代码：
```python
import asyncio

@asyncio.coroutine
def countdown(number, n):
    while n > 0:
        print('T-minus', n, '({})'.format(number))
        yield from asyncio.sleep(1)
        n -= 1

loop = asyncio.get_event_loop()
tasks = [
    asyncio.ensure_future(countdown("A", 2)),
    asyncio.ensure_future(countdown("B", 3))]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()

# 输出：
# T-minus 2 (A)
# T-minus 3 (B)
# T-minus 1 (A)
# T-minus 2 (B)
# T-minus 1 (B)
```
`asyncio.coroutine`装饰器用于标记一个协程函数，该函数会使用`asyncio`模块以及其事件循环。
通过这个coroutine的具体定义（与生成器提供的API相匹配），您可以在任何`asyncio.Future`对象上使用`yield from`，它将其传递给事件循环，在等待某事发生时暂停执行协程（`Future`对象是`asyncio`的一个实现细节，这不重要）。一旦`Future`对象到达事件循环，就会在那里进行监视，直到Future对象完成它需要做的任何事情。一旦`Future`完成它的事情，事件循环注意到并且暂停等待`Future`结果的协程再次开始，其结果使用其`send()`方法发送回协程。
<font size=1>小插曲：在3.4版本还提出了asyncio.async()，然而在3.4.4就被弃用，被asyncio.ensure_future()代替</font>
###3.5版本写法
将`@asyncio.coroutine`变成`async`，将`yield from`变成`await`（这种用法更推荐使用，因为这种新用法是原生协程，而不是基于生成器的协程）。注意不标注`async`就不能用`await`！(3.5新增的`types.coroutine`为`asyncio.coroutine`的新名称，但二者都是基于生成器的协程[已被弃用](https://docs.python.org/3/library/asyncio-task.html#asyncio.coroutine)，将在3.10版本中移除)
```python
async def countdown(number, n):
    while n > 0:
        print('T-minus', n, '({})'.format(number))
        await asyncio.sleep(1)
        n -= 1
```
`inspect.iscoroutine(obj)`判断是否为async协程函数(3.5+)：其实就是是否加了`async`
`inspect.isawaitable(obj)`判断是否可以使用await表达式：其实就是是否加了`@types.coroutine`或者`async`
#asyncio模块极详解（纯3.7式写法）
**新特性**：
`asyncio.run(coroutine())`：`get_event_loop()` + `ensure_future()` + `run_until_complete()`
`asyncio.create_task()`：`asyncio.ensure_future()`的[更高等级](https://stackoverflow.com/questions/36342899/asyncio-ensure-future-vs-baseeventloop-create-task-vs-simple-coroutine)函数
`asyncio.gather(*awaitable()`：同时执行各个awaitable对象，与await连用
`async for`：异步生成器(3.6+)
简单输出例子：
```python
#!/usr/bin/env python3
# countasync.py

import asyncio

async def count():
    print("One")
    await asyncio.sleep(1)  # 不可以使用time.sleep,因为这必须是非阻塞调用而不是阻塞调用
    print("Two")

async def main():
    await asyncio.gather(count(), count(), count())

if __name__ == "__main__":
    import time
    s = time.perf_counter()  # 获得当前时间秒
    asyncio.run(main())
    elapsed = time.perf_counter() - s
    print(f"{__file__} executed in {elapsed:0.2f} seconds.")
```
一直输出随机数，复杂例子：
```python
#!/usr/bin/env python3
# rand.py

import asyncio
import random

# ANSI colors
c = (
    "\033[0m",   # End of color
    "\033[36m",  # Cyan
    "\033[91m",  # Red
    "\033[35m",  # Magenta
)

async def makerandom(idx: int, threshold: int = 6) -> int:
    print(c[idx + 1] + f"Initiated makerandom({idx}).")
    i = random.randint(0, 10)
    while i <= threshold:
        print(c[idx + 1] + f"makerandom({idx}) == {i} too low; retrying.")
        await asyncio.sleep(idx + 1)
        i = random.randint(0, 10)
    print(c[idx + 1] + f"---> Finished: makerandom({idx}) == {i}" + c[0])
    return i

async def main():
    res = await asyncio.gather(*(makerandom(i, 10 - i - 1) for i in range(3)))
    return res

if __name__ == "__main__":
    random.seed(444)
    r1, r2, r3 = asyncio.run(main())
    print()
    print(f"r1: {r1}, r2: {r2}, r3: {r3}")
```

Asyncio设计模式：链接协程[Chaining Coroutines](https://realpython.com/async-io-python/)、使用队列[Using a Queue](https://realpython.com/async-io-python/)

**使用**`async for`:
```python
async def mygen(u: int = 10):
    """Yield powers of 2."""
    i = 0
    while i < u:
        yield 2 ** i
        i += 1
        await asyncio.sleep(0.1)

async def main():
    # This does *not* introduce concurrent execution
    # It is meant to show syntax only
    g = [i async for i in mygen()]
    f = [j async for j in mygen() if not (j // 3 % 5)]
    return g, f

g, f = asyncio.run(main())
print(g, f)
```
**新特性**：
`asyncio.get_running_loop()`返回当前OS线程中的运行事件循环。
使用`create_task()`来安排协程对象的执行
```python
>>> import asyncio

>>> async def coro(seq) -> list:
...     """'IO' wait time is proportional to the max element."""
...     await asyncio.sleep(max(seq))
...     return list(reversed(seq))
...
>>> async def main():
...     # This is a bit redundant in the case of one task
...     # We could use `await coro([3, 2, 1])` on its own
...     t = asyncio.create_task(coro([3, 2, 1]))  # Python 3.7+
...     await t
...     print(f't: type {type(t)}')
...     print(f't done: {t.done()}')
...
>>> t = asyncio.run(main())
t: type <class '_asyncio.Task'>
t done: True
```

#时间轴
* 3.3: 表达式的yield from允许生成器委托。

* 3.4: asyncio是在Python标准库中引入的，具有临时API状态。

* 3.5: async和wait成为Python语法的一部分，用于表示和等待协程。它们还不是保留关键字。(您仍然可以定义名为async、await的函数和变量。)

* 3.6: 引入了异步生成器和异步理解。asyncio的API被声明为稳定的，而不是临时的。

* 3.7: async和wait变成了保留关键字。(它们不能用作标识符。)它们用于替换asyncio.coroutine()装饰器。在asyncio包中引入了asyncio.run()和其他一些特性。

#附录：
[EventLoop的两种不同实现](https://docs.python.org/3/library/asyncio-eventloop.html#event-loop-implementations)(第二章仅限Windows)

`asyncio`模块函数大全：
[`asyncio.run(coro, *, debug=False)`](https://docs.python.org/3/library/asyncio-task.html#asyncio.run)
[`asyncio.create_task(coro)`](https://docs.python.org/3/library/asyncio-task.html#asyncio.create_task)
[`coroutine asyncio.sleep(delay, result=None, *, loop=None)`](https://docs.python.org/3/library/asyncio-task.html#asyncio.sleep)
[`awaitable asyncio.gather(*aws, loop=None, return_exceptions=False)`](https://docs.python.org/3/library/asyncio-task.html#asyncio.gather)
[`awaitable asyncio.shield(aw, *, loop=None)`](https://docs.python.org/3/library/asyncio-task.html#asyncio.shield)
[`coroutine asyncio.wait_for(aw, timeout, *, loop=None)`](https://docs.python.org/3/library/asyncio-task.html#asyncio.wait_for)
[`coroutine asyncio.wait(aws, *, loop=None, timeout=None, return_when=ALL_COMPLETED)`](https://docs.python.org/3/library/asyncio-task.html#asyncio.wait)
[`asyncio.as_completed(aws, *, loop=None, timeout=None)`](https://docs.python.org/3/library/asyncio-task.html#asyncio.as_completed)
[`asyncio.run_coroutine_threadsafe(coro, loop)`](https://docs.python.org/3/library/asyncio-task.html#asyncio.run_coroutine_threadsafe)
[`asyncio.current_task(loop=None)`](https://docs.python.org/3/library/asyncio-task.html#asyncio.current_task)
[`asyncio.all_tasks(loop=None)`](https://docs.python.org/3/library/asyncio-task.html#asyncio.all_tasks)
```python
# https://docs.python.org/3/library/asyncio-task.html#asyncio.Task
class asyncio.Task(coro, *, loop=None):
    cancel()
    cancelled()
    done()
    result()
    exception()
    add_done_callback(callback, *, context=None)
    remove_done_callback(callback)
    get_stack(*, limit=None)
    print_stack(*, limit=None, file=None)
    classmethod all_tasks(loop=None)
    classmethod current_task(loop=None)
```

[loop.call_at](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.call_at)
[loop.call_soon](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.call_soon)
[loop.call_later](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.call_later)
[loop.call_soon_threadsafe](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.call_soon_threadsafe)

`gather`和`wait`不同之处：
1.gather任务无法取消。
2.返回值是一个结果列表
3.可以按照传入参数的顺序，顺序输出。
```python
import asyncio

async def num(n):
    try:
        await asyncio.sleep(n*0.1)
        return n
    except asyncio.CancelledError:
        print(f"数字{n}被取消")
        raise

async def main():
    tasks = [num(i) for i in range(10)]
    complete, pending = await asyncio.wait(tasks, timeout=0.5)
    for i in complete:
        print("当前数字",i.result())
    if pending:
        print("取消未完成的任务")
        for p in pending:
            p.cancel()

if __name__ == '__main__':
    asyncio.run(main())

# 当前数字 1
# 当前数字 2
# 当前数字 0
# 当前数字 4
# 当前数字 3
# 取消未完成的任务
# 数字5被取消
# 数字9被取消
# 数字6被取消
# 数字8被取消
# 数字7被取消
```
gather通常被用来阶段性的一个操作。gather会等待最耗时的那个完成之后才返回结果，耗时总时间取决于其中任务最长时间的那个。

循环倒计时程序(3.5):
```python
import datetime
import heapq
import types
import time


class Task:

    """Represent how long a coroutine should wait before starting again.

    Comparison operators are implemented for use by heapq. Two-item
    tuples unfortunately don't work because when the datetime.datetime
    instances are equal, comparison falls to the coroutine and they don't
    implement comparison methods, triggering an exception.
    
    Think of this as being like asyncio.Task/curio.Task.
    """

    def __init__(self, wait_until, coro):
        self.coro = coro
        self.waiting_until = wait_until

    def __eq__(self, other):
        return self.waiting_until == other.waiting_until

    def __lt__(self, other):
        return self.waiting_until < other.waiting_until


class SleepingLoop:

    """An event loop focused on delaying execution of coroutines.

    Think of this as being like asyncio.BaseEventLoop/curio.Kernel.
    """

    def __init__(self, *coros):
        self._new = coros
        self._waiting = []

    def run_until_complete(self):
        # Start all the coroutines.
        for coro in self._new:
            wait_for = coro.send(None)
            heapq.heappush(self._waiting, Task(wait_for, coro))
        # Keep running until there is no more work to do.
        while self._waiting:
            now = datetime.datetime.now()
            # Get the coroutine with the soonest resumption time.
            task = heapq.heappop(self._waiting)
            if now < task.waiting_until:
                # We're ahead of schedule; wait until it's time to resume.
                delta = task.waiting_until - now
                time.sleep(delta.total_seconds())
                now = datetime.datetime.now()
            try:
                # It's time to resume the coroutine.
                wait_until = task.coro.send(now)
                heapq.heappush(self._waiting, Task(wait_until, task.coro))
            except StopIteration:
                # The coroutine is done.
                pass


@types.coroutine
def sleep(seconds):
    """Pause a coroutine for the specified number of seconds.

    Think of this as being like asyncio.sleep()/curio.sleep().
    """
    now = datetime.datetime.now()
    wait_until = now + datetime.timedelta(seconds=seconds)
    # Make all coroutines on the call stack pause; the need to use `yield`
    # necessitates this be generator-based and not an async-based coroutine.
    actual = yield wait_until
    # Resume the execution stack, sending back how long we actually waited.
    return actual - now


async def countdown(label, length, *, delay=0):
    """Countdown a launch for `length` seconds, waiting `delay` seconds.

    This is what a user would typically write.
    """
    print(label, 'waiting', delay, 'seconds before starting countdown')
    delta = await sleep(delay)
    print(label, 'starting after waiting', delta)
    while length:
        print(label, 'T-minus', length)
        waited = await sleep(1)
        length -= 1
    print(label, 'lift-off!')


def main():
    """Start the event loop, counting down 3 separate launches.

    This is what a user would typically write.
    """
    loop = SleepingLoop(countdown('A', 5), countdown('B', 3, delay=2),
                        countdown('C', 4, delay=1))
    start = datetime.datetime.now()
    loop.run_until_complete()
    print('Total elapsed time is', datetime.datetime.now() - start)


if __name__ == '__main__':
    main()
```
基于Asyncio协程的高并发爬虫(3.5):
```python
import re
import asyncio
import aiohttp
import aiomysql
from pyquery import PyQuery

# https://www.lfd.uci.edu/


stop_flag = False
start_url = "http://www.jobbole.com/"
waitting_urls = []
seen_urls = set()
sem = asyncio.Semaphore(3)


async def fetch(url, session):
    """
    发送http请求
    :param url:
    :return:
    """
    async with sem:                 # 并发度控制
        await asyncio.sleep(1)      # 爬取速度控制
        try:
            async with session.get(url) as resp:
                print('url statis: {0}'.format(resp.status))
                if resp.status in [200, 201]:
                    data = await resp.text()
                    return data
        except Exception as e:
            print(e)


def extract_urls(html):
    """
    从请求页面中获取下次要请求url
    :param html:
    :return:
    """
    urls = []
    pq = PyQuery(html)
    for link in pq.items('a'):
        url = link.attr('href')
        if url and url.startswith('http') and url not in seen_urls:
            urls.append(url)
            waitting_urls.append(url)
    return urls


async def article_handler(url, session, pool):
    """
    获取文章详情并解析入库
    :param url:
    :param session:
    :return:
    """
    html = await fetch(url, session)
    seen_urls.add(url)
    extract_urls(html)
    pq = PyQuery(html)
    title = pq('title').text()  # 省略其他字段
    print(title)
    async with pool.acquire() as conn:
        async with conn.cursor() as cur:
            await cur.execute("sql")
            insert_sql = """
                INSERT INTO xxx
            """
            print(cur.description)
            await cur.execute(insert_sql)


async def init_urls(url, session):
    """
    解析页面，
    :param url:
    :param session:
    :return:
    """

    html = await fetch(url, session)
    seen_urls.add(url)
    extract_urls(html)


async def consumer(pool, session):
    # async with aiohttp.ClientSession() as session:      # 发送http请求需要的session
    while not stop_flag:
        if len(waitting_urls) == 0:
            await asyncio.sleep(0.5)
            continue
        url = waitting_urls.pop()
        print('start get url: ' + url)

        # 详情页协程，解析页面内容、入库
        if re.match('http://.*?jobbole.com/\d+/', url):
            if url not in seen_urls:
                asyncio.ensure_future(article_handler(url, session, pool))

        # 非详情页协程，进一步提取出详情页的url
        else:
            if url not in seen_urls:
                asyncio.ensure_future(init_urls(url, session))


async def main(loop):
    # 等待Mysql连接池建立
    pool = await aiomysql.create_pool(
        host='', port='', user='', password='', db='mysql', loop=loop, charset='utf8', autocommit=True
    )
    async with aiohttp.ClientSession() as session:      # 发送http请求需要的session
        html = await fetch(start_url, session)
        seen_urls.add(start_url)
        extract_urls(html)

        # consumer协程从url获取，动态向asyncio提交article_handler和init_urls协程
        asyncio.ensure_future(consumer(pool, session))


if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    asyncio.ensure_future(main(loop))
    loop.run_forever()
```