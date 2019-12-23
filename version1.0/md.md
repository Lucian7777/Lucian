
# main
- addsig屏蔽SIGPIPE信号
```c
void addsig( int sig, void( handler )(int), bool restart = true )
{
    struct sigaction sa;
    memset( &sa, '\0', sizeof( sa ) );
    sa.sa_handler = handler;
    if( restart )
    {
        sa.sa_flags |= SA_RESTART; //重新调用被该信号终止的系统调用
    }
    sigfillset( &sa.sa_mask );
    assert( sigaction( sig, &sa, NULL ) != -1 );
}
```
一旦给信号设置了SA_RESTART标记，那么当执行某个阻塞系统调用时，收到该信号时，进程不会返回，而是重新执行该系统调用。

# locker.h
实现线程同步机制，分别有信号量、互斥锁和条件变量。
## 信号量
基本API
```c
#include<semaphore.h>
//信号量初始化，pshared为0表示局部信号量，否则该信号量可以在多个进程之间共享，value指定初值
int sem_init(sem_t *sem, int pshared, unsigned int value);
//销毁信号量，释放占用的内核资源
int sem_destroy(sem_t *sem);
// 以原子操作将信号量的值减一。如果信号量为0，sem_wait阻塞，知道有非零值
int sem_wait(sem_t *sem);
// 立即返回，当信号量为0返回-1设置EAGAIN
int sem_trywait(sem_t *sem);
// 以原子方式将信号量加一
int sem_post(sem_t *sem);
```
## 互斥锁(互斥量)
基本API
```c
#include<pthread.h>
//初始化互斥锁，mutexattr指定互斥锁属性
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *mutexattr);
//销毁互斥锁，释放其占用的内核资源
int pthread_mutex_destroy(pthread_mutex_t *mutex);
// 加锁，阻塞
int pthread_mutex_lock(pthread_mutex_t *mutex);
// 立即返回，已被加锁返回EBUSY
int pthread_mutex_trylock(pthread_mutex_t *mutex);
//解锁
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```
互斥锁属性：
```c
#include<pthread.h>
int pthread_mutexattr_init(pthread_mutexattr_t *attr);
int pthread_mutexattr_destroy(pthread_mutexattr_t *attr);
// 获取pshared属性
int pthread_mutexattr_getpshared(const pthread_mutexattr_t *attr, int *pshared);
int pthread_mutexattr_setpshared(pthread_mutexattr_t *attr, int *pshared);
// 获取type属性
int pthread_mutexattr_getptype(const pthread_mutexattr_t *attr, int *type);
int pthread_mutexattr_setptype(pthread_mutexattr_t *attr, int type);
```
pshared: PTHREAD_PROCESS_SHARED 、PTHREAD_PROCESS_PRIVATE<br/>type: PTHREAD_MUTEX_NORMAL PTHREAD_MUTEX_ERRORCHECK PTHREAD_MUTEX_RECURSIVE PTHREAD_MUTEX_DEFAULT
## 条件变量
基本API
```c
#include<pthread.h>
int pthread_cond_init(pthread_cond_t *cond, const pthread_condattr_t *cond_attr);
int pthread_cond_destory(pthread_cond_t *cond);
//以广播的方式唤醒所有等待目标条件变量得线程
int pthread_cond_broadcast(pthread_cond_t *cond);
int pthread_cond_signal(pthread_cond_t *cond);
//q确保互斥锁mutex加锁
int pthread_cond_wait(pthread_cond_t *cond, phtread_mutex_t *mutex);

```
# threadpool
相较于动态创建进程（线程）的优点：
1. 动态创建进程（线程）是比较耗费时间，这将导致响应慢
2. 动态创建进程（线程）通常只用来为一个客户服务，导致产生大量细微进程，进程（线程）直接切换消耗CPU时间
3. 动态创建子进程时当前进程的完整映像。需处理文件描述符，堆内存等系统资源，否则子进程可能复制这些资源，从而使系统可用资源急剧下降，影响服务器性能。

- pthread_detach用于主线程与子线程分离
- pthread_create中worker需是静态函数，该静态函数需要调用run()，所以将类的对象传递给该静态函数，然后静态函数引用这个对象。
- 因为工作队列是共享的，操作时需要加锁。


# httpConnecttion
- init中setsockopt中SO_REUSEADDR使处于TIME_WAIT状态下socket可以被重复使用
- 
