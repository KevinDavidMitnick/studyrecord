## 删除 MDS步骤
	1. 停止cpeh mds 进程
    
    service ceph stop mds
    
    2. 将mds状态标记为失效
    
    ceph mds fail 0 
    
    3. 删除mds
    
    ceph mds rm 0 mds.cloud1
    
    4. 删除cephfs文件系统
    
    ceph fs ls 查看文件系统
    
    ceph fs rm cephfs --yes-i-really-mean-t 
    
    5. 检查状态ceph -w，已经删除mds
    
## 手动停止ceph的自动数据恢复和回填
	 ceph osd set nobackfill
    
    ceph osd set norecover
    
    ceh osd set noout 
    
    同时参考链接http://docs.ceph.org.cn/rados/troubleshooting/troubleshooting-osd/
    
## benchmark测试ceph性能(测试吞吐)
	测试写       rados bench -p bak-t-306d3dc79398474182011aa1ea03047b 10 write --no-cleanup
   
	测试顺序读     rados bench -p bak-t-306d3dc79398474182011aa1ea03047b 10 seq
   
  	测试随机读   rados bench -p bak-t-306d3dc79398474182011aa1ea03047b 10 rand
    
	删除 之前的测试   rados -p bak-t-306d3dc79398474182011aa1ea03047b cleanup --prefix benchmark
    
 ## FIO 测试性能（测试IOPS）
 	4K随机写测试IOPS，测试写IOPS  fio -ioengine=libaio -bs=4k -direct=1 -thread -rw=randwrite -size=30G -filename=/dev/vdb -name="EBS 4KB randwrite test" -iodepth=32 -runtime=60
    
	4K随机读测试，测试读IOPS fio -ioengine=libaio -bs=4k -direct=1 -thread -rw=randread -size=30G -filename=/dev/vdb -name="EBS 4KB randwrite test" -iodepth=16 -runtime=60
 	
	512K顺序读，测试磁盘读吞吐
   fio -ioengine=libaio -bs=512k -direct=1 -thread -rw=write -size=30G -filename=/dev/vdb -name="EBS 512KB seqwrite test" -iodepth=64 -runtime=60
   
   512K顺序写，测试磁盘写吞吐
   fio -ioengine=libaio -bs=512k -direct=1 -thread -rw=write -size=40G -filename=/dev/vdb -name="EBS 512KB seqwrite test" -iodepth=64 -runtime=60
   
## 查询ceph状态和基础信息
 	ceph health 或者ceph health detail 
    
    ceph -s 查看统计状态
    
    ceph -w 查看实时状态
    
## 查询ceph存储空间
 
 	ceph df 看到每个池的使用情况
    
    rados df 看到池使用详细情况
    
## 查看文件系统osd的时间延迟

	ceph osd perf 
    
## 查看ceph池状态，并设置副本数

	ceph osd dump 
  
  	设置池副本数位2   ceph osd pool set bak-t-6090d4d547264a57a7d1627835d54320  size 2
  
  	设置池最小副本数1  ceph osd pool set bak-t-6090d4d547264a57a7d1627835d54320  min size 1
    
## 设置pg过多问题

	ceph tell osd.* '--mon_pg_warn_max_per_osd 401 '用于设置pgs过多问题
    
## 查看ceph存储空间使用情况

	ceph df 或者ceph df detail 可以看到每个池的使用情况
    
   rados df 可以看到池使用详细信息
   
   ceph osd df 可以看到每个osd的使用情况（某个osd使用超过80%，则需妥善处置）
   
