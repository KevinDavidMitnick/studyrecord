## 使用cloud-init方式
	1. 类似aws方法，可参考openstack中对cloud-init的使用，可完成诸如密码诸如，hostname修改，ssh key注入等功能
    
   2. 可用户镜像首次使用时，客户对虚拟机的自定义配置
   
##  使用qemu-guest-ga 软件
	https://github.com/aspirer/study/tree/master/nvs_monitor/kvm-monitor如下是一个开源地址，用于监控虚拟机内部信息 
    qemu guest agent相关patch（获取磁盘设备的realpath，以及获取指定目录所在磁盘的空间信息）：
	https://github.com/aspirer/study/blob/master/qemu-guest-agent/statvfs-realpath.patch	
	如果你的虚拟机是debian os，并且在虚拟机内安装了1.5.0+dfsg-5版本的qemu guest agent，可以直接用下面这个可执行的二进制文件替换	掉/usr/sbin/qemu-ga，即可使用这两个patch的功能。
	https://raw.github.com/aspirer/study/master/qemu-guest-agent/qemu-ga.statvfs.realpath

	支持的监控项包括：
	<metric name="cpuUsage" unit="Percent"/>
	<metric name="memUsage" unit="Megabytes"/>
	<metric name="networkReceive" unit="Kilobytes/Second"/>
	<metric name="networkTransfer" unit="Kilobytes/Second"/>
	<metric name="diskUsage" unit="Megabytes"/>
	<metric name="diskReadRequest" unit="Count/Second"/>
	<metric name="diskWriteRequest" unit="Count/Second"/>
	<metric name="diskReadRate" unit="Kilobytes/Second"/>
	<metric name="diskWriteRate" unit="Kilobytes/Second"/>
	<metric name="diskReadDelay" unit="Milliseconds/Count"/>
	<metric name="diskWriteDelay" unit="Milliseconds/Count"/>
	<metric name="diskPartition" unit="all partitions infos"/>
	<metric name="loadavg_5" unit="Percent"/>
	<metric name="memUsageRate" unit="Percent"/>
    
## 使用qemu-guest-agent（qga）
	原理分析：　　qga是一个运行在虚拟机内部的普通应用程序（可执行文件名称默认为qemu-ga，服务名称默认为qemu-guest-agent），其目的是实现一种宿主机和虚拟机进行交互的方式，这种方式不依赖于网络，而是依赖于virtio-serial（默认首选方式）或者isa-serial，而QEMU则提供了串口设备的模拟及数据交换的通道，最终呈现出来的是一个串口设备（虚拟机内部）和一个unix socket文件（宿主机上）。

 
　　qga通过读写串口设备与宿主机上的socket通道进行交互，宿主机上可以使用普通的unix socket读写方式对socket文件进行读写，最终实现与qga的交互，交互的协议与qmp（QEMU Monitor Protocol）相同（简单来说就是使用JSON格式进行数据交换），串口设备的速率通常都较低，所以比较适合小数据量的交换。
 
QEMU virtio串口设备模拟参数：
/usr/bin/kvm(QEMU) \
……\
-device virtio-serial-pci,id=virtio-serial0,bus=pci.0,addr=0x6 \
-device isa-serial,chardev=charserial1,id=serial1 \
-chardev socket,id=charchannel0,path=/var/lib/libvirt/qemu/test.agent,server,nowait \
-device virtserialport,bus=virtio-serial0.0,nr=1,chardev=charchannel0,id=channel0,\
name=com.163.spice.0
 
通过上面的参数就可以在宿主机上生成一个unix socket文件，路径为：/var/lib/libvirt/qemu/test.agent，同时在虚拟机内部生成一个serial设备，名字为com.163.spice.0，设备路径为：/dev/vport0p1，映射出来的可读性比较好的路径为：/dev/virtio-ports/com.163.spice.0，可以在运行qga的时候通过-p参数指定读写这个设备。
也可以通的XML文件来配置这个串口设备：
<channel type='unix'>
   <source mode='bind' path='/var/lib/libvirt/qemu/test.agent'/>
   <target type='virtio' name='com.163.spice.0'/>
 </channel>
 
注意: libvirt-qemu:kvm用户要有权限读写'/var/lib/libvirt/qemu/test.agent'
 
已有功能 
目前qga最新版本为1.5.50，linux已经实现下面的所有功能，windows仅支持加*的那些功能：
 
Ø guest-sync-delimited*：宿主机发送一个int数字给qga，qga返回这个数字，并且在后续返回字符串响应中加入ascii码为0xff的字符，其作用是检查宿主机与qga通信的同步状态，主要用在宿主机上多客户端与qga通信的情况下客户端间切换过程的状态同步检查，比如有两个客户端A、B，qga发送给A的响应，由于A已经退出，目前B连接到qga的socket，所以这个响应可能被B收到，如果B连接到socket之后，立即发送该请求给qga，响应中加入了这个同步码就能区分是A的响应还是B的响应；在qga返回宿主机客户端发送的int数字之前，qga返回的所有响应都要忽略；


Ø guest-sync*：与上面相同，只是不在响应中加入0xff字符；


Ø guest-ping*：Ping the guest agent, a non-error return implies success；


Ø guest-get-time*：获取虚拟机时间（返回值为相对于1970-01-01 in UTC，Time in nanoseconds.）；


Ø guest-set-time*：设置虚拟机时间（输入为相对于1970-01-01 in UTC，Time in nanoseconds.）；


Ø guest-info*：返回qga支持的所有命令；


Ø guest-shutdown*：关闭虚拟机（支持halt、powerdown、reboot，默认动作为powerdown）；


Ø guest-file-open：打开虚拟机内的某个文件（返回文件句柄）；


Ø guest-file-close：关闭打开的虚拟机内的文件；


Ø guest-file-read：根据文件句柄读取虚拟机内的文件内容（返回base64格式的文件内容）；


Ø guest-file-write：根据文件句柄写入文件内容到虚拟机内的文件；


Ø guest-file-seek：Seek to a position in the file, as with fseek(), and return the current file position afterward. Also encapsulates ftell()'s functionality, just Set offset=0, whence=SEEK_CUR；


Ø guest-file-flush：Write file changes bufferred in userspace to disk/kernel buffers；


Ø guest-fsfreeze-status：Get guest fsfreeze state. error state indicates；


Ø guest-fsfreeze-freeze：Sync and freeze all freezable, local guest filesystems；


Ø guest-fsfreeze-thaw：Unfreeze all frozen guest filesystems；


Ø guest-fstrim：Discard (or "trim") blocks which are not in use by the filesystem；


Ø guest-suspend-disk*：Suspend guest to disk；


Ø guest-suspend-ram*：Suspend guest to ram；


Ø guest-suspend-hybrid：Save guest state to disk and suspend to ram（This command requires the pm-utils package to be installed in the guest.）；


Ø guest-network-get-interfaces：Get list of guest IP addresses, MAC addresses and netmasks；


Ø guest-get-vcpus：Retrieve the list of the guest's logical processors；


guest-set-vcpus：Attempt to reconfigure (currently: enable/disable) logical processors inside the guest。


功能扩展方式
 
    qga功能扩展十分方便，只需要在qapi-schema.json文件中定义好功能名称、输入输出数据类型，然后在commands-posix.c里面增加对应的功能函数即可，下面的补丁即在qga中增加一个通过statvfs获取虚拟机磁盘空间信息的功能：
 

