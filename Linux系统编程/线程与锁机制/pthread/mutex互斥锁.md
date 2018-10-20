# mutex互斥锁

## 初始化互斥锁

```c
#include <pthread.h>
pthread_mutex_t fastmutex=PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t recmutex=PTHREAD_RECURSIVE_MUTEX_INITIALIZER_NP;
pthread_mutex_t errchkmutex=PTHREAD_ERRORCHECK_MUTEX_INITIALIZER_NP;
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutex_attr *attr);
```

pthread_mutex_init函数用来初始化一个mutex指向的互斥锁，这个互斥锁的属性由attr指定，attr为NULL时使用默认属性。

## 销毁互斥锁

```c
#include <pthread.h>
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```

## 锁定

```c
#include <pthread.h>
int pthread_mutex_lock(pthread_mutex_t *mutex);
```

```c
#include <pthread.h>
int pthread_mutex_trylock(pthread_mutex_t *mutex);
```

lock与trylock不同之处在于：

1. lock函数加锁时，若互斥锁已被锁定，则尝试加锁的线程被阻塞，直到互斥锁被其他线程释放；
2. trylock函数加锁时，若互斥锁已被锁定，则返回EBUSY错误码，不阻塞。

## 解锁

```c
#include <pthread.h>
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

## 分析

​	Mutex属于sleep-waiting类型的锁。假设线程A和线程B，分别运行在Core0和Core1上，A想通过Pthread_mutex_lock获得一个临界区的锁，而该锁被B持有，则A被阻塞，Core0会进行上下文切换将A置于等待队列中，此时Core0运行其他线程而不必忙等待。这点与spinlock不同，若A使用pthread_spin_lock，则A一直在Core0上进行锁请求，直到获取该锁。

