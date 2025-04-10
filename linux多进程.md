# 进程概述

## 说明

本部分笔记及源码出自`slide/02Linux多进程开发/01 进程概述`

## 程序和进程

- `程序`是包含一系列**信息**的文件，这些信息描述了如何在运行时创建一个`进程`
  - **二进制格式标识**：每个程序文件都包含用于描述可执行文件格式的元信息。内核利用此信息来解释文件中的其他信息，Linux中为ELF可执行连接格式
  - **机器语言指令**：对程序算法进行编码
  - **程序入口地址**：标识程序开始执行时的起始指令位置
  - **数据**：程序文件包含的变量初始值和程序使用的字面量值（比如字符串）
  - **符号表及重定位表**：描述程序中函数和变量的位置及名称。这些表格有多重用途，其中包括调试和运行时的符号解析（动态链接）
  - **共享库和动态链接信息**：程序文件所包含的一些字段，列出了程序运行时需要使用的共享库，以及加载共享库的动态连接器的路径名
  - 其他信息：程序文件还包含许多其他信息，用以描述如何创建进程
- **`进程`是正在运行的`程序`的实例**。是一个具有一定独立功能的程序关于某个数据集合的一次运行活动。它是操作系统动态执行的基本单元，在传统的操作系统中，进程既是基本的分配单元，也是基本的执行单元
- 可以用**一个程序来创建多个进程**，进程是由内核定义的抽象实体，并为该实体分配用以执行程序的各项系统资源。从内核的角度看，进程由用户内存空间和一系列内核数据结构组成，其中用户内存空间包含了程序代码及代码所使用的变量，而内核数据结构则用于维护进程状态信息。记录在内核数据结构中的信息包括许多与进程相关的标识号（IDs）、虚拟内存表、打开文件的描述符表、信号传递及处理的有关信息、进程资源使用及限制、当前工作目录和大量的其他信息

## 单道、多道程序设计

- `单道程序`，即在计算机内存中只允许一个的程序运行
- `多道程序`设计技术是在计算机内存中同时存放几道相互独立的程序，使它们**在管理程序控制下，相互穿插运行**，两个或两个以上程序在计算机系统中同处于开始到结束之间的状态，这些程序共享计算机系统资源。**引入多道程序设计技术的根本目的是为了提高 CPU 的利用率**
- 对于一个**单 CPU 系统**来说，程序同时处于运行状态只是一种宏观上的概念，他们虽然都已经开始运行，但**就微观而言，任意时刻，CPU 上运行的程序只有一个**
- 在多道程序设计模型中，多个进程轮流使用 CPU。而当下常见 **CPU 为纳秒级**，1秒可以执行大约 10 亿条指令。由于**人眼的反应速度是毫秒级**，所以看似同时在运行

## 时间片

- `时间片（timeslice）`又称为`量子（quantum）`或`处理器片（processor slice）`是操作系统分配给每个正在运行的进程微观上的一段 CPU 时间。事实上，虽然一台计算机通常可能有多个 CPU，但是同一个 CPU 永远不可能真正地同时运行多个任务。在只考虑一个 CPU 的情况下，这些进程“看起来像”同时运行的，实则是轮番穿插地运行，由于时间片通常很短（在 Linux 上为 `5ms－800ms`），用户不会感觉到
- **时间片由操作系统内核的调度程序分配给每个进程**。首先，内核会给每个进程分配相等的初始时间片，然后每个进程轮番地执行相应的时间，当所有进程都处于时间片耗尽的状态时，内核会重新为每个进程计算并分配时间片，如此往复

## 并行和并发

- `并行(parallel)`：指在同一时刻，有多条指令在多个处理器上同时执行
- `并发(concurrency)`：指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行

## 进程控制块（PCB） 

- 为了管理进程，内核必须对每个进程所做的事情进行清楚的描述。内核为每个进程分配一个`PCB(Processing Control Block)进程控制块`，维护进程相关的信息，Linux 内核的进程控制块是 `task_struct` 结构体

- 在 `/usr/src/linux-headers-xxx/include/linux/sched.h` 文件中可以查看 `struct task_struct` 结构体定义，其中`linux-headers-xxx`需要替换为该目录下相应的版本

- 需要掌握的`struct task_struct` 结构体成员

  - **进程id**：系统中每个进程有唯一的 id，用 `pid_t` 类型表示，其实就是一个非负整数

  - **进程的状态**：有`就绪`、`运行`、`挂起`、`停止`等状态

  - 进程切换时需要**保存和恢复的一些CPU寄存器**

  - 描述**虚拟地址空间**的信息

  - 描述**控制终端**的信息

  - 当前工作目录（Current Working Directory） 

  - `umask 掩码`

  - 文件描述符表，包含很多指向 file 结构体的指针

  - 和信号相关的信息

  - 用户 id 和组 id

  - 会话（Session）和进程组

  - 进程可以使用的资源上限（Resource Limit），在Linux中可用`ulimit -a`查看资源上限

    ![image-20210916220259062](D:/BaiduNetdiskDownload/02Linux多进程开发/image-20210916220259062.png)

# 进程状态

## 说明

本部分笔记及源码出自`slide/02Linux多进程开发/02 进程状态及转换`

## 基本概念

- 进程状态反映进程执行过程的变化，这些状态随着进程的执行和外界条件的变化而转换
- 分为`三态模型`和`五态模型`

## 三态模型

- `运行态`：进程占有处理器正在运行
- `就绪态`：进程具备运行条件，等待系统分配处理器以便运行。当进程已分配到除CPU以外的所有必要资源后，只要再获得CPU，便可立即执行。在一个系统中处于就绪状态的进程可能有多个，通常将它们排成一个队列，称为就绪队列
- `阻塞态`：又称为等待(wait)态或睡眠(sleep)态，指进程不具备运行条件，正在等待某个事件的完成

## 五态模型

