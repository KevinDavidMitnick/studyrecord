-  **virsh shutdown 关闭虚拟机失效解决**
	
在测试KVM的时候，发现在virsh执行shutdown命令后，虚拟机没有任何反应。只能使用destroy进制强制关闭。在查看shutdown的帮助后发现，shutdown使用发送acpi指令来控制虚拟机的电源，而KVM虚拟机安装linux系统时默认是没有安装acpi服务的，所以并不会做处理，而windows会自动安装acpi服务。
   - 解决办法：
    	在虚拟机安装acpi服务并重启。yum install -y acpid 或者apt-get install acpid
   - 原理分析:
		Acpid是一个用户空间的服务进程，它充当Linux内核与应用程序之间通信的接口，负责将kernel中的电源管理事件转发给应用程序。
		ACPId服务与内核的通信方式：acpid用poll函数挂在/proc/acpi/event文件上。内核在drivers/acpi /event.c中实现了该文件的接口，
		一旦总线事件列表(acpi_bus_event_list)上有电源管理事件发生，内核就会唤醒挂在/proc /acpi/event上的acpid，acpid再					从/proc/acpi/event中读取相应的事件。
		acpid与应用程序的通信方式有两种，
		其一是通过本地socket，其文件名为/var/run/acpid.socket，应用程序只要连接到这个socket上，不用发送任何命令就可以接收到acpid			转发的电源管理事件。
		其二是通过配置文件。在acpid收到来自内核的电源管理事件时，根据配置文件中的规则执行指定的命令。
		ACPId服务配置文件为/etc/acpi/events/power.conf，下面是一个示例:
		event=button/power.*
		action=/sbin/shutdown-hnow
		ACPId服务事件的格式为：
		device_classbus_idtypedata。device_class和bus_id是字符串，type和data是十六制整数。在配置文件中可以使用通配符，来匹配指定的事件。

- virsh iface-bridge eth0 br0 创建网桥(也可以通过brctl 对网桥进行管理)

- virt-install 可以用命令行来创建虚拟机virt-install --connect qemu:///system --virt-type kvm 
	--name rhel5 --ram 512 --disk path=/var/lib/libvirt/images/xxx.img（空）
   ,size=8  --graphics vnc --cdrom /tmp/boot.iso  --os-variant rhel5
	--network bridge=br0,model=virtio  --force


- virsh capabilities 可以返回当前虚拟机支持的平台包括cpu，磁盘等类型。

- virsh uri，用来查看当前主机hypervisor的连接路径（qemu:///system）

- virsh define创建虚拟机，根据定义好的xml格式创建虚拟机而不会自动自动，通过virsh define定义的虚拟机是持久域，只有持久域的虚拟机		才能做本地快照

- virsh create创建虚拟机，会自动启动虚拟机，这种方式创建的虚拟机是临时域，不能用来创建本地快照

- virsh undefine删除虚拟机域，会删除/etc/libvirt/qemu/下的xml配置文件

- qemu-img convert 命令可以用来对镜像格式进行转换，目前可以在vmware，vbox和kvm格式之间互相转换

- qemu-img resize 命令可以用来调整本地虚拟机磁盘大小，但是只能调大，不能调小。

- virsh attach-device $domain scsi1.xml --live --persistent用来对某些设备进行热插拔，设备定义在scsi1.xml文件中（virsh detach-device用来卸载）

- virsh dumpxml 可以用来生成虚拟机的xml配置

- virt-ls 可以列出虚拟机中目录下的文件或目录，需要事先安装包libguestfs-tools

- virt-what 可以用来检测当前系统是不是一个虚拟机,如果不是虚拟机,执行virt-what将不会有任何输出,如果是虚拟机,它会打印一系列关于虚拟机的’facts’(如kvm)

- virt-host-validate 这个命令可以用来检测宿主机是否正确配置以运行虚拟化,如果没有加参数,它会检查它所知道的所有的虚拟化驱动,可选的可以加qemu或lxc做限制

- virt-top 可以用来查看虚拟机内部的磁盘/CPU/网络性能，需要事先安装virt-top命令

- virt-cat 可以用来在宿主机查看虚拟机中的文件,例如virt-cat -a /opt/image/centos2.img /etc/passwd

- virt-edit 可以用vim来编辑虚拟机中的文件

- virt-copy-out 可以把虚拟机中的文件复制到宿主机

- virt-copy-in 用来将文件复制到虚拟机中（如果虚拟机正在运行，此命令可能出错）
