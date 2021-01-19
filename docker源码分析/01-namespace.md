# namespace 学习

Linux Namespace提供了一种内核级别隔离系统资源的方法，通过将系统的全局资源放在不同的Namespace中，来实现资源隔离的目的。不同Namespace的程序，可以享有一份独立的系统资源。目前Linux中提供了六类系统资源的隔离机制，分别是：

+ Mount: 隔离文件系统挂载点
+ UTS: 隔离主机名和域名信息
+ IPC: 隔离进程间通信
+ PID: 隔离进程的ID
+ Network: 隔离网络资源
+ User: 隔离用户和用户组的ID

涉及到Namespace的操作接口包括clone()、setns()、unshare()以及还有/proc下的部分文件。为了使用特定的Namespace，在使用这些接口的时候需要指定以下一个或多个参数：

+ CLONE_NEWNS: 用于指定Mount Namespace
+ CLONE_NEWUTS: 用于指定UTS Namespace
+ CLONE_NEWIPC: 用于指定IPC Namespace
+ CLONE_NEWPID: 用于指定PID Namespace
+ CLONE_NEWNET: 用于指定Network Namespace
+ CLONE_NEWUSER: 用于指定User Namespace

## clone系统调用
> int clone(int (*child_func)(void *), void *child_stack, int flags, void *arg);

它通过flags参数来控制创建进程时的特性，比如新创建的进程是否与父进程共享虚拟内存等。比如可以传入CLONE_NEWNS标志使得新创建的进程拥有独立的Mount Namespace，也可以传入多个flags使得新创建的进程拥有多种特性，比如：
> flags = CLONE_NEWNS | CLONE_NEWUTS | CLONE_NEWIPC;

传入这个flags那么新创建的进程将同时拥有独立的Mount Namespace、UTS Namespace和IPC Namespace。

## 通过/proc文件查看已存在的Namespace
在3.8内核开始，用户可以在 /proc/$pid/ns 文件下看到本进程所属的Namespace的文件信息。例如PID为2704进程的情况

这里需要注意的是：只/proc/$pid/ns/对应的Namespace文件被打开，并且该文件描述符存在，即使该PID所属的进程被销毁，这个Namespace会依然存在。可以通过挂载的方式打开文件描述符：

```
touch ~/mnt
mount --bind /proc/2704/mnt ~/mnt
```
这样就可以保留住PID为2704的进程的Mount Namespace了，即使2704进程被销毁或者退出，ID为4026531840的Mount Namespace依然会存在。

## setns加入已存在的 namepspace
setns()函数可以把进程加入到指定的Namespace中，它的函数描述如下：
> int setns(int fd, int nstype);

它的参数描述如下：
+ fd参数：表示文件描述符，前面提到可以通过打开/proc/$pid/ns/的方式将指定的Namespace保留下来，也就是说可以通过文件描述符的方式来索引到某个Namespace。
+ nstype参数：用来检查fd关联Namespace是否与nstype表明的Namespace一致，如果填0的话表示不进行该项检查。

通过在程序中调用setns来将进程加入到指定的Namespace中。

## unshare脱离到新的Namespace
unshare()系统调用用于将当前进程和所在的Namespace分离，并加入到一个新的Namespace中，相对于setns()系统调用来说，unshare()不用关联之前存在的Namespace，只需要指定需要分离的Namespace就行，该调用会自动创建一个新的Namespace。
unshare()的函数描述如下：
> int unshare(int flags);

其中flags用于指明要分离的资源类别，它支持的flags与clone系统调用支持的flags类似，这里简要的叙述一下几种标志：
+ CLONE_FILES: 子进程一般会共享父进程的文件描述符，如果子进程不想共享父进程的文件描述符了，可以通过这个flag来取消共享。
+ CLONE_FS: 使当前进程不再与其他进程共享文件系统信息。
+ CLONE_SYSVSEM: 取消与其他进程共享SYS V信号量。
+ CLONE_NEWIPC: 创建新的IPC Namespace，并将该进程加入进来。

