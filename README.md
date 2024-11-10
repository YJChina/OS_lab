# OS_lab1

## 步骤一(直接测试）

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












# 实验二 进程通信与内存管理

## 进程的软中断通信

### **编写实验代码需要考虑的问题：**

**(1)父进程向子进程发送信号时，如何确保子进程已经准备好接收信号？**

使用`signal`系统调用，子进程等待接收父进程发送的16号或17号软中断信号

**(2)如何阻塞住子进程，让子进程等待父进程发来信号？**

使用`waiting()`函数阻塞子进程，如果没有收到来自父进程的kill中断信号，`flag`保持为0，`waiting`阻塞子进程。

### **程序代码**

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <stdlib.h>
#include <signal.h>
int flag = 0;
void inter_handler(int sig)
{
    flag = 1;
    printf("%d stop test\n",sig);
}
void waiting()
{ 
    while (flag != 1);
}
int main()
{
    pid_t pid1 = -1, pid2 = -1;
    
    while (pid1 == -1)
        pid1 = fork();
    if (pid1 > 0)
    {
        while (pid2 == -1)
            pid2 = fork();
        if (pid2 > 0)
        {
            
            signal(SIGINT, inter_handler);
            signal(SIGQUIT, inter_handler);
            alarm(5);
            signal(SIGALRM,inter_handler);
            waiting();
            //sleep(5);

            kill(pid1, 16);     
            kill(pid2, 17);
                
            wait(NULL);
            wait(NULL);
            printf("\nParent process is killed!!\n");
        
        }
        else
        {
            signal(SIGINT,SIG_IGN);
            signal(SIGQUIT,SIG_IGN);
            flag = 0;
            signal(17, inter_handler);
            waiting();
            printf("\nChild process2 is killed by parent!!\n");
            
        }
    }
    else
    {
        signal(SIGINT,SIG_IGN);
        signal(SIGQUIT,SIG_IGN);
        flag = 0;
        signal(16, inter_handler);
        waiting();
        printf("\nChild process1 is killed by parent!!\n");
       
    }
    return 0;
}


```

### 运行结果截图

Ctrl+C软中断

![image-20231114084306648](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231114084306648.png)

Ctrl+\软中断

![image-20231114084512017](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231114084512017.png)

SIGALRM软中断（时间超过5秒父进程发送SIGALRM信号）

![image-20231114084223852](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231114084223852.png)

### 问题

当去掉下面代码的换行符时，父进程的软中断会在子进程`kill`之后`print`

```c
void inter_handler(int sig)
{
    flag = 1;
    printf("%d stop test\n",sig);
}
```



![image-20231114090232343](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231114090232343.png)

## 进程的管道通信

### 程序代码

```c
#include <unistd.h>
#include <signal.h>
#include <stdio.h>
#include <sys/wait.h>

int pid1, pid2; // 定义两个进程变量

int main()
{
    int fd[2];
    char InPipe[4001]; // 定义读缓冲区
    char c1 = '1', c2 = '2';
    pipe(fd); // 创建管道

    while ((pid1 = fork()) == -1)
        ; // 如果进程 1 创建不成功,则空循环

    if (pid1 == 0)
    { // 如果子进程 1 创建成功,pid1 为进程号
        lockf(fd[1], 1, 0); // 锁定管道写入端
        for (int i = 0; i < 2000; ++i)
        {
            write(fd[1], &c1, 1);
        }
        sleep(5);          // 等待读进程读出数据
        lockf(fd[1], 0, 0); // 解除管道的锁定
    }
    else
    {
        while ((pid2 = fork()) == -1)
            ; // 若进程 2 创建不成功,则空循环

        if (pid2 == 0)
        {
            lockf(fd[1], 1, 0);
            for (int i = 0; i < 2000; ++i)
            {
                write(fd[1], &c2, 1);
            }
            sleep(5);
            lockf(fd[1], 0, 0);
        }
        else
        {
            wait(0); // 等待子进程 1 结束
            wait(0); // 等待子进程 2 结束

            close(fd[1]); // 关闭写入端，以便读取数据

            int bytesRead = read(fd[0], InPipe, sizeof(InPipe)); // 从管道中读出数据
            InPipe[bytesRead] = '\0'; // 加字符串结束符

            printf("%s\n", InPipe); // 显示读出的数据

        }
    }
}

