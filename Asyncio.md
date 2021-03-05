# 参考文档：https://www.jianshu.com/p/b5e347b3a17c  https://www.lylinux.net/article/2019/6/9/57.html

# asyncio的基本概念
    
    在 Python 3.4 时候引进了协程的概念，它使用一种单线程单进程的的方式实现并发
    
    异步最适合的就是那种等待io，不需要消耗cpu的操作.比如爬虫，或者第三方接口的调用
    
# Coroutine 协程：
    
    协程函数: 定义形式为 async def 的函数
    
    协程对象：指一个使用async关键字定义的函数，它的调用不会立即执行函数，而是会返回一个协程对象。
              协程对象需要注册到事件循环，由事件循环调用。
              
    协程的本质还是一个生成器
    
## 定义一个协程函数
    
    async def func():
        pass
        
## 返回一个协程对象

    调用协程函数
    
    result = func()
    
    print(result)           # <coroutine object func at 0x0000020AECD8C2B0>
                  
# Eventloop 事件循环：
    
        程序开启一个无限的循环，把一些函数（协程）注册到事件循环上。当满足事件发生的时候，调用相应的协程函数。
        
        Eventloop实例提供了注册、取消、执行任务和回调 的方法。 简单来说，就是我们可以把一些异步函数注册到这个事件循环上，
        
        事件循环回循环执行这些函数（每次只能执行一个），如果当前正在执行的函数在等待I/O返回，那么事件循环就会暂停它的执行去执行其他函数。
        当某个函数完成I/O后会恢复，等到下次循环到它的时候就会继续执行。
        
## while 循环

    事件循环，可以把他当做是一个while循环，这个while循环在周期性的运行并执行一些任务，在特定条件下终止循环。
    
    任务列表 = [ 任务1, 任务2, 任务3,... ]
    
    while True:
        可执行的任务列表，已完成的任务列表 = 去任务列表中检查所有的任务，将'可执行'和'已完成'的任务返回
        
        for 就绪任务 in 已准备就绪的任务列表:
            执行已就绪的任务
        
        for 已完成的任务 in 已完成的任务列表:
            在任务列表中移除 已完成的任务
        
        如果 任务列表 中的任务都已完成，则终止循环
        
## 获取和创建事件循环

### get_event_loop
    
    获取当前事件循环。 如果当前 OS 线程没有设置当前事件循环并且 set_event_loop() 还没有被调用，
    asyncio 将创建一个新的事件循环并将其设置为当前循环。并且在主线程中多次调用始终返回该 event loop；而在其他线程中调用 get_event_loop() 则会报错
    
    loop = asyncio.get_event_loop()
    
### get_running_loop

    get_running_loop() 是python3.7之后新增的函数，用于获取当前正在运行的loop
    
    如果当前主线程中没有正在运行的loop，如果没有就会报RuntimeError 错误。
    
    get_running_loop 只能在协程里面用，用来捕获当前的loop
    
    因此 在协程和回调中应尽量使用 get_running_loop() 函数而非 get_event_loop()
    
    
### 使用示例

    import asyncio
    
    
    async def main():
    
        await asyncio.sleep(5)
    
        print("------------:Done")
    
        myloop = asyncio.get_running_loop()
    
        print('current loop id is {}'.format(id(myloop)))
    
    
    # loop = asyncio.get_running_loop()      报错,需定义在协程里面
    loop = asyncio.get_event_loop()
    
    print('current loop id is {}'.format(id(loop)))
    
    try:
        loop.run_until_complete(main())
    except KeyboardInterrupt:
        print('key board inpterrupt!')
    finally:
        try:
            # 清理任何没有完全消耗的异步生成器。
            loop.run_until_complete(loop.shutdown_asyncgens())
        finally:
            loop.close()
    
    current loop id is 140412318011008
    
    ------------:Done
    
    current loop id is 140412318011008
    
    两个loop的id是一样的，是同一个对象
    
### new_event_loop()和set_event_loop(loop)

    创建一个新的事件循环, 通常在子线程中创建
    
    将 loop 设置为当前 OS 线程的当前事件循环。
    
    
    同一个线程里，两个 event loop 无法同时 run。 对于多线程程序：
    
        1. 主线程用 loop=get_event_loop().      （不允许通过 asyncio.get_event_loop 在主线程之外创建循环）
        
        2. 子线程 首先loop=new_event_loop(),然后 set_event_loop(loop)
        
            new_event_loop()是创建一个event loop对象，而set_event_loop(eventloop对象)是将event loop对象指定为当前协程的event loop，
            一个协程内只允许运行一个event loop，不要一个协程有两个event loop交替运行。
    
