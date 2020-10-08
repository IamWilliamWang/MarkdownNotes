#Python简单多线程
##threading包提供简单多线程操作
`threading.current_thread()`: 返回当前的线程变量。
`threading.enumerate()`: 返回一个包含正在运行的线程的list。正在运行指线程启动后、结束前，不包括启动前和终止后的线程。
`threading.active_count()`: 返回正在运行的线程数量，与len(threading.enumerate())有相同的结果。
`threading.TIMEOUT_MAX`: timeout字段的上限值

###threading.Thread类的常用方法：
1.`threading.Thread.__init__(self,target=None,name=None,args=())`
target是ThreadFunction名，name是线程名，args是调用函数的参数一定要是tuple！

2.`run()`，通常需要重写，编写代码实现做需要的功能。 

3.`getName()`，获得线程对象名称。 

4.`setName()`，设置线程对象名称。 

5.`start()`，启动线程。 

6.`join([timeout])`，等待另一线程结束后再运行。 

7.`setDaemon(bool)`，设置子线程是否随主线程一起结束，必须在start()之前调用。默认为False。 

8.`isDaemon()`，判断线程是否随主线程一起结束。 

9.`is_alive()`，检查线程是否在运行中。

##threading包提供的线程同步锁
###Lock与RLock
最简单的同步机制就是锁。锁对象由`threading.RLock`类创建。线程可以使用锁的`acquire()`方法获得锁，这样锁就进入`locked`状态。每次只有一个线程可以获得锁。如果当另一个线程试图获得这个锁的时候，就会被系统变为`blocked`状态，直到那个拥有锁的线程调用锁的`release()`方法来释放锁，这样锁就会进入`unlocked`状态。`blocked`状态的线程就会收到一个通知，并有权利获得锁。

如果多个线程处于`blocked`状态，所有线程都会先解除`blocked`状态，然后系统选择一个线程来获得锁，其他的线程继续`blocked`。python的`threading module`是在建立在`thread module`基础之上的一个module，在`thread module`中，python提供了用户级的线程同步工具Lock对象。

而在`threading module`中，python又提供了`Lock`对象的变种: <u>`RLock`</u>对象。<u>`RLock`</u>对象内部维护着一个`Lock`对象，它是一种可重入的对象。对于`Lock`对象而言，如果一个线程连续两次进行`acquire`操作，那么由于第一次`acquire`之后没有`release`，第二次`acquire`将挂起线程。这会导致`Lock`对象永远不会`release`，使得线程死锁。

`RLock`对象允许一个线程多次对其进行`acquire`操作，因为在其内部通过一个counter变量维护着线程`acquire`的次数。而且每一次的`acquire`操作必须有一个`release`操作与之对应，在所有的`release`操作完成之后，别的线程才能申请该`RLock`对象。

我们把修改共享数据的代码称为临界区。必须将所有临界区都封闭在同一个锁对象的`acquire`和`release`之间。

方法展示：
```python
lock = threading.Lock() # Lock对象  
lock.acquire()  
lock.acquire()  # 在同一线程中被多次acquire，产生了死琐
lock.release()  
rLock = threading.RLock()  # RLock对象  
rLock.acquire()  
rLock.acquire() # 在同一线程内，程序不会堵塞。  
rLock.release()  
rLock.release() 
```
##_thread包提供了低级别的、原始的线程以及一个简单的锁
开启线程的方法
```python
import _thread
_thread.start_new_thread (function, args[, kwargs])
```
> 该_thread模块是Python的低级线程API。除非您确实需要，否则不建议直接使用。该threading模块是一个高级API，建立在_thread。该Thread.start方法实际上是使用thread.start_new_thread。
该daemon属性Thread必须在调用之前设置start，指定线程是否应该是一个守护进程。当没有剩下活着的非守护程序线程时，整个Python程序退出。默认情况下，daemon是False，所以线程不是守护进程，因此进程将等待其所有非守护进程线程退出，这是您正在观察的行为。

