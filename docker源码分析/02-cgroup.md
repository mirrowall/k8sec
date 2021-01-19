# CGroup 分析

## 子系统
+ cpu: 限制进程的 cpu 使用率。
+ cpuacct 子系统，可以统计 cgroups 中的进程的 cpu 使用报告。
+ cpuset: 为cgroups中的进程分配单独的cpu节点或者内存节点。
+ memory: 限制进程的memory使用量。
+ blkio: 限制进程的块设备io。
+ devices: 控制进程能够访问某些设备。
+ net_cls: 标记cgroups中进程的网络数据包，然后可以使用tc模块（traffic control）对数据包进行控制。
+ net_prio: 限制进程网络流量的优先级。
+ huge_tlb: 限制HugeTLB的使用。
+ freezer:挂起或者恢复cgroups中的进程。
+ ns: 控制cgroups中的进程使用不同的namespace。
## cgroups文件系统
Linux通过文件的方式，将cgroups的功能和配置暴露给用户，这得益于Linux的虚拟文件系统（VFS）。VFS将具体文件系统的细节隐藏起来，给用户态提供一个统一的文件系统API接口，cgroups和VFS之间的链接部分，称之为cgroups文件系统。

比如挂在 cpu、cpuset、memory 三个子系统到 /cgroups/cpu_mem 目录下

mount -t cgroup -o cpu,cpuset,memory cpu_mem /cgroups/cpu_mem
关于虚拟文件系统机制，见浅谈Linux虚拟文件系统机制

## cgroups子系统
这里简单介绍几个常见子系统的概念和用法，包括cpu、cpuacct、cpuset、memory、blkio。

### cpu子系统
cpu子系统限制对CPU的访问，每个参数独立存在于cgroups虚拟文件系统的伪文件中，参数解释如下：

+ cpu.shares: cgroup对时间的分配。比如cgroup A设置的是1，cgroup B设置的是2，那么B中的任务获取cpu的时间，是A中任务的2倍。
+ cpu.cfs_period_us: 完全公平调度器的调整时间配额的周期。
+ cpu.cfs_quota_us: 完全公平调度器的周期当中可以占用的时间。
+ cpu.stat 统计值
+ nr_periods 进入周期的次数
+ nr_throttled 运行时间被调整的次数
+ throttled_time 用于调整的时间
+ 
### cpuacct子系统
子系统生成cgroup任务所使用的CPU资源报告，不做资源限制功能。

+ cpuacct.usage: 该cgroup中所有任务总共使用的CPU时间（ns纳秒）
+ cpuacct.stat: 该cgroup中所有任务总共使用的CPU时间，区分user和system时间。
+ cpuacct.usage_percpu: 该cgroup中所有任务使用各个CPU核数的时间。

通过cpuacct如何计算CPU利用率呢？可以通过cpuacct.usage来计算整体的CPU利用率，计算如下：
```
# 1. 获取当前时间（纳秒）
tstart=$(date +%s%N)
# 2. 获取cpuacct.usage
cstart=$(cat /xxx/cpuacct.usage)
# 3. 间隔5s统计一下
sleep 5
# 4. 再次采点
tstop=$(date +%s%N)
cstop=$(cat /xxx/cpuacct.usage)
# 5. 计算利用率
($cstop - $cstart) / ($tstop - $tstart) * 100
```
### cpuset子系统
适用于分配独立的CPU节点和Mem节点，比如将进程绑定在指定的CPU或者内存节点上运行，各参数解释如下：

+ cpuset.cpus: 可以使用的cpu节点
+ cpuset.mems: 可以使用的mem节点
+ cpuset.memory_migrate: 内存节点改变是否要迁移？
+ cpuset.cpu_exclusive: 此cgroup里的任务是否独享cpu？
+ cpuset.mem_exclusive： 此cgroup里的任务是否独享mem节点？
+ cpuset.mem_hardwall: 限制内核内存分配的节点（mems是用户态的分配）
+ cpuset.memory_pressure: 计算换页的压力。
+ cpuset.memory_spread_page: 将page cache分配到各个节点中，而不是当前内存节点。
+ cpuset.memory_spread_slab: 将slab对象(inode和dentry)分散到节点中。
+ cpuset.sched_load_balance: 打开cpu set中的cpu的负载均衡。
+ cpuset.sched_relax_domain_level: the searching range when migrating tasks
+ cpuset.memory_pressure_enabled: 是否需要计算 memory_pressure?