### 使用示例

    import asyncio
    import threading
    from functools import partial
    
    
    def _worker(worker, *args, **kwargs):
    
        # 不允许通过 asyncio.get_event_loop 在主线程之外创建循环
        # 因此，我们必须通过 asyncio.set_event_loop（asyncio.new_event_loop（））创建一个线程本地事件循环。
        loop = asyncio.new_event_loop()
        asyncio.set_event_loop(loop)
        try:
            loop.run_until_complete(worker(*args, **kwargs))
        finally:
            loop.close()
    
    
    def create_event_loop_thread(worker, *args, **kwargs):
        return threading.Thread(target=partial(_worker, worker), args=args, kwargs=kwargs)
    
    
    async def print_coro(*args, **kwargs):
    
        loop_id = id(asyncio.get_running_loop())
        print(f"Inside the print coro on {threading.get_ident()} loop id is {loop_id}:", (args, kwargs))
    
    
    def start_threads(*threads):
        [t.start() for t in threads if isinstance(t, threading.Thread)]
    
    
    def join_threads(*threads):
        [t.join() for t in threads if isinstance(t, threading.Thread)]
    
    
    def main():
        workers = [create_event_loop_thread(print_coro, i) for i in range(3)]
        start_threads(*workers)
        join_threads(*workers)
    
    
    if __name__ == '__main__':
        main()

    
    
    
## 运行和停止事件循环

    loop.run_until_complete(future)      运行直到 future ( Future 的实例 ) 被完成 既可以接收 Future 对象，也可以是 coroutine 对象，
    
    loop.run_forever()                   运行事件循环直到 stop() 被调用
    
    loop.stop()                          停止事件循环
    
    loop.is_running()                    返回 True 如果事件循环当前正在运行
    
    loop.is_closed()                     如果事件循环已经被关闭，返回 True
    
    loop.close()                         关闭事件循环
    
    
### 示例

    def async_task_start(func_obj):
    
        loop = asyncio.get_event_loop()
    
        async_task = asyncio.ensure_future(func_obj)
    
        try:
            loop.run_until_complete(async_task)
    
            if async_task.done():
                print(async_task.result())
        except:
            loop.stop()
            raise
        finally:
    
            try:
                loop.run_until_complete(loop.shutdown_asyncgens())
            finally:
                loop.close()
# Task 对象

    Task对象被用来在事件循环中运行协程
    
    一个协程对象就是一个原生可以挂起的函数，任务则是对协程进一步封装，其中包含任务的各种状态。Task 对象是 Future 的子类
    
     一个事件循环每次运行一个Task对象。而一个Task对象会等待一个Future对象完成，该事件循环会运行其他Task、回调或执行IO操作。
     
## asyncio.ensure_future
    
    创建Task任务, 参数除了是协程对象, 还可以是Future对象或者awaitable对象：
    
        1. 如果参数是协程，其底层使用loop.create_task，返回Task对象
        2. 如果是Future对象会直接返回
        3. 如果是一个awaitable对象，会await这个对象的__await__方法，再执行一次ensure_future，最后返回Task或者Future
        
    ensure_future方法主要就是确保这是一个Future对象 
    
##  loop.create_task

    Python3.7 开始加入, 接收的参数需要是一个协程对象

##  asyncio.create_task
      Python3.7 开始加入, 接收的参数需要是一个协程对象
      
      
## 示例：
    async def func(n):
    
        print(f"func1-------------{n}")
    
        await asyncio.sleep(0.8)
    
        print(f"func2-------------{n}")
    
        return
    
    if __name__ == '__main__':
    
        loop = asyncio.get_event_loop()
    
        # 创建协程，将协程封装到一个Task对象中并立即添加到事件循环的任务列表中，等待事件循环去执行（默认是就绪状态）
    
        # 1.
        async_task = asyncio.ensure_future(func(1))
        
        # 2.
        # async_task = loop.create_task(func(1))
        
        # 3. Python3.7 以后
        # async_task = asyncio.create_task(func(1))
    
        try:
            loop.run_until_complete(async_task)
    
            if async_task.done():
                print(async_task.result())
        except:
            loop.stop()
            raise
        finally:
    
            try:
                loop.run_until_complete(loop.shutdown_asyncgens())
            finally:
                loop.close()
    
