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