**参数说明:**
function: 线程函数。
args: 传递给线程函数的参数,他必须是个tuple类型。
kwargs: 可选参数。

代码范例
```python
import _thread
import time
 # 为线程定义一个函数
def print_time(threadName, delay):
   count = 0
   while count < 5:
      time.sleep(delay)
      count += 1
      print("%s: %s" % (threadName, time.ctime(time.time())))
 
# 创建两个线程
try:
   _thread.start_new_thread(print_time, ("Thread-1", 2, ) )
   _thread.start_new_thread(print_time, ("Thread-2", 4, ) )
except:
   print("Error: unable to start thread")
```
在线程函数中调用_thread.exit()会抛出SystemExit exception，并退出线程

#线程优先级队列（Queue）
Python的Queue模块中提供了同步的、线程安全的队列类，包括FIFO（先入先出)队列Queue，LIFO（后入先出）队列LifoQueue，和优先级队列PriorityQueue。这些队列都实现了锁原语，能够在多线程中直接使用。可以使用队列来实现线程间的同步。

Queue模块中的常用方法:
`Queue.qsize()` 返回队列的大小
`Queue.empty()` 如果队列为空，返回True,反之False
`Queue.full()` 如果队列满了，返回True,反之False
Queue.full 与 maxsize 大小对应
`Queue.get([block[, timeout]])`获取队列，timeout等待时间
`Queue.get_nowait()` 相当Queue.get(False)
`Queue.put(item)` 写入队列，timeout等待时间
`Queue.put_nowait(item)` 相当Queue.put(item, False)
`Queue.task_done()` 在完成一项工作之后，Queue.task_done()函数向任务已经完成的队列发送一个信号
`Queue.join()` 实际上意味着等到队列为空，再执行别的操作

