# rw读写锁

> 读写锁适合于对数据结构的读次数比写次数多得多的情况.因为,读模式锁定时可以共享,以写模式锁定时意味着独占,所以读写锁又叫共享-独占锁。

读写锁是自旋锁的一个变种。

## 初始化

```c
#include <pthread.h>
int pthread_rwlock_init(pthread_rwlock_t *rwlock, const pthread_rwlockattr_t *attr);
```

## 销毁

```c
#include <pthread.h>
int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
```

## 加锁

```c
#include <pthread.h>
int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);
int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
```