```

### 运行结果

#### 有锁情况

![image-20231114111512728](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231114111512728.png)

#### 无锁情况

![image-20231114111342072](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231114111342072.png)

有锁情况保证了写入的原子性，一个进程写入的时候，另一个子进程无法写入。

无锁会导致进程间的竞争，多个进程同时写入，造成输入数据的混乱。

## 内存的分配与回收

### 程序代码

```c
#include <stdio.h>
#include <stdlib.h>
#include <stddef.h>

#define PROCESS_NAME_LEN 32 
#define MIN_SLICE 10 
#define DEFAULT_MEM_SIZE 4096 
#define DEFAULT_MEM_START 0 

/* 内存分配算法 */
#define MA_FF 1
#define MA_BF 2
#define MA_WF 3

struct free_block_type{
    int size;
    int start_addr;
    struct free_block_type *next;
};

struct free_block_type *free_block;

struct allocated_block{
    int pid; 
    int size;
    int start_addr;
    int request_size;
    char process_name[PROCESS_NAME_LEN];
    struct allocated_block *next;
};

struct allocated_block *allocated_block_head = NULL;

int mem_size = DEFAULT_MEM_SIZE; 
int ma_algorithm = MA_FF; 
static int pid = 0; 
int flag = 0; 

struct free_block_type *init_free_block(int mem_size);
void display_menu();
int set_mem_size();
void set_algorithm();
void kill_process();
int new_process();
int allocate_mem(struct allocated_block *ab);
int free_mem(struct allocated_block *ab);
int dispose(struct allocated_block *free_ab);
void display_mem_usage();
void compaction();
int find_free_block(int, struct free_block_type**, struct free_block_type**);
void do_exit();


int main(){
    char choice; 
    pid=0;
    free_block = init_free_block(mem_size); 
    while(1) {
        display_menu(); 
        choice=getchar(); 
        
        switch(choice){
            case '1': set_mem_size(); break; 
            case '2': set_algorithm();flag = 1; break;
            case '3': new_process(); flag = 1; break;
            case '4': kill_process(); flag = 1; break;
            case '5': display_mem_usage(); flag = 1; break; 
            case '0': do_exit(); exit(0); 
            default: break; 
        }
        while (getchar() != '\n')
            continue;
    }
    return 0;
}


struct free_block_type* init_free_block(int mem_size) {
    struct free_block_type *fb;
    fb = (struct free_block_type *)malloc(sizeof(struct free_block_type));
    if(fb == NULL) {
        printf("No mem\n");
        return NULL;
    }
    fb->size = mem_size;
    fb->start_addr = DEFAULT_MEM_START;
    fb->next = NULL;
    return fb;
}


void display_menu() {
    printf("\n");
    printf("1 - Set memory size (default=%d)\n", DEFAULT_MEM_SIZE);
    printf("2 - Select memory allocation algorithm\n");
    printf("3 - New process \n");
    printf("4 - Terminate a process \n");
    printf("5 - Display memory usage \n");
    printf("0 - Exit\n");
}


int set_mem_size(){
    int size;
    if(flag != 0){ 
        printf("Cannot set memory size again\n");
        return 0;
    }
    printf("Total memory size =");
    scanf("%d", &size);
    if(size > 0) {
        mem_size = size;
        free_block->size = mem_size;
    }
    flag = 1; 
    return 1;
}


void set_algorithm(){
    int algorithm;
    printf("Usage:\n");
    while (1) {
        printf("\t1 - First Fit\n");
        printf("\t2 - Best Fit \n");
        printf("\t3 - Worst Fit \n");
        scanf("%d", &algorithm);
        if(algorithm >= 1 && algorithm <= 3) {
            ma_algorithm = algorithm;
            break;
        } else 
            printf("unknown algorithm! Usage: \n");
    }
}