## 删除所有节点和ceph相关的软件包

	ceph-deploy uninstall ceph
    
    ceph-deploy purge node
    
    ceph-de[ploy purgedata node
    
## 查看mon状态信息

	ceph mon stat
    
## 查看mon的选举信息

	ceph quorum_status
    
## 查看mon的映射信息

	ceph mon dump
    
## 删除一个mon
	
    ceph mon remove node1

## 获得一个正在运行的mon map 

	ceph mon getmap -o map.txt
    
## 查看获得的mon map 信息

	monmaptool --print map.txt
    
## 把mon map 注入到新加的节点

	ceph-mon -i node4 --inject-monmap map.txt
 
## 查看mon的admin_socket 

	 ceph-conf --name mon.node1 --show-config-value admin_socket   /var/run/ceph/ceph-mon.node1.asok 
    
## 查看某个mon的详细状态

	ceph daemon mon.node1  mon_status
	
## 查看mds状态

	ceph mds stat
    
## 查看mds 的映射信息

	ceph mds dump
    
## 删除一个mds节点
	
    ceph mds rm 0 mds.node1
    
## 查看osd 运行状态

	ceph osd stat 
    
## 查看osd映射信息

	ceph osd dump 
	
## 查看osd的目录树

	ceph osd tree
    
## 设置osd的权重
  	
    ceph osd crush reweight 0sd.3 1.0
    
## 把一个osd节点逐出集群

	ceph osd out osd.3
    
## 把一个osd加入集群

	ceph  osd in osd.3
    
## 停止osd

	ceph osd pause
    
##  再次开启osd

	ceph osd unpause
    
## 查看pg组的映射关系

	ceph pg dump 或者 ceph pg dump --format plain
    
## 查看一个pg的map

	ceph pg map 0.3f
    
 ## 查看pg状态
 	
    ceph pg stat 
    
 ## 查询一个pg的详细信息
 
 	cep pg 0.35 query
    
 ## 查看pg中的stuck状态
 
 	ceph pg dump_stuck inactive|unclean|stale
  
 ## 显示一个集群中所有pg的统计信息
 
 	ceph pg dump --format plain
    
 ## 恢复一个丢似乎都pg
 
 	ceph pg 0.35 mark_unfound_lost revert
    
 ## 查看 ceph 集群中的pool数量
 
 	ceph osd lspools 或者 ceph osd pool ls 
    
 ## 在ceph集群中创建一个pool
 
 	ceph osd pool create test 100 100 
    
 ## 为一个ceph pool 配置配额
 
 	ceph osd pool set-quota data max_objects 10000
    
 ## 在集群中删除一个pool
 
 	ceph osd pool delete test test --yes-i-really-really-mean-it 
    
 ## 显示集群中pool的详细信息
 
 	rados df 
    
 ## 给一个pool创建快照
 
 	ceph osd pool mksnap data snap_test
    
 ## 删除pool的快照
 
 	ceph osd pool rmsnap data snap_test
    
 ## 查看data池的pg数量
 
 	ceph osd pool get data pg_num
    
 ## 设自豪data池的最大存储空间为1T
 
 	 ceph osd pool set data target_max_bytes 100000000000000
     
 ## 设置data池的副本数是3
 
 	ceph osd pool set data size 3 
    
 ## 设置data池能接受写操作的最小副本为2
 
 	ceph osd pool set data min_size 2 
    
 ## 查看集群中所有pool的副本尺寸
 
 	ceph osd dump | grep 'replicated size'
    
 ## 设置一个pool的pg数量
   
    ceph osd pool set data pg_num 100 
    
 ## 设置一个pool的pgp数量
 
 	ceph osd pool set data pgp_num 100 
    
## 使用rados创建一个池
 
 	rados mkpool test
    
## 查看ceph pool 中的ceph object 

	rados ls -p volumes | more 
    
## 删除rados中的某个对象

	rados rm test-object -p test
    
## 查看一个pool中的所有镜像

	rbd ls images 或者 rbd ls volumes
    
 ## 查看ceph的pool中某个镜像信息
 
	rbd info -p images --image 74cb427c-cee9-47d0-b467-af217a67e60a
    或者 rbd info rbd/testimage
    
 ## 给一个镜像创建一个快照
 
 	rbd snap create test/myimage@myimage
    
## 查询镜像的快照

	rbd snap ls -p test myimage 或者 rbd info test/myimage@myimage
    
## 删除一个镜像文件的一个快照  

	rbd snap rm test/myimage@myimage 

## 解除镜像保护

	rbd snap unprotect test/myimage@myimage
    
## 把ceph pool 中的镜像导出
	
   rbd export -p images --image 74cb427c-cee9-47d0-b467-af217a67e60a /root/aaa.img
	
## 把一个镜像导入ceph

	rbd import /root/aaa.img -p images --image 74cb427c-cee9-47d0-b467-af217a67e60a  
	
 	

   
   
