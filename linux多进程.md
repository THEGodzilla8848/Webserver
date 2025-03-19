# exec函数族

作用：调用进程内部执行一个可执行文件

现象：调用成功后不会返回，调用失败返回-1。调用成功后会将进程中除内核区以外的替换为目标路径下的文件。

## 函数

### execl(const char* path,const char* arg, ...)



```c++
/*
    #include <unistd.h>

    int execl(const char *path, const char *arg, ...);
        -参数：
            -path:需要执行的文件的路径或名称
            -arg:是执行可执行文件所需的参数列表
                第一个参数一般是可执行程序的名称
                第二个往后是参数列表
                参数最后以NULL结束
        -返回值：
            只有调用失败返回-1
*/
#include <unistd.h>
#include <stdio.h>
int main(){
    //创建一个子进程，在子进程中执行execl函数
    pid_t pid =fork();
    if(pid>0){
        //父进程
        printf("i am parent process,pid :%d\n",getpid());
        sleep(1);
    }
    else if(pid==0){
        //子进程
        execl("hello","hello",NULL);
        printf("i am child process");
    }
    for(int i=0;i<4;i++){
        printf("i = %d,pid = %d\n",i,getpid());
    }
    return 0;

}
```



### execlp(const char *file, const char *arg, ...);

```c++
/*
    #include <unistd.h>

    int execlp(const char *file, const char *arg, ...);
        -会到环境变量中查找指定的可执行文件，找到就执行，找不到就执行不成功
        -参数：
            -file:需要执行的文件的名称
            -arg:是执行可执行文件所需的参数列表
                第一个参数一般是可执行程序的名称
                第二个往后是参数列表
                参数最后以NULL结束
        -返回值：
            只有调用失败返回-1
*/
#include <unistd.h>
#include <stdio.h>
int main(){
    //创建一个子进程，在子进程中执行execl函数
    pid_t pid =fork();
    if(pid>0){
        //父进程
        printf("i am parent process,pid :%d\n",getpid());
        sleep(1);
    }
    else if(pid==0){
        //子进程
        execlp("ps","ps","aux",NULL);
        perror("execlp");
        printf("i am child process\n");
    }
    for(int i=0;i<4;i++){
        printf("i = %d,pid = %d\n",i,getpid());
    }
    return 0;

}
```





# 进程控制

## 进程退出

![](C:\Users\11048\Desktop\Linux多进程\{79835C44-2B64-49ad-89C6-BBAF7DF3896D}.png)

```c
/*
    #include<stdlib.h>
    void exit(int status);

    #include<unistd.h>
    void _exit(int status);

    status参数：是进程退出的一个状态信息，父进程回收子进程资源的时候可以获取到
  
*/
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
int main(){
    printf("hello\n");
    printf("world");

    //exit(0);
    _exit(0);
    return 0;
}
```



## 孤儿进程

- 父进程运行结束，但子进程还在运行（未运行结束），这样的子进程就称为`孤儿进程（Orphan Process）`
- 每当出现一个孤儿进程的时候，内核就把孤儿进程的父进程设置为 init ，而 init 进程会循环地 wait() 它的已经退出的子进程。
- 孤儿进程并不会有什么危害

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main() 
{

    // 创建子进程
    pid_t pid = fork();

    // 判断是父进程还是子进程
    if(pid > 0) {
       
        // 如果大于0，返回的是创建的子进程的进程号，当前是父进程
        printf("i am parent process, pid : %d, ppid : %d\n", getpid(), getppid());

      
    } else if(pid == 0) {
        // 当前是子进程
        printf("i am child process, pid : %d, ppid : %d\n", getpid(),getppid());
       
       
    }

    // for循环
    for(int i = 0; i < 3; i++) {
        printf("i : %d , pid : %d\n", i , getpid());
        
    }

    return 0;
}
```



## 僵尸进程

- 每个进程结束之后，都会释放自己地址空间中的用户区数据，内核区的 PCB 没有办法自己释放掉，需要父进程去释放
- 进程终止时，父进程尚未回收，子进程残留资源（PCB）存放于内核中，变成`僵尸（Zombie）进程`
- **僵尸进程不能被 `kill -9` 杀死**，这样就会导致一个问题，如果父进程不调用 `wait()` 或 `waitpid()` 的话，那么保留的那段信息就不会释放，其进程号就会一直被占用，但是系统所能使用的进程号是有限的，如果大量的产生僵尸进程，将因为没有可用的进程号而导致系统不能产生新的进程，此即为僵尸进程的危害，应当避免

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main() 
{

    // 创建子进程
    pid_t pid = fork();

    // 判断是父进程还是子进程
    if(pid > 0) {
       while(1){
        // 如果大于0，返回的是创建的子进程的进程号，当前是父进程
        printf("i am parent process, pid : %d, ppid : %d\n", getpid(), getppid());
        sleep(1);
       }
    } else if(pid == 0) {
        
        // 当前是子进程
        printf("i am child process, pid : %d, ppid : %d\n", getpid(),getppid());
       
       
    }

    // for循环
    for(int i = 0; i < 3; i++) {
        printf("i : %d , pid : %d\n", i , getpid());
        
    }

    return 0;
}
```