__注意事项__

这里需要注意的是：unshare()和setns()系统调用对PID Namespace的处理不太相同，当unshare PID namespace时，调用进程会为它的子进程分配一个新的PID Namespace，但是调用进程本身不会被移到新的Namespace中。而且调用进程第一个创建的子进程在新Namespace中的PID为1，并成为新Namespace中的init进程。

setns()系统调用也是类似的，调用者进程并不会进入新的PID Namespace，而是随后创建的子进程会进入。

为什么创建其他的Namespace时unshare()和setns()会直接进入新的Namespace，而唯独PID Namespace不是如此呢？

因为调用getpid()函数得到的PID是根据调用者所在的PID Namespace而决定返回哪个PID，进入新的PID namespace会导致PID产生变化。而对用户态的程序和库函数来说，他们都认为进程的PID是一个常量，PID的变化会引起这些进程奔溃。

换句话说，一旦程序进程创建以后，那么它的PID namespace的关系就确定下来了，进程不会变更他们对应的PID namespace。

### Mount Namespace
Mount Namespace用来隔离文件系统的挂载点，不同Mount Namespace的进程拥有不同的挂载点，同时也拥有了不同的文件系统视图。Mount Namespace是历史上第一个支持的Namespace，它通过CLONE_NEWNS来标识的。

### UTS Namespace
UTS Namespace提供了主机名和域名的隔离，也就是struct utsname里的nodename和domainname两个字段。不同Namespace中可以拥有独立的主机名和域名。

那么为什么需要对主机名和域名进行隔离呢？因为主机名和域名可以用来代替IP地址，如果没有这一层隔离，同一主机上不同的容器的网络访问就可能出问题。

### IPC Namespace
IPC Namespace是对进程间通信的隔离，进程间通信常见的方法有信号量、消息队列和共享内存。IPC Namespace主要针对的是SystemV IPC和Posix消息队列，这些IPC机制都会用到标识符，比如用标识符来区分不同的消息队列，IPC Namespace要达到的目标是相同的标识符在不同的Namepspace中代表不同的通信介质(比如信号量、消息队列和共享内存)。

### PID Namespace
PID Namespace对进程PID重新标号，即不同的Namespace下的进程可以有同一个PID。内核为所有的PID Namespace维护了一个树状结构，最顶层的是系统初始化创建的，被称为Root Namespace，由它创建的新的PID Namespace成为它的Child namespace，原先的PID Namespace成为新创建的Parent Namespace，这种情况下不同的PID Namespace形成一个等级体系：父节点可以看到子节点中的进程，可以通过信号对子节点的进程产生影响，反过来子节点无法看到父节点PID Namespace里面的进程。

__有一点需要注意的是__，在新创建的PID Namespace中通过ps命令仍有可能看到Parent Namespace中的进程，这是因为ps命令是从/proc文件系统下读取进程的信息，如果只想看到本Namespace具有的进程信息，还需要重新挂载/proc文件系统：
> mount -t proc proc /proc

### Network Namespace
Network Namespace主要是用来提供关于网络资源的隔离，包括网络设备（网卡、网桥）、IPV4或IPV6协议栈、路由表、防火墙、端口等信息，不同Namespace种可以拥有独立的网络资源。

一个物理网络设备最多只能存在一个Network Namespace中，如果该Namespace被销毁后，这个物理设备不会回到它的Parent Namespace，而是会回到Root Namespace中。

如果需要打通不同Namespace的通信，可以通过创建vnet pair虚拟网络设备对的形式。虚拟网络设备对：有两端，类似于管道，分别放置在不同的Namespace中，从一端传入的数据，可以从另一端读取。

### User Namespace
User Namespace允许Namespace间可以映射用户和用户组ID，这意味着一个进程在Namespace里面的用户和用户组ID可以与Namespace外面的用户和用户组ID不同。值得一提的是，一个普通进程(Namespace外面的用户ID非0)在Namespace里面的用户和用户组ID可以为0，换句话说这个普通进程在Namespace里面可以拥有root特权的权限。