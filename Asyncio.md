# 参考文档：https://www.jianshu.com/p/b5e347b3a17c  https://www.lylinux.net/article/2019/6/9/57.html

# asyncio的基本概念
    
    在 Python 3.4 时候引进了协程的概念，它使用一种单线程单进程的的方式实现并发
    
    异步最适合的就是那种等待io，不需要消耗cpu的操作.比如爬虫，或者第三方接口的调用
    
## async/await 关键字：
        python3.5 用于定义协程的关键字
        
        async：将一个函数定义为一个协程
        
        await： 将协程的控制权让出，以便loop调用其他的协程。
                针对耗时的操作进行挂起，协程遇到await，事件循环将会挂起该协程，执行别的协程，直到其他的协程也挂起或者执行完毕，再进行下一个协程的执行。
                耗时的操作一般是一些IO操作，例如网络请求，文件读取等。我们使用asyncio.sleep函数来模拟IO操作。协程的目的也是让这些IO操作异步化
    
## 1. Coroutine 协程：
        协程函数: 定义形式为 async def 的函数
        协程对象：指一个使用async关键字定义的函数，它的调用不会立即执行函数，而是会返回一个协程对象。
                  协程对象需要注册到事件循环，由事件循环调用。
                  
        它的本质还是一个生成器
                  
## 2. Eventloop 事件循环：
    
        程序开启一个无限的循环，把一些函数（协程）注册到事件循环上。当满足事件发生的时候，调用相应的协程函数。
        Eventloop实例提供了注册、取消、执行任务和回调 的方法。 简单来说，就是我们可以把一些异步函数注册到这个事件循环上，事件循环回循环执行这些函数（每次只能执行一个），
        如果当前正在执行的函数在等待I/O返回，那么事件循环就会暂停它的执行去执行其他函数。当某个函数完成I/O后会恢复，等到下次循环到它的时候就会继续执行。
    
## 3. Task 任务:
        
        Task对象被用来在事件循环中运行协程。
        
        一个协程对象就是一个原生可以挂起的函数，任务则是对协程进一步封装，其中包含任务的各种状态。Task 对象是 Future 的子类
        
        一个事件循环每次运行一个Task对象。而一个Task对象会等待一个Future对象完成，该事件循环会运行其他Task、回调或执行IO操作。
    
## 4. Future：
        Future是表示一个“未来”对象，类似于javascript中的promise，当异步操作结束后会把最终结果设置到这个Future对象上，Future是对协程的封装
        它和task上没有本质的区别
        
# 协程完整的工作流程：
## 1. 流程：
        1. 定义/创建协程对象
        2. 定义事件循环对象容器
        3. 将协程转为Task任务
        4. 将Task任务扔进事件循环对象中并触发

## 2. 协程实现：
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
            
## 3. 绑定回调函数(add_done_callback)
    
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
            
