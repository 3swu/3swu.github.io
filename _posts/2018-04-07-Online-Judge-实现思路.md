---
author: wulei
comments: true
date: 2018-04-07 23:16:00+00:00
link: http://wulei.kim/2018/04/07/oj/
slug: oj
title: Online Judge 实现思路
tags:
- OJ
---

清明节给自己挖了一个新坑：开发一个Online Judge系统，因为我们学校没有搞ACM的人，自然也没有Online Judge系统，所以开发一个OJ系统是一个很酷的事情，对我来说也具有很大的挑战。已经有一些开源的OJ系统，但是造轮子多有趣。。为了这个坑，清明节这几天都没有学习。。这几天实验性的写了一部分Judge的核心部分。不知道这个坑什么时候能填上，所以这里先记录下我大致设想的思路。

### 总体架构

整个OJ分为两部分，自然是Web部分和核心的评测机部分。Web部分实现从前端得到用户的源文件，将操作和源代码存入数据库中，然后调用评测机部分，评测机经过编译，运行，与预期输出的对比，从而判定结果。Web端主要要实现的功能有用户管理，包括用户权限的管理，题目管理，小组（小组是我设想的类似班级的东西，一个小组包括一个题集，属于这个小组的用户去完成这个题集）管理，比赛管理。

评测机是系统的核心。评测机运行在LInux上，评测机的实现需要与系统调用打交道，所以并不能实现跨平台，而我选择Linux，主要是因为对Windows系统开发很不熟悉。主要包括代码的编译，运行，结果评定。流程如下：编译->fork一个子进程运行用户程序->返回结果。在服务器上运行别人的代码是一件极其危险的事情，所以评测机的安全非常重要。在父进程中要时刻监视子进程的状态，包括时间，内存和系统调用。通过在网上收集的资料，我暂时选用的ptrace来监视子进程的状态。ptrace可以让子进程产生每一个系统调用的时候阻塞，父进程得到子进程的系统调用号，通过比对安全的系统调用白名单判断子进程是否产生了危险的系统调用。

### Web端

Web端没什么好说的，现在是设想的是用Python+Flask或者Django，至于使用Flask还是Django，我现在更偏爱Flask，因为Flask轻量一些，而且暂时对Django不是很了解，数据库用Mysql，可以加一层Redis提高性能和并发量。通过Socket与评测机通信，这样有一个缺点，就是这样属于阻塞式IO，性能很低，只有通过Web端每一个线程管理一个连接。而且还有一个致命的缺点，就是这样设计上只能有一个评测机，如果遇到比赛，用户量激增的时候，就无法扩充评测机，更好的做法是使用消息队列，Web端只需要将操作添加到队列中，评测机从队列中取出操作。

### 评测机

评测机的第一部分是编译，第一阶段我想只先实现C和C++的评测，这两个都用GCC编译就行。评测机的安全管理很重要，就是要实现一个安全的沙箱。

#### 危险系统调用的监视

ptrace是一个强大的工具，据说GDB就是通过ptrace实现断点调试等功能的，一个简单的通过ptrace实现监视系统调用的程序如下:

``` c++
#include <iostream>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/ptrace.h>
#include <sys/reg.h>

using namespace std;

int main(){
    pid_t pid;
    switch (pid = fork()){
        case -1:
            exit(EXIT_FAILURE);
            break;

        case 0: //子进程
            ptrace(PTRACE_TRACEME, 0, NULL, NULL);
            execve("...", NULL, NULL); //子进程调用另一个程序

        default: //父进程
            while(1){
                int status;
                int flag = 0;
                wait(&status);
                if(WIFEXITED(status))
                    return 0;
                if(flag == 0){
                    long syscall_id = ptrace(PTRACE_PEEKUSER, pid, ORIG_RAX * 8, NULL);
                    cout << "Process system call ID = " << syscall_id;
                    flag = 1;
                }
                if(flag == 1){
                    long return_value = ptrace(PTRACE_PEEKUSER, pid, RAX * 8, NULL);
                    cout << " with return value = " << return_value << endl;
                    flag = 0;
                }
                ptrace(PTRACE_SYSCALL, pid, NULL, NULL);
            }
    }
}
```

这个程序实现了监视子进程的每一个系统调用，给出调用号和系统调用的返回值：