- 除`新建态`和`终止态`，其余三个状态与`三态模型`一致
- `新建态`：进程刚被创建时的状态，尚未进入就绪队列
- `终止态`：进程完成任务到达正常结束点，或出现无法克服的错误而异常终止，或被操作系统及有终止权的进程所终止时所处的状态。进入终止态的进程以后不再执行，但依然保留在操作系统中等待善后。一旦其他进程完成了对终止态进程的信息抽取之后，操作系统将删除该进程

## 进程相关命令

### 查看进程-静态

- `ps`命令用来查看进程（静态），可以使用`man ps`查看使用说明

- 常用参数含义

  - a：显示终端上的所有进程，包括其他用户的进程
  - u：显示进程的详细信息
  - x：显示没有控制终端的进程
  - j：列出与作业控制相关的信息

- `ps -aux`或`ps aux`
  - `USER`：进程所属用户

  - `PID`：进程ID

  - `%CPU`：CPU使用占比

  - `%MEM`：内存使用占比

  - `TTY`：进程所属终端，在终端直接执行`tty`可查看当前`Terminal`所属终端（因为此时我还打开了另外两个终端）

  - `STAT`：进程状态

    - D ：不可中断 Uninterruptible（usually IO）
    - R：正在运行，或在队列中的进程
    - S(大写) ：处于休眠状态
    - T：停止或被追踪
    - Z：僵尸进程
    - W：进入内存交换（从内核2.6开始无效）
    - X：死掉的进程
    - <：高优先级
    - N：低优先级
    - s：包含子进程
    - \+：位于前台的进程组

  - `START`：进程开始执行时间

  - `TIME`：进程执行持续时间

  - `COMMAND`：进程执行命令

- `ps -ajx`或`ps ajx`

  - `PPID`：该进程的父进程ID
  - `PGID`：该进程所属组ID
  - `SID`：该进程所属会话(session)ID，多个组构成会话

### 查看进程-动态

- `top`

- 可以在使用 top 命令时加上 -d 来指定显示信息更新的时间间隔

- 在 top 命令执行后，可以按以下按键对显示的结果进行排序

  - M：根据内存使用量排序
  - P：根据 CPU 占有率排序
  - T：根据进程运行时间长短排序
  - U：根据用户名来筛选进程
  - K：输入指定的 PID 杀死进程

### 杀死进程

- `kill [-signal] pid`

- `kill -l`：列出所有信号

- `kill -9 进程ID`等价于`kill –SIGKILL 进程ID`

- `killall name`：根据进程名杀死进程

##  进程号和相关函数

- 每个进程都由进程号来标识，其类型为 `pid_t（整型）`，进程号的范围：`0～32767`。进程号总是唯一的，但可以重用。当一个进程终止后，其进程号就可以再次使用
- **任何进程（除 init 进程）都是由另一个进程创建**，该进程称为被创建进程的父进程，对应的进程号称为父进程号（PPID）
- **进程组是一个或多个进程的集合**。他们之间相互关联，进程组可以接收同一终端的各种信号，关联的进程有一个进程组号（PGID）。默认情况下，当前的进程号会当做当前的进程组号
- 进程号和进程组相关函数
  - `pid_t getpid(void);`：获取进程ID
  - `pid_t getppid(void);`：获取进程的父进程ID
  - `pid_t getpgid(pid_t pid);`：获取进程的组ID

# 进程创建

## 说明

本部分笔记及源码出自`slide/02Linux多进程开发/03 进程创建`

## 进程创建：fork

- 可通过`man 2 fork`查看帮助

- `pid_t fork(void);`

  ```c
  /*
      #include <sys/types.h>
      #include <unistd.h>
  
      pid_t fork(void);
          函数的作用：用于创建子进程。
          返回值：
              fork()的返回值会返回两次。一次是在父进程中，一次是在子进程中。
              在父进程中返回创建的子进程的ID,
              在子进程中返回0
              如何区分父进程和子进程：通过fork的返回值。
              在父进程中返回-1，表示创建子进程失败，并且设置errno
  */
  
  #include <sys/types.h>
  #include <unistd.h>
  #include <stdio.h>
  
  int main() 
  {
      int num = 10;
  
      // 创建子进程
      pid_t pid = fork();
  
      // 判断是父进程还是子进程
      if(pid > 0) {
          printf("pid : %d\n", pid);
          // 如果大于0，返回的是创建的子进程的进程号，当前是父进程
          printf("i am parent process, pid : %d, ppid : %d\n", getpid(), getppid());
  
          printf("parent num : %d\n", num);
          num += 10;
          printf("parent num += 10 : %d\n", num);
      } else if(pid == 0) {
          // 当前是子进程
          printf("i am child process, pid : %d, ppid : %d\n", getpid(),getppid());
         
          printf("child num : %d\n", num);
          num += 100;
          printf("child num += 100 : %d\n", num);
      }
  
      // for循环
      for(int i = 0; i < 3; i++) {
          printf("i : %d , pid : %d\n", i , getpid());
          sleep(1);
      }
  
      return 0;
  }
  ```


## fork工作原理

- Linux 的 `fork()` 使用是通过**写时拷贝 (copy- on-write) 实现**。写时拷贝是一种可以推迟甚至避免拷贝数据的技术

- 内核此时并不复制整个进程的地址空间，而是让**父子进程共享同一个地址空间**，只有在**需要写入的时候**才会复制地址空间，从而使各个进程拥有各自的地址空间。即**资源的复制是在需要写入的时候才会进行，在此之前，只有以只读方式共享**（示例程序中`num`的作用）

- **fork之后父子进程共享文件**。fork产生的子进程与父进程**有相同的文件描述符，指向相同的文件表**，引用计数增加，共享文件偏移指针

- 使用**虚拟地址空间**，由于用的是**写时拷贝 (copy- on-write) **，下图**不完全准确，但可帮助理解**


## 父子进程关系

### 区别

- **fork()函数的返回值不同**。父进程中: >0 返回的是子进程的ID，子进程中: =0
- **pcb中的一些数据不同**。pcb中存的是**当前进程的ID(pid)**，**当前进程的父ID(ppid)**和**信号集**

### 共同点

