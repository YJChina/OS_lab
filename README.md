# OS_lab1

## 步骤一

代码

```c
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>
int main()
{
pid_t pid, pid1;
/* fork a child process */
pid = fork();  
if (pid < 0) { /* error occurred */
fprintf(stderr, "Fork Failed");
return 1;
}
else if (pid == 0) { /* child process */
pid1 = getpid();
printf("child: pid = %d",pid); /* A */
printf("child: pid1 = %d",pid1); /* B */
}
else { /* parent process */
pid1 = getpid();
printf("parent: pid = %d",pid); /* C */
printf("parent: pid1 = %d",pid1); /* D */
wait(NULL);
}
return 0;
}
```

运行结果

![image-20231016144953964](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231016144953964.png)



程序创建一个子进程，父进程和子进程分别输出pid和pid1，pid1为各自进程的PID,子进程的PID为fork()的返回值0,父进程的PID为子进程的PID

## 步骤二

代码

```c
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>
int main()
{
pid_t pid, pid1;
/* fork a child process */
pid = fork();
if (pid < 0) { /* error occurred */
fprintf(stderr, "Fork Failed");
return 1;
}
else if (pid == 0) { /* child process */
pid1 = getpid();
printf("child: pid = %d",pid); /* A */
printf("child: pid1 = %d",pid1); /* B */
}
else { /* parent process */
pid1 = getpid();
printf("parent: pid = %d",pid); /* C */
printf("parent: pid1 = %d",pid1); /* D */
}
return 0;
}
```

运行结果

![image-20231016151013731](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231016151013731.png)

wait()父进程等待子进程结束,避免他成为僵尸进程,去掉wait,父进程将不会等待子进程结束

## 步骤三

运行结果

![image-20231016152408067](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231016152408067.png)

子进程和父进程地址空间相同,子进程将value修改为1,地址空间和父进程相同

## 步骤四

运行结果

![image-20231016152848375](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231016152848375.png)

运行结果与步骤三相同最后在return前给value值加5父进程和子进程分别返回4和6, 地址空间都保持不变

## 步骤五

### system()

代码

```c
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>
int main()
{
pid_t pid, pid1;
int value = 0;
/* fork a child process */
pid = fork();
if (pid < 0) { /* error occurred */
fprintf(stderr, "Fork Failed");
return 1;
}
else if (pid == 0) { /* child process */
pid1 = getpid();
printf("child process1 PID: %d\n",pid1);
system("./system");
printf("child process PID: %d\n",pid1);
}
else { /* parent process */
pid1 = getpid();
printf("parent process PID = %d\n",pid1);}
wait(NULL);return 0;}
```

system_call代码

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t pid = getpid();
    printf("system_call PID：%d\n", pid);
    return 0;
}
```

运行结果

![image-20231016153800560](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231016153800560.png)

父子进程各自输出自己的PID,子进程调用system系统调用,输出当前(system)进程的PID,system调用完成,子进程继续执行,输出子进程的PID(再次)

### exec族函数

代码

```c
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>
int main()
{
pid_t pid, pid1;
int value = 0;
/* fork a child process */
pid = fork();
if (pid < 0) { /* error occurred */
fprintf(stderr, "Fork Failed");
return 1;
}
else if (pid == 0) { /* child process */
pid1 = getpid();
printf("child process1 PID: %d\n",pid1);
execl("./system","system",NULL);
printf("child process PID: %d\n",pid1);
}
else { /* parent process */
pid1 = getpid();
printf("parent process PID = %d\n",pid1);}
wait(NULL);
return 0;}
```

运行结果

![image-20231016155142624](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231016155142624.png)



子进程调用 `execl()` 函数来执行 `system` ，该程序的路径是 `/system`。

`execl()` 执行成功，子进程被 `/system` 替代，后续的代码不会被执行, 只会输出system程序的PID, 即子进程的PID

# 实验1-2

## 步骤一

代码

```c
#include <stdio.h>
#include <pthread.h>

#define NUM_THREADS 2
#define NUM_OPERATIONS 1000000

int shared_variable = 0;

void* thread_function(void* thread_id) {
    long tid = (long)thread_id;
    for (int i = 0; i < NUM_OPERATIONS; i++) {
        shared_variable++;
    }
    printf("thread%ld create success!\n", tid);
    pthread_exit(NULL);
}

int main() {
    pthread_t threads[NUM_THREADS];

    for (long i = 0; i < NUM_THREADS; i++) {
        int result = pthread_create(&threads[i], NULL, thread_function, (void*)i);
        if (result) {
            printf("Error creating thread %ld. Return code: %d\n", i, result);
            return 1;
        }
    }

    for (long i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }

    printf("variable result: %d\n", shared_variable);

    return 0;
}
```

运行结果

![image-20231016191009814](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231016191009814.png)

两个线程分别对shared_variable加 `NUM_OPERATIONS`次,

但是因为缺少同步机制, 所以得到的`variable result`不确定

## 步骤二

```c
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>

#define NUM_THREADS 2
#define NUM_OPERATIONS 1000000

int shared_variable = 0;
sem_t semaphore;

void* thread_function(void* thread_id) {
    long tid = (long)thread_id;
    for (int i = 0; i < NUM_OPERATIONS; i++) {
        sem_wait(&semaphore); // 等待信号量
        shared_variable++;
        sem_post(&semaphore); // 发信号量
    }
    printf("thread%ld create success!\n", tid);
    pthread_exit(NULL);
}

