Docker 

����ʼ�

һ
1.1 Docker ��û�д�ͳ�������Hypervisor �㡣��ΪDocker�ǻ������������������⻯������ڴ�ͳ�����⻯������ʡȥ��Hypervisor��Ŀ��������������⻯����
�ǻ����ں˵�Cgroup��NameSpace�����������߼����ں�����ںϣ������ںܶ෽�棬�������ܷǳ��ӽ��������
��ͨ�ŷ��棬Docker����ֱ�����ں˽�������ͨ��һ�����ӵײ�Ĺ���Libcontainer���ں˽�����Libcontainer�������ײ���������棬��ͨ��cloneϵͳ����ֱ��
����������ͨ��pivot_root ϵͳ���ý�������������ͨ��ֱ�Ӳ���cgroup�ļ�ʵ�ֶ���Դ�Ĺܿء���Docker���������ҵ�����Ĵ���

1.2 Docker������һ�������ǶԵײ㾵��Ĵ���Ӧ�ã�����ͬ���������Թ���ײ��ֻ������ͨ��д���Լ����е����ݺ����Ӿ���㡣�����ľ����ֲ���²�ľ���
��һ�������Ϊ����������ʹ�á�

1.3 ����͹���

docker �ͻ���
docker daemon
docker container
docker image
docker ����
registery
1.4
Docker�ڹ��ܺ͸����У�������һ���������ݡ�Dockerͨ��Libcontainer ʵ���˶������������ڵĹ�����Ϣ���úͲ�ѯ���Լ���غ�ͨ�ŵȹ��ܡ�
1.5 Registry
Registry��һ����ž���Ĳֿ⣬��ͨ���ᱻ�����ڷ����������ƶ�

2.1 
�Դ˶������ĸ��ͣ�����ļ���ϵͳ�ĸ������ͨ������Ĵ���������䴴��ԭ��

����һ��
  pid = clone(fun,stack,flags,clone_arg):
    (flags:CLONE_NEWPID | CLONE_NEWS)|
    CLONE NEW_USER | CLONE_NEWNET
    CLONE_NEWIPC | CLONE_NEWUTS
    
    
�������
   echo $pis > /sys/fs/cgroup/cpu/tasks
   echo $pid > /sys/fs/cgroup/cpuset/tasks
   echo $pid > /sys/fs/cgroup/blkio/tasks
   echo $pid > /sys/fs/cgroup/memory/tasks
   echo $pid > /sys/fs/cgroup/devices/tasks
��������
fun()
{
   pivot_root ("path_of_rootfs",path);
   exec("/bin/bash");
   
   ����һ��ͨ��cloneϵͳ�ĵ��ã�������һ��Namespace��Ӧ��clone flag������һ���µ��ӽ��̣��ý���ӵ���Լ���NameSpace
   ���������������pid д�����Cgroup��ϵͳ�У��������̾��ܵ���Cgroup�Ŀ���
   ����������fun������ͨ��pivot_root ϵͳ�ĵ��ã�ʹ���̽���һ��rootfs��֮����ͨ��exec ϵͳ����
   
   
   
   
   Cgroup
   
   ����Դ��QOS����Щ��Դ��Ҫ���� CPU MEMOTY BLOCK/IO BANDWITH��Cgroup ʵ����һ��ͨ�ý��̵ķֲ��ܣ�����ͬ����Դ�ľ���������Ǹ���Cgroup
   ��ϵͳʵ�ֵġ�
   Cgroup��ԭ���ӿ���ͨ��cgroupfs�ṩ�ģ�����sysfs��һ�������ļ�ϵͳ��
  -Cpuset��ϵͳ
  cpuset.cpus
  cpuset.mems
  ���Խ����̰󶨵�����ĳ��CPU��
  -CPU ��ϵͳ
  �������ƽ��̵�CPU ռ���ʣ�cpu.shares��������cgroupfs�ĺ�Ŀ¼�´�������Cgroup(C1,C2),���ҽ�cpu.shares �ֱ���Ϊ512 1024����ô��C1 C2 ����CPU 
  ��C2 �����C1 �õ���һ����CPU ռ����
  -CPU ��������
  �������ʹ�õĽӿ���cpu.cfs_period_us ��cpu.cfs_quota_us���������ӿڵĵ�λ����΢����Խ�period����Ϊ1�룬��quota����Ϊ0.5����ôcgroup��de
  ������һ�������ֻ������0.5�룬Ȼ��ͻᱻǿ��˯��
  ʵʱ���̵�CPU��������
  cpu.rt_period_us cpu.rt_runtime_us
  
  Cpuacct
  cpuaccut��ϵͳ����ͳ�Ƹ���Cgroup��CPU ʹ����������½ӿڣ�
  cpuact.stat �������Cgroup�ֱ����û�̬���ں�̬���ĵ�CPUʱ�䣬��λ��USER_HZ��USER_HZ��x86��һ����100����USER_HZ ����0.01s
  cpustat.usage �������Cgroup ���ĵ���CPU ʱ�䣬��λ������
  -Mmemory ��ϵͳ
  memory��ϵͳ��������Cgroup����ʹ�õ��ڴ�����
  �ӿ� memort.limit_in_bytes
  echo 1G > memory.limit_in_bytes
  mem.memsw.limit_in_bytes ���趨�ڴ���Ͻ�������ʹ�õ�����
  memory.oom_control������Ϊ0����ô���ڴ�ʹ�õ������޺󣬲���ɱ�����̣�������������ֱ�����ڴ��ͷź�����ṩʹ��
  memory.stat �� �㱨�ڴ�ʹ�����
  -blkio ��ϵͳ
  blkio.weight ����Ȩ�ط�Χ
  blkio.weight_device
  �Ծ�����豸����Ȩ�أ����ֵ�Ḳ��������blkio.weight��
  ���磺/dev/sda
  echo 8:0 100 > blkio.weiht_device
  
  blkio.throttle.read_bps_device: �Ծ�����豸������ÿ����̴������ޣ����� /dev/sda ��������Ϊ 1MB/s
  # echo ��8:0 1048576�� > blkio.throttle.read_bps_device
  blkio.throttle.write_bps_device
  blkio.throttle.read_iops
  blkio.throttle.write_iops
  
  -Device ��ϵͳ
  
  devices��ϵͳ����������Cgroup�Ľ��̶���Щ�豸�з���Ȩ�ޡ�
  device.list ֻ���ļ�����ʾĿǰ�������ʵ��豸�б�
   -- ���ͣ� ������a  b c ���ֱ��ʾ�����豸 �ַ��豸 ���豸
   -- �豸�� ����ʽΪmajor:minor
   -- Ȩ�ޣ� r w m �ɶ� ��д �ɴ���
   "a *:* rmw" ��ʾ�����豸�����Ա�����
   ��c 1:3 r" 
   
   2.4 Namespace
   Namespace�ǽ��ں�ȫ����Դ����װ��ĿǰLinux�ں��ܹ�ʵ����6��NameSpace��
   IPC
   NETWORK
   mount
   pid
   uts
   user
   
  
  
  
  
  
  
  
  
   
   


}   

    