- 在某些状态下，即**子进程刚被创建出来，还没有执行任何的写数据的操作**。此时**用户区的数据**和**文件描述符表**父进程和子进程一样

### 父子进程对变量共享说明

- 刚开始的时候，是一样的，共享的。如果修改了数据，不共享了
- 读时共享（子进程被创建，两个进程没有做任何的写的操作），写时拷贝

## GDB 多进程调试

- 在以下调试程序**第10行**及**第20行**打断点，后续说明都基于这两个断点

- 打断点及查看

- 使用 GDB 调试的时候，GDB 默认只能跟踪一个进程，可以在 fork 函数调用之前，通过指令设置 GDB 调试工具跟踪父进程或者是跟踪子进程，**默认跟踪父进程**

- 查看当前跟踪的进程：`show follow-fork-mode`

- 设置调试父进程或者子进程：`set follow-fork-mode [parent（默认）| child]`

  - 调试父进程，子进程循环会自动执行，完毕后需要输入`n`继续执行父进程

  - 调试子进程，父进程循环会自动执行，完毕后需要输入`n`继续执行子进程

- 查看调试模式：`show detach-on-fork`

- 设置调试模式：`set detach-on-fork [on | off]`

  - 默认为 on，表示调试当前进程的时候，其它的进程继续运行，如果为 off，调试当前进程的时候，其它进程被 GDB 挂起

  - 注：在设置为`off`时，执行程序会报以下错误，原因是**gdb 8.x版本存在bug**

  - 以下正常执行的`gdb`版本为`v7.11.1`（截图来源于视频），与设置为`on`的区别在于，**`for`循环是否打印**

- 查看调试的进程：`info inferiors`，此时调试进程为`parent`，需要执行后才会显示进程

  - 当`detach-on-fork`为`on`时，只会显示一个进程（==因为另一个进程已经执行完毕，销毁==，猜测）

  - 当`detach-on-fork`为`off`时，会显示两个进程

- 切换当前调试的进程：`inferior Num`

- 使进程脱离 GDB 调试：`detach inferiors Num`

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



### alarm & setitimer

- 区别：`alarm`只能定一次时，`setitimer`可以周期性定时

- `unsigned int alarm(unsigned int seconds);`
  - 使用`man 2 alarm`查看帮助
  - 功能：设置定时器（闹钟）。函数调用，开始倒计时，当倒计时为0的时候，函数会给当前的进程发送一个信号：`SIGALARM`
  - 参数：`seconds`，倒计时的时长，单位：秒。如果参数为0，定时器无效（不进行倒计时，不发信号）
  - 取消一个定时器，通过`alarm(0)`
  - 返回值
    - 之前没有定时器，返回0
    - 之前有定时器，返回之前的定时器剩余的时间
- `SIGALARM` ：默认终止**当前的进程**，每一个进程都有且只有唯一的一个定时器
- 定时器，与进程的状态无关（自然定时法）。无论进程处于什么状态，alarm都会计时，即**函数不阻塞**

```c
/*
    #include <unistd.h>
    unsigned int alarm(unsigned int seconds);
        -功能：设置定时器，函数调用，开启倒计时，当倒计时为0时，函数
        会发送一个信号：SIGALARM
        -参数：
            seconds:倒计时的时长，单位：秒
        -返回值：倒计时剩余时间

    -SIGALARM：默认终止当前进程，每一个进程有且只有唯一一个定时器



*/
#include <unistd.h>
#include <stdio.h>

int main(){
    int seconds=alarm(5);
    printf("seconds = %d\n",seconds);

    sleep(2);
    seconds = alarm(2);
    printf("seconds = %d\n",seconds);

    while(1){

    }
    return 0;
}
```

- `int setitimer(int which, const struct itimerval *new_val, struct itimerval *old_value);`

  - 使用`man 2 setitimer`查看帮助
  - 功能：设置定时器（闹钟）。可以替代alarm函数。精度微妙us，可以实现周期性定时
  - 参数
    - `which` : 定时器以什么时间计时
      - `ITIMER_REAL`: 真实时间，时间到达，发送 `SIGALRM` (常用)
      - `ITIMER_VIRTUAL`: 用户时间，时间到达，发送 `SIGVTALRM`
      - `ITIMER_PROF`: 以该进程在用户态和内核态下所消耗的时间来计算，时间到达，发送 `SIGPROF`
    - `new_value`: 设置定时器的属性
    - `old_value` ：记录上一次的定时的时间参数，一般不使用，指定NULL
  - 返回值：成功 0，失败 -1 并设置错误号

- `struct itimerval`

  ```c
  struct itimerval {      // 定时器的结构体
      struct timeval it_interval;  // 每个阶段的时间，间隔时间
      struct timeval it_value;     // 延迟多长时间执行定时器
  };
  
  struct timeval {        // 时间的结构体
      time_t      tv_sec;     //  秒数     
      suseconds_t tv_usec;    //  微秒    
  };
  
  // 过it_value秒后，每隔it_interval秒定时一次
  ```

- 实现**过3秒以后，每隔2秒钟定时一次**=>因为没有**信号捕捉**，所以还没有实现这样的效果

```c
/*
    #include <sys/time.h>
    int setitimer(int which, const struct itimerval *new_value,struct itimerval *old_value);
        -功能：设置定时器，可以代替alarm函数，精度微秒
        -参数：
            -which：定时器以什么时间计时
                ITIMER_REAL：真实时间，时间到达，发送SIGALARM
                ITIMER_VIRTUAL：用户时间
                ITIMER_PROF：该进程在用户态和内核态下消耗的时间
            -new_value:设置定时器属性
            struct itimerval {
               struct timeval it_interval; 
               struct timeval it_value;    
            };

           struct timeval {
               time_t      tv_sec;         
               suseconds_t tv_usec;       
           };

            -old_value:
                记录上一次定时的参数
*/              

#include <sys/time.h>
#include <stdio.h>

int main(){
    struct itimerval new_value;
    new_value.it_interval.tv_sec=2;
    new_value.it_interval.tv_usec=0;

    new_value.it_value.tv_sec=3;
    new_value.it_value.tv_usec=0;
   int ret=setitimer(ITIMER_REAL, &new_value,NULL);

    getchar();
    return 0;
}
```

