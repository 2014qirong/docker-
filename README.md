Docker 

读书笔记

一
1.1 Docker 并没有传统虚拟机的Hypervisor 层。因为Docker是基于容器的轻量级虚拟化，相对于传统的虚拟化技术，省去了Hypervisor层的开销，而且其虚拟化技术
是基于内核的Cgroup和NameSpace技术，处理逻辑与内核深度融合，所以在很多方面，它的性能非常接近物理机器
在通信方面，Docker不会直接与内核交互，它通过一个更加底层的工具Libcontainer与内核交互。Libcontainer是真正底层的容器引擎，它通过clone系统调用直接
创建容器，通过pivot_root 系统调用进入容器，并且通过直接操作cgroup文件实现对资源的管控。而Docker本身侧重于业务层面的处理

1.2 Docker的另外一个优势是对底层镜像的创新应用，即不同的容器可以共享底层的只读镜像，通过写入自己独有的内容后增加镜像层。新增的镜像又层和下层的镜像
又一起可以作为基础镜像来使用。

1.3 组件和功能

docker 客户端
docker daemon
docker container
docker image
docker 镜像
registery
1.4
Docker在功能和概念中，容器是一个核心内容。Docker通过Libcontainer 实现了对容器生命周期的管理信息设置和查询，以及监控和通信等功能。
1.5 Registry
Registry是一个存放镜像的仓库，它通常会被部署在服务器或者云端

2.1 
对此对容器的概念还停留在文件和系统的概念可以通过抽象的代码来理解其创建原理

代码一：
  pid = clone(fun,stack,flags,clone_arg):
    (flags:CLONE_NEWPID | CLONE_NEWS)|
    CLONE NEW_USER | CLONE_NEWNET
    CLONE_NEWIPC | CLONE_NEWUTS
    
    
代码二：
   echo $pis > /sys/fs/cgroup/cpu/tasks
   echo $pid > /sys/fs/cgroup/cpuset/tasks
   echo $pid > /sys/fs/cgroup/blkio/tasks
   echo $pid > /sys/fs/cgroup/memory/tasks
   echo $pid > /sys/fs/cgroup/devices/tasks
代码三：
fun()
{
   pivot_root ("path_of_rootfs",path);
   exec("/bin/bash");
   
   代码一，通过clone系统的调用，并传入一个Namespace对应的clone flag，创建一个新的子进程，该进程拥有自己的NameSpace
   代码二，将产生的pid 写入各个Cgroup子系统中，这样进程就受到了Cgroup的控制
   代码三，在fun函数中通过pivot_root 系统的调用，使进程进入一个rootfs，之后再通过exec 系统调用
   
   
   
   
   Cgroup
   
   对资源做QOS，这些资源主要包括 CPU MEMOTY BLOCK/IO BANDWITH。Cgroup 实现了一个通用进程的分层框架，而不同的资源的具体管理则是各个Cgroup
   子系统实现的。
   Cgroup的原生接口是通过cgroupfs提供的，类似sysfs是一种虚拟文件系统。
  -Cpuset子系统
  cpuset.cpus
  cpuset.mems
  可以将进程绑定到具体某个CPU上
  -CPU 子系统
  用于限制进程的CPU 占用率，cpu.shares，假设在cgroupfs的很目录下创建两个Cgroup(C1,C2),并且将cpu.shares 分别设为512 1024，那么当C1 C2 争夺CPU 
  ，C2 将会比C1 得到多一倍的CPU 占用率
  -CPU 带宽限制
  这个特性使用的接口是cpu.cfs_period_us 和cpu.cfs_quota_us，这连个接口的单位都是微妙，可以将period设置为1秒，将quota设置为0.5，那么cgroup中de
  进程在一秒内最多只能运行0.5秒，然后就会被强制睡眠
  实时进程的CPU带宽限制
  cpu.rt_period_us cpu.rt_runtime_us
  
  Cpuacct
  cpuaccut子系统用来统计各个Cgroup的CPU 使用情况，如下接口：
  cpuact.stat 报告这个Cgroup分别在用户态和内核态消耗的CPU时间，单位是USER_HZ，USER_HZ在x86上一般是100，即USER_HZ 等于0.01s
  cpustat.usage 报告这个Cgroup 消耗的总CPU 时间，单位是纳秒
  -Mmemory 子系统
  memory子系统用来限制Cgroup所能使用的内存上限
  接口 memort.limit_in_bytes
  echo 1G > memory.limit_in_bytes
  mem.memsw.limit_in_bytes ：设定内存加上交换分区使用的总量
  memory.oom_control：设置为0，那么在内存使用到达上限后，不会杀死进程，而是阻塞进程直到有内存释放后可以提供使用
  memory.stat ： 汇报内存使用情况
  -blkio 子系统
  blkio.weight 设置权重范围
  blkio.weight_device
  对具体的设备设置权重，这个值会覆盖上述的blkio.weight。
  例如：/dev/sda
  echo 8:0 100 > blkio.weiht_device
  
  blkio.throttle.read_bps_device: 对具体的设备，设置每秒磁盘带宽上限，例如 /dev/sda 上限设置为 1MB/s
  # echo “8:0 1048576” > blkio.throttle.read_bps_device
  blkio.throttle.write_bps_device
  blkio.throttle.read_iops
  blkio.throttle.write_iops
  
  -Device 子系统
  
  devices子系统是用来控制Cgroup的进程对那些设备有访问权限。
  device.list 只读文件，显示目前允许被访问的设备列表：
   -- 类型： 可以是a  b c ，分别表示所有设备 字符设备 块设备
   -- 设备号 ：格式为major:minor
   -- 权限： r w m 可读 可写 可创建
   "a *:* rmw" 表示所有设备都可以被访问
   “c 1:3 r" 
   
   2.4 Namespace
   Namespace是将内核全部资源做封装。目前Linux内核总共实现了6种NameSpace：
   IPC
   NETWORK
   mount
   pid
   uts
   user
   
  
  
  
  
  
  
  
  
   
   


}   

    