# 进程回收

- 父进程可以通过$wait()$或$waitpid()$得到它的退出状态同时彻底清除掉这个进程
- $wait()$和$waitpid()$函数的功能一样，区别在于
  - `wait()` 函数会阻塞
  - `waitpid()` 可以设置是否阻塞，`waitpid()` 还可以指定等待哪个子进程结束
- 注意：**一次`wait`或`waitpid`调用只能清理一个子进程，清理多个子进程应使用循环**



### 退出信息相关宏函数

- `WIFEXITED(status)`：非0，进程正常退出
- `WEXITSTATUS(status)`：如果上宏为真，获取进程退出的状态（exit的参数）
- `WIFSIGNALED(status)`：非0，进程异常终止
- `WTERMSIG(status)`：如果上宏为真，获取使进程终止的信号编号
- `WIFSTOPPED(status)`：非0，进程处于暂停状态
- `WSTOPSIG(status)`：如果上宏为真，获取使进程暂停的信号的编号
- `WIFCONTINUED(status)`：非0，进程暂停后已经继续运行

### wait()

- 可通过`man 2 wait`查看帮助
- `pid_t wait(int *wstatus);`
  - 功能：等待任意一个子进程结束，如果任意一个子进程结束了，此函数会回收子进程的资源
  - 参数
    - `int *wstatus`：进程退出时的状态信息，传入的是一个int类型的地址，传出参数。
  - 返回值
    - 成功：返回被回收的子进程的id
    - 失败：-1 (所有的子进程都结束，调用函数失败)
- 其他说明
  - 调用wait函数的进程会被挂起（阻塞），直到它的一个子进程退出或者收到一个不能被忽略的信号时才被唤醒（相当于继续往下执行）
  - 如果没有子进程了，函数立刻返回，返回-1；如果子进程都已经结束了，也会立即返回，返回-1

```c
int main() 
{
    // 有一个父进程，创建5个子进程（兄弟）
    pid_t pid;

    // 创建5个子进程
    for(int i = 0; i < 5; i++) {
        pid = fork();
        // 避免嵌套重复生成子进程
        if(pid == 0) {
            break;
        }
    }

    if(pid > 0) {
        // 父进程
        while(1) {
            printf("parent, pid = %d\n", getpid());
            // int ret = wait(NULL);
            int st;
            int ret = wait(&st);

            if(ret == -1) {
                break;
            }

            if(WIFEXITED(st)) {
                // 是不是正常退出
                printf("退出的状态码：%d\n", WEXITSTATUS(st));
            }
            if(WIFSIGNALED(st)) {
                // 是不是异常终止
                printf("被哪个信号干掉了：%d\n", WTERMSIG(st));
            }

            printf("child die, pid = %d\n", ret);

            sleep(1);
        }

    } else if (pid == 0){
        // 子进程
         while(1) {
            printf("child, pid = %d\n",getpid());    
            sleep(1);       
         }

        exit(0);
    }

    return 0; // exit(0)
}
```



### waitpid()

- 可通过`man 2 wait`查看帮助
- `pid_t waitpid(pid_t pid, int *wstatus, int options);`
  - 功能：回收指定进程号的子进程，可以设置是否阻塞
  - 参数
    - pid:
      - `pid > 0` : 回收某个子进程的pid
      - `pid = 0` : 回收当前进程组的所有子进程
      - `pid = -1` : 回收所有的子进程，相当于 wait() （最常用）
      - `pid < -1` : 某个进程组的组id的绝对值，回收指定进程组中的子进程
    - options：设置阻塞或者非阻塞
      - 0 : 阻塞
      - WNOHANG : 非阻塞
    - 返回值
      - &gt; 0 : 返回子进程的id
      - 0 : options=WNOHANG, 表示还有子进程活着
      - -1 ：错误，或者没有子进程了