![](C:\Users\11048\Desktop\Linux多进程\{116310A2-DEA7-4f86-9BEA-2A2DC15A4BD1}.png)

## 信号捕捉函数

### signal

- `sighandler_t signal(int signum, sighandler_t handler);`

  - 使用`man 2 signal`查看帮助
  - 功能：设置某个信号的捕捉行为
  - 参数
    - `signum`: 要捕捉的信号
    - `handler`: 捕捉到信号要如何处理
      - `SIG_IGN` ： 忽略信号
      - `SIG_DFL` ： 使用信号默认的行为
      - `自定义回调函数`
    - 返回值
      - 成功，返回上一次注册的信号处理函数的地址。第一次调用返回NULL
      - 失败，返回SIG_ERR，设置错误号
      - 注意：返回值定义在宏`__USE_GNU`中，需要指定或者直接在程序中使用`typedef __sighandler_t sighandler_t;`
    - `SIGKILL`和`SIGSTOP`不能被捕捉，不能被忽略

- 完善**过3秒以后，每隔2秒钟定时一次的定时器功能**

```c
/*
    #include <signal.h>
    typedef void (*sighandler_t)(int);
    sighandler_t signal(int signum, sighandler_t handler);
        -功能：去设置某个信号捕捉行为
        -参数：
            -signum:要捕捉的信号
            -handler:要如何处理信号
                -SIG_IGN：忽略信号
                -SIG_DFL：使用信号默认行为
                -回调函数：内核调用，程序员只负责写，捕捉到信号后如何处理
        -返回值：
            成功：返回上一次注册的信号处理函数的地址，第一次调用返回NULL
            失败：返回SIG_ERR，设置错误号



    -SIGKILL SIGSTOP不能被捕捉不能被忽略

*/
#include <sys/time.h>
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>

void myalarm(int num){
    printf("捕捉到信号的编号是： %d\n",num);


}
int main(){
    //捕捉信号
    //signal(SIGALRM,SIG_IGN);
    signal(SIGALRM,myalarm);
    struct itimerval new_value;
    new_value.it_interval.tv_sec=2;
    new_value.it_interval.tv_usec=0;

    new_value.it_value.tv_sec=3;
    new_value.it_value.tv_usec=0;
   int ret=setitimer(ITIMER_REAL, &new_value,NULL);

    getchar();
    return 0;
}
```

### sigaction

- `int sigaction(int signum, const struct sigaction *act,struct sigaction *oldact);`

  - 使用`man 2 sigaction`查看帮助
  - 功能：检查或者改变信号的处理，即信号捕捉
  - 参数
    - `signum` : 需要捕捉的信号的编号或者宏值（信号的名称）
    - `act` ：捕捉到信号之后的处理动作
    - `oldact` : 上一次对信号捕捉相关的设置，一般不使用，设置为NULL
  - 返回值：成功返回0， 失败返回-1

- `struct sigaction`

  ```c
  struct sigaction {
      // 函数指针，指向的函数就是信号捕捉到之后的处理函数
      void     (*sa_handler)(int);
      // 不常用
      void     (*sa_sigaction)(int, siginfo_t *, void *);
      // 临时阻塞信号集，在信号捕捉函数执行过程中，临时阻塞某些信号。
      sigset_t   sa_mask;
      // 使用哪一个信号处理对捕捉到的信号进行处理
      // 这个值可以是0，表示使用sa_handler,也可以是SA_SIGINFO表示使用sa_sigaction
      int        sa_flags;
      // 被废弃掉了
      void     (*sa_restorer)(void);
  };
  ```

```c
/*
    #include <signal.h>
    int sigaction(int signum, const struct sigaction *act,
                struct sigaction *oldact);


        -功能：检查或设置信号的处理
        -参数：
            -signum:需要捕捉的信号编号
            -act:捕捉信号之后的处理动作
            -oldact:上一次对信号的处理动作，一般不使用，传递NULL
        -返回值：成功0，失败-1

        struct sigaction {
            //函数指针：指向的函数就是信号捕捉到之后的处理函数
               void     (*sa_handler)(int);
               //不常用
               void     (*sa_sigaction)(int, siginfo_t *, void *);
            //临时阻塞信号集
               sigset_t   sa_mask;
               //使用哪一个信号处理对捕捉到的信号进行处理
               int        sa_flags;
               void     (*sa_restorer)(void);
        };


*/

#include <sys/time.h>
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>

void myalarm(int num){
    printf("捕捉到信号的编号是： %d\n",num);


}
int main(){
    //捕捉信号
    //signal(SIGALRM,SIG_IGN);
    struct sigaction act;
    act.sa_flags=0;
    act.sa_handler=myalarm;
    sigemptyset(&act.sa_mask);
    sigaction(SIGALRM,&act,NULL);
    struct itimerval new_value;
    new_value.it_interval.tv_sec=2;
    new_value.it_interval.tv_usec=0;

    new_value.it_value.tv_sec=3;
    new_value.it_value.tv_usec=0;
   int ret=setitimer(ITIMER_REAL, &new_value,NULL);
    
    //getchar();
    while(1){
        
    }
    return 0;
}
```



### signal和sigaction区别

- 参数区别
- 版本区别，`signal`在不同版本Linux中，行为不一致，所以推荐使用`sigaction`（`ubutun`下两者一致）

### ==未解决==

- `signal`中可以使用一个`getchar()`阻塞信号，而`sigaction`中调用几次回调函数，就要使用多少个`getchar()`

## 信号集

### 基本概念

- 使用`man 3 sigset`查看帮助