int new_process(){
    struct allocated_block *ab;
    int size; 
    int ret;
    ab = (struct allocated_block *)malloc(sizeof(struct allocated_block));
    if(!ab) exit(-5);
    ab->next = NULL;
    pid++;
    sprintf(ab->process_name, "PROCESS-%02d", pid);
    ab->pid = pid;
    printf("Memory for %s:", ab->process_name);
    scanf("%d", &size);
    if(size > 0) ab->request_size = size;
    ret = allocate_mem(ab); 
    if((ret == 1) &&(allocated_block_head == NULL)){
        allocated_block_head = ab;
        return 1; 
    }
    else if (ret == 1) {
        ab->next = allocated_block_head;
        allocated_block_head = ab;
        return 2; 
    }
    else if(ret == -1){ 
        printf("Allocation fail\n");
        free(ab);
        return -1;
    }
    return 3;
}


int allocate_mem(struct allocated_block *ab){
    struct free_block_type *target, *pre;
    int request_size = ab->request_size;
    int total_size;

    total_size = find_free_block(request_size, &target, &pre);
    
    if (!target && total_size < request_size) 
        return -1;

    if (!target && total_size >= request_size) {
        compaction();
        target = free_block;
    }
    
    ab->start_addr = target->start_addr;
    int remain_size;
    if ((remain_size = target->size - request_size) >= MIN_SLICE) {
        target->size = remain_size;
        target->start_addr += request_size;
        ab->size = request_size;
    } else {
        ab->size = target->size;
        if (target == free_block) {
            free_block = target->next;
            free(target);
        } else {
            pre->next = target->next;
            free(target);
        }
    }
    return 1;
}
int find_free_block(int request_size, struct free_block_type** target, struct free_block_type** pre) {
    struct free_block_type* fbt = free_block, *previous = NULL;
    int total_size = 0;
    *target = NULL;
    *pre = NULL;
    if (ma_algorithm == MA_FF) {
        for (; fbt; previous = fbt, fbt = fbt->next) {
            total_size += fbt->size;
            if (fbt->size >= request_size) {
                *target = fbt;
                *pre = previous;
                return 0;
            }
        }
        return total_size;
    } else if (ma_algorithm == MA_BF) {
        int min_size = mem_size + 1;
        for (; fbt; previous = fbt, fbt = fbt->next) {
            total_size += fbt->size;
            if (fbt->size >= request_size) {
                if (fbt->size < min_size) {
                    *target = fbt;
                    *pre = previous;
                    min_size = fbt->size;
                }
            }
        }
        if (*target) return 0;
        return total_size;
    } else if (ma_algorithm == MA_WF) {
        int max_size = 0;
        for (; fbt; previous = fbt, fbt = fbt->next) {
            total_size += fbt->size;
            if (fbt->size >= request_size) {
                if (fbt->size > max_size) {
                    *target = fbt;
                    *pre = previous;
                    max_size = fbt->size;
                }
            }
        }
        if (*target) return 0;
        return total_size;
    }
}


void compaction() {
    struct free_block_type* fbt, *tmp;
    struct allocated_block* ab;
    int total_size = 0;

    for (fbt = free_block; fbt;) {
        total_size += fbt->size;
        if (fbt != free_block) {
            tmp = fbt;
            fbt = fbt->next;
            free(tmp);
        }
        else
            fbt = fbt->next;
    }

    if (free_block) {
        free_block->start_addr = 0;
        free_block->size = total_size;
        free_block->next = NULL;
    }

    int start_addr = total_size;
    for (ab = allocated_block_head; ab; ab = ab->next) {
        ab->start_addr = start_addr;
        start_addr += ab->size;
    }
}


void kill_process(){
    struct allocated_block *ab;
    int pid;
    printf("Kill Process, pid=");
    scanf("%d", &pid);
    for (ab = allocated_block_head; ab; ab = ab->next) {
        if (ab->pid == pid)
            break;
    }
    if(ab != NULL){
        free_mem(ab); 
        dispose(ab); 
    }
}