#虚类Executor
含有的函数有：
`submit(fn, *args, **kwargs)`
`map(func, *iterables, timeout=None, chunksize=1)`
`shutdown(wait=True)`
详细用途见下文
#ThreadPoolExecutor线程池
<font size=2>注：线程池、进程池有关的[concurrent.futures](https://docs.python.org/3/library/concurrent.futures.html)包官方文档。</font>
`ThreadPoolExecutor`是Executor的子类，它[使用](https://stackoverflow.com/questions/46045956/whats-the-difference-between-threadpool-vs-pool-in-python-multiprocessing-modul)`multiprocessing.pool.ThreadPool`(官网无此类的文档)进行线程的异步操作。
`ThreadPoolExecutor`常用类函数：
1. `executor = ThreadPoolExecutor(max_workers)`构造函数
`ThreadPoolExecutor`构造实例的时候，传入`max_workers`参数来设置线程池中最多能同时运行的线程数目。
2. `future = executor.submit(fn, *args)`新线程的执行
使用`submit`函数来提交线程需要执行的任务（函数名和参数）到线程池中，并返回该任务的句柄（类似于文件、画图），注意`submit()`不是阻塞的，而是立即返回。
3. `executor.map(func, *iterables)`循环执行线程
将序列中的每个元素都执行同一个函数func，输出顺序和传入列表的顺序相同。
```python
def get(obj, obj2):
    return str(obj)+str(obj2)

executor = ThreadPoolExecutor(max_workers=2)
strs = [i for i in range(1,10)]
for data in executor.map(get, strs, strs): # 因为get要接受两个参数，所以传入两个[]。有几个参数传入几个[]，总map次数取各len([])的最小值
    print(data)
```
4. `executor.shutdown(wait=True)`关闭线程池，拒绝新线程加入
拒绝新线程加入线程池，释放线程池占用资源。如果wait为True，会一直阻塞到全部执行完毕，如果wait为False，则函数直接返回False。
如果`shutdown`后再调用`submit`或者`map`则会产生RuntimeError。
建议使用`with ThreadPoolExecutor(max_workers=4) as executor:`，已经内含了`wait`和`shutdown`操作！

`Future`常用类函数：
1. `future.done()`判断线程是否执行完毕
通过submit函数返回的任务句柄，能够使用done()方法判断该任务是否结束。
2. `future.running()`判断线程是否在运行中
3. `future.cancel()`取消线程执行
使用cancel()方法可以取消提交的任务，如果任务已经在线程池中运行了，就取消不了。
4. `future.result(timeout=None)`获得线程执行返回值
使用result()方法可以获取任务的返回值。查看内部代码，发现这个方法是**阻塞的**。超过timeout会报<u>TimeoutError</u>错。
5. `future.add_done_callback(fn)`设置回调函数
将callable的fn附加到future。fn会在future执行完毕或者取消后**立即**被调用，传入参数为future。
添加的callables按添加顺序调用，并且始终在属于添加它们的进程的线程中调用。如果callable引发Exception子类，则会记录并忽略它。如果callable引发BaseException子类，则行为未定义。
```python
executor = ThreadPoolExecutor(max_workers=2)
# 通过submit函数提交执行的函数到线程池中，submit函数立即返回，不阻塞
task1 = executor.submit(get_html, 3)
task2 = executor.submit(get_html, 2)
# done方法用于判定某个任务是否完成
print(task1.done())
# cancel方法用于取消某个任务,该任务没有放入线程池中才能取消成功
print(task2.cancel())
time.sleep(4)
print(task1.done())
# result方法可以获取task的执行结果
print(task1.result())
```
##concurrent.futures包中的独立函数(wait、as_completed)
###as_completed(fs)
`as_completed(fs, timeout=None)`方法可以一次取出所有任务的结果。
```python
for future in as_completed(all_future):
    print(future.result())
```
`as_completed(fs)`方法是一个生成器，在没有任务完成的时候，会阻塞，在有某个任务完成的时候，会`yield`这个任务，就能执行for循环下面的语句，然后继续阻塞住，循环到所有的任务结束。先完成的任务会先通知主线程。
###future.wait(fs, return_when=ALL_COMPLETED)
`future.wait(fs, timeout=None, return_when=ALL_COMPLETED)`方法可以让主线程阻塞，直到满足设定的要求。
`wait`方法接收3个参数，等待的任务序列、超时时间以及等待条件：
等待条件`return_when`默认为`ALL_COMPLETED`，表明要等待所有的任务都结束。可以看到运行结果中，确实是所有任务都完成了，主线程才打印出main。等待条件还可以设置为`FIRST_COMPLETED`，表示第一个任务完成就停止等待。
```python
from concurrent.futures import ThreadPoolExecutor, wait, ALL_COMPLETED, FIRST_COMPLETED

# 参数times用来模拟网络请求的时间
def get_html(times):
    print("get page {}s finished".format(times))
    return times

executor = ThreadPoolExecutor(max_workers=2)
urls = [3, 2, 4] # 并不是真的url
all_task = [executor.submit(get_html, url) for url in urls] # 注意submit里不要用(url)
wait(all_task, return_when=ALL_COMPLETED)
```
#ProcessPoolExecutor进程池
`ProcessPoolExecutor`是`Executor`的子类，它使用`multiprocessing.Pool`来进行线程的异步操作，是`Pool`的一个包装器
常用类函数：
`ProcessPoolExecutor(max_workers)`等等，与`ThreadPoolExecutor`完全一致

**线程池与进程池的选择**:由于GIL限制，建议：IO密集的任务，用`ThreadPoolExecutor`；CPU密集任务，用`ProcessPoolExcutor`。混合作业的任务取决于工作量，通常由于流程隔离带来的优势更偏向于`multiprocessing.Pool`
#multiprocessing模块
Python2常用模块，了解即可
##multiprocessing.pool.Pool
常用类函数
1. `multiprocessing.pool.Pool([processes[, initializer[, initargs[, maxtasksperchild[, context]]]]])`
创建进程池

2. `apply(func[, args=()[, kwds={}]])`
该函数用于传递不定参数，主进程会被阻塞直到函数执行结束

3. `apply_async(func[, args=()[, kwds={}[, callback=None]]])`
与`apply`用法一样，但它是非阻塞且支持结果返回进行回调。

4. `map(func, iterable[, chunksize=None])`
`Pool`类中的`map`方法，与内置的map函数用法行为基本一致，它会使进程阻塞直到返回结果。 
注意，虽然第二个参数是一个迭代器，但在实际使用中，必须在整个队列都就绪后，程序才会运行子进程。

5. `close()`
关闭进程池（pool），使其不在接受新的任务。

6. `terminate()`
结束工作进程，不在处理未处理的任务。

7. `join()`
主进程阻塞等待子进程的退出，join方法必须在close或terminate之后使用。

##multiprocessing.Process(group=None, target=None, name=None, args=(), kwargs={})
参数说明： 
group：进程所属组。基本不用 
target：表示调用对象。 
args：表示调用对象的位置参数元组。 
name：别名 
kwargs：表示调用对象的字典。
创建进程的简单实例(Python2)：
```python
#coding=utf-8
import multiprocessing

def do(n) :
  #获取当前线程的名字
  name = multiprocessing.current_process().name
  print name,'starting'
  print "worker ", n
  return 

if __name__ == '__main__' :
  numList = []
  for i in xrange(5) :
    p = multiprocessing.Process(target=do, args=(i,))
    numList.append(p)
    p.start()
    p.join()
    print "Process end."
```
#Semaphore与Condition
##Semaphore信号量
常用类函数：
1. `threading.Semaphore(value=1)`创建新的信号量
2. `acquire(blocking=True, timeout=None)`获取信号
参数`blocking`：
`blocking=True`时：如果调用时计数器大于零，则将其减1并立即返回。如果在调用时计数器为零，则阻塞并等待，直到其他线程调用`release()`使其大于零。这是通过适当的互锁来完成的，因此如果多个`acquire()`被阻塞，`release()`将只唤醒其中一个，这个过程会随机选择一个，因此不应该依赖阻塞线程的被唤醒顺序。返回值为True。
`blocking=False`时，不会阻塞。如果调用`acquire()`时计数器为零，则会立即返回`False`
参数`timeout`：
最多阻塞`timeout`秒。如果在该时间段内没有获取锁，则返回False，否则返回True。
3. `release()`释放信号
释放信号，使计数器递增1。当计数器为零并有另一个线程等待计数器大于零时，唤醒该线程。

tip: `threading.BoundedSemaphore`实现有界信号对象。有界信号对象确保计数器不超过初始值`value`，否则抛出`ValueError`。大多数情况下，该对象用于保护有限容量的资源。
##Condition条件变量
常用类函数：
1. `Condition([lock/rlock])`构造一个新的条件变量，使用传入的lock或rlock
2. `acquire([timeout])`线程锁
直接执行lock.acquire
3. `release()`释放锁
直接执行lock.release
4. `wait([timeout])`线程进入等待池等待通知
调用这个方法将使线程进入Condition的等待池等待通知，并释放锁。使用前线程必须已获得锁定，否则将抛出异常。 
5. `notify(n=1)`从等待池挑选n(默认1)个线程并通知
调用这个方法将从等待池挑选一个线程并通知，收到通知的线程将自动调用acquire()尝试获得锁定（进入锁定池）；其他线程仍然在等待池中。调用这个方法不会释放锁定。使用前线程必须已获得锁定，否则将抛出异常。 
6. `notifyAll()`通知等待池中所有的线程
调用这个方法将通知等待池中所有的线程，这些线程都将进入锁定池尝试获得锁定。调用这个方法不会释放锁定。使用前线程必须已获得锁定，否则将抛出异常。

例子：
```python
import threading
import time

con = threading.Condition()
num = 0

# 生产者
class Producer(threading.Thread):

    def __init__(self):
        threading.Thread.__init__(self)
    def run(self):
        # 锁定线程
        global num
        con.acquire()
        while True:
            print("开始添加产品")
            num += 5
            print("产品个数：%s" % str(num))
            if num >= 10:
                print("产品已经有10个，无法继续添加！")
                con.notify()  # 唤醒等待的线程
                con.wait()  # 等待通知
        # 释放锁
        con.release()

# 消费者
class Consumers(threading.Thread):
    def __init__(self):
        threading.Thread.__init__(self)

    def run(self):
        con.acquire()
        global num
        while True:
            print("开始消耗产品")
            num -= 1
            print("产品剩余数量：%s" %str(num))
            if num <= 0:
                print("已经没有任何产品！")
                con.notify()  # 唤醒其它线程
                con.wait()  # 等待通知
        # 释放锁
        con.release()

p = Producer()
c = Consumers()
p.start()
c.start()
```

#deque
collections.deque双向队列。不像queue.Queue一样完全线程安全，[deque只有append、appendleft、pop和popleft是线程安全的](https://stackoverflow.com/questions/8554153/is-this-deque-thread-safe-in-python)（文档中只说了append和popleft线程安全）
常用方法：
1. 创建双向队列
```python
import collections
d = collections.deque()
```
2. append(往右边添加一个元素)
```python
import collections
d = collections.deque()
d.append(1)
d.append(2)
print(d)
```
3. appendleft（往左边添加一个元素）
```python
import collections
d = collections.deque()
d.append(1)
d.appendleft(2)
print(d)
```
4. clear(清空队列)
```python
import collections
d = collections.deque()
d.append(1)
d.clear()
print(d)
```
5. copy(浅拷贝)
```python
import collections
d = collections.deque()
d.append(1)
new_d = d.copy()
print(new_d)
```
6. count(返回指定元素的出现次数)
```python
import collections
d = collections.deque()
d.append(1)
d.append(1)
print(d.count(1))
```
7. extend(从队列右边扩展一个列表的元素)
```python
import collections
d = collections.deque()
d.append(1)
d.extend([3,4,5])
print(d)
```
8. extendleft(从队列左边扩展一个列表的元素)
```python
import collections
d = collections.deque()
d.append(1)
d.extendleft([3,4,5])
print(d)
```
9. index（查找某个元素的索引位置）
```python
import collections
d = collections.deque()
d.extend(['a','b','c','d','e'])
print(d)
print(d.index('e'))
print(d.index('c',0,3))  #指定查找区间
```
10. insert（在指定位置插入元素）
```python
import collections
d = collections.deque()
d.extend(['a','b','c','d','e'])
d.insert(2,'z')
print(d)
```
11. pop（获取最右边一个元素，并在队列中删除）
```python
import collections
d = collections.deque()
d.extend(['a','b','c','d','e'])
x = d.pop()
print(x,d)
```
12. popleft（获取最左边一个元素，并在队列中删除）
```python
import collections
d = collections.deque()
d.extend(['a','b','c','d','e'])
x = d.popleft()
print(x,d)
```
13. remove（删除指定元素）
```python
import collections
d = collections.deque()
d.extend(['a','b','c','d','e'])
d.remove('c')
print(d)
```
14. reverse（队列反转）
```python
import collections
d = collections.deque()
d.extend(['a','b','c','d','e'])
d.reverse()
print(d)
```
15. rotate（把右边元素放到左边）
```python
import collections
d = collections.deque()
d.extend(['a','b','c','d','e'])
d.rotate(2)   #指定次数，默认1次
print(d)
```
16. maxlen
类内的唯一参数：限制队列的最大长度

#未分类：
1. python两个字典合并：
```python
new_dict = dict1.copy()
new_dict.update(dict2)
```
2. `dis.dis(function)`：反汇编function函数
3. `f'{name}'`：字符串format的便捷用法
```python
import time
t0 = time.time()
time.sleep(1)
name = 'processing'

# 以 f开头表示在字符串内支持大括号内的python 表达式
print(f'{name} done in {time.time() - t0:.2f} s') 
```
4. python format：
`"{} {}".format("hello", "world")    # 不设置指定位置，按默认顺序({0}{1})`
输出：hello world
`"{1} {0} {1}".format("hello", "world")  # 设置指定位置`
`"网站名：{name}, 地址 {url}".format(name="菜鸟教程", url="www.runoob.com")`
高级用法：
<table class="reference">
<tbody><tr><th width="10%">数字</th><th width="30%">格式</th><th width="30%">输出
</th><th width="30%">描述</th></tr>
<tr><td> 3.1415926 </td>
    <td> {:.2f} </td>
    <td> 3.14 </td>
    <td> 保留小数点后两位 </td>
</tr>
<tr><td> 3.1415926 </td>
    <td> {:+.2f} </td>
    <td> +3.14 </td>
    <td> 带符号保留小数点后两位 </td>
</tr>
<tr><td> -1 </td>
    <td> {:+.2f} </td>
    <td> -1.00 </td>
    <td> 带符号保留小数点后两位 </td>
</tr>
<tr><td> 2.71828 </td>
    <td> {:.0f} </td>
    <td> 3 </td>
    <td> 不带小数 </td>
</tr>
<tr><td> 5 </td>
    <td> {:0&gt;2d} </td>
    <td> 05 </td>
    <td> 数字补零 (填充左边, 宽度为2) </td>
</tr>
<tr><td> 5 </td>
    <td> {:x&lt;4d} </td>
    <td> 5xxx </td>
    <td> 数字补x (填充右边, 宽度为4) </td>
</tr>
<tr><td> 10 </td>
    <td> {:x&lt;4d} </td>
    <td> 10xx </td>
    <td> 数字补x (填充右边, 宽度为4) </td>
</tr>
<tr><td> 1000000 </td>
    <td> {:,} </td>
    <td> 1,000,000 </td>
    <td> 以逗号分隔的数字格式 </td>
</tr>
<tr><td> 0.25 </td>
    <td> {:.2%} </td>
    <td> 25.00% </td>
    <td> 百分比格式 </td>
</tr>
<tr><td> 1000000000 </td>
    <td> {:.2e} </td>
    <td> 1.00e+09 </td>
    <td> 指数记法 </td>
</tr>
<tr><td> 13 </td>
    <td> {:10d} </td>
    <td>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;13</td>
    <td> 右对齐 (默认, 宽度为10) </td>
</tr>
<tr><td> 13 </td>
    <td> {:&lt;10d} </td>
    <td> 13 </td>
    <td> 左对齐 (宽度为10)</td>
</tr>
<tr><td> 13 </td>
    <td> {:^10d} </td>
    <td> &nbsp;&nbsp;&nbsp;&nbsp;13 </td>
    <td> 中间对齐 (宽度为10) </td>
</tr>
<tr><td> 11 </td>
    <td><pre>'{:b}'.format(11)
'{:d}'.format(11)
'{:o}'.format(11)
'{:x}'.format(11)
'{:#x}'.format(11)
'{:#X}'.format(11)</pre></td>
    <td><pre>1011
11
13
b
0xb
0XB
</pre></td>
    <td> 进制</td>
</tr>
</tbody></table>

`^`, `<`, `>` 分别是居中、左对齐、右对齐，后面带宽度， `:` 号后面带填充的字符，只能是一个字符，不指定则默认是用空格填充。

`+` 表示在正数前显示 `+`，负数前显示 `-`；` `（空格）表示在正数前加空格

b、d、o、x 分别是二进制、十进制、八进制、十六进制。

此外我们可以使用大括号 `{}` 来转义大括号