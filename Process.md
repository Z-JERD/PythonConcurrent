# 进程
## 参考文档：https://juejin.im/post/5cce9e20f265da036b4a76d6

# 单进程

## 进程的建立

    multiprocessing.Process(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)

    1. target:  函数名字，需要调用的函数
    2. args:    函数需要的参数，以 tuple 的形式传入
    3. daemon:  设置为True时，则主线程不必等待子进程，主进程结束则所有结束
    4. star():  启动进程
    5. join()   实现进程间的同步，等待所有进程退出
    6. close()  阻止多余的进程涌入进程池 Pool 造成进程阻塞

## 实现
    import time
    import multiprocessing


    def process_task(data):
         res = data ** 3

         while res > 0:
             res -= 1

         return


    def main():
        start_time = time.time()
        p = multiprocessing.Process(target=process_task, args=(100,), daemon=None)
        p.start()
        p.join()

        print("--------end")
        print("-----耗时：%s " % (time.time() - start_time))


    if __name__ == "__main__":
        main()

# 守护进程
## join()
    join()是感知一个子进程的结束, 子进程结束后再执行后续的主进程
### 1. 不使用join
    import time
    import multiprocessing
    
    
    def process_task(data):
         res = data ** 3
    
         while res > 0:
             res -= 1
    
         print("==============: 子进程结束")
    
         return
    
    
    def main():
        start_time = time.time()
        p = multiprocessing.Process(target=process_task, args=(100,), daemon=None)
        p.start()
    
        print("--------end")
        print("-----耗时：%s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        main()
        """
        --------end
        -----耗时：0.042887210845947266 
        ==============: 子进程结束
        """

### 2. 使用join
    import time
    import multiprocessing
    
    
    def process_task(data):
         res = data ** 3
    
         while res > 0:
             res -= 1
    
         print("==============: 子进程结束")
    
         return
    
    
    def main():
        start_time = time.time()
        p = multiprocessing.Process(target=process_task, args=(100,), daemon=None)
        p.start()
        p.join()
    
        print("--------end")
        print("-----耗时：%s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        main()
        """
        ==============: 子进程结束
        --------end
        -----耗时：0.20245671272277832 
        """

## 守护进程
    
    守护进程会随着主进程的代码执行完毕而结束（运行完毕并非终止运行）
    
    进程运行的机制：
        主进程在其代码结束后就已经算运行完毕了（守护进程被迫结束运行在此时被回收）,
        然后主进程会一直等非守护的子进程都运行完毕后回收子进程的资源(否则会产生僵尸进程)，才会结束
    主进程创建守护进程
    　　其一：守护进程会在主进程代码执行结束后就终止
    　　其二：守护进程内无法再开启子进程,否则抛出异常
### 守护进程实现
    Process中daemon设置为True 或者生成的进程实例设置daemon=True
    
    import time
    import multiprocessing
    
    
    def process_task(data):
         res = data ** 3
    
         while res > 0:
             res -= 1
    
         print("==============: 子进程结束")
    
         return
    
    
    def main():
        start_time = time.time()
        """
        p = multiprocessing.Process(target=process_task, args=(100,), daemon=None)
        p.daemon = True
        """
        p = multiprocessing.Process(target=process_task, args=(100,), daemon=True)
        p.start()
    
        print("--------end")
        print("-----耗时：%s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        main()
        """
        --------end
        -----耗时：0.04319334030151367 
        """
    
# 多进程
    
    多进程实现多个CPU同时执行任务。进程是Python中最小的资源分配单元，也就是进程中间的数据，内存是不共享的，
    每启动一个进程，都要独立分配资源和拷贝访问的数据，所以进程的启动和销毁的代价是比较大的

## 多进程实现方式1：
    import os
    import multiprocessing


    def process_task(data):
        """
        进程任务
        """
        print('测试多进程任务：%s, 当前进程号：%s, 父进程号为: %s' % (data, os.getpid(), os.getppid()))
        res = data ** 3

        while res > 0:
            res -= 1
        print('多进程任务结束：%s, 当前进程号为: %s' % (data, os.getpid()))

        return


    def main():
        process_data = [100, 200, 300]
        task_list = []

        print("主进程:%s, 父进程:%s" % (os.getpid(), os.getppid()))

        for v in process_data:
            p_t = multiprocessing.Process(target=process_task, args=(v, ), daemon=None)
            p_t.start()
            task_list.append(p_t)

        for v in task_list:
            v.join()


    if __name__ == "__main__":
        main()

## 多进程实现方式2
    import os
    import multiprocessing
    
    class MyProcess(multiprocessing.Process):
        def __init__(self, data):
            super(MyProcess, self).__init__()
            self.data = data

        def run(self):
            print('测试多进程任务：%s, 当前进程号：%s, 父进程号为: %s' % (self.data, os.getpid(), os.getppid()))
            res = self.data ** 3

            while res > 0:
                res -= 1
            print('多进程任务结束：%s, 当前进程号为: %s' % (self.data, os.getpid()))

        @classmethod
        def main(cls):
            process_data = [100, 200, 300]
            task_list = []

            print("主进程:%s, 父进程:%s" % (os.getpid(), os.getppid()))

            for v in process_data:
                p_t = MyProcess(v)
                p_t.start()
                task_list.append(p_t)

            for v in task_list:
                v.join()


    if __name__ == "__main__":
        MyProcess.main()

# 进程间的通信
    
    进程之间是相互独立的，每启动一个新的进程相当于把数据进行了一次克隆，子进程里的数据修改无法影响到主进程中的数据，
    不同子进程之间的数据也不能共享

## 数据不共享实例

    子进程里的数据修改无法影响到主进程中的数据

    class MyProcess(object):

        def __init__(self):
            self.sh_dock_code = [31, 32, 33, 34]

        def process_task(self, data):
            print('测试多进程任务：%s, 当前进程号：%s, 父进程号为: %s' % (data, os.getpid(), os.getppid()))
            res = data ** 3

            while res > 0:
                res -= 1

            self.sh_dock_code = [31, 32, data]

            print('多进程任务结束：%s, 当前进程号为: %s, sh_dock_code is %s' % (data, os.getpid(), self.sh_dock_code))

        def main(self):
            process_data = [100, 200, 300]
            task_list = []

            print("主进程:%s, 父进程:%s" % (os.getpid(), os.getppid()))

            for v in process_data:
                p_t = multiprocessing.Process(target=self.process_task, args=(v,), daemon=None)
                p_t.start()
                task_list.append(p_t)

            for v in task_list:
                v.join()

            print("sh_dock_code is %s " % self.sh_dock_code)


    if __name__ == "__main__":
        MyProcess().main()
        
## 1. 队列 Queue
    
    队列类似于一条管道，元素先进先出,进put(arg)，取get( )。需要注意的是：队列都是在内存中操作
### 队列实现数据交互
    import os
    import multiprocessing
    
    
    def process_task(data, queue):
        """
        进程任务
        queue: 队列
        """
        print('测试多进程任务：%s, 当前进程号：%s, 父进程号为: %s' % (data, os.getpid(), os.getppid()))
        res = data ** 3
        #queue.put("data：%s deal result is %s" % (data, res))
        queue.put({data: res})
    
        while res > 0:
            res -= 1
        print('多进程任务结束：%s, 当前进程号为: %s' % (data, os.getpid()))
    
        return
    
    
    def main():
        process_data = [100, 200, 300]
        task_list = []
        queue = multiprocessing.Queue()
    
        print("主进程:%s, 父进程:%s" % (os.getpid(), os.getppid()))
    
        for v in process_data:
            p_t = multiprocessing.Process(target=process_task, args=(v, queue), daemon=None)
            p_t.start()
            task_list.append(p_t)
    
        for v in task_list:
            v.join()
    
        while not queue.empty():
    
            print(queue.get())
    
    
    if __name__ == "__main__":
        main()
        
        
### 队列实现生产者消费者模型
    import multiprocessing
    
    
    def consumer(queue):
        while True:
            task = queue.get()
            if task is None:
                break
            print(task)
    
        return
    
    
    def producer(queue):
    
        for i in range(5):
            queue.put("work：%s" % i)
    
        return
    
    
    def main():
        queue = multiprocessing.Queue()
        p_t1 = multiprocessing.Process(target=consumer, args=(queue,), daemon=None)
        p_t2 = multiprocessing.Process(target=producer, args=(queue, ), daemon=None)
        p_t1.start()
        p_t2.start()
    
        p_t2.join()         # 保证生产者全部生产完毕,才应该发送结束信号
        queue.put(None)  
        p_t1.join()
    
    
    if __name__ == "__main__":
        main()

## 2. 管道 Pipe
    Pipe():在进程之间创建一条双向管道，并返回元组（conn1,conn2）,其中conn1，conn2表示管道两端的连接对象
   
### Pipe 实现数据交互
    import multiprocessing
    
    
    def process_task(data, p):
        """
        进程任务
        p: Pipe生成的双向管道
        conn1,conn2都传过去。不用的就给关闭掉
        """
        conn1, conn2 = p
        conn2.close()
    
        conn1.send('你好主进程, the data is %s' % data)
        print(conn1.recv())
        conn1.close()
    
        return
    
    
    def main():
        conn1, conn2 = multiprocessing.Pipe()
        data = 100
        p_t = multiprocessing.Process(target=process_task, args=(data, (conn1, conn2)), daemon=None)
        p_t.start()
        print(conn2.recv())
        conn2.send('你好子进程')
        print("======================================")
        p_t.join()
    
        conn1.close()
        conn2.close()
    
    
    if __name__ == "__main__":
        main()
        
### Pipe 实现生产者消费者模型
    import multiprocessing
    
    
    def consumer(name, p):
        conn1, conn2 = p
        conn1.close()
        while True:
            try:
                ret = conn2.recv()
                print("%s work is %s" % (name, ret))
            except Exception:
                print("===============")
                break
    
        conn2.close()
    
    
    def producer(p):
        conn1, conn2 = p
        conn2.close()
        for i in range(5):
            conn1.send("doing：%s" % i)
        conn1.close()
    
    
    def main():
        conn1, conn2 = multiprocessing.Pipe()
        data = "process pipe"
        p_t1 = multiprocessing.Process(target=consumer, args=(data, (conn1, conn2)), daemon=None)
        p_t2 = multiprocessing.Process(target=producer, args=((conn1, conn2), ), daemon=None)
        p_t1.start()
        p_t2.start()
    
        conn1.close()
        conn2.close()
    
    
    if __name__ == "__main__":
        main()

## 3. Managers
    Queue和Pipe只是实现了数据交互，并没实现数据共享，即一个进程去更改另一个进程的数据。那么要用到Managers
###  Managers实现数据共享
    import os
    import multiprocessing
    
    
    def process_task(data, queue, share_data):
        """
        进程任务
        queue: 队列
        """
        print('测试多进程任务：%s, 当前进程号：%s, 父进程号为: %s' % (data, os.getpid(), os.getppid()))
        res = data ** 3
        share_data["count"] -= 1
        share_data[data] = res
    
        while res > 0:
            res -= 1
    
        print('多进程任务结束：%s, 当前进程号为: %s' % (data, os.getpid()))
    
        return
    
    
    def main():
        process_m = multiprocessing.Manager()
        process_share_data = process_m.dict({"count": 10})
        process_data = [100, 200, 300]
        task_list = []
        queue = multiprocessing.Queue()
    
        print("主进程:%s, 父进程:%s" % (os.getpid(), os.getppid()))
    
        for v in process_data:
            p_t = multiprocessing.Process(target=process_task, args=(v, queue, process_share_data), daemon=None)
            p_t.start()
            task_list.append(p_t)
    
        for v in task_list:
            v.join()
    
        print(process_share_data)
    
    
    if __name__ == "__main__":
        main()   
  
# 进程锁 Lock
    当多个进程需要访问共享资源的时候，Lock可以用来避免访问的冲突
    当多个进程使用同一份数据资源的时候，就会引发数据安全或顺序混乱问题
    加锁可以保证多个进程修改同一块数据时，同一时间只能有一个任务可以进行修改，即串行的修改 
 
## 不加锁
    import time
    import multiprocessing
    
    
    def process_task(data, share_data):
        """
        进程任务
        queue: 队列
        """
        res = data ** 3
        share_data[data] = res
    
        for _ in range(5):
            time.sleep(0.1)
            share_data["count"] += data
            print("share_data count is %s " % share_data["count"])
    
        return
    
    
    def main():
        process_m = multiprocessing.Manager()
        process_share_data = process_m.dict({"count": 500})
        process_data = [100, 200]
        task_list = []
        queue = multiprocessing.Queue()
    
        for v in process_data:
            p_t = multiprocessing.Process(target=process_task, args=(v, process_share_data), daemon=None)
            p_t.start()
            task_list.append(p_t)
    
        for v in task_list:
            v.join()
    
        print(process_share_data)
    
    
    if __name__ == "__main__":
        main()
        """
        进程1和进程2在相互抢着使用共享内存v。
        share_data count is 600 
        share_data count is 800 
        share_data count is 900 
        share_data count is 1100 
        share_data count is 1200 
        share_data count is 1400 
        share_data count is 1500 
        share_data count is 1700 
        share_data count is 1800 
        share_data count is 2000 
    
        """   
## 加锁
    import time
    import multiprocessing
    
    
    def process_task(data, share_data, lc):
        """
        进程任务
        queue: 队列
        """
        lc.acquire()  # 锁住
        res = data ** 3
        share_data[data] = res
    
        for _ in range(5):
            time.sleep(0.1)
            share_data["count"] += data
            print("share_data count is %s " % share_data["count"])
        lc.release()    # 释放
        return
    
    
    def main():
        process_m = multiprocessing.Manager()
        process_share_data = process_m.dict({"count": 500})
        lc = multiprocessing.Lock()
        process_data = [100, 200]
        task_list = []
    
        for v in process_data:
            p_t = multiprocessing.Process(target=process_task, args=(v, process_share_data, lc), daemon=None)
            p_t.start()
            task_list.append(p_t)
    
        for v in task_list:
            v.join()
    
        print(process_share_data)
    
    
    if __name__ == "__main__":
        main()
        """
        进程锁保证了进程p1的完整运行，然后才进行了进程p2的运行
        share_data count is 600 
        share_data count is 700 
        share_data count is 800 
        share_data count is 900 
        share_data count is 1000 
        share_data count is 1200 
        share_data count is 1400 
        share_data count is 1600 
        share_data count is 1800 
        share_data count is 2000 
        {'count': 2000, 100: 1000000, 200: 8000000}
    
        """    

# 信号量 Semaphore
    信号量Semaphore是一个计数器，控制对公共资源或者临界区域的访问量，信号量可以指定同时访问资源或者进入临界区域的进程数。
    每次有一个进程获得信号量时，计数器－1，若计数器为0时，其他进程就停止访问信号量，一直阻塞直到其他进程释放信号量。
## 实现
    import multiprocessing
    
    
    def process_task(data, sem):
        """
        进程任务
        """
        print("--------data: %s enter work" % data)
        sem.acquire()
        print("data: %s start work" % data)
        res = data ** 3
    
        while res > 0:
            res -= 1
        print("data: %s done work" % data)
        sem.release()
    
        return data ** 3
    
    
    def main():
    
        process_data = [100, 200, 300, 400, 500]
        sem = multiprocessing.Semaphore(3)
    
        task_list = []
    
        for v in process_data:
            p_t = multiprocessing.Process(target=process_task, args=(v, sem), daemon=None)
            p_t.start()
            task_list.append(p_t)
    
        for v in task_list:
            v.join()
    
    
    if __name__ == "__main__":
        main()

# 进程池   multiprocessing.Pool()

## 进程池的概念：
    创建进程需要消耗时间，销毁进程也需要消耗时间
    进程池内部维护一个进程序列，当使用时，则去进程池中获取一个进程，如果进程池序列中没有可供使用的进进程，那么程序就会等待，直到进程池中有可用进程为止
    
    好处：
        1. 节省了开闭进程的时间 
        2. 一定程度上能够实现并发效果
        
    进程池和信号量的区别：
        进程池控制的是操作系统的调度 
        信号量控制的是进程
        
    查看CPU数量：os.cpu_count()
    一般开进程池的数目是: cpu个数 + 1
        
## 1. apply
    同步，当一个任务结束后，下个任务才会执行。一般不使用
    
    import os
    import time
    import multiprocessing
    
    
    def process_task(data):
        """
        进程任务
        queue: 队列
        """
        print('测试多进程任务：%s, 当前进程号：%s, 父进程号为: %s' % (data, os.getpid(), os.getppid()))
        res = data ** 3
    
        while res > 0:
            res -= 1
    
        print('多进程任务结束：%s, 当前进程号为: %s' % (data, os.getpid()))
    
        time.sleep(1)
    
        return
    
    
    def main():
        start_time = time.time()
        process_pool = multiprocessing.Pool()
        process_data = [100, 200, 300]
    
        print("主进程:%s, 父进程:%s" % (os.getpid(), os.getppid()))
    
        for v in process_data:
            process_pool.apply(func=process_task, args=(v,))  # 第一个子进程执行完func之后，第二个子进程才会开启
    
        print("user_time: %s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        main()
        
## 2. apply_async
    异步 需要手动close 和 join
    当func被注册进入一个进程之后,程序就继续向下执行, 需要先close后join来保持多进程和主进程代码的同步性
    Pool对象调用join()方法会等待所有子进程执行完毕，调用join()之前必须先调用close()，调用close()之后就不能继续添加新的Process了
    
    import os
    import time
    import multiprocessing
    
    
    def process_task(data):
        """
        进程任务
        queue: 队列
        """
        print('测试多进程任务：%s, 当前进程号：%s, 父进程号为: %s' % (data, os.getpid(), os.getppid()))
        res = data ** 3
    
        while res > 0:
            res -= 1
    
        print('多进程任务结束：%s, 当前进程号为: %s' % (data, os.getpid()))
    
        time.sleep(1)
    
        return
    
    
    def main():
        start_time = time.time()
        process_pool = multiprocessing.Pool()
        process_data = [100, 200, 300]
    
        print("主进程:%s, 父进程:%s" % (os.getpid(), os.getppid()))
    
        for v in process_data:
            process_pool.apply_async(func=process_task, args=(v,))  # 第一个子进程执行完func之后，第二个子进程才会开启
    
        process_pool.close()
        process_pool.join()
        print("user_time: %s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        main()
## 3. map
    Pool类中的map方法，与内置的map函数用法基本一致，它会使进程阻塞直到结果返回
 
    import os
    import time
    import multiprocessing
    
    
    def process_task(info):
        """
        进程任务
        queue: 队列
        """
        data, length = info
        print('测试多进程任务：%s, 当前进程号：%s, 父进程号为: %s' % (data, os.getpid(), os.getppid()))
        res = data ** 3
    
        while res > 0:
            res -= 1
    
        print('多进程任务结束：%s, 当前进程号为: %s' % (data, os.getpid()))
    
        time.sleep(1)
    
        return
    
    
    def main():
        start_time = time.time()
        process_pool = multiprocessing.Pool()
        process_data = [100, 200, 300]
    
        print("主进程:%s, 父进程:%s" % (os.getpid(), os.getppid()))
        info = [(v, 5) for v in process_data]
    
        process_pool.map(process_task, info)
    
        process_pool.close()
        process_pool.join()
        print("user_time: %s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        main()
        
  
 
## 4. 进程中的返回值
### apply_async 调用每个对象下的get方法去获取结果
    import os
    import time
    import multiprocessing
    
    
    def process_task(info):
        """
        进程任务
        queue: 队列
        """
        data, length = info
        print('测试多进程任务：%s, 当前进程号：%s, 父进程号为: %s' % (data, os.getpid(), os.getppid()))
        res = data ** 3
    
        while res > 0:
            res -= 1
    
        print('多进程任务结束：%s, 当前进程号为: %s' % (data, os.getpid()))
    
        time.sleep(1)
    
        return data ** 3
    
    
    def main():
        start_time = time.time()
        process_pool = multiprocessing.Pool()
        process_data = [100, 200, 300]
    
        print("主进程:%s, 父进程:%s" % (os.getpid(), os.getppid()))
       
        result_list = []
        for v in process_data:
            res = process_pool.apply_async(func=process_task, args=((v, 5),))
            result_list.append(res)
    
        process_pool.close()
        process_pool.join()
    
        for v in result_list:
            print(v.get())
    
        print("user_time: %s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        main()
### map 返回的结果放到一个列表中
    import os
    import time
    import multiprocessing
    
    
    def process_task(info):
        """
        进程任务
        queue: 队列
        """
        data, length = info
        print('测试多进程任务：%s, 当前进程号：%s, 父进程号为: %s' % (data, os.getpid(), os.getppid()))
        res = data ** 3
    
        while res > 0:
            res -= 1
    
        print('多进程任务结束：%s, 当前进程号为: %s' % (data, os.getpid()))
    
        time.sleep(1)
    
        return data ** 3
    
    
    def main():
        start_time = time.time()
        process_pool = multiprocessing.Pool()
        process_data = [100, 200, 300]
    
        print("主进程:%s, 父进程:%s" % (os.getpid(), os.getppid()))
        info = [(v, 5) for v in process_data]
        res = process_pool.map(process_task, info)
    
        process_pool.close()
        process_pool.join()
    
        print(res)
    
        print("user_time: %s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        main()
## 5. apply_async 的回调函数
    pool.apply_async里面不能单独给回调函数传值，回调函数接受的参数就是执行函数的返回值
    回调函数就是是在主进程执行的,且用不了return 
    
    import os
    import time
    import multiprocessing
    
    
    def process_task(info):
        """
        进程任务
        queue: 队列
        """
        data, length = info
        print('测试多进程任务：%s, 当前进程号：%s, 父进程号为: %s' % (data, os.getpid(), os.getppid()))
        res = data ** 3
    
        while res > 0:
            res -= 1
    
        print('多进程任务结束：%s, 当前进程号为: %s' % (data, os.getpid()))
    
        time.sleep(1)
    
        return data ** 3
    
    
    def process_callback(data):
        print("----------- the data is %s" % data)
        print("当前进程号为: %s", os.getpid())
    
    
    def main():
        start_time = time.time()
        process_pool = multiprocessing.Pool()
        process_data = [100, 200, 300]
    
        print("主进程:%s, 父进程:%s" % (os.getpid(), os.getppid()))
        result_list = []
        for v in process_data:
            res = process_pool.apply_async(func=process_task, args=((v, 5),), callback=process_callback)
            result_list.append(res)
    
        process_pool.close()
        process_pool.join()
    
        for v in result_list:
            print(v.get())
    
        print("user_time: %s " % (time.time() - start_time))
    
    
    if __name__ == "__main__":
        main()
    
    
    
# 进程池  ProcessPoolExecutor 