int free_mem(struct allocated_block *ab){
    int algorithm = ma_algorithm;
    struct free_block_type *fbt, *pre, *work;
    fbt = (struct free_block_type*) malloc(sizeof(struct free_block_type));
    if(!fbt) return -1;
    fbt->size = ab->size;
    fbt->start_addr = ab->start_addr;
    //sorted by start address
    //insert to head
    if (!free_block || free_block->start_addr > fbt->start_addr) {
        //merge insert and coalesce step
        if (free_block && fbt->size + fbt->start_addr == free_block->start_addr) {
            //coalesce
            free_block->size += fbt->size;
            free_block->start_addr = fbt->start_addr;
        } else {
            //do not coalesce
            fbt->next = free_block;
            free_block = fbt;
        }
        return 1;
    }
    //find position to insert
    for (pre = free_block, work = pre->next; 
        work && work->start_addr < fbt->start_addr; pre = work, work = work->next);
    if (work && pre->start_addr + pre->size == fbt->start_addr && fbt->size + fbt->start_addr == work->start_addr) {
        pre->size += (fbt->size + work->size);
        pre->next = work->next;
        free(fbt);
        free(work);
    } else if (pre->start_addr + pre->size == fbt->start_addr) {
        pre->size += fbt->size;
        free(fbt);
    } else if (work && fbt->start_addr + fbt->size == work->start_addr) {
        work->start_addr = fbt->start_addr;
        work->size += fbt->size;
        free(fbt);
    } else {
        fbt->next = work;
        pre->next = fbt;
    }    

    return 1;
}


int dispose(struct allocated_block *free_ab){
    struct allocated_block *pre, *ab;
    if(free_ab == allocated_block_head) { 
        allocated_block_head = allocated_block_head->next;
        free(free_ab);
        return 1;
    }
    pre = allocated_block_head;
    ab = allocated_block_head->next;
    while(ab != free_ab){ pre = ab; ab = ab->next; }
    pre->next = ab->next;
    free(ab);
    return 2;
}


void display_mem_usage(){
    struct free_block_type *fbt = free_block;
    struct allocated_block *ab = allocated_block_head;
    printf("----------------------------------------------------------\n");
    printf("Free Memory:\n");
    printf("%20s %20s\n", " start_addr", " size");
    while(fbt != NULL) {
        printf("%20d %20d\n", fbt->start_addr, fbt->size);
        fbt = fbt->next;
    }
    
    printf("\nUsed Memory:\n");
    printf("%10s %20s %10s %10s %10s\n", "PID", "ProcessName", "start_addr", " request size", " actual size");
    while(ab != NULL) {
        printf("%10d %20s %10d %10d %10d\n", ab->pid, ab->process_name,
        ab->start_addr, ab->request_size, ab->size);
        ab = ab->next;
    }
    printf("----------------------------------------------------------\n");
}


void do_exit() {
    struct allocated_block *ab = allocated_block_head;
    while (ab != NULL) {
        struct allocated_block *temp = ab;
        ab = ab->next;
        free(temp);
    }
    
    struct free_block_type *fbt = free_block;
    while (fbt != NULL) {
        struct free_block_type *temp = fbt;
        fbt = fbt->next;
        free(temp);
    }
}
```

### 运行结果

**分配内存大小**

![image-20231114164108400](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231114164108400.png)

**选择内存分配算法**

![image-20231114164242594](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231114164242594.png)

**创建进程，分配内存**

![image-20231114164353806](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231114164353806.png)

![image-20231114164420584](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231114164420584.png)

**结束进程，释放内存**

![image-20231114164531720](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231114164531720.png)

**测试紧凑效果**

![image-20231114164911105](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231114164911105.png)

**没有连续的大小为7000的`free block`,执行紧凑函数，合并外碎片，让内存有足够大小来创建进程**

![image-20231114164938599](C:\Users\nightgoodl\AppData\Roaming\Typora\typora-user-images\image-20231114164938599.png)