```c
/*
    #include <sys/types.h>
    #include <sys/wait.h>


    pid_t waitpid(pid_t pid, int *wstatus, int options);
        功能：回收指定进程号的子进程，可以设置是否阻塞
        参数：
            pid:
                pid>0:某个子进程PID
                pid=0:回收当前进程组的所有子进程
                pid=-1:回收所有子进程
                pid<-1:回收某个进程组的组id的绝对值
            options:设置阻塞或者非阻塞
            返回值：
                >0:返回子进程id;
                =0:表示还有子进程；
                =-1:表示错误或者没有子进程

*/

#include <sys/types.h>
#include <sys/wait.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
int main() 
{
    // 有一个父进程，创建5个子进程（兄弟）
    pid_t pid;

    // 创建5个子进程
    for(int i = 0; i < 5; i++) {
        pid = fork();
        // 避免嵌套重复生成子进程
        if(pid == 0) {
            break;
        }
    }

    if(pid > 0) {
        // 父进程
        while(1) {
            printf("parent, pid = %d\n", getpid());
            // int ret = wait(NULL);
            sleep(1);
            int st;
            //int ret = waitpid(-1,&st,0);
            int ret = waitpid(-1,&st,WNOHANG);
            if(ret == -1) {
                break;
            }
            else if(ret == 0){
                continue;
            }
            else if(ret>0){
            if(WIFEXITED(st)) {
                // 是不是正常退出
                printf("退出的状态码：%d\n", WEXITSTATUS(st));
            }
            if(WIFSIGNALED(st)) {
                // 是不是异常终止
                printf("被哪个信号干掉了：%d\n", WTERMSIG(st));
            }

            printf("child die, pid = %d\n", ret);
            }
            
        }

    } else if (pid == 0){
        // 子进程
         while(1) {
            printf("child, pid = %d\n",getpid());    
            sleep(1);       
         }

        exit(0);
    }

    return 0; // exit(0)
}
```





# 进程间通信之管道及内存映射

## 进程间通讯概念

- 进程是一个独立的资源分配单元，不同进程（这里所说的进程通常指的是用户进程）之间的资源是独立的，没有关联，不能在一个进程中直接访问另一个进程的资源
- 但是，进程不是孤立的，不同的进程需要进行信息的交互和状态的传递等，因此需要`进程间通信( IPC：Inter Processes Communication)`
- 进程间通信的目的
  - 数据传输：一个进程需要将它的数据发送给另一个进程
  - 通知事件：一个进程需要向另一个或一组进程发送消息，通知它（它们）发生了某种事件（如进程终止时要通知父进程）
  - 资源共享：多个进程之间共享同样的资源。为了做到这一点，需要内核提供互斥和同步机制
  - 进程控制：有些进程希望完全控制另一个进程的执行（如 Debug 进程），此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变



## Linux 进程间通信的方式

![](C:\Users\11048\Desktop\Linux多进程\{11EED3B8-76EE-4ecc-A2B8-DDBE7F352FBE}.png)



## 管道

### 管道特点

- 管道其实是一个在**内核内存中维护的缓冲器**，这个缓冲器的存储能力是有限的，不同的操作系统大小不一定相同
- 管道拥有文件的特质：读操作、写操作
  - **匿名管道**没有文件实体
  - **有名管道**有文件实体，但不存储数据。可以按照操作文件的方式对管道进行操作
- **一个管道是一个字节流**，使用管道时不存在消息或者消息边界的概念，从管道读取数据的进程可以读取任意大小的数据块，而不管写入进程写入管道的数据块的大小是多少
- 通过管道传递的数据是顺序的，从管道中读取出来的字节的顺序和它们被写入管道的顺序是完全一样的
- 在管道中的数据的传递方向是单向的，一端用于写入，一端用于读取，管道是**半双工**的
- 从管道读数据是一次性操作，数据一旦被读走，它就从管道中被抛弃，释放空间以便写更多的数据，**在管道中无法使用 lseek() 来随机的访问数据**
- `匿名管道`只能在**具有公共祖先的进程（父进程与子进程，或者两个兄弟进程，具有亲缘关系）之间使用**



### 匿名管道

#### 概念及使用

