# 协程

# 协程的概念
    
    是单线程下的并发.本质上是一个线程,能够在多个任务之间切换来节省一些IO时间
    
## 协程的本质：
    
    协程是遵循某些规则的生成器
    
## 协程的作用：
    
    在执行函数A时，可以随时中断，去执行函数B，然后中断继续执行函数A（可以自由切换）。但这一过程并不是函数调用（没有调用语句），
    这一整个过程看似像多线程，然而协程只有一个线程执行.
    
## 协程的特点：
    能够在一个线程中实现并发效果的概念
    能够规避一些任务中的IO操作
    在任务的执行过程中,检测到IO就切换到其他任务
 
### 优点：
    协程相比于多线程的优势 切换的效率更快
### 缺点：
    协程的本质是单线程下，无法利用多核。可以是一个程序开启多个进程，每个进程内开启多个线程，每个线程内开启协程
    协程指的是单个线程，因而一旦协程出现阻塞，将会阻塞整个线程

## 协程的调度：
    进程/线程之间的调度靠操作系统
    协程是由用户程序自己控制调度的
    
## 协程的效率高
    协程由于由程序主动控制切换，没有线程切换的开销，所以执行效率极高。
    协程中任务之间的切换也消耗时间,但是开销要远远小于进程线程之间的切换
    
    对于IO密集型任务非常适用，如果是cpu密集型，推荐多进程+协程的方式
    

# yield 实现协程
    
    生产者生产消息后，直接通过yield跳转到消费者开始执行，待消费者执行完毕后，切换加生产者继续生产，效率较高。
    
## 代码实现生产者消费者模型
    import time
    
    
    def producer(con):
    
        next(con)
    
        n = 0
    
        while n < 10:
            n += 1
            print("[producer]-->%s" % n)
    
            cr = con.send(n)
    
            print("[producer] consumer return:%s" % cr)
    
        con.close()
    
    
    def consumer():
    
        r = None
    
        while True:
            n = yield r
    
            print("[consumer]<-- %s" % n)
    
            time.sleep(1)
    
            r = "ok"
    
    
    if __name__ == "__main__":
        con = consumer()
        producer(con)
        
        
# greenlet 实现协程
    pip install greenlet

    from greenlet import greenlet
    greenlet 是python的第三方模块, 通过switch()方法手动实现切换
## 实现方式1：
    import time
    from greenlet import greenlet
    
    def task1():
        print("task1 start ---->", time.ctime())  # 遇到了io，执行到 g2.switch()，就切换到task2函数了
        g2.switch()
        time.sleep(5)
        print("task1 end ---->", time.ctime())
    
    def task2():
    
        print("task2 start ---->", time.ctime())
        g1.switch()
    
        time.sleep(3)
    
        print("task2 end ---->", time.ctime())
    
    
    if __name__ == "__main__":
        s_time = time.time()
        g1 = greenlet(task1)
        g2 = greenlet(task2)
    
        g1.switch()
    
        print("=====use time is： %s " % (time.time() - s_time))

## 实现方式2： 
    import time
    import queue
    from greenlet import greenlet
    
    que = queue.Queue()
    
    
    def producer(name):
    
        n = 0
    
        while n < 10:
            n += 1
    
            print("[producer: %s ]-->%s" % (name, n))
    
            que.put(n)
    
            gc.switch("顾客xxx")
    
            time.sleep(0.5)
    
    
    def consumer(name):
        while True:
            if not que.empty():
                data = que.get()
                print("[consumer: %s ]<-- %s" % (name, data))
                gp.switch()
            time.sleep(1)
    
    
    if __name__ == "__main__":
        s_time = time.time()
    
        gp = greenlet(producer)
        gc = greenlet(consumer)
    
        gp.switch("厂商xxxx")
    
        print("=====use time is： %s " % (time.time() - s_time))
        
# gevent 实现协程
    gevent遇到io主动切换 使用gevent的时候必须用spawn和join
## 实现方式1：
    import time
    from greenlet import greenlet
    import gevent
    
    
    def task1():
    
        print("task1 start ---->", time.ctime())  # 遇到了io，执行到 g2.switch()，就切换到task2函数了
        gevent.sleep(1)
        print("task1 end ---->", time.ctime())
    
    
    def task2():
    
        print("task2 start ---->", time.ctime())
        gevent.sleep(0.5)
        print("task2 end ---->", time.ctime())
    
    
    if __name__ == "__main__":
        s_time = time.time()
        g1 = gevent.spawn(task1)
        g2 = gevent.spawn(task2)
    
        gevent.joinall([g1, g2])
    
        print("=====use time is： %s " % (time.time() - s_time))
        """
            task1 start ----> Tue Jun 30 19:23:06 2020
            task2 start ----> Tue Jun 30 19:23:06 2020
            task2 end ----> Tue Jun 30 19:23:07 2020
            task1 end ----> Tue Jun 30 19:23:07 2020
            =====use time is： 1.017209529876709 
        """
        
# asyncio
