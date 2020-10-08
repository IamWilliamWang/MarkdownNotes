#Python生成器
简单实例代码
```python
def println(n):
    for i in range(1, n + 1):
        sendKey = yield i
        print('生成数字为 ' + str(i))
        if sendKey is not None:
            print('获得主函数发送的 '+str(sendKey))


generator = println(5)
print(next(generator))  # 等价于print(generator.__next__())
generator.send('Sended string.')
for element in generator:
    print('获得生成器元素 ' + str(element))

# 生成数字为 1
# 获得主函数发送的 Sended string.
# 生成数字为 2
# 获得生成器元素 3
# 生成数字为 3
# 获得生成器元素 4
# 生成数字为 4
# 获得生成器元素 5
# 生成数字为 5
```
解释：构建函数生成generator，这时函数没有执行。
然后调用`next(generator)`开始执行函数，由于有`yield`关键字，执行到`yield i`会暂停执行函数并返回当前值i=1，所有第一行输出是1。
调用`.send`意思是向`yield`位置发送value，并赋值给sendKey。然后输出“生成数字为 1”和“获得主函数发送的 Sended string.”，再执行到`yield i`继续暂停。
接着将生成器进行迭代，也就是每次自动执行`next()`并赋值给element。反复执行到最后，由于这期间没有`.send()`所以在迭代期间sendKey恒等于None
#协程库asyncio
由最初的`yield`和`send`升级为`@asyncio.coroutine`和`yield from`。
简单实例代码
```python
import asyncio
 
@asyncio.coroutine
def say_hi(n):
    print("start:", n)
    r = yield from asyncio.sleep(2)
    print("end:", n)
 
loop = asyncio.get_event_loop()
tasks = [say_hi(0), say_hi(1)]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
 
# start: 1
# start: 0
# 停顿两秒
# end: 1
# end: 0
```
1. `@asyncio.coroutine`把一个generator标记为coroutine类型，然后，我们就把这个`coroutine`扔到`EventLoop`中执行。
2. `yield from`语法可以让我们方便地调用另一个generator。由于`asyncio.sleep()`也是一个coroutine，所以线程不会等待`asyncio.sleep()`，而是直接中断并执行下一个消息循环。当`asyncio.sleep()`返回时，线程就可以从`yield from`拿到返回值（此处是None），然后接着执行下一行语句。
3. `asyncio.sleep(1)`相当于一个耗时1秒的IO操作，在此期间，主线程并未等待，而是去执行`EventLoop`中其他可以执行的coroutine了，因此可以实现并发执行。

`asyncio`中`get_event_loop()`就是事件循环，而装饰器`@asyncio.coroutine`标记了一个协程，并`yield from`语法实现协程切换。在**Python3.5**中，**新增了async和await**的新语法，代替装饰器和`yield from`。上例可以用新增语法完全代替。如下
```python
# 新语法将@asyncio.coroutine换成async,将yield from换成await
async def say_hi(n):
    print("start:", n)
    r = await asyncio.sleep(2)
    print("end:", n)
```
**协程的缺点**：
（1）使用协程，只能使用单线程，多线程的便利就一点都用不到。例如，I/O阻塞程序，CPU仍然会将整个任务挂起直到操作完成。
（2） 一旦使用协程，大部分python库并不能很好的兼容，这就会导致要改写大量的标准库函数。