int main() {
    sem_init(&semaphore, 0, 1); // 初始化信号量，初值为1

    pthread_t threads[NUM_THREADS];

    for (long i = 0; i < NUM_THREADS; i++) {
        int result = pthread_create(&threads[i], NULL, thread_function, (void*)i);
        if (result) {
            printf("Error creating thread %ld. Return code: %d\n", i, result);
            return 1;
        }
    }

    for (long i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }

    sem_destroy(&semaphore); // 销毁信号量

    printf("variable result: %d\n", shared_variable);

    return 0;
}
```

运行结果

![image-20231016194232513](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231016194232513.png)

使用信号量来保护共享变量, `sem_init()` 初始化一个二值信号量，初值为1，保证了只有一个线程可以访问共享变量。

在 `thread_function` 中，使用 `sem_wait()` 等待信号量，然后进行共享变量的操作，之后使用 `sem_post()` 发信号量。

这样没有竞态条件的影响下, 共享变量的值为2000000

## 步骤三

代码

```c
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>
#include <sys/syscall.h>
#define NUM_THREADS 2

sem_t semaphore;

void* thread_function(void* thread_id) {
    long tid = (long)thread_id;
    //pthread_t ttid = pthread_self();
    for (int i = 0; i < NUM_OPERATIONS; i++) {
        sem_wait(&semaphore); // 等待信号量
        shared_variable++;
        sem_post(&semaphore); // 发信号量
    }
    printf("thread%ld create success!\n", tid);
    printf("thread%ld tid = %ld,pid = %ld\n",tid,(long int)syscall(SYS_gettid),getpid());
    system("./system");
    printf("thread%ld systemcall return\n",tid);
    pthread_exit(NULL);
}

int main() {
    sem_init(&semaphore, 0, 1); // 初始化信号量，初值为1

    pthread_t threads[NUM_THREADS];

    for (long i = 0; i < NUM_THREADS; i++) {
        int result = pthread_create(&threads[i], NULL, thread_function, (void*)i);
        if (result) {
            printf("Error creating thread %ld. Return code: %d\n", i, result);
            return 1;
        }
    }

    for (long i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }

    sem_destroy(&semaphore); // 销毁信号量
    return 0;
}
```

运行结果

![image-20231016211946209](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231016211946209.png)



代码

```c
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>
#include <sys/syscall.h>
#define NUM_THREADS 2
#define NUM_OPERATIONS 1000000

int shared_variable = 0;
sem_t semaphore;

void* thread_function(void* thread_id) {
    long tid = (long)thread_id;
    //pthread_t ttid = pthread_self();
    for (int i = 0; i < NUM_OPERATIONS; i++) {
        sem_wait(&semaphore); // 等待信号量
        shared_variable++;
        sem_post(&semaphore); // 发信号量
    }
    printf("thread%ld create success!\n", tid);
    printf("thread%ld tid = %ld,pid = %ld\n",tid,(long int)syscall(SYS_gettid),getpid());
    execl("./system","system",NULL);
    printf("thread%ld systemcall return\n",tid);
    pthread_exit(NULL);
}

int main() {
    sem_init(&semaphore, 0, 1); // 初始化信号量，初值为1

    pthread_t threads[NUM_THREADS];

    for (long i = 0; i < NUM_THREADS; i++) {
        int result = pthread_create(&threads[i], NULL, thread_function, (void*)i);
        if (result) {
            printf("Error creating thread %ld. Return code: %d\n", i, result);
            return 1;
        }
    }

    for (long i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }

    sem_destroy(&semaphore); // 销毁信号量
    return 0;
}
```

运行结果

![image-20231016212741617](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231016212741617.png)

在同一进程中, 不同线程的`tid`不同,但`pid`相同,调用`execl()`程序,  来执行 `system` ，该程序的路径是 `/system`。

`execl()` 执行成功，子进程被 `/system` 替代，后续的代码不会被执行, 只会输出system程序的PID, 即子进程的PID。

因此子进程未执行的另一个线程也不会执行而是直接被`system`程序替代,  故只会输出一个线程`tid`和`pid`,另一个线程将不会被创建

# 实验1-3

## 步骤一

补全代码

```c
#include <stdio.h>
#include <pthread.h>

// 定义自旋锁结构体
typedef struct {
    int flag;
} spinlock_t;

// 初始化自旋锁
void spinlock_init(spinlock_t *lock) {
    lock->flag = 0;
}

// 获取自旋锁
void spinlock_lock(spinlock_t *lock) {
    while (__sync_lock_test_and_set(&lock->flag, 1)) {
        // 自旋等待
    }
}

// 释放自旋锁
void spinlock_unlock(spinlock_t *lock) {
    __sync_lock_release(&lock->flag);
}

// 共享变量
int shared_value = 0;

// 线程函数
void *thread_function(void *arg) {
    spinlock_t *lock = (spinlock_t *)arg;
    for (int i = 0; i < 5000; ++i) {
        spinlock_lock(lock);
        shared_value++;
        spinlock_unlock(lock);
    } return NULL;
}

int main() {
    pthread_t thread1, thread2;
    spinlock_t lock;

    // 输出共享变量的值
    printf("Shared value：%d\n", shared_value);

    // 初始化自旋锁
    spinlock_init(&lock);

    // 创建两个线程
    pthread_create(&thread1, NULL, thread_function, &lock);
    pthread_create(&thread2, NULL, thread_function, &lock);

    // 等待线程结束
    pthread_join(thread1, NULL);
    printf("tread1 create success!\n");
    pthread_join(thread2, NULL);
    printf("tread2 create success!\n");

    // 输出共享变量的值
    printf("Shared value：%d\n", shared_value);

    return 0;
}
```



## 步骤二

运行结果

![image-20231016201838191](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231016201838191.png)

程序创建两个线程每个线程都会执行 `thread_function` 函数, 在 `thread_function` 中, 使用自旋锁确保了对共享变量的安全访问。每个线程会进行5000次的累加操作。最终得到共享变量的值为10000