- 许多信号相关的系统调用都需要能表示一组不同的信号，多个信号可使用一个称之为信号集的数据结构来表示，其系统数据类型为 `sigset_t`
- 在 PCB 中有两个非常重要的信号集。一个称之为 `阻塞信号集` ，另一个称之为`未决信号集`。这两个信号集都是**内核使用位图机制来实现**的。但操作系统不允许我们直接对这两个信号集进行位操作。而需自定义另外一个集合，借助信号集操作函数来对 PCB 中的这两个信号集进行修改
- 信号的 `未决` 是一种状态，指的是**从信号的产生到信号被处理前的这一段时间**
- 信号的 `阻塞` 是一个开关动作，指的是**阻止信号被处理，但不是阻止信号产生**。信号的阻塞就是让系统暂时保留信号留待以后发送。由于另外有办法让系统忽略信号，所以一般情况下信号的阻塞只是暂时的，只是为了防止信号打断敏感的操作

### 阻塞信号集与非阻塞信号集说明

1. 用户通过键盘  `Ctrl + C`, 产生2号信号 `SIGINT` (信号被创建)
2. 信号产生但是没有被处理 （未决）
   - 在内核中将所有的没有被处理的信号存储在一个集合中 （未决信号集）
   - `SIGINT`信号状态被存储在第二个标志位上
     - 这个标志位的值为0， 说明信号不是未决状态
     - 这个标志位的值为1， 说明信号处于未决状态
3. 这个未决状态的信号，需要被处理，处理之前需要和另一个信号集（阻塞信号集），进行比较
   - 阻塞信号集默认不阻塞任何的信号
   - 如果想要阻塞某些信号需要用户调用系统的API
4. 在处理的时候和阻塞信号集中的标志位进行查询，看是不是对该信号设置阻塞了
   - 如果没有阻塞，这个信号就被处理
   - 如果阻塞了，这个信号就继续处于未决状态，直到阻塞解除，这个信号就被处理

### 操作自定义信号集函数(sigemptyset等)

- 使用`man 3 sigemptyset`查看帮助
- `int sigemptyset(sigset_t *set);`
  - 功能：清空信号集中的数据，将信号集中的所有的标志位置为0
  - 参数：`set`，传出参数，需要操作的信号集
  - 返回值：成功返回0， 失败返回-1
- `int sigfillset(sigset_t *set);`
  - 功能：将信号集中的所有的标志位置为1
  - 参数：`set`，传出参数，需要操作的信号集
  - 返回值：成功返回0， 失败返回-1
- `int sigaddset(sigset_t *set, int signum);`
  - 功能：设置信号集中的某一个信号对应的标志位为1，表示阻塞这个信号
  - 参数
    - `set`：传出参数，需要操作的信号集
    - `signum`：需要设置阻塞的那个信号
  - 返回值：成功返回0， 失败返回-1
- `int sigdelset(sigset_t *set, int signum);`
  - 功能：设置信号集中的某一个信号对应的标志位为0，表示不阻塞这个信号
  - 参数
    - `set`：传出参数，需要操作的信号集
    - `signum`：需要设置不阻塞的那个信号
  - 返回值：成功返回0， 失败返回-1
- `int sigismember(const sigset_t *set, int signum);`
  - 功能：判断某个信号是否阻塞
  - 参数
    - `set`：传入参数，需要操作的信号集
    - `signum`：需要判断的那个信号
  - 返回值
    - 1 ： `signum`被阻塞
    - 0 ： `signum`不阻塞
    - -1 ： 失败



```c
/*
    int sigemptyset(sigset_t *set);
        -功能：清空信号集中的数据，将信号集所有标志位记为0
        -参数：set传出参数，需要处理的信号集
        -返回值：0成功，-1失败
    int sigfillset(sigset_t *set)
        -功能：将信号集所有标志位记为1
        -参数：set传出参数，需要处理的信号集
        -返回值：0成功，-1失败
    int sigaddset(sigset_t *set,int signum)
        -功能：设置信号集中的某一个信号对应的标志位为1，表示阻塞这个信号
        -参数：
            -set:传出参数，需要处理的信号集
            -signum:需要设置阻塞的那个信号
        -返回值：0成功，-1失败
    int sigdelset(sigset_t *set,int signum)
        -功能：设置信号集中的某一个信号对应的标志位为0，表示不阻塞这个信号
        -参数：
            -set传出参数，需要处理的信号集
            -signum:需要设置不阻塞的那个信号
        -返回值：0成功，-1失败
    int sigismember(const sigset_t *set,int signum);
        -功能：判断这个信号是否阻塞
        -参数：
            -set:传出参数，需要处理的信号集
            -signum:需要设置判断的那个信号
        返回值：1阻塞，0不阻塞，-1失败

*/
#include <sys/time.h>
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
int main(){
    sigset_t set;
    sigemptyset(&set);

    int ret=sigismember(&set,SIGINT);
    if(ret==0){
        printf("信号未阻塞\n");
    }
    else if(ret==1){
        printf("信号阻塞\n");
    }
    sigaddset(&set,SIGINT);
    ret=sigismember(&set,SIGINT);
    if(ret==0){
        printf("信号未阻塞\n");
    }
    else if(ret==1){
        printf("信号阻塞\n");
    }

    sigdelset(&set,SIGINT);
    ret=sigismember(&set,SIGINT);
    if(ret==0){
        printf("信号未阻塞\n");
    }
    else if(ret==1){
        printf("信号阻塞\n");
    }
    return 0;
}
```

### 操作内核信号集函数(sigprocmask & sigpending)

- `int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);`
  - 使用`man 2 sigprocmask`查看帮助
  - 功能：将自定义信号集中的数据设置到内核中（设置阻塞，解除阻塞，替换）
  - 参数
    - `how` : 如何对内核阻塞信号集进行处理
      - `SIG_BLOCK`: 将用户设置的阻塞信号集添加到内核中，内核中原来的数据不变。假设内核中默认的阻塞信号集是mask， 相当于`mask | set`
      - `SIG_UNBLOCK`: 根据用户设置的数据，对内核中的数据进行解除阻塞。相当于`mask &= ~set`
      - `SIG_SETMASK`：覆盖内核中原来的值
    - `set` ：已经初始化好的用户自定义的信号集
    - `oldset` : 保存设置之前的内核中的阻塞信号集的状态，一般不使用，设置为 NULL 即可
  - 返回值：成功返回0， 失败返回-1