## 4.协程并发的实现
    async def demo_exp(x):
    
            print("Waiting: ", x)
    
            await asyncio.sleep(x)                  # 模拟io
    
            return "暂停了{time}秒".format(time=x)
    
    def main():
            start_time = time.time()
    
            # 创建多个协程
            async_coroutine_1 = demo_exp(1)
            async_coroutine_2 = demo_exp(5)
            async_coroutine_3 = demo_exp(8)
    
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
    
            loop.run_until_complete(asyncio.wait(async_task))
    
            for v in async_task:
                    if v.done():
                            print('task result is:', v.result())
                    else:
                            print('task  is running:')
    
            print('TIME: ', time.time() - start_time)
    
    
    if __name__ == "__main__":
            main()
            """
            Waiting:  1
            Waiting:  5
            Waiting:  8
            task result is: 暂停了1秒
            task result is: 暂停了5秒
            task result is: 暂停了8秒
            TIME:  8.003252506256104
            
            时间为8s左右。如果是同步顺序的任务，那么至少需要14s
            """
         
        
 ## 5. 应用场景
     爬取某个平台下的所有图片的时候，我们需要下载图片，如果你一个个地下载会出现这样的情况：
    
        如果某个请求堵塞，整个队列都会被堵塞
        如果是小文件，单线程下载太慢
        
      在请求第一个图片获得数据的时候，它会切换请求第二个图片或其他图片，等第一个图片获得所有数据后再切换回来。
      从而实现多线程批量下载的功能
      
    import asyncio
    import aiohttp
    import os
    
    async def get_image(web_server, url, file_dir):
            name = url.split('/')[-1]
            image_obj = await web_server.get(url)
            image_code = await image_obj.read()
    
            file_path = os.path.join(file_dir, str(name))
    
            with open(file_path, 'wb') as f:
                    f.write(image_code)
    
            return url
    
    
    async def main_task(loop, url, file_dir="image"):
    
            if not os.path.exists(file_dir):
                    os.mkdir(file_dir)
    
            async with aiohttp.ClientSession() as web_server:
                    tasks = [
                            asyncio.ensure_future(get_image(web_server, v, file_dir))for v in url
                    ]
    
                    await asyncio.wait(tasks)
    
    
    if __name__ == "__main__":
            url = [
                    "https://wx1.sinaimg.cn/mw690/4e73feeegy1gfcvebt1pqj20yi1pcarx.jpg",
                    "https://wx2.sinaimg.cn/mw690/4e73feeegy1gfcvebb3urj20yi1pcqq5.jpg"
            ]
    
            loop = asyncio.get_event_loop()
            loop.run_until_complete(main_task(loop, url))
            
# 概念区别
## 1. await与yield对比:
    
    1. await用于挂起阻塞的异步调用接口。其作用在一定程度上类似于yield。（都能实现暂停的效果），但是功能上却不兼容。
           就是不能在生成器中使用await，也不能在async 定义的协程中使用yield。

    2.yield from 后面可接 可迭代对象，也可接future对象/协程对象；
      await 后面必须要接 future对象/协程对象
      
###  asyncio.sleep(n)

    asyncio自带的工具函数，他可以模拟IO阻塞，他返回的是一个协程对象
   
    import asyncio
    
    from collections import Generator
    from asyncio.futures import Future
    from asyncio.tasks import Task
    
    
    def main():
            func = asyncio.sleep(2)
    
            print(isinstance(func, Future))         #   False
            print(isinstance(func, Generator))      #   True
            
### 验证

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
            
## 2. asyncio.gather和asyncio.wait的区别
    
    asyncio.gather能收集协程的结果，而且会按照输入协程的顺序保存对应协程的执行结果
    asyncio.wait的返回值有两项，第一项是完成的任务列表，第二项表示等待完成的任务列表
    
     1. asyncio.gather
                      
            result_list = loop.run_until_complete(asyncio.gather(*async_task))
            print(result_list)  # ['暂停了1秒', '暂停了5秒']
        
     2. asyncio.wait
            
            done,pending = loop.run_until_complete(asyncio.wait(async_task))
            print(done)
            print(pending)
            
             """
            {<Task finished coro=<demo_exp() done, defined at C:/Users/86134/Desktop/ownfile/note/demo.py:68> result='暂停了1秒'>,
             <Task finished coro=<demo_exp() done, defined at C:/Users/86134/Desktop/ownfile/note/demo.py:68> result='暂停了5秒'>}
            set()
            """
            
## 3. asyncio.create_task和loop.create_task以及asyncio.ensure_future
    
    这三种方法都可以创建Task,从Python3.7开始可以统一的使用更高阶的asyncio.create_task.其实asyncio.create_task
    就是用的loop.create_task. loop.create_task接受的参数需要是一个协程，
    
    但是asyncio.ensure_future除了接受协程，还可以是Future对象或者awaitable对象：

        1. 如果参数是协程，其底层使用loop.create_task，返回Task对象
        2. 如果是Future对象会直接返回
        3. 如果是一个awaitable对象，会await这个对象的__await__方法，再执行一次ensure_future，最后返回Task或者Future。           
      
        ensure_future方法主要就是确保这是一个Future对象
        

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