- `管道`也叫`无名（匿名）管道`，它是是 UNIX 系统 IPC（进程间通信）的最古老形式，所有的 UNIX 系统都支持这种通信机制
- 统计一个目录中文件的数目命令：`ls | wc –l`，为了执行该命令，shell 创建了两个进程来分别执行 ls 和 wc

- 查看帮助：`man 2 pipe`
- 创建匿名管道：`int pipe(int pipefd[2]);`
- 查看管道缓冲大小命令：`ulimit –a `（共8个，每个521byte，即4k）
- 查看管道缓冲大小函数：`long fpathconf(int fd, int name);`

#### 创建匿名管道

- `int pipe(int pipefd[2])`

  - 功能：创建一个匿名管道，用来进程间通信。

  - 参数：int pipefd[2]

    这个数组是一个传出参数。

    - `pipefd[0]` 对应的是管道的读端
    - `pipefd[1]` 对应的是管道的写端

  - 返回值：成功 0，失败 -1

- 注意

  - 管道默认是阻塞的：如果管道中没有数据，read阻塞，如果管道满了，write阻塞
  - 匿名管道只能用于具有关系的进程之间的通信（父子进程，兄弟进程）

```c
/*
    #include <unistd.h>
    int pipe(int pipefd[2]);
        功能：创建一个匿名管道
        参数：int pipefd[2]这个数组是一个传出参数
            pipefd[0]对应读端，pipefd[1]对应写端
        返回值：
            成功返回0，失败返回-1；
    管道默认是阻塞的
    注意：匿名管道只能用于有关系进程间的通信

*/
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
int main(){
    //在fork之前创建管道
    int pipefd[2];
    int ret=pipe(pipefd);
    if(ret==-1){
        perror("pipe");
        exit(0);
    }

    //创建子进程
    pid_t pid=fork();
    if(pid>0){
        //父进程
        //read pipe
        printf("i am parent, pid:%d\n",getpid());
        char buff[1024]={0};
        while(1){
            int len=read(pipefd[0],buff,sizeof(buff));
            printf("parent recv:%s, pid:%d\n",buff,getpid());

            char* str="hello i am parent";
            write(pipefd[1],str,strlen(str));
            sleep(1);
        }
    }
    else if(pid==0){
        //child
        //write pipe
        printf("i am child, pid:%d\n",getpid());
        char buff[1024]={0};
        while(1){
            char* str="hello i am child";
            write(pipefd[1],str,strlen(str));
            sleep(1);

            int len=read(pipefd[0],buff,sizeof(buff));
            printf("child recv:%s, pid:%d\n",buff,getpid());
        }

    }
    return 0;
}
```

#### 实例：自建管道实现shell命令(`ps aux`)

```c
/*
    实现 ps aux | grep xxx 父子进程间通信

    子进程：ps aux，结束后，将数据发送给子进程
    父进程:获取到数据，过滤

    pipe()
    execlp()
    子进程将标准输出stdout_fileno重定向到管道的写端。dup2
*/
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <wait.h>
int main(){
    //创建管道
    int fd[2];
    int ret=pipe(fd);
    if(ret==-1){
        perror("pipe");
        exit(0);
    }
    //child process
    pid_t pid=fork();
    if(pid>0){
        //parent
        close(fd[1]);
        char buf[1024]={0};
        int len=-1;
        while((len=read(fd[0],buf,sizeof(buf)-1))>0){
            printf("%s",buf);
            memset(buf,0,1024);
        }
        wait(NULL);
    }
    else if(pid==0){
        //child
        close(fd[0]);
        dup2(fd[1],STDOUT_FILENO);
        execlp("ps","ps","aux",NULL);
    }
    return 0;
}
```

#### 设置管道非阻塞

```
int flags = fcntl(fd[0], F_GETFL);  // 获取原来的flag
flags |= O_NONBLOCK;            // 修改flag的值
fcntl(fd[0], F_SETFL, flags);   // 设置新的flag
```

#### 读写特点总结

- 读管道
  - 管道中有数据，read返回实际读到的字节数
  - 管道中无数据
    - 写端被全部关闭，read返回0（相当于读到文件的末尾）
    - 写端没有完全关闭，read阻塞等待
- 写管道
  - 管道读端全部被关闭，进程异常终止（进程收到`SIGPIPE`信号）
  - 管道读端没有全部关闭：
    - 管道已满，write阻塞
    - 管道没有满，write将数据写入，并返回实际写入的字节数



#### 创建有名管道