+ Linux Man Pages给出ptrace的原型如下：

  ``` c
  #include <sys/ptrace.h>

  long ptrace(enum __ptrace_request request, pid_t pid, void *addr, void *data);

  ```

  request参数决定了函数的功能，也决定了后面三个参数的意义。在上面的程序中，request参数为PTRACE_TRACEME时，子进程进入被监视状态，相当于子进程请求父进程监视自己，在评测机中通常在execv用户程序之前使用，达到父进程能够监视的效果；request参数为PTRACE_PEEKUSER时，可以获取子进程寄存器中的某个值，在32位系统中，系统调用号存在ORIG_REX寄存器中，系统调用的返回值存在REX寄存器中，而在64位系统中，系统调用号存在ORIG_RAX中，系统调用的返回值存在RAX寄存器中。上面程序中`RAX * 8`中的8是寄存器的宽度，在64位系统中为8位，而在32位系统中的宽度为4位。

+ 最后request参数为PTRACE_SYSCALL时使父进程再次监视子进程。

#### 程序资源的限制

对于用户程序资源的限制，使用Linux提供的rlimit工具实现。通过getrlimit函数与setrlimit函数可以限制进程的资源，如下：

``` c
...
struct rlimit r;
getrlimit(RLIMIT_CPU, &r);
r.rlim_cur = r.rlim_max = time_limit / 1000;
setrlimit(RLIMIT_CPU, &r);
getrlimit(RLIMIT_AS, &r);
r.rlim_cur = r.rlim_max = memory_limit * 1024 * 1.25;
setrlimit(RLIMIT_AS, &r);
...
```

+ `struct rlimit`是`<sys/resource.h>`定义的一个结构体，包含两个元素，`rlim_cur`为软限制，`rlim_max`为硬限制。

#### 进程实际占用资源的获取

通过上述方法限制了进程的资源之后，可能由于系统进程切换的原因导致子进程实际在CPU上执行的时间超过我们限制的时间，并且内存的占用也会超过限制。这就需要父进程在子进程运行过程中监视实际的资源占用量。父进程在子进程每次状态改变的时候查询实际使用的资源量，如果发现实际使用的资源量大于题目的限制，则将用户程序kill掉。

##### 实际运行时间的获取

Linux提供了一组函数：`wait3`和`wait4`，这两个函数与`wait`与`waitpid`类似，都是等待子进程的状态改变。前两个函数不同的是，他们不仅能得到子进程的状态，还能获得子进程的实际资源占用，在Linux Man Pages中，对这两个函数的原型定义如下 :

``` c
pid_t wait3(int *status, int options, struct rusage *rusage);

pid_t wait4(pid_t pid, int *status, int options, struct rusage *rusage);
```

`wait3`函数与`wait`函数类似，会在父进程的所有子进程都改变状态之后返回，而`wait4`和`waitpid`函数类似，给以通过给定一个pid实现单独等待某一个子进程。

+ `options`参数为进程等待选项，`WNOHANG`时表示立即返回，而当`WUNTRACED`时表示等待子进程状态改变后返回。

+ `struct rusage`是记录死亡进程资源使用的结构体，结构体具体信息这里不给出了，我们主要在其中获得进程的CPU使用时间，实现如下：

  ``` c
  wait4(child_pid, &status, WUNTRACED, &usage);
  time_usage = usage.ru_utime.tv_sec * 1000 + usage.ru_utime.tv_usec / 1000 + usage.ru_stime.tv_sec * 1000 +usage.ru_stime.tv_usec / 1000;
  ```

  `rusage`结构体中的`ru_utime`和`ru_stime`是`time_val`类型的结构体，其中的`tv_sec`和`tv_usec`分别为秒和微秒(10的负六次方秒)。

##### 实际内存占用的获取

通常我们要查询一个进程的内存占用情况时会在shell中使用top命令或ps命令。查资料发现，而这些命令是通过系统目录中`/proc/{pid}/status`文件查询得到的，所以我们也可以通过这个方法，在status这个文件中的VmSize行中，可以轻松获得该进程的内存使用量。

#### 系统调用的白名单

使用了开源的hustoj中的系统调用白名单。

#### 最后

项目实时上传到[Github](https://github.com/wuleiaty/online-judge) 
