## 介绍：
	 supervisor 是一款进程管理工具，当进程因为各种原因而中断的时候，它可以自动启动该进程。
    
    这个工具主要两个命令:
   
    supervisord:    supervisor的服务端部分，启动supervisor就是用这个命令
   
    supervisorctl:   启动supervisor的命令行窗口个
   
## 安装:
	 1. yum install -y python-setuptools
     
    2. easy_install supervisor (或者pip install supervisor,或者通过python setup.py install 安装supervisor源码包)
    
 ## 生成配置文件:
 	 1.  echo_supervisord_conf > /etc/supervisord.conf
    在supervisord.conf中最后增加
     
     
     
     