# Future 对象：
        
        Future是对协程的封装, Future是表示一个"未来"对象。
        Future对象提供了很多任务方法（如完成后的回调，取消，设置任务结果等等），但是一般情况下开发者不需要操作Future这种底层对象，
        而是直接用Future的子类Task协同的调度协程来实现并发
        
        Future为我们提供了异步编程中的 最终结果 的处理（Task类也具备状态处理的功能）
        
        Future对象的常用方法：
        
            result()                           返回Future执行的结果返回值
             
            done()                             如果Future1执行完毕，则返回 True 
            
            add_done_callback(callback, *, context=None)  在Future完成之后，给它添加一个回调方法
            
            cancelled()                        判断任务是否取消
            
            cancel()                           取消任务
            
            set_result(result)                 标记Future已经执行完毕，并且设置它的返回值
            
            set_exception(exception)           标记Future已经执行完毕，并且触发一个异常
            
            get_loop()                         返回Future所绑定的事件循环
            
 
# 协程工作流程

## 协程的4种状态
    
    Pending
    
    Running
    
    Done
    
    Cacelled
    
    创建future的时候，task为pending，事件循环调用执行的时候当然就是running，调用完毕自然就是done，
    如果需要停止事件循环，中途需要取消，就需要先把task取消，即为cancelled
    
    
## 创建/执行流程

    1. 定义/创建协程对象
    
    2. 定义事件循环对象容器
    
    3. 将协程转为Task任务
        
    4. 将Task任务扔进事件循环对象中并触发
    
## 协程示例

        async def demo_exp(data):

            print("The demo_exp data is ", data)

            return "OK"

        def main():

            async_coroutine = demo_exp("Hello World")

            loop = asyncio.get_event_loop()

            """
            创建Task的三种方法：
                    1. async_task = asyncio.ensure_future(async_coroutine)  ensure_future3.7之前
                    2. async_task = asyncio.create_task(async_coroutine)  ensure_future3.7之后
                    3. async_task = loop.create_task(async_coroutine)  ensure_future3.7之前

            """

            async_task = asyncio.ensure_future(async_coroutine)

            loop.run_until_complete(async_task)

            # 查看Task对象是否调用完毕
            
            print(async_task.done())
            
            # 查看协程的返回值。未调用完毕查看报错
            if async_task.done():
                print(async_task.result())

        if __name__ == "__main__":
            main()
       
## 绑定回调函数(add_done_callback)
    
    import asyncio
    import functools
    
    
    async def demo_exp(data):
    
            await asyncio.sleep(1)                  # 模拟io
    
            print("The demo_exp data is ", data)
    
            return "OK"
    
    
    def exp_callback_1(future):
    
            data = future.result()
            
            res = "callback_1: %s" % data
    
            print(res)
    
            return True


    def exp_callback_2(future, info):
    
            data = future.result()
            res = "callback_1: %s_%s" % (data, info)
    
            print(res)
    
            return True


    def main():
    
            async_coroutine = demo_exp("Hello World")
    
            loop = asyncio.get_event_loop()
    
            async_task = asyncio.ensure_future(async_coroutine)
    
            # 1. callback无参数
            async_task.add_done_callback(exp_callback_1)
    
            # 2. callback无参数
            async_task.add_done_callback(functools.partial(exp_callback_2, info="Python"))
    
            loop.run_until_complete(async_task)


    if __name__ == "__main__":
            main()
            
## 协程并发的实现
    async def demo_exp(x):
    
            print("Waiting: ", x)
    
            await asyncio.sleep(x)                  # 模拟io
    
            return "暂停了{time}秒".format(time=x)
                
###  asyncio.gather

    asyncio.gather能收集协程的结果，而且会按照输入协程的顺序保存对应协程的执行结果
    返回的是所有已完成 Task 的 result
    
    async def demo_exp(data):
        await asyncio.sleep(data)  # 模拟io
    
        print("The demo_exp data is ", data)
    
        return "OK"


    def main():
        start_time = time.time()
    
        # 创建多个协程
        async_coroutine_1 = demo_exp(1)
        async_coroutine_2 = demo_exp(2)
        async_coroutine_3 = demo_exp(4)
    
        loop = asyncio.get_event_loop()
    
        # 创建Task对象
        async_task = [
            asyncio.ensure_future(async_coroutine_1),
            asyncio.ensure_future(async_coroutine_2),
            asyncio.ensure_future(async_coroutine_3),
    
        ]
        # Task 注册到loop中
        """
        loop中注册多个Task
                1. asyncio.wait
                        loop.run_until_complete(asyncio.wait(async_task))
                2.async_task
                        loop.run_until_complete(asyncio.gather(*async_task))
    
        """
    
        response = loop.run_until_complete(asyncio.gather(*async_task))
    
        for v in response:
            print('task result is:', v)
    
        print('TIME: ', time.time() - start_time)
    
    
    if __name__ == "__main__":
        main()
        """
        The demo_exp data is  1
        The demo_exp data is  2
        The demo_exp data is  4
        task result is: OK
        task result is: OK
        task result is: OK
        TIME:  4.003488063812256
        """

