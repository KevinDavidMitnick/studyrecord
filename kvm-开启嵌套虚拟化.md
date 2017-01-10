1. 确认主机是否支持nested
	lscpu 发现是否有vmx特性，有就能支持。如果要开启还需要在bios中将VT设置为enabled
      
2. 启嵌套虚拟化的方法（两种方法都可）
 	在grub中的kernel那行末尾加上kvm-intel.nested=1作为启动参数加入。
 		
   将 options kvm-intel nested=1 加在文件 /etc/modprobe.d/kvm-intel.conf的末尾
   临时加载模块的办法: rmmod kvm_intel   modprobe kvm-intel nested=1 
   注意:如果是esxi的嵌套虚拟化(使用esxi6.0安装)，需要做如下设置：
	echo 1 > /sys/module/kvm/parameters/ignore_msrs
	cat /sys/module/kvm/parameters/ignore_msrs
	Y
 