- `int sigpending(sigset_t *set);`
  - 使用`man 2 sigpending`查看帮助
  - 功能：获取内核中的未决信号集
  - 参数：set，传出参数，保存的是内核中的未决信号集中的信息
  - 返回值：成功返回0， 失败返回-1

```c
/*
         #include <signal.h>
       int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
            -功能：将自定义信号集中的数据设置到内核中（设置阻塞，解除阻塞，替换）
            -参数：
                -how：如何对内核阻塞信号集进行处理
                    SIG_BLOCK：将用户设置的阻塞信号集添加到内核中，内核中原来数据不变
                           mask|set
                    
                    SIG_UNBLOCK：根据用户设置的数据，对内核中数据进行解除阻塞
                           mask &= ~set 

                    SIG_SETMASK:覆盖内核中原来的值
            -set:用户初始化好的用户自定义信号集
            -oldset:保存的之前内核中的信号集状态，可以是NULL
            -返回值： 成功0，失败-1
        int sigpending(sigset_t *set);
            -功能：获取内核中的未决信号集
            -参数：set，传出参数，保存的是内核中未决信号集中的信息

*/

#include <sys/time.h>
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <stdio.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
int main(){
    sigset_t set;
    sigemptyset(&set);
    sigaddset(&set,SIGINT);
    sigaddset(&set,SIGQUIT);
    sigprocmask(SIG_BLOCK,&set,NULL);
    int num=0;
    while(1){
        num++;
        sigset_t pendingset;
        sigemptyset(&pendingset);
        sigpending(&pendingset);
        for(int i=0;i<=32;i++){
            if(sigismember(&pendingset,i)==1){
                printf("1");
            }
            else{
                printf("0");
            }
        }
        printf("\n");
        sleep(3);
        if(num==10){
            //解除阻塞
            sigprocmask(SIG_UNBLOCK,&set,NULL);
        }
    }
    return 0;
}
```

## SIGCHLD信号

### 基本介绍

- 作用：解决**僵尸进程问题**，能够在不阻塞父进程的情况下，回收子进程的资源

### 实例：僵尸问题解决

```c
/*
    SIGCHLD信号产生的三个条件：
        1、子进程结束
        2、子进程暂停
        3、子进程继续运行
        都会给父进程发送信号，父进程默认忽略该信号
    
    使用SIGCHLD解决僵尸进程问题

*/
#include <sys/time.h>
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <stdio.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
void myfun(int num){
    printf("捕捉到的信号： %d\n",num);
    while(1){
        int ret=waitpid(-1,NULL,WNOHANG);
        if(ret>0){
            printf("child die, pid=%d\n",ret);
        }
        else if(ret==0){
            break;
        }
        else{
            break;
        }
    }
}
int main(){
    sigset_t set;
    sigemptyset(&set);
    sigaddset(&set,SIGCHLD);
    sigprocmask(SIG_BLOCK,&set,NULL);
    pid_t pid;
    for(int i=0;i<20;i++){
        pid=fork();
        if(pid==0){
            break;
        }

    }
    if(pid>0){
        struct sigaction act;
        act.sa_flags=0;
        act.sa_handler=myfun;
        sigemptyset(&act.sa_mask);
        sigaction(SIGCHLD,&act,NULL);
        sigprocmask(SIG_UNBLOCK,&set,NULL);
        while(1){
            printf("parent process pid : %d\n",getpid());
            sleep(2);
        }
    }
    else if(pid==0){
        printf("child process pid : %d\n",getpid());
    }
    return 0;
}
```

# 进程间通信之共享内存

## 说明

本部分笔记及源码出自`slide/02Linux多进程开发/08 进程间通信之共享内存`

## 基本概念

- **共享内存允许两个或者多个进程共享物理内存的同一块区域（通常被称为段）**。由于一个共享内存段会称为一个进程用户空间的一部分，因此这种 `IPC` 机制无需内核介入。所有需要做的就是让一个进程将数据复制进共享内存中，并且这部分数据会对其他所有共享同一个段的进程可用
- 与管道等要求发送进程将数据从用户空间的缓冲区复制进内核内存和接收进程将数据从内核内存复制进用户空间的缓冲区的做法相比，这种 `IPC` 技术的速度更快

## 共享内存使用步骤

1. 调用 `shmget()` 创建一个新共享内存段或取得一个既有共享内存段的标识符（即由其他进程创建的共享内存段）。这个调用将返回后续调用中需要用到的共享内存标识符
2. 使用 `shmat()`来附上共享内存段，即使该段成为调用进程的虚拟内存的一部分
3. 此刻在程序中可以像对待其他可用内存那样对待这个共享内存段。为引用这块共享内存，程序需要使用由 `shmat()` 调用返回的 `addr` 值，它是一个指向进程的虚拟地址空间中该共享内存段的起点的指针
4. 调用 `shmdt()` 来分离共享内存段。在这个调用之后，进程就无法再引用这块共享内存了。这一步是可选的，并且在进程终止时会自动完成这一步
5. 调用 `shmctl()` 来删除共享内存段。只有当当前所有附加内存段的进程都与之分离之后内存段才会销毁。只有一个进程需要执行这一步

## 共享内存操作函数

- `int shmget(key_t key, size_t size, int shmflg);`
  - 使用`man 2 shmget`查看帮助
  - 功能：创建一个新的共享内存段（新创建的内存段中的数据都会被初始化为0），或者获取一个既有的共享内存段的标识
  - 参数
    - `key`：`key_t`类型是一个整形，通过这个找到或者创建一个共享内存。一般使用**16进制**表示，非0值
    - `size`：共享内存的大小
    - `shmflg`：属性
      - 访问权限
      - 附加属性：创建/判断共享内存是不是存在
        - 创建：`IPC_CREAT`
        - 判断共享内存是否存在： `IPC_EXCL` , 需要和`IPC_CREAT`一起使用，即`IPC_CREAT | IPC_EXCL | 0664`
  - 返回值
    - 失败：-1 并设置错误号
    - 成功：>0 返回共享内存的引用的ID，后面操作共享内存都是通过这个值

