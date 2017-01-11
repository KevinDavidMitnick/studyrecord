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
 	
   512K顺序读，测试吞吐
   fio -ioengine=libaio -bs=512k -direct=1 -thread -rw=write -size=30G -filename=/dev/vdb -name="EBS 512KB seqwrite test" -iodepth=64 -runtime=60
    
    
