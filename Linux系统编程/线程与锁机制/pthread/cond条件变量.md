# Cond条件变量

> **条件变量是用来等待而不是用来上锁的。条件变量用来自动阻塞一个线程，直到某特殊情况发生为止**。**通常条件变量和互斥锁同时使用。**

## 初始化Cond锁

```c
#include <pthread.h>
int pthread_cond_init(pthread_cond_t *cond, const pthread_cond_attr *attr);
```

初始化由cond指定的条件变量，这个变量属性由参数attr指定。若为NULL则使用默认设置。

## 销毁

```c
#include <pthread.h>
int pthread_cond_destroy(pthread_cond_t *cond);
```

## 等待锁

```c
#include <pthread.h>
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
```

此函数释放由参数mutex指向的互斥锁，并且使调用线程关于参数cond指向的条件变量阻塞。被阻塞进程可以被pthread_cond_signal、pthread_cond_broadcast或者由fork和传递信号引起的中断唤醒。*在被唤醒之后，pthread_cond_wait会再次将mutex锁定，然后读取资源*。

```c
#include <pthread.h>
int pthread_cond_timewait(pthread_cond_t *cond, pthread_mutex_t *mutex, const struct timespaec *abstime);
```

timewait在经过由abstime指定的时间时不阻塞。

```c
#include <pthread.h>
int pthread_cond_signal(pthread_cond_t *cond);
```

使得关于由参数cond指向的条件变量阻塞的线程退出阻塞状态。

````c
#include <pthread.h>
int pthread_cond_broadcast(pthread_cond_t *cond);
````

使得所有由参数cond指向的条件变量阻塞的线程退出阻塞状态。