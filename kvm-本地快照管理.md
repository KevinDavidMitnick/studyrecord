## 离线外置快照:
	 命令:qemu-img create -f qcow2 -b centos.qcow2 snapshot.qcow2
     
    说明: -b指定的是原始镜像，快照创建完后，当前目录会生成一个 snapshot.qcow2文件
    可以通过qemu-img info查看快照信息，快照的后端文件（backing file）指向的是centos.qcow2文件
    
    使用新的快照文件（snapshot.qcow2）启动虚拟机，并进行相关操作。之后的操作都将作用在snapshot.qcow2文件上,如果想将快照里的相关操作合并到原始镜像中，可以通过下面命令:qemu-img commit -f qcow2 snapshot.qcow2,此命令执行完后，会将snapshot.qcow2和其对应的后端文件进行合并，合并完后snapshot.qcow2文件依然留存。
    
## 离线内置快照:
	内置snapshot指的是生成快照的相关信息保存在原始镜像中，而非创建新的快照文件。内置快照利用qcow2文件本身的特性进行实现的。(通过打标签的方式实现)
    qemu-img snapshot -c tag1 centos.qcow2,tag1指的是快照标签名称.打完标签后，可以通过qemu-img snapshot -l 命令查看快照信息.qemu-img snapshot -a 1 centos.qcow2这个命令可以用来恢复快照到指定的标签，“1”对应的是tag1的ID。qemu-img snapshot -d 1 centos.qcow2这个命令用来删除指定的快照
    
 ## 在线快照:
 	在QEMU内部有一个Guest Agent Daemon（qemu-guest-agent），用来接收QEMU monitor发来的命令，并执行对应的处理函数。在线快照可以通过QEMU Monitor发来的命令进行。
   1. 查看block信息
		命令：（qemu）info block
		说明：可以看出centos.qcow2对应的设备名为ide0-hd0

	2.进行内置快照
		命令：（qemu）savevm s1
		说明：创建一个内置快照，名为s1

		命令：（qemu）info snapshot
		说明：查看内置快照信息

		命令：（qemu）loadvm s1
		说明：回到s1时刻状态

		命令：（qemu）delvm s1
		说明：删除内置快照s1


	3.进行外置快照
		命令：（qemu） snapshot_blkdev
		说明：为设备blockX创建一个快照文件，创建完以后，可以通过info block查看信息。新的信息中file参数已经变为snapshot文件，并新增了backing_file参数，其值指向原始镜像文件。

		命令：（qemu）commit
		说明：通过commit命令可以将snapshot合并（merge）到源镜像文件里。但是之后继续操作修改的还是snapshot文件，可以继续合并。

## 块拷贝  block copy
	Block Copy指的是虚拟机存储迁移。迁移时，采用Snapshot+Block stream完成存储迁移。首先通过对虚拟机进行在线外部快照，然后通过BlockStream技术合并快照，完成存储热迁移。BlockStream可以将backing file合并至active，与前面提及到的BlockCommit正好相反，可以通过下面命令完成。
	命令：block_stream <设备名>
	说明：可使用info block查看设备信息，发现backing_file文件已经没有，存储已经完全Copy到目的文件了。

参考文献
http://wiki.qemu.org/Features/Snapshots


