## **CPU优化方案**
1. 采用巨型内存页技术
 	- x86默认内存页面大小是4KB，但是也可以使用2MB或者1GB的巨型页。系统的巨型页面可以传到虚拟机中，kvm虚拟机可以通过分配巨型页提高性能。大大提高了CPU cache中存放的page table所覆盖的内存大小，从而提高了TLB命中率。进程的虚拟内存地址段先连接到page tables然后再连接到物理内存。所以在访问内存时需要先访问page tables得到虚拟内存和物理内存的映射关系，然后再访问物理内存。CPU cache中有一部分TLB（Translation Lookaside Buffer）用来存放部分page table以提高这种装换的速度。因为page size变大了，所以同样大小的TLB，所覆盖的内存大小也变大了。提高了TBL命中率，也就是提高了地址转换的速度。
  系统进程是通过虚拟地址访问内存，但是CPU必须把它转换程物理内存地址才能真正访问内存。为了提高这个转换效率，CPU会缓存最近的虚拟内存地址和物理内存地址的映射关系，并保存在一个由CPU维护的映射表中。为了尽量提高内存的访问速度，需要在映射表中保存尽量多的映射关系。
    centos7默认启用了巨型页，查看状态：
    cat /sys/kernel/mm/transparent_hugepage/enabled
    [always] madvise never
    临时修改配置以开启巨型内存页:echo never  >/sys/kernel/mm/transparent_hugepage/enabled 
    参数说明如下：
	never：关闭，不使用透明内存。
	alway：尽量使用透明内存，扫描内存，有512个4KB页面可以整合，就整合成一个2MB的页面。
	madvise：避免改变内存占用。
    
  - 使用情况监控：
	可以查看/sys/kernel/mm/transparent_hugepage/khugepaged下的信息。
	pages_to_scan（默认4096=16MB）：一个扫描周期被扫描的内存页数。
	scan_sleep_millisecs（默认10?000=10sec）：多长时间扫描一次。
	alloc_sleep_millisecs（默认60?000=60sec）：多长时间整理一次碎片。
 	也可以查看/proc/meminfo信息(未使用巨型内存页的时候如下:)
 	1. grep Huge /proc/meminfo  
	2. AnonHugePages:    266240 kB  
	3. HugePages_Total:       0  
	4. HugePages_Free:        0  
	5. HugePages_Rsvd:        0  
	6. HugePages_Surp:        0  
	7. Hugepagesize:       2048 kB 

  -  修改宿主机巨型页数量:
    1. 巨型页默认大小是2MB，可以通过如下命令修改可以使用的巨型页数量：
	 echo 25000 > /proc/sys/vm/nr_hugepages 
	 2. 或者使用sysctl命令，N是要设置的巨型页数量：
	 sysctl vm.nr_hugepages=N （这里的N值设置为=虚拟机所需内存总和/页面大小）
    3. 挂载巨型页:mount -t hugetlbfs hugetlbfs /dev/hugepages
    4. 重启libvirt服务和虚拟机:systemctl restart libvirtd && virsh start base
    
  - 在虚拟机xml文件中添加如下配置,以启动巨型内存页
    <memoryBacking>
    	<hugepages/>
    </memoryBacking>
  - 启动虚拟机之后，再次验证巨型内存页面是否开启:
  
	cat /proc/meminfo | grep Huge  
	2. AnonHugePages:      2048 kB  
	3. HugePages_Total:       0  
	4. HugePages_Free:        0  
	5. HugePages_Rsvd:        0  
	6. HugePages_Surp:        0  
	7. Hugepagesize:       2048 kB 




   
