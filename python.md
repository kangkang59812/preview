##### GIL

GIL对I/O绑定多线程程序的性能影响不大,因为线程在等待I/O时共享锁.

GIL对计算型绑定多线程程序有影响,例如: 使用线程处理部分图像的程序,不仅会因锁定而成为单线程,而且还会看到执行时间的增加,这种增加是由锁的获取和释放开销的结果.

从release GIL到acquire GIL之间几乎是没有间隙的。所以当其他在其他核心上的线程被唤醒时，大部分情况下主线程已经又再一次获取到GIL了。这个时候被唤醒执行的线程只能白白的浪费CPU时间，看着另一个线程拿着GIL欢快的执行着

解决办法：

**多处理与多线程：**最流行的方法是使用多处理方法，其中您使用多个进程而不是线程。每个Python进程都有自己的Python解释器和内存空间，因此GIL不会成为问题（可以用管道，fifo通信https://www.cnblogs.com/yssjun/p/11438850.html）

```python
import time

COUNT = 50000000
def countdown(n):
    while n>0:
        n -= 1

if __name__ == '__main__':
    pool = Pool(processes=2) #两个进程
    start = time.time()
    r1 = pool.apply_async(countdown, [COUNT//2]) # 第二个是 可变参数列表，第三个是字典参数
    r2 = pool.apply_async(countdown, [COUNT//2])
    pool.close() #进程池close的时候并未关闭进程池，只是会把状态改为不可再插入元素的状态，完全关闭进程池使用
    pool.join()
    end = time.time()
    print('Time taken in seconds -', end - start)
```

##### 多进程

https://www.cnblogs.com/shaosks/p/10281190.html

#### 常用函数：

```python
import datetime
import random
# 日期，时间
date = datetime.date(year=int(1998),month=int(2),day=int(12)) # 1998-02-12
time = datetime.time(12,12,50) # 12:12:50

# 列表相关
#打乱
alist = [1,2,3,4,5]
random.shuffle(alist) # in-place
#切片超出范围不会报错，单独的索引会
list = ['a','b','c','d','e']
print(list[10:]) # 返回[]不会报错，list[10]会报错
  
  
#排序字典
sorted(d.items(),key=lambda x:x[1])

#集合
A,B 中相同元素： print(set(A)&set(B))
A,B 中不同元素:  print(set(A)^set(B))
  

```

#### Time

time.time()是统计的wall time(即墙上时钟)，也就是系统时钟的时间戳（1970纪元后经过的浮点秒数）。所以两次调用的时间差即为系统经过的总时间。
time.clock()是统计cpu时间 的工具，这在统计某一程序或函数的执行速度最为合适。两次调用time.clock()函数的差值即为程序运行的cpu时间。