- shell命令创建：`mkfifo 文件名`，可通过`man 1 mkfifo`查看帮助
- 函数创建：`int mkfifo(const char *pathname, mode_t mode);`，可通过`man 3 mkfifo`查看帮助

```c
/*
    创建fifo文件
    1、通过命令：mkfifo 名字
    2、通过函数：int mkfifo(const char *pathname, mode_t mode);

    #include <sys/types.h>
    #include <sys/stat.h>
    int mkfifo(const char *pathname, mode_t mode);
        参数：
            -pathname:管道名称的路径
            -made:文件的权限 和 open的mode是一样的
        返回值：
            成功返回0，失败返回-1；


*/

#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(){
    int ret1=access("fifo1",F_OK);
    if(ret1==-1){
        printf("管道不存在，创建管道\n");
    
        int ret=mkfifo("fifo1",0664);
        if(ret==-1){
            perror("mkfifo");
            exit(0);
        }
    }
    return 0;
}
```

#### 实例：两进程通过有名管道通信（单一发送）

- 写端

```c
//从管道中写数据


#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
int main(){
    int ret1=access("test",F_OK);
    if(ret1==-1){
        printf("管道不存在，创建管道\n");
    
        int ret=mkfifo("test",0664);
        if(ret==-1){
            perror("mkfifo");
            exit(0);
        }
    }
    int fd = open("test",O_WRONLY);
    if(fd==-1){
        perror("open");
        exit(0);
    }

    for(int i=0;i<100;i++){
        char buf[1024];
        sprintf(buf,"hello, %d\n",i);
        printf("write data : %s\n",buf);
        write(fd,buf,strlen(buf));
        sleep(1);
    }
    close(fd);
    return 0;
}
```



- 读端

```c
//从管道中读数据

#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
int main(){
    //1、打开管道
    int fd = open("test",O_RDONLY);
    if(fd==-1){
        perror("open");
        exit(0);
    }
    //读数据
    while(1){
        char buf[1024]={0};
        int len = read(fd,buf,sizeof(buf));
        if(len==0){
            printf("写端关闭\n");
            break;
        }
        printf("recv buf:%s\n",buf);
    }
    close(fd);
    return 0;
}
```



#### 实例：简易版聊天功能（连续发送）

- 功能：两个进程相互发送数据及接收数据，能够连续发送及接收
- 思路
  - 由于两个进程并没有亲缘关系，所以只能使用有名管道实现
  - 需要两个管道
    - 一个管道用于进程A的写与进程B的读
    - 一个管道用于进程B的写与进程A的读
  - 需要父子进程，实现连续发送及接收
    - 父进程负责写入数据到管道
    - 子进程负责从管道读取数据

- 进程A

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>

int main(){
    int ret=access("fifo1",F_OK);
    if(ret==-1){
        printf("管道不存在\n");
        int ret1=mkfifo("fifo1",0664);
        if(ret1==-1){
            perror("mkfifo");
            exit(0);
        }

    }
    int ret2=access("fifo2",F_OK);
    if(ret2==-1){
        printf("管道不存在\n");
        int ret3=mkfifo("fifo2",0664);
        if(ret3==-1){
            perror("mkfifo");
            exit(0);
        }

    }
    int fd=open("fifo1",O_WRONLY);
    if(fd==-1){
        perror("open");
        exit(0);
    }
    printf("打开fifo1成功\n");

    int fd1=open("fifo2",O_RDONLY);
    if(fd1==-1){
        perror("open");
        exit(0);
    }
    printf("打开fifo2成功\n");

    char buf[128]={0};
    while(1){
        memset(buf,0,sizeof(buf));
        fgets(buf,128,stdin);
        int ret=write(fd,buf,strlen(buf));
        if(ret==-1){
            perror("write");
            exit(0);
        }
        memset(buf,0,sizeof(buf));
        ret = read(fd1,buf,sizeof(buf));
        if(ret<=0){
            perror("read");
            break;
        }
        printf("buf: %s\n",buf);
    }
    close(fd);
    close(fd1);
    return 0;
}
```



- 进程B

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>

int main(){
    int ret=access("fifo1",F_OK);
    if(ret==-1){
        printf("管道不存在\n");
        int ret1=mkfifo("fifo1",0664);
        if(ret1==-1){
            perror("mkfifo");
            exit(0);
        }

    }
    int ret2=access("fifo2",F_OK);
    if(ret2==-1){
        printf("管道不存在\n");
        int ret3=mkfifo("fifo2",0664);
        if(ret3==-1){
            perror("mkfifo");
            exit(0);
        }

    }
    int fd=open("fifo1",O_RDONLY);
    if(fd==-1){
        perror("open");
        exit(0);
    }
    printf("打开fifo1成功\n");

    int fd1=open("fifo2",O_WRONLY);
    if(fd1==-1){
        perror("open");
        exit(0);
    }
    printf("打开fifo2成功\n");

    char buf[128]={0};
    while(1){
    
        memset(buf,0,sizeof(buf));
        ret = read(fd,buf,sizeof(buf));
        if(ret<=0){
            perror("read");
            break;
        }
        printf("buf: %s\n",buf);

        memset(buf,0,sizeof(buf));
        fgets(buf,128,stdin);
        int ret=write(fd1,buf,strlen(buf));
        if(ret==-1){
            perror("write");
            exit(0);
        }
    }
    close(fd);
    close(fd1);
    return 0;
}
```