### memory子系统
memory子系统主要涉及内存一些的限制和操作，主要有以下参数：

+ memory.usage_in_bytes # 当前内存中的使用量
+ memory.memsw.usage_in_bytes # 当前内存和交换空间中的使用量
+ memory.limit_in_bytes # 设置or查看内存使用量
+ memory.memsw.limit_in_bytes # 设置or查看 内存加交换空间使用量
+ memory.failcnt # 查看内存使用量被限制的次数
+ memory.memsw.failcnt # - 查看内存和交换空间使用量被限制的次数
+ memory.max_usage_in_bytes # 查看内存最大使用量
+ memory.memsw.max_usage_in_bytes # 查看最大内存和交换空间使用量
+ memory.soft_limit_in_bytes # 设置or查看内存的soft limit
+ memory.stat # 统计信息
+ memory.use_hierarchy # 设置or查看层级统计的功能
+ memory.force_empty # 触发强制page回收
+ memory.pressure_level # 设置内存压力通知
+ memory.swappiness # 设置or查看vmscan swappiness 参数
+ memory.move_charge_at_immigrate # 设置or查看 controls of moving charges?
+ memory.oom_control # 设置or查看内存超限控制信息(OOM killer)
+ memory.numa_stat # 每个numa节点的内存使用数量
+ memory.kmem.limit_in_bytes # 设置or查看 内核内存限制的硬限
+ memory.kmem.usage_in_bytes # 读取当前内核内存的分配
+ memory.kmem.failcnt # 读取当前内核内存分配受限的次数
+ memory.kmem.max_usage_in_bytes # 读取最大内核内存使用量
+ memory.kmem.tcp.limit_in_bytes # 设置tcp 缓存内存的hard limit
+ memory.kmem.tcp.usage_in_bytes # 读取tcp 缓存内存的使用量
+ memory.kmem.tcp.failcnt # tcp 缓存内存分配的受限次数
+ memory.kmem.tcp.max_usage_in_bytes # tcp 缓存内存的最大使用量

### blkio子系统 - block io
主要用于控制设备IO的访问。有两种限制方式：权重和上限，权重是给不同的应用一个权重值，按百分比使用IO资源，上限是控制应用读写速率的最大值。

按权重分配IO资源：

+ blkio.weight：填写 100-1000 的一个整数值，作为相对权重比率，作为通用的设备分配比。
+ blkio.weight_device： 针对特定设备的权重比，写入格式为 device_types:node_numbers weight，空格前的参数段指定设备，weight参数与blkio.weight相同并覆盖原有的通用分配比。

按上限限制读写速度：

+ blkio.throttle.write_bps_device：按每秒写入块设备的数据量设定上限，格式device_types:node_numbers bytes_per_second。
+ blkio.throttle.read_iops_device：按每秒读操作次数设定上限，格式device_types:node_numbers operations_per_second。
+ blkio.throttle.write_iops_device：按每秒写操作次数设定上限，格式device_types:node_numbers operations_per_second
  
针对特定操作 (read, write, sync, 或 async) 设定读写速度上限

+ blkio.throttle.io_serviced：针对特定操作按每秒操作次数设定上限，格式device_types:node_numbers operation operations_per_second
+ blkio.throttle.io_service_bytes：针对特定操作按每秒数据量设定上限，格式device_types:node_numbers operation bytes_per_second


# Cgroups用法

