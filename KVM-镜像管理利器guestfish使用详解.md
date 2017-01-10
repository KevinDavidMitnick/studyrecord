1. KVM虚拟化与guestfish套件我们来看看guestfish套件是什么
guestfish是一套虚拟机镜像管理的利器，提供一系列对镜像管理的工具，也提供对外的API。

	guestfish主要包含以下工具：

	guestfish interactive shell  挂载镜像，并提供一个交互的shell。
    
	guestmount mount guest filesystem in hos 将镜像挂载到指定的目录。
    
	guestumount unmount guest filesystem 卸载镜像目录。
    
	virt-alignment-scan 镜像块对齐扫描。
    
	virt-builder ― quick image builder 快速镜像创建。
    
	virt-cat(1) ― display a file 显示镜像中文件内容。
    
	virt-copy-in(1) ― copy files and directories into a VM 拷贝文件到镜像内部。
    
	virt-copy-out(1) ― copy files and directories out of a VM 拷贝镜像文件出来。
    
	virt-customize(1) ― customize virtual machines 定制虚拟机镜像
    
	virt-df(1) ― free space 查看虚拟机镜像空间使用情况。
    
	virt-diff(1) ― differences 不启动虚拟机的情况下，比较虚拟机内部两份文件差别。
    
	virt-edit(1) ― edit a file 编辑虚拟机内部文件。
    
	virt-filesystems(1) ― display information about filesystems, devices, LVM 显示镜像文件系统信息。
    
	virt-format(1) ― erase and make blank disks 格式化镜像内部磁盘。
    
	virt-inspector(1) ― inspect VM images 镜像信息测试。
    
	virt-list-filesystems(1) ― list filesystems 列出镜像文件系统。
    
	virt-list-partitions(1) ― list partitions 列出镜像分区信息。
    
	virt-log(1) ― display log files 显示镜像日志。
    
	virt-ls(1) ― list files 列出镜像文件。
    
	virt-make-fs(1) ― make a filesystem 镜像中创建文件系统。
    
	virt-p2v(1) ― convert physical machine to run on KVM 物理机转虚拟机。
    
	virt-p2v-make-disk(1) ― make P2V ISO 创建物理机转虚拟机ISO光盘。
    
	virt-p2v-make-kickstart(1) ― make P2V kickstart 创建物理机转虚拟机kickstart文件。
    
	virt-rescue(1) ― rescue shell 进去虚拟机救援模式。
    
	virt-resize(1) ― resize virtual machines 虚拟机分区大小修改。
    
	virt-sparsify(1) ― make virtual machines sparse (thin-provisioned) 镜像稀疏空洞消除。
    
	virt-sysprep(1) ― unconfigure a virtual machine before cloning 镜像初始化。
    
	virt-tar(1) ― archive and upload files 文件打包并传入传出镜像。
    
	virt-tar-in(1) ― archive and upload files 文件打包并传入镜像。
    
	virt-tar-out(1) ― archive and download files 文件打包并传出镜像。
    
	virt-v2v(1) ― convert guest to run on KVM 其他格式虚拟机镜像转KVM镜像。
    
	virt-win-reg(1) ― export and merge Windows Registry keys windows注册表导入镜像。
    
	libguestfs-test-tool(1) ― test libguestfs 测试libguestfs
    
	libguestfs-make-fixed-appliance(1) ― make libguestfs fixed appliance
    
	hivex(3) ― extract Windows Registry hive 解压windows注册表文件。
    
	hivexregedit(1) ― merge and export Registry changes from regedit-format files 合并、并导出注册表文件内容。
    
	hivexsh(1) ― Windows Registry hive shell window注册表修改交互的shell。
    
	hivexml(1) ― convert Windows Registry hive to XML 将window注册表转化为xml
    
	hivexget(1) ― extract data from Windows Registry hive 得到注册表键值。
    
	guestfsd(8) ― guestfs daemon guestfs服务。
    
   virt-df -- 用来查看虚拟机磁盘使用情况

	virt-tar-in -a disk.img data.tar /destination 压缩文件拷贝进虚拟机并解压

	virt-tar-out -a disk.img /dir files.tar	压缩文件拷贝进虚拟机并解压

	virt-tar -x domname /home home.tar	将虚拟机的home目录拷贝出来并打包

	virt-tar -u domname uploadstuff.tar /tmp 上传本地的压缩文件到虚拟机并解压
    