## 内存映射

### 概念

- `内存映射（Memory-mapped I/O）`是将**磁盘文件的数据映射到内存**，用户通过修改内存就能修改磁盘文件

- 内存映射相关系统调用，使用`man 2 mmap`查看帮助
  - `void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);`
    - 功能：将一个文件或者设备的数据映射到内存中
    - 参数
      - `addr`：设置为 NULL时, 由内核指定（推荐做法）
      - `length` : 要映射的数据的长度，这个值**不能为0。建议使用文件的长度**，获取文件的长度：`stat `，`lseek`
      - `prot` : 对申请的内存映射区的操作权限
        - `PROT_EXEC` ：可执行的权限
        - `PROT_READ` ：读权限
        - `PROT_WRITE` ：写权限
        - `PROT_NONE` ：没有权限
      - `flags`
        - `MAP_SHARED` : 映射区的数据会自动和磁盘文件进行同步，进程间通信，必须要设置这个选项
        - `MAP_PRIVATE` ：不同步，内存映射区的数据改变了，对原来的文件不会修改，会重新创建一个新的文件。（`copy on write`）
      - `fd`: 需要映射的那个文件的文件描述符，通过`open`得到，`open`的是一个磁盘文件
      - `offset`：偏移量，一般进行特殊指定（指定为0即可），如果使用必须指定的是 `4k` 的整数倍，0表示不偏移
    - 返回值：返回创建的内存的首地址。失败返回`MAP_FAILED(即(void *) -1)`
  - `int munmap(void *addr, size_t length);`
    -  功能：释放内存映射
    -  参数
       - `addr` : 要释放的内存的首地址
       - `length` : 要释放的内存的大小，要和`mmap`函数中的length参数的值一样

### 进程间通信种类

- 有关系的进程（父子进程）
  - 还没有子进程的时候，通过唯一的父进程，先创建内存映射区
  - 有了内存映射区以后，创建子进程
  - 父子进程共享创建的内存映射区
- 没有关系的进程间通信
  - 准备一个大小不是0的磁盘文件
  - 进程1 通过磁盘文件创建内存映射区，得到一个操作这块内存的指针
  - 进程2 通过磁盘文件创建内存映射区，得到一个操作这块内存的指针
  - 使用内存映射区通信

### 实例：父子进程通信

- 思路
  1. 打开指定文件并获取文件长度
  2. 创建内存映射区
  3. 父子进程功能，父进程负责收数据，子进程负责发数据
  4. 回收资源

- code

```c
/*
    #include <sys/mman.h>
    void *mmap(void *addr, size_t length, int prot, int flags,int fd, off_t offset);
        -功能：将一个文件或设备映射到内存中
        -参数：
            -void *addr:NULL,由内核指定
            -length:映射数据的长度，不能为0
                获取文件长度：stat,lseek
            -prot:对申请的内存映射区的操作权限
                PROT_EXEC  可执行权限

                PROT_READ  读权限

                PROT_WRITE 写权限

                PROT_NONE  无权限
            -flag:
                - MAP_SHARED:映射区数据会自动和磁盘文件同步
                - MAP_PRIVATE：不同步，映射区数据修改，原来的文件不会修改
            -fd :需映射的文件描述符
                注意：open权限不能于prot权限有冲突
            -offset：偏移量，必须为4k整数倍
        -返回值：创建的内存的首地址
                失败返回一个失败的宏
    int munmap(void *addr, size_t length);
        -功能：释放内存映射
        -参数：
            -*addr：释放内存的首地址
            -length:释放的长度

*/


#include <stdio.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(){
    int fd=open("test.txt",O_RDWR);
    int size=lseek(fd,0,SEEK_END);
    void *ptr=mmap(NULL,size,PROT_READ|PROT_WRITE,MAP_SHARED,fd,0);
    if(ptr == MAP_FAILED){
        perror("mmap");
        exit(0);
    }
    pid_t pid=fork();
    if(pid>0){
        wait(NULL);
        strcpy((char*)ptr,"nihao ,son!!!");
    } 
    else if(pid==0){
        char buf[64];
        strcpy(buf,(char*)ptr);
        printf("read data : %s\n",buf);
    }
    munmap(ptr,size);
    
    return 0;
}
```

