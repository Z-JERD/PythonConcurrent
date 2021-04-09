# 线程
## 参考文档：https://mp.weixin.qq.com/s/Hgp-x-T3ss4IiVk2_4VUrA

# 单线程
    1. 使用线程可以把占据长时间的程序中的任务放到后台去处理
    2. 在一些等待的任务实现上如用户输入、文件读写和网络收发数据等，线程就比较有用了。在这种情况下我们可以释放一些珍贵的资源如内存占用等等
## 线程的建立
    threading.Thread(group=None, target=None, name=None, args=(), kwargs=None, *, daemon=None)
    
    1. target:  函数名字，需要调用的函数
    2. args:    函数需要的参数，以 tuple 的形式传入
    3. daemon:  设置为True时，则主线程不必等待子线程，主线程结束则所有结束
    4. star():  启动线程
    5. join()   实现线程间的同步，等待所有线程退出。在子线程完成运行之前，这个子线程的父线程将一直被阻塞
    6. close()  阻止多余的线程涌入进程池 Pool 造成线程阻塞
    7. setDaemon(True)  将线程声明为守护线程  必须在start() 方法调用之前设置
    
## 实现1：主线程和子线程同时运行
    因为主线程和子线程是同时跑的。但是主进程要等非守护子线程结束之后，主线程才会退出。
    
    import os
    import time
    import threading
    
    
    def thread_task(data):
    
        print("------子线程data:%s enter thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        time.sleep(2)
    
        print("------子线程data:%s exit thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        return
    
    
    def main():
        start_time = time.time()
        t = threading.Thread(target=thread_task, args=(100,), daemon=None)
        t.start()
    
        print("--------主线程end 父进程号为: %s, 线程号为：%s" % (os.getpid(), threading.currentThread().ident))
        print("-----耗时：%s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        main()
        
## 实现2：子线程运行结束，主线程结束
    import os
    import time
    import threading
    
    
    def thread_task(data):
    
        print("------子线程data:%s enter thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        time.sleep(2)
    
        print("------子线程data:%s exit thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        return
    
    
    def main():
        start_time = time.time()
        t = threading.Thread(target=thread_task, args=(100,), daemon=None)
        t.start()
        t.join()
    
        print("--------主线程end 父进程号为: %s, 线程号为：%s" % (os.getpid(), threading.currentThread().ident))
        print("-----耗时：%s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        main()

# 守护线程
## 线程运行机制：
    主线程在其他非守护线程运行完毕后才算运行完毕（守护线程在此时就被回收）。
    因为主线程的结束意味着进程的结束，进程整体的资源都将被回收，而进程必须保证非守护线程都运行完毕后才能结束。
 
## 守护线程概念：
    默认的子线程都是主线程的非守护子线程。如果某个子线程设置为守护线程，主线程就不用管这个子线程了，
    当所有其他非守护线程结束，主线程就会退出，而守护线程不管运行完毕都将和主线程一起退出
   
## 守护线程实现
    import os
    import time
    import threading
    
    
    def thread_task(data):
    
        print("------子线程data:%s enter thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        time.sleep(2)
    
        print("------子线程data:%s exit thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        return
    
    
    def thread_task_2(data):
    
        print("------子线程data:%s enter thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        time.sleep(5)
    
        print("------子线程data:%s exit thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        return
    
    
    def main():
        start_time = time.time()
        
        t = threading.Thread(target=thread_task, args=(100,), daemon=None)
        t_2 = threading.Thread(target=thread_task_2, args=(200,), daemon=None)
        
        t_2.setDaemon(True)
        
        t.start()
        t_2.start()
    
        print("--------主线程end 父进程号为: %s, 线程号为：%s" % (os.getpid(), threading.currentThread().ident))
        print("-----耗时：%s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        main()
    

# 多线程
## 实现方式 1:
    import os
    import time
    import threading
    
    
    def thread_task(data):
    
        print("------子线程data:%s enter thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        time.sleep(2)
    
        print("------子线程data:%s exit thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        return data ** 2
    
    
    def main():
        start_time = time.time()
        thread_data = [100, 200]
    
        thread_list = []
    
        for v in thread_data:
            t = threading.Thread(target=thread_task, args=(100,), daemon=None)
            t.start()
            thread_list.append(t)
    
        for v in thread_list:
            v.join()
    
        print("--------主线程end 父进程号为: %s, 线程号为：%s" % (os.getpid(), threading.currentThread().ident))
        print("-----耗时：%s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        main()
## 实现方式2：
    import os
    import time
    import threading
    
    
    def thread_task(data):
    
        print("------子线程data:%s enter thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        time.sleep(2)
    
        print("------子线程data:%s exit thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        return data ** 2
    
    
    class MyThread(threading.Thread):
        def __init__(self, func, args):
            threading.Thread.__init__(self)   # 不要忘记调用Thread的初始化方法
            self.func = func
            self.args = args
            self.result = None
    
        def run(self):
            self.result = self.func(self.args)
    
        def getResult(self):
            return self.result
    
        @classmethod
        def main(cls):
            start_time = time.time()
            thread_data = [100, 200]
    
            thread_list = []
    
            for v in thread_data:
                t = MyThread(thread_task, v)
                t.start()
                thread_list.append(t)
    
            for v in thread_list:
                v.join()
    
            for v in thread_list:
                print("------result is %s " % v.getResult())
    
            print("--------主线程end 父进程号为: %s, 线程号为：%s" % (os.getpid(), threading.currentThread().ident))
            print("-----耗时：%s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        MyThread.main()
        
 # threading.local()

    多线程操作时,为每个线程创建一个值。使得线程之间操作自己的值,互相不影响。
    
    一旦在主线程实例化了一个local，它会一直活在主线程中，并且又主线程启动的子线程调用这个local实例时，
    它的值将会保存在相应的子线程中。
    
    使用场景:为每个线程绑定一个资源

## 实例：

    import time
    import threading


    info = "Python Thread Global"


    localVal = threading.local()

    localVal.var = "Python Thread Local"


    def thread_task(data):

        print("{}------>{}".format(info, data))

        localVal.var = f"Local var is {data}"

        time.sleep(2)

        print(localVal.var)

        return data ** 2


    def main():
        start_time = time.time()
        thread_data = [100, 200]

        thread_list = []

        for v in thread_data:
            t = threading.Thread(target=thread_task, args=(v,), daemon=None)
            t.start()
            thread_list.append(t)

        for v in thread_list:
            v.join()

        print(localVal.var)

        print("-----耗时：%s " % (time.time() - start_time))


    if __name__ == "__main__":
        main()
        

# 线程的同步
    
    线程间访问同一变量需要同步
    
    线程间加锁会导致性能损失
    
    加锁可能产生死锁，迭代死锁和互相调用死锁
    
    可重入锁可以避免迭代死锁
    
## Lock互斥锁
    多线程使用共享资源时，不加锁会造成数据不安全。线程中有了GIL锁为什么还会出现这种情况：
    GIL锁锁的是线程。保证同一时刻只有一个线程在运行。当A线程运行，拿到共享资源后，还没来得及处理，时间片切换到B线程，B线程拿到
    的是未处理的资源，因此造成数据不安全。需要加锁来避免线程间冲突
    
### 未使用Lock
    import os
    import time
    import threading
    
    
    num = 10
    
    
    def thread_task(data):
    
        global num
        num1 = num
    
        print("------子线程data:%s enter thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        time.sleep(0.5)
    
        num = num1 - 1
    
        print("------子线程data:%s exit thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        return data ** 2
    
    
    def main():
        start_time = time.time()
    
        thread_list = []
    
        for v in range(10):
            t = threading.Thread(target=thread_task, args=(v,), daemon=None)
            t.start()
            thread_list.append(t)
    
        for v in thread_list:
            v.join()
    
        print("--------主线程end 父进程号为: %s, 线程号为：%s" % (os.getpid(), threading.currentThread().ident))
        print("-----耗时：%s " % (time.time() - start_time))
        print("--------the num is %s " % num)
    
    
    if __name__ == "__main__":
        main()
        """
       -----耗时：0.5043702125549316 
        --------the num is 9 
        
        增加了一个sleep()操作，当在没有锁的情况下线程将在这里被释放出来，让给下一线程运行，而num值还没有被修改，
        所以后面线程的num1的取值都是10
        """
        
 
 
### 使用Lock
    import os
    import time
    import threading
    
    
    num = 10
    
    
    def thread_task(data, lock):
        """
            :param data:
            :param lock:
            :return:
            获取锁 如果忘记释放锁，就可能会导致死锁，可用 上下文管理器来实现 with
        
             global num
        
            num1 = num
        
            with lock:
        
                print("------子线程data:%s enter thread task  父进程号为: %s, 线程号为：%s " % (
                data, os.getpid(), threading.currentThread().ident))
        
                time.sleep(0.5)
        
                num = num1 - 1
        
                print("------子线程data:%s exit thread task  父进程号为: %s, 线程号为：%s" % (
                data, os.getpid(), threading.currentThread().ident))
    
            return data ** 2
        """
    
        global num
    
        lock.acquire()
        num1 = num
    
        print("------子线程data:%s enter thread task  父进程号为: %s, 线程号为：%s " % (data, os.getpid(), threading.currentThread().ident))
    
        time.sleep(0.5)
    
        num = num1 - 1
    
        print("------子线程data:%s exit thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
        lock.release()
        return data ** 2
    
    
    def main():
        lock = threading.Lock()
        start_time = time.time()
    
        thread_list = []
    
        for v in range(10):
            t = threading.Thread(target=thread_task, args=(v, lock), daemon=None)
            t.start()
            thread_list.append(t)
    
        for v in thread_list:
            v.join()
    
        print("--------主线程end 父进程号为: %s, 线程号为：%s" % (os.getpid(), threading.currentThread().ident))
        print("-----耗时：%s " % (time.time() - start_time))
        print("--------the num is %s " % num)
    
    
    if __name__ == "__main__":
        main()
        """
        -----耗时：5.010795593261719 
        --------the num is 0 
        """
 
 
## 死锁:

    死锁主要在两种情况下发生:
    
        1. 迭代死锁:
            
            一个线程“迭代”请求同一个资源 ，会造成死锁
        
        2. 互相调用死锁:
             
             两个线程中都会调用相同的资源，互相等待对方结束的情况 。假设A线程需要资源a,b，B线程也需要资源a,b，
             而线程A先获取资源a后，再获取资源b， B线程先获取资源b，再获取资源a。在A线程获取资源a，B线程获取资源b后，
             A线程在等待B线程释放资源b，而B线程在等待A线程释放资源a，从而死锁就发生了

### 迭代死锁：

    total = 0
    
    lock = threading.Lock()
    
    lock.acquire()
    lock.acquire()
    
    total += 1
    
    lock.release()
    lock.release()   
    
    一次请求资源后还未 release ，再次acquire，最终无法释放，造成死锁
    
### 互相调用死锁
    
    import time
    import threading
    
    
    def thread_task_1(noodles_lock, fork_lock, name):
    
        noodles_lock.acquire()
    
        print("======= %s 获取到了面条" % name)
    
        time.sleep(0.5)
    
        fork_lock.acquire()
    
        print("======= %s 获取到了叉子" % name)
    
        print("======= %s 任务完成" % name)
    
        fork_lock.release()
    
        noodles_lock.release()
    
    
    def thread_task_2(noodles_lock, fork_lock, name):
        
        fork_lock.acquire()
    
        print("======= %s 获取到了叉子" % name)
    
        time.sleep(0.5)
    
        noodles_lock.acquire()
    
        print("======= %s 获取到了面条" % name)
    
        print("======= %s 任务完成" % name)
        
        noodles_lock.release()
    
        fork_lock.release()
    
        
    
    
    def main():
        """
        吃面的任务： 需拿到面条和叉子后任务才能进行
        :return:
        主进程开始后, person1执行任务1， 拿到noodles_lock, person2执行任务2， 拿到叉子 fork_lock
                     然后person1去拿 fork_lock， 但 fork_lock已被person2拿走, 因此person1一直停在请求获取fork_lock这一步
                     走不到person1去释放fork_lock。同理因此person2一直停在请求获取noodles_lock这一步.两者相互等待
        """
    
        noodles_lock = threading.Lock()
        fork_lock = threading.Lock()
    
        t1 = threading.Thread(target=thread_task_1, args=(noodles_lock, fork_lock, "person1"))
        t2 = threading.Thread(target=thread_task_2, args=(noodles_lock, fork_lock, "person2"))
    
        t1.start()
        t2.start()
    
        t1.join()
        t2.join()
    
        print("=========主线程结束")
    
    
    if __name__ == "__main__":
        main()
        
## RLock 递归锁
    
    为了支持同一个线程中多次请求同一资源，Python 提供了可重入锁(RLock)。这个RLock内部维护着一个锁(Lock)和一个计数器(counter)变量，
    counter 记录了acquire 的次数，从而使得资源可以被多次acquire。直到一个线程所有 acquire都被release(计数器counter变为0)，其他的线程才能获得资源。

### RLock解决迭代死锁：
    total = 0
    
    lock = threading.RLock()
    
    lock.acquire()
    lock.acquire()
    
    total += 1
    
    lock.release()
    lock.release() 
    
### RLock的实现
    import time
    import threading
    
    
    def thread_task_1(noodles_lock, fork_lock, name):
    
        noodles_lock.acquire()
    
        print("======= %s 获取到了面条" % name)
    
        time.sleep(0.5)
    
        fork_lock.acquire()
    
        print("======= %s 获取到了叉子" % name)
    
        print("======= %s 任务完成" % name)
    
        fork_lock.release()
    
        noodles_lock.release()
    
    
    def thread_task_2(noodles_lock, fork_lock, name):
        fork_lock.acquire()
    
        print("======= %s 获取到了叉子" % name)
    
        time.sleep(0.5)
    
        noodles_lock.acquire()
    
        print("======= %s 获取到了面条" % name)
    
        print("======= %s 任务完成" % name)
    
        fork_lock.release()
    
        noodles_lock.release()
    
    
    def main():
        """
        吃面的任务： 需拿到面条和叉子后任务才能进行
        
        noodles_lock = fork_lock = threading.RLock()

            noodles_lock 和 fork_lock 是同一资源 id相同

            print(id(noodles_lock))         1904251945808
            print(id(fork_lock))            1904251945808

        noodles_lock = threading.RLock()

        fork_lock = threading.RLock()

            noodles_lock 和 fork_lock 非同一资源 

            print(id(noodles_lock))         2233250267984
            print(id(fork_lock))            2233281886608

            使用此方法仍会产生死锁
        
        """
    
        noodles_lock = fork_lock = threading.RLock()   # 一个钥匙串上的两把钥匙
    
        t1 = threading.Thread(target=thread_task_1, args=(noodles_lock, fork_lock, "person1"))
        t2 = threading.Thread(target=thread_task_2, args=(noodles_lock, fork_lock, "person2"))
    
        t1.start()
        t2.start()
    
        t1.join()
        t2.join()
    
        print("=========主线程结束")
        
        """
            ======= person1 获取到了面条
            ======= person1 获取到了叉子
            ======= person1 任务完成
            ======= person2 获取到了叉子
            ======= person2 获取到了面条
            ======= person2 任务完成
            =========主线程结束
        """
    
    
    if __name__ == "__main__":
        main()

### RLock中互斥锁和计数器的工作原理：
    
    在threading内部，RLock实现方式有两种，一种是调用_thread模块下的RLock，它是用C语言写的，另外一种是用Python语言写的
    两者实现原理是一致的
    
    流程：
    
        当一个线程通过acquire()获取一个锁时， 首先会判断拥有锁的线程和调用acquire()的线程是否是同一个线程，如果是同一个线程
        那么计数器+1，如果两个线程不一致时，那么会通过调用底层锁（_allocate_lock())进行阻塞自己（也可能是获得锁）
        
    当某个线程内部多次调用可重入锁时，仅仅在第一次获取锁对象时调用了_thread模块中锁的acquire()方法，
    第二次，第三次...只是让计数器加1了而已；而当其他线程获取该锁时，因为调用了 _trhead模块中 _allocate_lock()方法阻塞了自己
    
### Lock和RLock的区别
   
    Rlock实例化之后该对象可以在一个线程一直去acquire()，没有release()，其他对象不能获取锁；
    Lock是互斥锁，同一时间只能在一个线程/进程acquire()一次，没有release()，其他对象不能获取锁。

    当同一个线程/进程需要用的2次（包含）以上的锁的时候，用Lock就可能出现死锁的问题。这时候就需要用递归锁去解决该问题
    
## 信号量(Semaphore)
    信号量是一个内部数据，它有一个内置的计数器，它标明能同时执行多少个线程
    在多线程环境下用于协调各个线程, 以保证它们能够正确、合理的使用公共资源

### 应用场景
    
    1. 在读写文件的时候，一般只能只有一个线程在写，而读可以有多个线程同时进行，如果需要限制同时读文件的线程个数，
       这时候就可以用到信号量了（如果用互斥锁，就是限制同一时刻只能有一个线程读取文件）
       
    2. 爬虫实例, 有一个UrlProducer线程，爬取url，多个htmlSpider线程，爬取url对应的网页。如果直接开20个htmlSpider线程，
       20个线程是同时执行的，现在要限制同时执行能执行三个，就可以使用信号量来控制

### 信号量实现
    import time
    import threading
    
    
    def thread_task(lock, data):
    
        lock.acquire()
    
        print("======= data %s done work" % data)
    
        time.sleep(0.5)
    
        lock.release()
    
    
    def main():
    
        sem_lock = threading.Semaphore(3)       # 创建信号量对象，3个线程并发
    
        task_list = []
    
        for i in range(1, 11):
    
            t = threading.Thread(target=thread_task, args=(sem_lock, i))
            t.start()
    
            task_list.append(t)
    
        for v in task_list:
            v.join()
    
        print("=========主线程结束")
    
    
    if __name__ == "__main__":
        main()
        
## Condition 条件变量
   
    使得线程等待，只有满足某条件时，才释放n个线程

### 定义条件变量
    condition = threading.Condition()
    
    acquire()           获得锁(线程锁)
    release()           释放锁
    wait(timeout)       挂起线程timeout秒(为None时时间无限)，直到收到notify通知或者超时才会被唤醒继续运行。必须在获得Lock下运行。
    notify(n=1)         通知挂起的线程开始运行，默认通知正在等待该condition的线程，可同时唤醒n个。必须在获得Lock下运行。
    notifyAll()         通知所有被挂起的线程开始运行。必须在获得Lock下运行。
   
### Condition 实现1
    
    线程produce产品生产成功后通知线程consume使用产品，线程consume使用完产品后通知线程produce继续生产产品。

    import threading
    import time
    
    # 商品
    product = None
    # 条件变量对象
    con = threading.Condition()
    
    
    # 生产方法
    def produce():
        global product  # 全局变量产品
        if con.acquire():
            while True:
                print('---执行，produce--')
                if product is None:
                    product = '手机配件'
                    print('---生产产品:%s---' % product)
                    con.notify()  # 唤醒消费线程
                # 等待通知
                con.wait()
                time.sleep(2)
    
    
    # 消费方法
    def consume():
        global product
        if con.acquire():
            while True:
                print('***执行，consume***')
                if product is not None:
                    print('***卖出产品:%s***' % product)
                    product = None
                    # 通知生产者，商品已经没了
                    con.notify()
                # 等待通知
                con.wait()
                time.sleep(2)
    
    
    if __name__=='__main__':
        t1 = threading.Thread(target=consume)
        t1.start()
        t2 = threading.Thread(target=produce)
        t2.start()
 
###  Condition 实现2
    
    生产商品数量达到一定条件后被消费   
    
    import threading
    import time
    
    
    num = 0
    money = 10
    # 条件变量对象
    con = threading.Condition()
    
    
    # 生产方法
    def produce():
        global num
        # 获取锁
        con.acquire()
        while True:
            num += 1
            print('生产了1个，现在有{0}个'.format(num))
            time.sleep(1)
            if num >= 5:
                print('已达到5个，停止生产')
                # 唤醒消费者费线程
                con.notify()
                # 等待-释放锁 或者 被唤醒-获取锁
                con.wait()
    
    
    # 消费方法
    def consume():
        global num, money
        while money > 0:
            con.acquire()
            if num <= 0:
                print('没货了，{0}通知生产者'.format(threading.current_thread().name))
                con.notify()
                con.wait()
            money -= 1
            num -= 1
            print('{0}消费了1个, 剩余{1}个'.format(threading.current_thread().name, num))
            con.release()
            time.sleep(1)
    
        print('{0}没钱了-停止消费'.format(threading.current_thread().name))
    
    
    if __name__=='__main__':
        t1 = threading.Thread(target=consume)
        t1.start()
        t2 = threading.Thread(target=produce)
        t2.start()
## Event 事件
    
    用于线程间通信，即程序中的其一个线程需要通过判断某个线程的状态来确定自己下一步的操作，就用到了event()对象
    
### 定义Event 事件
    
    event = threading.Event()  
    
    event()对象有个状态值，他的默认值为 Flase
    wait(timeout=None) 挂起线程timeout秒(None时间无限)，直到超时或收到event()信号开关为True时才唤醒程序。
    set() Even状态值设为True
    clear() Even状态值设为 False
    isSet() 返回Even对象的状态值
    
### Event 实现
    import time
    import threading
    
    
    event = threading.Event()
    
    
    def thread_task_1():
    
        print("======= task 1 start work")
    
        event.wait()
    
        print("======= task 1 continue to do work ")
    
        time.sleep(0.5)
    
        print("======= task 1  done work ")
    
    
    def thread_task_2():
    
        print("======= task 2 start work")
    
        time.sleep(2)
        print("======= task 2 done work")
    
        event.set()
    
    
    def main():
        """
        吃面的任务： 需拿到面条和叉子后任务才能进行
        """
    
        t1 = threading.Thread(target=thread_task_1)
        t2 = threading.Thread(target=thread_task_2)
    
        t1.start()
        t2.start()
    
        t1.join()
        t2.join()
    
        print("=========主线程结束")
    
    
    if __name__ == "__main__":
        main()
        

# 线程池 threadpool
    Python的第三方模块  
    开线程池的数目是: cpu 个数*5
    
# 线程池 ThreadPoolExecutor
    
    从Python3.2开始，标准库为我们提供了 concurrent.futures 模块，它提供了 ThreadPoolExecutor (线程池)和ProcessPoolExecutor (进程池)
    
## 线程池的基本使用
    Exectuor 提供了如下常用方法：
        submit(fn, *args, **kwargs)：将 fn 函数提交给线程池
        
        map(func, *iterables, timeout=None, chunksize=1)：该函数类似于全局函数 map(func, *iterables)，只是该函数将会启动多个线程，
            以异步方式立即对 iterables 执行 map 处理。
        
        shutdown(wait=True)：关闭线程池。
    
    程序将 task 函数提交（submit）给线程池后，submit 方法会返回一个 Future 对象，Future 类主要用于获取线程的状态，以及返回值。
 
### Future 对象
    cancel()：                   取消线程任务。如果该任务正在执行，不可取消，则该方法返回 False；否则，程序会取消该任务，并返回 True。
    cancelled()：                返回线程任务是否被成功取消。
    running()：                  线程任务正在执行 该方法返回 True。
    done()：                     线程任务被成功行完成，则该方法返回 True。
    result(timeout=None)：       线程任务最后返回的结果。如果 Future 代表的线程任务还未完成，该方法将会阻塞当前线程，其中 timeout 参数指定最多阻塞多少秒。
    exception(timeout=None)：    获取该 Future 代表的线程任务所引发的异常。如果该任务成功完成，没有异常，则该方法返回 None。
    add_done_callback(fn)：      为该 Future 代表的线程任务注册一个“回调函数”，当该任务成功完成时，程序会自动触发该 fn 函数
    
### 线程池几基本实现
    import os
    import threading
    import time
    from concurrent.futures import ThreadPoolExecutor
    
    
    def thread_task(data):
        print(
            "------子线程data:%s enter thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        time.sleep(data / 1000)
    
        print(
            "------子线程data:%s exit thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        return data**2
    
    
    def main():
    
        data_list = [100, 200]
    
        task_list = []
    
        """
        线程池实现了上下文管理协议（Context Manage Protocol），因此，程序可以使用 with 语句来管理线程池，这样即可避免手动关闭线程池
         thread_pool = ThreadPoolExecutor(max_workers=5)
         for v in data_list:
                t = thread_pool.submit(thread_task, v)
                task_list.append(t)
         
         thread_pool.shutdown(wait=True)
         
        """
    
        with ThreadPoolExecutor(max_workers=5) as thread_pool:
            for v in data_list:
                t = thread_pool.submit(thread_task, v)
                task_list.append(t)
    
        for v in task_list:
            while True:
                if v.done():
                    print("status is %s, the result is %s " % (v.done(), v.result()))
                    break
                else:
                    print("status is %s " % (v.done(),))
    
                time.sleep(0.1)
    
    
    if __name__ == "__main__":
        main()

## as_completed
    判断任务是否结束, 当某个任务结束了, 就给主线程返回结果
    as_completed() 当子线程中的任务执行完后，直接用 result() 获取返回结果
    
### as_completed获取返回值
    import os
    import threading
    import time
    from concurrent.futures import ThreadPoolExecutor, as_completed
    
    
    def thread_task(data):
        print(
            "------子线程data:%s enter thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        time.sleep(data / 1000)
    
        print(
            "------子线程data:%s exit thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        return data**2
    
    
    def main():
    
        data_list = [100, 200]
    
        task_list = []
    
        """
         thread_pool = ThreadPoolExecutor(max_workers=5)
         for v in data_list:
                t = thread_pool.submit(thread_task, v)
                task_list.append(t)
         
         thread_pool.shutdown(wait=True)
         
        """
    
        with ThreadPoolExecutor(max_workers=5) as thread_pool:
            for v in data_list:
                t = thread_pool.submit(thread_task, v)
                task_list.append(t)
    
            for v in as_completed(task_list):
                print("status is %s, the result is %s " % (v.done(), v.result()))
    
    
    if __name__ == "__main__":
        main()
## add_done_callback
### 回调函数的实现
    import os
    import threading
    import time
    from concurrent.futures import ThreadPoolExecutor, as_completed


    def thread_task(data):
        print(
            "------子线程data:%s enter thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))

        time.sleep(data / 100 + 2)

        print(
            "------子线程data:%s exit thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))

        return data**2


    def thread_task_callback(future):

        """
            回调在子线程中进行
        """
        print(" thread id is %s done task callback, the result is %s " % (threading.currentThread().ident, future.result()))

        time.sleep(1)

        return


    def main():

        """
         print("-------主线程 thread task：%s" % threading.currentThread().ident)

         放在 with 的外层, 子线程执行完后，主线程才会继续

         放在with的内层， 子线程开启后，主线程就会继续执行
        """

        data_list = [100, 200]

        with ThreadPoolExecutor(max_workers=5) as thread_pool:

            for v in data_list:
                t = thread_pool.submit(thread_task, v)
                t.add_done_callback(thread_task_callback)

            print("-------主线程 thread task：%s" % threading.currentThread().ident)


    if __name__ == "__main__":
        main()


## map
    map() 方法的返回值将会收集每个线程任务的返回结果

### map方法的实现
    import os
    import threading
    import time
    from concurrent.futures import ThreadPoolExecutor, as_completed
    
    
    def thread_task(data):
        print(
            "------子线程data:%s enter thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        time.sleep(data / 1000)
    
        print(
            "------子线程data:%s exit thread task  父进程号为: %s, 线程号为：%s" % (data, os.getpid(), threading.currentThread().ident))
    
        return data**2
    
    
    def main():
    
        data_list = [100, 200]
    
        task_list = []
    
        with ThreadPoolExecutor(max_workers=5) as thread_pool:
    
            p_res = thread_pool.map(thread_task, data_list)
    
            for v in p_res:
                print("the result is %s " % v)
    
    if __name__ == "__main__":
        main()