###  asyncio.wait

    asyncio.wait 会返回两个值：done 和 pending，done 为已完成的协程 Task，pending 为超时未完成的协程 Task 
    需通过 future.result 调用 Task 的 result  
    
    done, pending = loop.run_until_complete(asyncio.wait(async_task))  
    
### wait 示例
    
    async def demo_exp(data):
        
        await asyncio.sleep(data)  # 模拟io
    
        print("The demo_exp data is ", data)
    
        return "OK"
    
    
    def main():
        start_time = time.time()
    
        # 创建多个协程
        async_coroutine_1 = demo_exp(1)
        async_coroutine_2 = demo_exp(2)
        async_coroutine_3 = demo_exp(4)
    
        loop = asyncio.get_event_loop()
    
        # 创建Task对象
        async_task = [
            asyncio.ensure_future(async_coroutine_1),
            asyncio.ensure_future(async_coroutine_2),
            asyncio.ensure_future(async_coroutine_3),
    
        ]
        # Task 注册到loop中
    
        done, pending = loop.run_until_complete(asyncio.wait(async_task))
    
        for v in done:
            
            # 需要调用future的result
    
            print('task result is:', v.result())
    
        print('TIME: ', time.time() - start_time)
    
    
    if __name__ == "__main__":
        main()
        """
        The demo_exp data is  1
        The demo_exp data is  2
        The demo_exp data is  4
        task result is: OK
        task result is: OK
        task result is: OK
        TIME:  4.003674745559692
        """
#  await

    将协程的控制权让出，以便loop调用其他的协程。
    针对耗时的操作进行挂起，协程遇到await，事件循环将会挂起该协程，执行别的协程，直到其他的协程也挂起或者执行完毕，再进行下一个协程的执行。
    耗时的操作一般是一些IO操作，例如网络请求，文件读取等。我们使用asyncio.sleep函数来模拟IO操作。协程的目的也是让这些IO操作异步化
    
##  await与yield对比:
    
    1. await用于挂起阻塞的异步调用接口。其作用在一定程度上类似于yield。（都能实现暂停的效果），但是功能上却不兼容。
           就是不能在生成器中使用await，也不能在async 定义的协程中使用yield。

    2.yield from 后面可接 可迭代对象，也可接future对象/协程对象；
      await 后面必须要接 future对象/协程对象
      
### 示例：

        # await 接协程
        async def hello_1(name):
            await  asyncio.sleep(2)
            print("Hello,",name)

        # await 接Future对象
        async def hello_2(name):
            await asyncio.ensure_future(asyncio.sleep(2))

        # yield from 接协程
        def hello_3(name):
            yield from asyncio.sleep(2)

        # yield from  接Future对象

        def hello_4(name):
            yield from asyncio.ensure_future(asyncio.sleep(2))
            
## asyncio.sleep(n)

    asyncio自带的工具函数，他可以模拟IO阻塞，他返回的是一个协程对象
   
    import asyncio
    
    from collections import Generator
    from asyncio.futures import Future
    from asyncio.tasks import Task
    
    
    def main():
            func = asyncio.sleep(2)
    
            print(isinstance(func, Future))         #   False
            print(isinstance(func, Generator))      #   True
            
##  await 协程对象

    async def func(n):
        print(f"func1-------------{n}")
    
        await asyncio.sleep(0.8)
    
        print(f"func2-------------{n}")
    
        return True
    
    
    async def main():
    
        # 1. await 协程对象
        # res = await func(2)
    
        # 2. await task对象
        # task = asyncio.ensure_future(func(2))
        #
        # res = await task
    
        # 3. await task 列表
    
        task_list = [
            asyncio.ensure_future(func(3)),
            asyncio.ensure_future(func(4))
        ]
    
        res = await asyncio.gather(*task_list)
    
        print(res)
    
    
    if __name__ == '__main__':
    
        loop = asyncio.get_event_loop()
        loop.run_until_complete(main())

# 协程的高阶应用