### 实例：文件拷贝

- 思路

  1. 需要两个文件，一个是有内容的文件（待拷贝文件），一个是空文件
  2. 由于有两个文件，需要两个内存映射区
  3. 然后将文件A的内存映射区内容拷贝给文件B的内存映射区
  4. 回收资源

- code

```c
//使用内存映射实现文件拷贝功能
/*
    思路：
        1、对原有文件创建内存映射
        2、创建一个新文件
        3、把新文件的数据映射到内存中
        4、通过内存拷贝将第一个文件的内存拷贝到新的文件
        5、释放资源
*/
#include <stdio.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
int main(){
    int fd=open("test.txt",O_RDWR);
    if(fd==-1){
        perror("open");
        exit(0);
    }
    int len=lseek(fd,0,SEEK_END);
    int fd1=open("cpy.txt",O_RDWR | O_CREAT, 0664);
    if(fd1==-1){
        perror("open");
        exit(0);
    }
    truncate("cpy.txt",len);
    write(fd1," ",1);
    void *ptr=mmap(NULL,len,PROT_READ | PROT_WRITE,MAP_SHARED,fd,0);
    void *ptr1=mmap(NULL,len,PROT_READ | PROT_WRITE,MAP_SHARED,fd1,0);
    if(ptr==MAP_FAILED){
        perror("mmap");
        exit(0);
    }
     if(ptr1==MAP_FAILED){
        perror("mmap");
        exit(0);
    }
    memcpy(ptr1,ptr,len);
    munmap(ptr1,len);
    munmap(ptr,len);

    close(fd1);
    close(fd);
    return 0;
}
```

- output

​	![](C:\Users\11048\Desktop\Linux多进程\{89D3E347-560C-4757-91EA-56FF884E62E1}.png)

![](C:\Users\11048\Desktop\Linux多进程\{0541663C-3F47-4138-9E76-BA4BCEB27183}.png)

### 实例：匿名内存映射

- 思路

  1. 匿名内存映射不存在文件实体，那么只能通过父子进程实现
  2. 父子进程操作同一块区域，重点在于内存映射区在创建时新增flags参数`MAP_ANONYMOUS`
  3. 父进程读，子进程写
- code

```c
/*
    匿名映射：不需要文件进程的一个内存映射
*/

#include <stdio.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(){
    int len=4096;
    void *ptr=mmap(NULL,len,PROT_READ | PROT_WRITE,MAP_SHARED | MAP_ANONYMOUS,-1,0);
    if(ptr==MAP_FAILED){
        perror("mmap");
        exit(0);

    }
    //父子进程通信
    pid_t pid=fork();
    if(pid>0){
        strcpy((char*)ptr,"hello world");
        wait(NULL);
    }
    else if(pid==0){
        sleep(1);
        printf("%s\n",(char*)ptr);
    }
    munmap(ptr,len);
    
    return 0;
}

```

- output

![](C:\Users\11048\Desktop\Linux多进程\{C6E6E575-710B-4f70-8B19-32C4B92BBD73}.png)

# 进程间通信之信号

## 基本概念

- 信号是 Linux 进程间通信的最古老的方式之一，是事件发生时对进程的通知机制，有时也称之为软件中断，它是在软件层次上对中断机制的一种模拟，是一种异步通信的方式。信号可以导致一个正在运行的进程被另一个正在运行的异步进程中断，转而处理某一个突发事件