2. guestfish安装与注意事项guestfish套件安装
	guestfish套件安装非常简单，一条命令就可以。
	yum install libguestfs-tools
	注意：
	默认安装是不安装windows系统支持的，如果需要修改windows系统镜像，需要再运行如下命令。
	yum install libguestfs-winsupport

3. 使用guestfish查看虚拟机信息虚拟机镜像信息查看，主要通过virt-inspector和virt-inspector2命令
	virt-inspector - Display OS version, kernel, drivers, mount points, applications, etc. in a virtual machine
    
	virt-inspector 显示os版本、内核、驱动、挂载点、应用等等。
    
	virt-inspector2 - Display operating system version and other information about a virtual machine
    
	virt-inspector2 显示os版本和其他信息。

	也可以通过--query输出一些固定内容，方便脚本判断。
	virt-inspector --query centos5332.qcow2（--xml可以输出为xml格式）
	windows=no
	linux=yes
	rhel=no
	fedora=no
	debian=no
	fullvirt=yes
	xen_domU_kernel=no
	xen_pv_drivers=yes
	virtio_drivers=yes
	kernel_arch=i386
	userspace_arch=i386

4. 使用guestfish查看虚拟机分区及文件系统虚拟机分区及文件系统查看主要使用三个命令
	virt-list-partitions 列出虚拟机镜像文件分区信息   
    
	virt-list-filesystems   列出虚拟机镜像文件文件系统，分区，块设备，lvm信息
    
	virt-alignment-scan	    查看虚拟机镜像分区是否块对齐

5. 去掉磁盘空洞--KVM虚拟镜像的稀疏问题RAW格式和QCOW2
	使用virt-sparsify，去除镜像空洞，语法为virt-sparsify  -x   /root/test.qcow2 --convert qcow2 /root/test2.qcow2

6. 用guestfish操作虚拟机内部文件（如果需要windows的支持，需要安装包libguestfs-winsupport.）：

7. guestfish修改镜像格式和大小修改镜像格式和大小主要使用以下命令
	virt-convert - convert virtual machines between formats	  转化虚拟机镜像格式

	virt-resize - Resize a virtual machine disk  修改虚拟机镜像磁盘

8. lv扩展
virt-resize --expand /dev/sda2 --LV-expand /dev/vg_guest/lv_root \
olddisk newdisk

扩展分区，并将raw格式转化成qcow2格式
qemu-img create -f qcow2 newdisk.qcow2 15G
virt-resize --expand /dev/sda2 olddisk newdisk.qcow2
注意：
1.  如果是扩展分区，目标磁盘文件必须大于原生磁盘；
2.  磁盘缩小比较复杂，一般要求缩小到的空间远大于文件系统的大小。
3.  guestfish挂载、修改、运行救援方式guestmount - Mount a guest filesystem on the host using FUSE and libguestfs
挂载镜像到某个目录
guestfish - the libguestfs Filesystem Interactive SHell
挂载镜像并得到一个交互的shell
virt-rescue - Run a rescue shell on a virtual machine
运行一个镜像的救援模式
示例
只读方式将镜像挂载到/mnt目录
guestmount -a windows.img -m /dev/sda1 --ro /mnt
将linux虚拟机的根目录挂载到宿主机的/mnt目录
guestmount -a linux.qcow2 -m /dev/sda2  /mnt

guestfish编辑镜像grub文件
guestfish --rw --add disk.img \--mount /dev/vg_guest/lv_root \--mount /dev/sda1:/boot \
          edit /boot/grub/grub.conf

进入镜像的救援模式
$ virt-rescue --suggest -d Fedora15