## 1. 协程的嵌套
    import asyncio
    
    
    async def demo_exp(x):
            print("Waiting: ", x)
    
            await asyncio.sleep(x)  # 模拟io
    
            return "暂停了{time}秒".format(time=x)
    
    
    async def main():
    
            # 创建多个协程
            async_coroutine_1 = demo_exp(1)
            async_coroutine_2 = demo_exp(5)
            async_coroutine_3 = demo_exp(8)
    
            # 创建Task对象
            async_task = [
                    asyncio.ensure_future(async_coroutine_1),
                    asyncio.ensure_future(async_coroutine_2),
                    asyncio.ensure_future(async_coroutine_3),
    
            ]
    
            """
                1.
                    results = await asyncio.gather(*async_task)
        
                    for result in results:
                        print('task result is: ', result)
                
                2. return await asyncio.gather(*async_task)
                
                3.  return await asyncio.wait(tasks)
                
            """
    
            dones, pendings = await asyncio.wait(async_task)
    
            for v in async_task:
                    
                    if v.done():
                            print('task result is:', v.result())
                    else:
                            print('task  is running:')
    
    
    if __name__ == "__main__":
            loop = asyncio.get_event_loop()
            loop.run_until_complete(main())
            
## 2. 不同线程的事件循环(同步)
    import asyncio
    import time
    from queue import Queue
    from threading import Thread
    
    
    def start_loop(loop):
            # 一个在后台永远运行的事件循环
            asyncio.set_event_loop(loop)   # 将 loop 设置为当前 OS 线程的当前事件循环。
            loop.run_forever()
    
    
    def do_sleep(x, queue, msg= ""):
            time.sleep(x)
            queue.put(msg)
    
    
    def main():
            """
            主线程同步执行
            # 由于是同步的，所以总共耗时6 + 3 = 9秒.
            :return:
            """
    
            queue = Queue()
    
            new_loop = asyncio.new_event_loop()
    
            t = Thread(target=start_loop, args=(new_loop,))
            t.start()
    
            print(time.ctime())
    
            # 动态添加两个协程，在主线程是同步的
            new_loop.call_soon_threadsafe(do_sleep, 6, queue, '第一个')
            new_loop.call_soon_threadsafe(do_sleep, 3, queue, "第二个")
    
            while True:
                    msg = queue.get()
                    print("{} 协程运行完..".format(msg))
                    print(time.ctime())
                    
    if __name__ == "__main__":
            main()
        
