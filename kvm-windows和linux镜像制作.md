## linux 镜像制作
1. 首先在http://mirrors.aliyun.com上选择操作系统版本，尽量选择minimal版本的镜像

2. 使用virt-manager，和刚才下载的镜像，创建一个虚拟机，命名为base.

3. 默认安装后的虚拟机位置在/var/lib/libvirt/images下面，也可能在自定义的别的位置。

4. 安装虚拟机之后，需要安装客户端如下:spice-vdagent 和qxl驱动(这两者自带在spice-tools中。)，利用virtmanager添加virtio类型的磁盘和网卡驱动，即可完成virtio网卡安装。

5. 使用virt-sysprep -d vm1 对linux进行封装，在这之前，可以安装自己喜欢的镜像。

6. 使用virt-sparsify --compress base.qcow2  base.img 生成base.img镜像文件，使用virt-xxx命令可能需要安装的工具包如下:libguestfs-tools

## Windows 镜像制作
1. 步骤同linux镜像制作的1-4步骤。

2. 实现windows开机修改密码功能（需要安装yum install libguestfs-winsupport）。
开始输入gpedit.msc进入组策略，选择计算机配置，脚本(启动/关机)选项中的启动，它会自动创建如下目录：
   C:\Windows\System32\GroupPolicy\Machine\Scripts\Startup
  添加start.bat文件，以生成相应的注册表信息，之后删除start.bat文件。
  
3. 开启远程桌面连接，校对time服务（xml中需要配置为localtime格式，才是东八区）。

6. 安装其他必须软件，然后采用vmware-view-virtual-desktops-windows-optimization对windows镜像进行服务精简和优化。

7. 采用sysprep对镜像进行封装。

8. 使用virt-sparsify对镜像进行稀疏文件压缩
