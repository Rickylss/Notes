# Linux无锁编程

>  https://www.ibm.com/developerworks/cn/linux/l-cn-lockfree/index.html

## Wait-free

​	Wait-free 是指任意线程的任何操作都可以在有限步之内结束，而不用关心其它线程的执行速度。 Wait-free 是基于 per-thread 的，可以认为是 starvation-free 的。非常遗憾的是实际情况并非如此，采用 Wait-free 的程序并不能保证 starvation-free，同时内存消耗也随线程数量而线性增长。目前只有极少数的非阻塞算法实现了这一点。

## Lock-free

​	Lock-Free 是指能够确保执行它的所有线程中至少有一个能够继续往下执行。由于每个线程不是 starvation-free 的，即有些线程可能会被任意地延迟，然而在每一步都至少有一个线程能够往下执行，因此系统作为一个整体是在持续执行的，可以认为是 system-wide 的。所有 Wait-free 的算法都是 Lock-Free 的。

## Obstruction-free

​	Obstruction-free 是指在任何时间点，一个孤立运行线程的每一个操作可以在有限步之内结束。只要没有竞争，线程就可以持续运行。一旦共享数据被修改，Obstruction-free 要求中止已经完成的部分操作，并进行回滚。 所有 Lock-Free 的算法都是 Obstruction-free 的。