**Table of Contents**
<!-- BEGIN MUNGE: GENERATED_TOC -->
  - [创建并挂载一个hierarchy](#创建并挂载一个hierarchy)
  - [在新建的hierarchy上的cgroup的根节点中扩展出两个子cgroup](#在新建的hierarchy上的cgroup的根节点中扩展出两个子cgroup)
  - [通过subsystem来限制cgroup中进程的资源](#通过subsystem来限制cgroup中进程的资源)
  - [Docker中的Cgroups](#docker中的cgroups)

<!-- END MUNGE: GENERATED_TOC -->

环境说明：
```shell
[root@fqhnode01 ~]# uname -a
Linux fqhnode01 3.10.0-514.6.1.el7.x86_64 #1 SMP Wed Jan 18 13:06:36 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
[root@fqhnode01 ~]# docker version
Client:
 Version:      1.13.1
```

## 创建并挂载一个hierarchy
```shell
[root@fqhnode01 home]# mkdir cgroups-test
[root@fqhnode01 home]# mount -t cgroup -o none,name=cgroups-test cgroups-test ./cgroups-test
[root@fqhnode01 home]# ll cgroups-test/
总用量 0
-rw-r--r--. 1 root root 0 11月  1 12:31 cgroup.clone_children
--w--w--w-. 1 root root 0 11月  1 12:31 cgroup.event_control
-rw-r--r--. 1 root root 0 11月  1 12:31 cgroup.procs
-r--r--r--. 1 root root 0 11月  1 12:31 cgroup.sane_behavior
-rw-r--r--. 1 root root 0 11月  1 12:31 notify_on_release
-rw-r--r--. 1 root root 0 11月  1 12:31 release_agent
-rw-r--r--. 1 root root 0 11月  1 12:31 tasks
```

## 在新建的hierarchy上的cgroup的根节点中扩展出两个子cgroup
```shell
[root@fqhnode01 cgroups-test]# cd cgroups-test/
[root@fqhnode01 cgroups-test]# mkdir cgroup-1 cgroup-2
[root@fqhnode01 cgroups-test]# ls
cgroup-1  cgroup.clone_children  cgroup.procs          notify_on_release  tasks
cgroup-2  cgroup.event_control   cgroup.sane_behavior  release_agent
[root@fqhnode01 cgroups-test]# 
[root@fqhnode01 cgroups-test]# tree
.
├── cgroup-1
│   ├── cgroup.clone_children
│   ├── cgroup.event_control
│   ├── cgroup.procs
│   ├── notify_on_release
│   └── tasks
├── cgroup-2
│   ├── cgroup.clone_children
│   ├── cgroup.event_control
│   ├── cgroup.procs
│   ├── notify_on_release
│   └── tasks
├── cgroup.clone_children
├── cgroup.event_control
├── cgroup.procs
├── cgroup.sane_behavior
├── notify_on_release
├── release_agent
└── tasks
2 directories, 17 files
```
可以看出，在一个cgroup的目录下创建文件夹时，系统Kernel会自动把该文件夹标记为这个cgroup的子cgroup，它们会继承父cgroup的属性。

几个文件功能说明：
- tasks，标识该cgroup下面的进程ID。如果把一个进程ID写入了tasks文件，就是把该进程加入了这个cgroup中。
- cgroup.procs 是树中当前cgroup中的进程组ID。如果是根节点，会包含所有的进程组ID。
- cgroup.clone_children，默认值为0。如果设置为1，子cgroup会继承父cgroup的cpuset配置。

## 往一个cgroup中添加和移动进程
1. 首先，查看一个进程目前所处的cgroup
```shell
[root@fqhnode01 cgroup-1]# echo $$
1019
[root@fqhnode01 cgroup-1]# cat /proc/1019/cgroup 
12:name=cgroups-test:/
11:devices:/
10:memory:/
9:hugetlb:/
8:cpuset:/
7:blkio:/
6:cpuacct,cpu:/
5:freezer:/
4:net_prio,net_cls:/
3:perf_event:/
2:pids:/
1:name=systemd:/user.slice/user-0.slice/session-1.scope
```
从`name=cgroups-test:/`可以看到当前进程（$$）位于hierarchy cgroups-test:/的根节点上。

2. 把进程移动到节点cgroup-1/中
```shell
[root@fqhnode01 cgroups-test]# cd cgroup-1/
[root@fqhnode01 cgroup-1]# cat tasks 
[root@fqhnode01 cgroup-1]# 
[root@fqhnode01 cgroup-1]# cat cgroup.procs 
[root@fqhnode01 cgroup-1]#
```
可以看到目前节点cgroup-1的tasks文件是空的。
```shell
[root@fqhnode01 cgroup-1]# echo $$ >> tasks
[root@fqhnode01 cgroup-1]# cat /proc/1019/cgroup 
12:name=cgroups-test:/cgroup-1
11:devices:/
10:memory:/
9:hugetlb:/
8:cpuset:/
7:blkio:/
6:cpuacct,cpu:/
5:freezer:/
4:net_prio,net_cls:/
3:perf_event:/
2:pids:/
1:name=systemd:/user.slice/user-0.slice/session-1.scope
[root@fqhnode01 cgroup-1]# cat cgroup.procs 
1019
1436
[root@fqhnode01 cgroup-1]# cat tasks 
1019
1437
```
可以看到，当前进程1019已经加入到cgroups-test:/cgroup-1中了。

需要注意的是，到目前为止，上面创建的hierarchy并没有关联到任何的subsystem，所以还是没有办法通过上面的cgroup节点来限制一个进程的资源占用。

## 通过subsystem来限制cgroup中进程的资源
系统已经默认为每一个subsystem创建了一个hierarchy，以memory的hierarchy为例子
```shell
# mount |grep memory
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
```
可以看到memory subsystem的hierarchy的目录是`/sys/fs/cgroup/memory`。 
这个目录就跟上面我们创建的目录类似，只不过它是系统默认创建的，已经和memory subsystem关联起来，可以限制其名下进程占用的内存。

1. 安装stress工具
```shell
yum install stress.x86_64
```

2. 不使用Cgroups限制，启动一个占用200M内存的stress进程
```shell
[root@fqhnode01 memory]# stress --vm-bytes 200m --vm-keep -m 1
```
测试结果如下，本机内存为2G，所以2G＊10%＝200M：
```shell
# top
 PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                     
 1851 root      20   0  212060 204796    132 R 96.3 10.0   0:18.27 stress  
```

3. 使用Cgroups进行限制，仅允许使用100M内存  

首先参照前面步骤，在memory 的根节点下新建一个cgroup节点test-limit-memory
```shell
[root@fqhnode01 memory]# cd /sys/fs/cgroup/memory
[root@fqhnode01 memory]# mkdir test-limit-memory
```
然后，设置该cgroup节点的最大内存占用为100M
```shell
[root@fqhnode01 memory]# cd test-limit-memory/
[root@fqhnode01 test-limit-memory]# cat memory.limit_in_bytes 
9223372036854771712
[root@fqhnode01 test-limit-memory]# echo "100m" >> ./memory.limit_in_bytes 
[root@fqhnode01 test-limit-memory]# 
[root@fqhnode01 test-limit-memory]# cat memory.limit_in_bytes 
104857600
```
然后，把当前进程移动到这个cgroup节点test-limit-memory中 
```shell
[root@fqhnode01 test-limit-memory]# cat tasks 
[root@fqhnode01 test-limit-memory]# 
[root@fqhnode01 test-limit-memory]# echo $$ >> tasks 
[root@fqhnode01 test-limit-memory]# cat tasks 
1019
1878
```
最后，再次运行`stress --vm-bytes 200m --vm-keep -m 1`，测试结果如下：
```shell
 PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                     
 1881 root      20   0  212060  87924    132 R 37.0  4.3   0:12.05 stress  
```
观察显示内存占用最大值只能达到5%，也就是100m。 成功地使用Cgroups把stree进程的内存占用限制到了100m的范围之内。

最后说一下上面创建的`/sys/fs/cgroup/memory/test-limit-memory`文件会在重启之后被系统自动删除。
如果要删除一个cgroup，用umount命令。

## Docker中的Cgroups
```shell
[root@fqhnode01 memory]# docker run -itd  -e is_leader=true -e node_name=aaa -m 128m 98c2ef0aa9bb
e19acd747074941e9062f2beb5163927b097296f31b7a0317ecb93387cc16466
```
docker 会自动在memory hierarchy的目录下新建一个cgroup节点
```shell
[root@fqhnode01 e19acd747074941e9062f2beb5163927b097296f31b7a0317ecb93387cc16466]# pwd
/sys/fs/cgroup/memory/docker/e19acd747074941e9062f2beb5163927b097296f31b7a0317ecb93387cc16466
[root@fqhnode01 e19acd747074941e9062f2beb5163927b097296f31b7a0317ecb93387cc16466]# cat memory.limit_in_bytes 
134217728
[root@fqhnode01 e19acd747074941e9062f2beb5163927b097296f31b7a0317ecb93387cc16466]# cat memory.usage_in_bytes 
66355200
```

- memory.limit_in_bytes 内存限制
- memory.usage_in_bytes 该cgroup中的进程已经使用的memory