- `void *shmat(int shmid, const void *shmaddr, int shmflg);`
  - 使用`man 2 shmat`查看帮助
  - 功能：和当前的进程进行关联
  - 参数
    - `shmid` : 共享内存的标识（ID），由`shmget`返回值获取
    - `shmaddr`: 申请的共享内存的起始地址，设置为NULL，表示由内核指定
    - `shmflg` : 对共享内存的操作
      - 读 ： `SHM_RDONLY`，必须要有读权限
      - 读写： 指定为0即为有读写权限
  - 返回值：成功：返回共享内存的首（起始）地址。  失败`(void *) -1`
- `int shmdt(const void *shmaddr);`
  - 使用`man 2 shmdt`查看帮助
  - 功能：解除当前进程和共享内存的关联
  - 参数：`shmaddr`：共享内存的首地址
  - 返回值：成功 0， 失败 -1
- `int shmctl(int shmid, int cmd, struct shmid_ds *buf);`
  - 使用`man 2 shmctl`查看帮助
  - 功能：对共享内存进行操作。删除共享内存，共享内存要删除才会消失，创建共享内存的进程被销毁了对共享内存是没有任何影响
  - 参数
    - `shmid`：共享内存的ID
    - `cmd` : 要做的操作
      - `IPC_STAT`：获取共享内存的当前的状态
      - `IPC_SET`：设置共享内存的状态
      - `IPC_RMID`：标记共享内存被销毁
    - buf：需要设置或者获取的共享内存的属性信息
      - `IPC_STAT`：`buf`存储数据
      - `IPC_SET`：`buf`中需要初始化数据，设置到内核中
      - `IPC_RMID`：没有用，设置为NULL
- `key_t ftok(const char *pathname, int proj_id);`
  - 使用`man 3 ftok`查看帮助
  - 功能：根据指定的路径名，和int值，生成一个共享内存的key
  - 参数
    - `pathname`：指定一个**存在的路径**
    - `proj_id`：int类型的值，但是系统调用只会使用其中的1个字节，范围 ： 0-255  一般指定一个字符 `'a'`
  - 返回值：`shmget`中用到的`key`

## 共享内存操作命令

### ipcs 

- `ipcs -a`：打印当前系统中**所有的**进程间通信方式的信息
- `ipcs -m`：打印出**使用共享内存**进行进程间通信的信息
- `ipcs -q`：打印出**使用消息队列**进行进程间通信的信息
- `ipcs -s`：打印出**使用信号**进行进程间通信的信息

### ipcrm

- `ipcrm -M shmkey`：移除用`shmkey`创建的**共享内存段**
- `ipcrm -m shmid`：移除用`shmid`标识的**共享内存段**
- `ipcrm -Q msgkey`：移除用`msqkey`创建的**消息队列**
- `ipcrm -q msqid`：移除用`msqid`标识的**消息队列**
- `ipcrm -S semkey`：移除用`semkey`创建的**信号**
- `ipcrm -s semid`：移除用`semid`标识的**信号**

## 实例：进程间通信（注意）

### 写端

```c
 #include <stdio.h>
 #include <sys/ipc.h>
 #include <sys/shm.h>
 #include <string.h>

 int main(){
    int shmid=shmget(100,4096,IPC_CREAT|0664);

    void * ptr=shmat(shmid,NULL,0);

    char * str="helloworld";
    memcpy(ptr,str,strlen(str)+1);

    printf("按任意键继续\n");
    getchar();
    shmdt(ptr);

    shmctl(shmid,IPC_RMID,NULL);
    return 0;
 }
```

### 读端

```c
 #include <stdio.h>
 #include <sys/ipc.h>
 #include <sys/shm.h>
 #include <string.h>

 int main(){
    int shmid=shmget(100,0,IPC_CREAT);

    void * ptr=shmat(shmid,NULL,0);

    printf("%s\n",(char*)ptr);

    printf("按任意键继续\n");
    getchar();
    shmdt(ptr);

    shmctl(shmid,IPC_RMID,NULL);
    return 0;
 }
```

## 总结

### 常见问题

- 操作系统如何知道一块共享内存被多少个进程关联？
  - 共享内存维护了一个结构体`struct shmid_ds`，这个结构体中有一个成员 `shm_nattch`
  - `shm_nattach` 记录了关联的进程个数
- 可不可以对共享内存进行多次删除 `shmctl`
  - 可以，因为执行`shmctl` 表示**标记删除共享内存（key变为0），不是直接删除**。当和共享内存关联的进程数为0的时候，就真正被删除
  - 如果一个进程和共享内存取消关联，那么这个进程就不能继续操作这个共享内存

### 共享内存与内存映射区别

- **共享内存**可以直接创建，**内存映射**需要磁盘文件（匿名映射除外）
- 共享内存效率更高
- **共享内存**所有的进程操作的是同一块共享内存，**内存映射**，每个进程在自己的虚拟地址空间中有一个独立的内存
- 数据安全
  - 进程突然退出：**共享内存**还存在，**内存映射区**消失
  - 运行进程的电脑死机(宕机)：**共享内存**中的数据消失，内存映射区的数据也消失 ，但由于磁盘文件中的数据还在，所以**可以说内存映射区的数据还存在**
- 生命周期
  - 共享内存
    - 进程退出时共享内存还在，只会标记删除
    - 只有当所有的关联的进程数为0或者关机时，才会真正删除
    - 如果一个进程退出，会自动和共享内存进行取消关联
  - 内存映射区：进程退出，内存映射区销毁

# 守护进程

## 前置知识

### 终端

- 在 `UNIX` 系统中，用户通过终端登录系统后得到一个 `shell` 进程，这个终端成为 shell 进程的`控制终端（Controlling Terminal）`，进程中，控制终端是保存在 PCB 中的信息，而 fork() 会复制 PCB 中的信息，因此由 shell 进程启动的其它进程的控制终端也是这个终端
- 默认情况下（没有重定向），每个进程的标准输入、标准输出和标准错误输出都指向控制终端
  - 进程从标准输入读也就是读用户的键盘输入
  - 进程往标准输出或标准错误输出写也就是输出到显示器上