- 发往进程的诸多信号，通常都是源于内核。引发内核为进程产生信号的各类事件如下

  - 对于前台进程，用户可以通过输入特殊的终端字符来给它发送信号。比如输入 `Ctrl+C` 通常会给进程发送一个中断信号
  - 硬件发生异常，即硬件检测到一个错误条件并通知内核，随即再由内核发送相应信号给相关进程。比如执行一条异常的机器语言指令，诸如被 0 除，或者引用了无法访问的内存区域
  - 系统状态变化，比如 alarm 定时器到期将引起 `SIGALRM` 信号，进程执行的 CPU 时间超限，或者该进程的某个子进程退出
  - 运行 kill 命令或调用 kill 函数

- 使用信号的两个主要目的是

  - 让进程知道已经发生了一个特定的事情
  - 强迫进程执行它自己代码中的信号处理程序

- 信号的特点

  - 简单
  - 不能携带大量信息
  - 满足某个特定条件才发送
  - 优先级比较高

- 查看系统定义的信号列表：`kill –l`，前 31 个信号为常规信号，其余为实时信号

  ![](C:\Users\11048\Desktop\Linux多进程\{2E7924EB-34BB-453b-AA18-7FC832068DB2}.png)

## 信号一览表及特点

- 可通过`man 7 signal`查看帮助
- 信号的 5 中默认处理动作
  - `Term`：终止进程
  - `Ign`：当前进程忽略掉这个信号
  - `Core`：终止进程，并生成一个Core文件
  - `Stop`：暂停当前进程
  - `Cont`：继续执行当前被暂停的进程
- 信号的几种状态：`产生`、`未决`、`递达`
- `SIGKILL` 和 `SIGSTOP` 信号不能被捕捉、阻塞或者忽略，只能执行默认动作

## 信号相关的函数

### core文件生成及调试

- 当进程异常终止时，会生成`core`文件（需要进行相应设置），可以通过`gdb`调试查看错误，调试以下程序

- code

  ```c
  #include <stdio.h>
  #include <string.h>
  
  int main()
  {
      char* buf;
      strcpy(buf, "core test");
      return 0;
  }
  ```

- 生成调试`core`文件需要做以下几步

  1. 使用`ulimit -a`查看资源上限

  2. 修改`core size`：`ulimit -c core-size`

  3. 在编译运行程序时加上`-g`选项使得能够被`gdb`调试，运行后生成`core`文件

  4. 调试`core`程序：`gdb test`进入`gdb`终端，使用`core-file core`可以查看`core`定位错误

### kill & raise & abort

- `int kill(pid_t pid, int sig);`
  - 使用`man 2 kill`查看帮助
  - 功能：给**任何的进程或者进程组**`pid`，发送**任何的信号** `sig`
  - 参数
    - `pid`
      - `> 0` : 将信号发送给指定的进程
      - `= 0` : 将信号发送给当前的进程组
      - `= -1` : 将信号发送给每一个有权限接收这个信号的进程
      - `< -1` : 这个`pid=某个进程组的ID取反`
    - `sig` : 需要发送的信号的编号或者是宏值，0表示不发送任何信号
  - 返回值：0成功，-1失败
- `int raise(int sig);`
  - 使用`man 3 raise`查看帮助
  - 功能：给**当前进程**发送信号
  - 参数：`sig` : 要发送的信号
  - 返回值：0成功，非0失败
- `void abort(void);`
  - 使用`man 3 abort`查看帮助
  - 功能： 发送`SIGABRT`信号给当前的进程，**杀死当前进程**
- code

```c
/*
    #include <sys/types.h>
    #include <signal.h>
    int kill(pid_t pid, int sig);
        -功能：给某个进程pid，发送某个信号sig
        -参数：
            -pid：需要发送给的进程
            -sig: 需要发送的信号的编号或者是宏值
    int raise(int sig);
        -功能：给当前进程发送信号
        -参数：
            -sig:要发送的信号
        -返回值：
            -成功 0
            -失败 非0  
    void abort(void);
        -功能：发送SIGABRT信号给当前进程，杀死当前进程

     
*/
 #include <sys/types.h>
 #include <signal.h>
 #include <stdio.h>
 #include <unistd.h>
 int main(){
    pid_t pid=fork();
    if(pid>0){
        printf("parent process\n");
        sleep(2);
        printf("kill child process now\n");
        kill(pid,SIGINT);
    }
    else if(pid==0){
        int i=0;
        for(i=0;i<5;i++){
            printf("child process\n");
            sleep(1);
        }
    }
    return 0;
 }
```

![](C:\Users\11048\Desktop\Linux多进程\{036F1D12-3D19-48d4-BB0F-B7E34A6E5B72}.png)
