# 工作队列

[ linux中断管理之工作队列](https://blog.csdn.net/wyy4045/article/details/104434338?spm=1001.2014.3001.5502)

[Linux内核工作队列workqueue分析](https://blog.csdn.net/u011037593/article/details/114795134)

```c
[include/linux/workqueue.h]

struct work_struct {
    // 低比特部分存放work的标志位，剩余的存放上一次运行worker_pool的ID号或			pool_workqueue的指针
    atomic_long_t data; 
    struct list_head entry;  // 串联工作任务的指针
    work_func_t func;  // 工作任务的回调函数
    ......
};
// 工作任务的回调函数类型
typedef void (*work_func_t)(struct work_struct *work);  

```



# 锁

[ Linux内核源码阅读之自旋锁的作用及其实现](https://blog.csdn.net/u011037593/article/details/79409633?spm=1001.2014.3001.5502)

[ Spinlock semaphore mutex的区别](https://blog.csdn.net/m0_37458552/article/details/105285683)

```c
spinlock //关闭抢占
mutex  //互斥
semaphore //同步
```



# 死锁

1.临界区执行中发生中断+本地中断申请锁：当本地中断也请求锁的时候，就会发生死锁

2.锁+中断函数sleep()/schedule(),发生调度后，可能会切换到另一个线程，当另一个线程请求锁的时候，发生死锁



# 中断

[为什么在中断里不能 sleep ](https://zhuanlan.zhihu.com/p/403276552)

1.中断上半部：在关中断的时候不能sleep,因为时钟中断无法触发，所以在中断上半部分的一定不能sleep

（spin_lock_irq  ）

2.中断下半部分：

(1).x86架构中断有独立的中断上下文，发生调度之后不能恢复。

(2).arm架构中断环境保存在中断发生的线程中，可以恢复到当前状态。

​      尽管可以切换，临界区加的是普通的锁，若在访问临界区时发生中断，中断后,call scheduler新的进程请求锁后可能发生死锁。





