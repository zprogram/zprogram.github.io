## Python3线程

很大一堆数据需要处理，加速效率使用多线程可以节省运算的时间。

## 多线程基础

- threading.active_count()  目前多少个激活的线程
- threading.enumerate() 枚举当前正在运行的多线程

- threading.current_thread() 当前程序运行的进程是哪个线程

- 基本用法

```
  target  ： 指定 这个线程去哪个函数里面去执行代码

  args：     指定将来调用 函数的时候   传递什么数据过去

                  args参数指定的一定是一个元组类型
 name属性是命名多线程的名字 
 start()是开始执行多线程
 join()是线程结束后再执行后来的语句才会用到
```

示例代码：

```
thread = threading.Thread(target=thread_job,)
thread.start()
```

thread_job函数是输出一个当前正在运行的线程名称，完整代码。

```
import threading

#def main():
#    print(threading.active_count())
#    print(threading.enumerate()) # see the thread list
#    print(threading.current_thread())

def thread_job():
    print('This is a thread of %s' % threading.current_thread())

def main():
    thread = threading.Thread(target=thread_job,)
    thread.start()
if __name__ == '__main__':
    main()
```

## join应用

想要等待所有多线程运行完毕后再显示出`all done\n`，才需要用到join语句。

```
import threading
import time
def thread_job():
    print('T1 start\n')
    for i in range(10):
        time.sleep(0.1)
    print('T1 finish\n')

def T2_job(temp):
    print('T2 start\n')
    print("-----in test2 temp=%s-----"% str(temp))
    print('T2 finish\n')
    
def main():
    added_thread = threading.Thread(target=thread_job, name='T1')
    thread2 = threading.Thread(target=T2_job, args=(g_nums,), name='T2')
    added_thread.start()
    thread2.start()
    thread2.join()
    added_thread.join()

    print('all done\n')

if __name__ == '__main__':
    main()
```

## Queue应用

多线程的结果是没有返回值的，可以把所有线程结果放到长队列中运算然后再取出来。

```

import threading
import time
from queue import Queue

def job(l,q):                # 执行函数
    for i in range(len(l)):
        l[i] = l[i]**2
    q.put(l)                 # 把结果加入到队列中

def multithreading():
    q = Queue()
    threads = []
    data = [[1,2,3],[3,4,5],[4,4,4],[5,5,5]]
    for i in range(4):       # 定义4个线程
        t = threading.Thread(target=job, args=(data[i], q))
        t.start()
        threads.append(t)
    for thread in threads:
        thread.join()        # 结束后再执行接下来的内容
    results = []
    for _ in range(4):       # 从4个线程里面得到结果，按顺序拿出结果
          results.append(q.get())
    print(results)

if __name__ == '__main__':
    multithreading()
```

## GIL

Python不是真正意义上的多线程而是在多个线程中快速切换。

```
import threading
from queue import Queue
import copy
import time

def job(l, q):
    res = sum(l)
    q.put(res)

def multithreading(l):
    q = Queue()
    threads = []
    for i in range(4):
        t = threading.Thread(target=job, args=(copy.copy(l), q), name='T%i' % i)
        t.start()
        threads.append(t)
    [t.join() for t in threads]
    total = 0
    for _ in range(4):
        total += q.get()
    print(total)

def normal(l):
    total = sum(l)
    print(total)

if __name__ == '__main__':
    l = list(range(1000000))
    s_t = time.time()
    normal(l*4)
    print('normal: ',time.time()-s_t)
    s_t = time.time()
    multithreading(l)
    print('multithreading: ', time.time()-s_t)
```

## lock

第一个线程处理完结果，然后第二个线程用的时候。会用到锁，在第一个线程处理时，第二个线程暂时无法操作。等第一个线程执行完毕之后才可以接下来的操作。



A是一个全局变量，当两个线程对同一个变量进行操作。那么就在操作A的时候锁住其他线程。然后当操作完毕之后再让第二个线程操作。

```
import threading

def job1():
    global A, lock     # 开始锁线程
    lock.acquire()
    for i in range(10):
        A += 1
        print('job1', A)
    lock.release()     # 释放锁

def job2():
    global A, lock
    lock.acquire()
    for i in range(10):
        A += 10
        print('job2', A)
    lock.release()

if __name__ == '__main__':
    lock = threading.Lock()
    A = 0
    t1 = threading.Thread(target=job1)
    t2 = threading.Thread(target=job2)
    t1.start()
    t2.start()
    t1.join()
    t2.join()
```



## 参考

https://www.cnblogs.com/17bdw/p/11564848.html