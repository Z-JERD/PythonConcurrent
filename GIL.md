# GIL锁
## GIL锁的概念

    GIL是一个互斥锁，它防止多个线程同时执行Python字节码。即任何一个时间点只有一个线程处于执行状态
    
    GIL即使在拥有多个CPU核的多线程框架下都只允许一次运行一个线程
    
    无论你启多少个线程，你有多少个cpu, Python在执行的时候同一时间只能有一个线程访问cpu
    
    使用线程性能提高, 是cpu线程切换（多道技术）带来的，而并不是真正的多cpu并行执行
    
    影响：python在多线程中不能实现真正的并行
    
## 并行和并发的概念
    并发：是指一个系统具有处理多个任务的能力（cpu切换，多道技术）
    并行：是指一个系统具有同时处理多个任务的能力（cpu同时处理多个任务
    
## GIL的由来
    
    GIL并不是Python的特性，它是在实现Python解析器(CPython)时所引入的一个概念。CPython的内存管理不是线程安全的
    
    CPython的解析器是使用引用计数来进行内存管理
    
    Python内部对变量或数据对象使用了引用计数器,通过计算引用个数,当个数为0时,变量或者数据对象就被自动释放
    这个引用计数器需要保护,当多个线程同时修改这个值时,可能会导致内存泄漏,  加锁处理，如果有多个锁的话，线程同步时就容易出现死锁
    
    当全局只有一个锁时，所有线程都在竞争一把锁，就不会出现相互等待对方锁的情况，编码的实现也更简单。
    此外只有一把锁时对单线程的影响其实并不是很大。所以 GIL是CPython开发者在早期Python生涯中面对困难问题的一种实用解决方案
   

## GIL 对多线程的影响

    计算密集型(CPU)：需要大量CPU算力来支持
    
        数学计算、图片处理、矩阵运算， 循环， 视频高清解码
        
    I/O密集型： 需要等待IO操作的时间,然后才进行下一步操作
        
        I/O 程序是进行数据传输,例如: 文件读写、网络请求、数据库访问
        
        执行流程：
            
            1.保存当前线程的状态到一个全局变量中
            
            2. 释放GIL
            
            3. ... 执行IO操作 ...
            
            4. 获取GIL
            
            5. 从全局变量中恢复之前的线程状态
            
## 为什么GIL没有被删除：

    GIL 确实可以移除，而且在过去的一段时间内 GIL 曾被移除过。但是有所的这些尝试全都破坏了现存的那些 C 扩展，
    这些 C 扩展严格依赖于 GIL 提供的线程安全环境 同时 删除GIL会使得Python在处理单线程任务方面性能下降

    每个线程执行前都要获取锁，那么有一个线程获取到锁一直占用不释放，怎么办？
    
        1. IO密集型的程序会主动释放锁
        
        2. 对于CPU密集型的程序或IO密集型和CPU混合的程序：
            
            早期的做法是Python会执行100条指令后就强制线程释放GIL让其它线程有可执行的机会
            
            目前：在固定连续使用时间后强迫线程释放GIL 
                
                sys.getswitchinterval()           # 0.005
                
                每个线程执行0.005秒后就释放GIL，用于线程的切换
            
            不足: 
                
                Python的GIL通过不让I/O密集型线程从计算密集型线程获取GIL而使I/O密集型线程陷入瘫痪
                
                Python中内嵌了一种机制, 在固定连续使用时间后强迫线程释放GIL，并且如果没人获取这个GIL，那么同一线程可以继续使用
                
                这个机制面临的问题是大多数计算密集型线程会在别的线程获取GIL之前再次获取GIL
                
            解决：
                Python3.2中解决了这个问题，添加了一种机制来查看其他线程请求GIL的访问数量，
                当数量下降时不允许当前线程在其他线程有机会运行之前重新获取GIL
                 
            
## 进程 线程 选择

    常见的内存管理方案有引用计数和垃圾回收，Python选择了前者，这保证了单线程的执行效率， 同时对编码实现也更加简单
    
    要移除GIL是不容易的，即使成功将GIL去除，对Python的来说是牺牲了单线程的执行效率
    
    Python中GIL对IO密集型程序可以较好的支持多线程并发，然而对CPU密集型程序来说就要使用多进程或使用其它不使用GIL的解析器
    
   
## GIL锁和互斥锁的关系(有了GIL为什么还要加锁)

    GIL锁: 保证同一时刻只有一个线程能使用到cpu
    互斥锁 : 多线程时,保证修改共享数据时有序的修改,不会产生数据修改混乱
    
### 示例：

    假设只有一个进程,这个进程中有两个线程 Thread1,Thread2, 要修改共享的数据date, 并且有互斥锁
    
    执行以下步骤：
    
    1. 多线程运行，假设Thread1获得GIL可以使用cpu，这时Thread1获得 互斥锁lock,Thread1可以改date数据(但并没有开始修改数据)
    
    2. Thread1线程在修改date数据前发生了 i/o操作 或者 ticks计数满100 (注意就是没有运行到修改data数据),这个时候 Thread1 让出了Gil,Gil锁可以被竞争
    
    3.Thread1 和 Thread2 开始竞争 Gil (注意:如果Thread1是因为 i/o 阻塞 让出的Gil Thread2必定拿到Gil,如果
      Thread1是因为ticks计数满100让出Gil 这个时候 Thread1 和 Thread2 公平竞争)
      
    4. 假设 Thread2正好获得了GIL, 运行代码去修改共享数据date,由于Thread1有互斥锁lock，所以Thread2无法更改共享数据date,
       这时Thread2让出Gil锁 , GIL锁再次发生竞争
       
    5. 假设Thread1又抢到GIL，由于其有互斥锁Lock所以其可以继续修改共享数据data,当Thread1修改完数据释放互斥锁lock,
       Thread2在获得GIL与lock后才可对data进行修改