## 3. 不同线程的事件循环(异步)
    import asyncio
    import time
    from queue import Queue
    from threading import Thread
    
    
    def start_loop(loop):
            # 一个在后台永远运行的事件循环
            asyncio.set_event_loop(loop)   # 将 loop 设置为当前 OS 线程的当前事件循环。
            loop.run_forever()
    
    async def do_sleep_demo(x, queue, msg=""):
            await asyncio.sleep(x)
            queue.put(msg)
    
    
    def main_demo():
            """主线程异步执行
              总共耗时max(6, 3)=6秒
            """
    
            queue = Queue()
    
            new_loop = asyncio.new_event_loop()
    
            t = Thread(target=start_loop, args=(new_loop,))
            t.start()
    
            print(time.ctime())
    
            asyncio.run_coroutine_threadsafe(do_sleep_demo(6, queue, '第一个'), new_loop)
            asyncio.run_coroutine_threadsafe(do_sleep_demo(3, queue, "第二个"), new_loop)
    
            while True:
                    msg = queue.get()
                    print("{} 协程运行完..".format(msg))
                    print(time.ctime()
    
    if __name__ == "__main__":
            main_demo()
            
## 4. get_event_loop和 new_event_loop
    一个线程内只允许运行一个eventloop，意味着不能有两个eventloop交替运行。
        get_event_loop()：用于主线程
        new_event_loop()：用于给非主线程创建eventloop。
                          然后set_event_loop(loop)
        new_event_loop()是创建一个eventloop对象，而set_event_loop(eventloop对象)是将eventloop对象指定为当前线程的eventloop

### 实例：
    import asyncio
    import time
    from queue import Queue
    from threading import Thread
    
    
    async def main_work(main_loop):
            while True:
                    print('main on loop:%s' % id(main_loop))
                    await asyncio.sleep(4)
    
    
    async def work_2(loop):
            while True:
                    print('work_2 on loop:%s' % id(loop))
                    await asyncio.sleep(2)
    
    
    async def work_4(loop):
            while True:
                    print('work_4 on loop:%s' % id(loop))
                    await asyncio.sleep(4)
    
    
    def thread_loop_task(loop):
            # 为子线程设置自己的事件循环
    
            asyncio.set_event_loop(loop)
    
            future = asyncio.gather(work_2(loop), work_4(loop))
    
            loop.run_until_complete(future)
    
    
    def main():
            """
            work_4 on loop:1834188545664
            work_2 on loop:1834188545664
            main on loop:1834195353496
            :return:
            """
            # 创建一个事件循环thread_loop
            thread_loop = asyncio.new_event_loop()
            # 将thread_loop作为参数传递给子线程
            t = Thread(target=thread_loop_task, args=(thread_loop,))
            t.daemon = True
            t.start()
    
            main_loop = asyncio.get_event_loop()
    
            main_loop.run_until_complete(main_work(main_loop ))
    
    
    if __name__ == "__main__":
            main()
            
# 协程的停止
## 协程的状态
        
        Pending：创建future，还未执行
        Running：事件循环正在调用执行任务
        Done：任务执行完毕
        Cancelled：Task被取消后的状态
        
## 1. 
    import asyncio
    
    
    async def demo_exp(x):
            print("Waiting: ", x)
    
            await asyncio.sleep(x)  # 模拟io
    
            return "暂停了{time}秒".format(time=x)
    
    
    def main_1():
            # 创建多个协程
            async_coroutine_1 = demo_exp(1)
            async_coroutine_2 = demo_exp(5)
            async_coroutine_3 = demo_exp(8)
    
            # 创建Task对象
            async_task = [
                    asyncio.ensure_future(async_coroutine_1),
                    asyncio.ensure_future(async_coroutine_2),
                    asyncio.ensure_future(async_coroutine_3),
    
            ]
    
            loop = asyncio.get_event_loop()
    
            try:
                    loop.run_until_complete(asyncio.wait(async_task))
            except KeyboardInterrupt as e:
                    print(asyncio.Task.all_tasks())
                    for task in asyncio.Task.all_tasks():
                            print(task.cancel())
                    loop.stop()
                    loop.run_forever()
            finally:
                    loop.close()
    
            """
            启动事件循环之后，马上ctrl+c，会触发run_until_complete的执行异常 KeyBorardInterrupt。然后通过循环asyncio.Task取消future
            True表示cannel成功，loop stop之后还需要再次开启事件循环，最后在close，不然还会抛出异常
            {
                     <Task pending coro=<wait() running at d:\python\package\Lib\asyncio\tasks.py:313> wait_for=<Future pending cb=[
                    <TaskWakeupMethWrapper object at 0x0000019C47C573D8>()]>>,
                    <Task finished coro=<demo_exp() done,
                    defined at demo.py:44> result='暂停了1秒'>,
                    <Task pending coro=<demo_exp() running at demo.py:47> wait_for=<Future pending cb=[
                        <TaskWakeupMethWrapper object at 0x0000019C47C57288>()]> cb=[
                            _wait.<locals>._on_
                            completion() at d:\python\package\Lib\asyncio\tasks.py:380]>,
                    <Task pending coro=<demo_exp() running at demo.py:47> wait_for=<Future pending cb=[
                        <TaskWakeupMethWrapper object at 0x0000019C47C57228>()]> cb=[
                            _wait.<locals>._on_completion() at d:\python\package\Lib\asyncio\tasks.py:380]>
                    }
            
            False
            True
            True
            
            暂停1秒的任务已经完成,暂停5秒，8秒的任务在running
    
            """
    
    
    
    
    
    if __name__ == "__main__":
       
            main_1()

## 2. 停止嵌套协程

    import asyncio
    
    
    async def demo_exp(x):
            print("Waiting: ", x)
    
            await asyncio.sleep(x)  # 模拟io
    
            return "暂停了{time}秒".format(time=x)
    
    async def main():
    
            # 创建多个协程
            async_coroutine_1 = demo_exp(1)
            async_coroutine_2 = demo_exp(5)
            async_coroutine_3 = demo_exp(8)
    
            # 创建Task对象
            async_task = [
                    asyncio.ensure_future(async_coroutine_1),
                    asyncio.ensure_future(async_coroutine_2),
                    asyncio.ensure_future(async_coroutine_3),
    
            ]
    
            dones, pendings = await asyncio.wait(async_task)
    
            for task in dones:
                    print('Task ret: ', task.result())
    
    
    if __name__ == "__main__":
            loop = asyncio.get_event_loop()
            try:
                    loop.run_until_complete(main())
            except KeyboardInterrupt as e:
                    print(asyncio.Task.all_tasks())
                    print(asyncio.gather(*asyncio.Task.all_tasks()).cancel())
                    loop.stop()
                    loop.run_forever()
            finally:
                    loop.close()

# 生产者消费者模型：
    import time
    import redis
    import asyncio
    from queue import Queue
    from threading import Thread
    
    
    def start_loop(loop):
        # 一个在后台永远运行的事件循环
        asyncio.set_event_loop(loop)
        loop.run_forever()
    
    
    async def do_sleep(x, queue):
        print("=================")
        await asyncio.sleep(x)
        queue.put("ok")
    
    
    def get_redis():
        # 未添加decode_responses，从redis中取出的数据为bytes类型
        connection_pool = redis.ConnectionPool(host='10.110.1.111', db=0, decode_responses=True)
        return redis.Redis(connection_pool=connection_pool)
    
    
    def consumer(rcon, queue, new_loop):
        while True:
            task = rcon.rpop("queue")
            if not task:
                print("+++++++++++++++")
                time.sleep(1)
                continue
            print("---------:", int(task))
            asyncio.run_coroutine_threadsafe(do_sleep(int(task), queue), new_loop)
         
            
    def main():
        """
        在Redis，分别发起了5s，3s，1s的任务。这三个任务，确实是并发执行的，1s的任务最先结束，三个任务完成总耗时5s
        :return: 
        """
        
        new_loop = asyncio.new_event_loop()
    
        # 定义一个线程，运行一个事件循环对象，用于实时接收新任务
        loop_thread = Thread(target=start_loop, args=(new_loop,))
        loop_thread.setDaemon(True)
        loop_thread.start()
        # 创建redis连接
        rcon = get_redis()
    
        queue = Queue()
    
        # 子线程：用于消费队列消息，并实时往事件对象容器中添加新任务
        consumer_thread = Thread(target=consumer, args=( rcon, queue, new_loop))
        consumer_thread.setDaemon(True)
        consumer_thread.start()
    
        while True:
            msg = queue.get()
            print("协程运行完..", msg)
            print("当前时间：", time.ctime())
        
    
    if __name__ == '__main__':
        main()

# uvloop

    uvloop 使得 asyncio 更快. 实际上，比nodejs，gevent，以及其他任何Python异步框架至少快两倍 ．
    uvloop asyncio 基于性能的测试接近于Go程序


## 代码实现

    import asyncio
    import uvloop
    asyncio.set_event_loop_policy(uvloop.EventLoopPolicy())
    
    在线程或者进程池中执行代码
    
    import concurrent.futures
    import asyncio
    import requests
    import BeautifulSoup
    
    
    class Spider(object):
        def __init__(self):
            self._urls = [
                "http://www.fover.cn/da/"
                "http://www.fover.cn/sha/"
                "http://www.fover.cn/bi/"
            ]
            self._loop = asyncio.get_event_loop()
            self._thread_pool = concurrent.futures.ThreadPoolExecutor(
                max_workers=10)
            self._process_pool = concurrent.futures.ProcessPoolExecutor()

    def crawl(self):
        a = []
        for url in self._urls:
            a.append(self._little_spider(url))
        self._loop.run_until_complete(asyncio.gather(*a))
        self._loop.close()
        self._thread_pool.shutdown()
        self._process_pool.shutdown()

    async def _little_spider(self, url):
        response = await self._loop.run_in_executor(
            self._thread_pool, self._request, url)
        urls = await self._loop.run_in_executor(self._process_pool,
                                                self._biu_soup,
                                                response.text)
        print(urls)

    def _request(self, url):
        return requests.get(url=url,timeout=10)

    @classmethod
    def _biu_soup(cls, response_str):
        
        # 使用进程池执行任务的时候，需要加上 @classmethod装饰，因为多进程不共享内存
       
        soup = BeautifulSoup(response_str, 'lxml')
        a_tags = soup.find_all("a")
        a = []
        for a_tag in a_tags:
            a.append(a_tag['href'])
        return a


    如果顺序执行要6s结束，如果是多线程执行4S结束，性能是有所提升的，但是我们要知道这里的性能提升实际上是由于cpu并发实现性能提升，
    也就是cpu线程切换（多道技术）带来的，而并不是真正的多cpu并行执行
    
    

# aysncio 示例
## 1. 异步Redis

    当通过python去操作redis时，链接、设置值、获取值 这些都涉及网络IO请求，
    使用asycio异步的方式可以在IO等待时去做一些其他任务，从而提升性能。

### 代码

    import aioredis
    import asyncio
    
    
    class Redis:
    
        _redis = None
    
        def __init__(self, host, port, db=0, encode="utf-8"):
    
            self.host = host
            self.port = port
            self.address = (host, port)
            self.db = db
            self.encode = encode
    
        async def get_redis_pool(self, *args, **kwargs):
            """连接redis连接池 最大连接数10"""
            if not self._redis:
                # 网络IO操作：创建redis连接
                self._redis = await aioredis.create_redis_pool(*args, **kwargs)
    
            return self._redis
    
        async def close(self):
            """关闭redis"""
            if self._redis:
                self._redis.close()
                # 网络IO操作：关闭redis连接
                await self._redis.wait_closed()
    
        async def hmset_value(self, key, value):
    
            if not self._redis:
    
                await self.get_redis_pool(self.address, db=self.db, encoding=self.encode)
    
            await self._redis.hmset_dict(key, **value)
    
            await self._redis.close()
    
        async def get_value(self, key):
    
            # 网络IO操作：去redis中获取值
    
            result = await self._redis.hgetall(key, encoding='utf-8')
    
            await self._redis.close()
    
            return result
    
        @staticmethod
        def async_task_start(func_obj):
    
            loop = asyncio.get_event_loop()
    
            async_task = asyncio.ensure_future(func_obj)
    
            try:
                loop.run_until_complete(async_task)
    
                if async_task.done():
                    print(async_task.result())
            except:
                loop.stop()
                raise
            finally:
    
                try:
                    loop.run_until_complete(loop.shutdown_asyncgens())
                finally:
                    loop.close()
    
    
    if __name__ == "__main__":
    
        host = '127.0.0.1'
        port = 6379
        db = 7
    
        application = Redis(host, port, db)
    
        application.async_task_start(application.hmset_value('car', {"key1": 1, "key2": 2}))
        application.async_task_start(application.get_value('car'))

## 2. 异步MySQL

    通过python去操作MySQL时，连接、执行SQL、关闭都涉及网络IO请求，使用asycio异步的方式可以在IO等待时去做一些其他任务，
    从而提升性能。
    
### 代码
    import asyncio
    import aiomysql
    
    
    async def select_value():
        # 网络IO操作：连接MySQL
        conn = await aiomysql.connect(host='127.0.0.1', port=3306, user='root', password='123', db='mysql', )
        
        # 网络IO操作：创建CURSOR
        cur = await conn.cursor()
        
        # 网络IO操作：执行SQL
        await cur.execute("SELECT Host,User FROM user")
        
        # 网络IO操作：获取SQL结果
        result = await cur.fetchall()
        
        # 网络IO操作：关闭链接
        await cur.close()
        conn.close()
    
    
    def async_task_start(func_obj):
    
        loop = asyncio.get_event_loop()
    
        try:
            loop.run_until_complete(func_obj)
        except:
            loop.stop()
            raise
        finally:
    
            try:
                loop.run_until_complete(loop.shutdown_asyncgens())
            finally:
                loop.close()
    
    
    if __name__ == '__main__':
        
        async_task_start(select_value())
        
## 3. 异步下载图片

    我们需要下载图片，如果你一个个地下载会出现这样的情况：
    
        如果某个请求堵塞，整个队列都会被堵塞
        如果是小文件，单线程下载太慢
        
    在请求第一个图片获得数据的时候，它会切换请求第二个图片或其他图片，等第一个图片获得所有数据后再切换回来。
    从而实现多线程批量下载的功能
    
### 代码

    import os
    import asyncio
    import aiohttp
    
    
    class AsyncImage:
    
        @staticmethod
        async def get_image(web_server, url, file_dir):
    
            file_name = url.split('/')[-1]
            file_path = os.path.join(file_dir, str(file_name))
    
            async with web_server.get(url, verify_ssl=False) as response:
    
                content = await response.content.read()
    
                with open(file_path, 'wb') as f:
                    f.write(content)
    
            return True
    
        async def image_task(self, url_list, file_dir="image"):
    
            if not os.path.exists(file_dir):
                os.mkdir(file_dir)
    
            async with aiohttp.ClientSession() as web_server:
    
                tasks = [
                    asyncio.ensure_future(self.get_image(web_server, v, file_dir)) for v in url_list
                ]
    
                await asyncio.wait(tasks)
    
        @staticmethod
        def async_task_start(func_obj):
    
            loop = asyncio.get_event_loop()
    
            async_task = asyncio.ensure_future(func_obj)
    
            try:
                loop.run_until_complete(async_task)
    
                if async_task.done():
                    print(async_task.result())
            except:
                loop.stop()
                raise
            finally:
    
                try:
                    loop.run_until_complete(loop.shutdown_asyncgens())
                finally:
                    loop.close()
    
    
    if __name__ == "__main__":
    
        url = [
            "https://wx1.sinaimg.cn/mw690/4e73feeegy1gfcvebt1pqj20yi1pcarx.jpg",
            "https://wx2.sinaimg.cn/mw690/4e73feeegy1gfcvebb3urj20yi1pcqq5.jpg"
        ]
    
        application = AsyncImage()
    
        application.async_task_start(application.image_task(url))