- 在控制终端输入一些特殊的控制键可以给前台进程发信号，例如 `Ctrl + C` 会产生 `SIGINT` 信号，`Ctrl + \` 会产生 `SIGQUIT` 信号

### 进程组

- **进程组**和**会话**在进程之间形成了一种两级层次关系
  - 进程组是一组相关进程的集合，会话是一组相关进程组的集合
  - 进程组和会话是为支持 shell 作业控制而定义的抽象概念，用户通过 shell 能够交互式地在前台或后台运行命令
- 进程组由一个或多个共享同一进程组标识符（`PGID`）的进程组成
- **一个进程组拥有一个进程组首进程，该进程是创建该组的进程，其进程 ID 为该进程组的 ID，新进程会继承其父进程所属的进程组 ID**
- 进程组拥有一个生命周期，其**开始时间为首进程创建组的时刻**，**结束时间为最后一个成员进程退出组的时刻**
- 一个进程可能会因为终止而退出进程组，也可能会因为加入了另外一个进程组而退出进程组
- 进程组首进程无需是最后一个离开进程组的成员

### 会话

- **会话**是一组进程组的集合
- **会话首进程是创建该新会话的进程，其进程 ID 会成为会话 ID。新进程会继承其父进程的会话 ID**
- 一个会话中的所有进程共享单个控制终端。控制终端会在会话首进程首次打开一个终端设备时被建立
- **一个终端最多可能会成为一个会话的控制终端**
- **在任一时刻，会话中的其中一个进程组会成为终端的前台进程组，其他进程组会成为后台进程组**。只有前台进程组中的进程才能从控制终端中读取输入。当用户在控制终端中输入终端字符生成信号后，该信号会被发送到前台进程组中的所有成员
- 当控制终端的连接建立起来之后，会话首进程会成为该终端的控制进程

### 进程组、会话操作函数

- `pid_t getpgrp(void);`
- `pid_t getpgid(pid_t pid);`
- `int setpgid(pid_t pid, pid_t pgid);`
- ` pid_t getsid(pid_t pid);`
- `pid_t setsid(void);`

## 守护进程概念

- `守护进程（Daemon Process）`，也就是通常说的 Daemon 进程（精灵进程），是Linux 中的后台服务进程。它是一个生存期较长的进程，通常独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。一般采用以 d 结尾的名字
- 守护进程特征
  - 生命周期很长，守护进程会在系统启动的时候被创建并一直运行直至系统被关闭
  - 它在后台运行并且不拥有控制终端。没有控制终端确保了内核永远不会为守护进程自动生成任何控制信号以及终端相关的信号（如 `SIGINT`、`SIGQUIT`）
- Linux 的大多数服务器就是用守护进程实现的。比如，Internet 服务器 `inetd`，Web 服务器 `httpd` 等

## 守护进程的创建步骤

1. 执行一个 `fork()`，之后父进程退出，子进程继续执行
2. 子进程调用 `setsid()` 开启一个新会话
3. 清除进程的 `umask` 以确保当守护进程创建文件和目录时拥有所需的权限
4. 修改进程的当前工作目录，通常会改为根目录（`/`）
5. 关闭守护进程从其父进程继承而来的所有打开着的文件描述符
6. 在关闭了文件描述符0、1、2之后，守护进程通常会打开`/dev/null` 并使用`dup2()` 使所有这些描述符指向这个设备
7. 核心业务逻辑

## 实例：守护进程实现每隔两秒获取时间并写入磁盘

```c
/*
    写一个守护进程，每隔2s获取一下系统时间，将这个时间写入磁盘中
*/
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/time.h>
#include <signal.h>
#include <time.h>
#include <string.h>
void work(int num){
    time_t tm=time(NULL);
    struct tm* loc=localtime(&tm);
    // char buf[1024];
    // sprintf(buf,"%d-%d-%d %d:%d:%d\n",loc->tm_year,loc->tm_mon,loc->tm_mday,loc->tm_hour,loc->tm_min,loc->tm_sec);
    // printf("%s\n",buf);
    char * str=asctime(loc);
    int fd=open("time.txt",O_RDWR|O_CREAT|O_APPEND,0664);
    write(fd,str,strlen(str));
    close(fd);
}
int main(){
    // 1、创建子进程，杀死父进程
    pid_t pid=fork();
    if(pid>0){
        exit(0);
    }
    //2、将子进程创建一个会话
    setsid();
    //3、设置掩码
    umask(022);
    //4、更改工作目录
    chdir("/home/nowcoder/");
    //5、关闭，重定向文件描述符
    // int fd=open("/dev/null",O_RDWR);
    // dup2(fd,STDIN_FILENO);
    // dup2(fd,STDOUT_FILENO);
    // dup2(fd,STDERR_FILENO);
    //6、业务逻辑
    struct sigaction act;
    act.sa_flags=0;
    act.sa_handler=work;
    sigemptyset(&act.sa_mask);
    sigaction(SIGALRM,&act,NULL);
    struct itimerval val;
    val.it_value.tv_sec=2;
    val.it_value.tv_usec=0;
    val.it_interval.tv_sec=2;
    val.it_interval.tv_usec=0;
    //设置定时器

    setitimer(ITIMER_REAL,&val,NULL);
    while(1){
        sleep(10);
    }
    return 0;
}
```

# 实用技巧

## 后台运行进程

- code

  ```c
  #include <stdio.h>
  #include <unistd.h>
  
  int main()
  {
      while (1) {
          printf("this is a test...\n");
          sleep(1);
      }
      return 0;
  }
  ```

- 进程切换到后台运行：`./test &`，切换到后台后，当前终端可以使用其他命令，此时无法通过`CTRL C`终止

- 后台进程切换到前台：`fg`，切换后，可以通过`CTRL C`终止
