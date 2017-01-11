## Zabbix 简介
	zabbix是一个基于WEB界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案。zabbix能监视各种网络参数，保证服务器系统的安全运营；并提供灵活的通知机制以让系统管理员快速定位/解决存在的各种问题。
    
## Zabbix 安装
	本文档在同一台机器上安装了zabbix-server 和zabbix-agent 自己监控自己。
    
    1. 安装配置LAMP（http://www.cnblogs.com/xqzt/p/5123748.html）
    
    2. zabbix配置yum源和ZabbixZone package repository and GPG key
    
    	yum install epel-release
        
    	rpm --import http://repo.zabbix.com/RPM-GPG-KEY-ZABBIX  
        
    	rpm -Uv http://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/zabbix-release-3.2-1.el7.noarch.rpm
    
    3. 安装Zabbix server and agent(agent是可选的)
    
    	yum install zabbix-server-mysql zabbix-web-mysql zabbix-agent zabbix-java-gateway
        
      - 编辑/etc/httpd/conf.d/zabbix
      
      vi /etc/httpd/conf.d/zabbix.conf
      
      - 更新时区
      
      php_value date.timezone Asia/Shanghai
      
      重启httpd
      
      systemcl restart httpd
      
    4. 创建MySQL 数据库和用户
      
      	- 登录mariadb 
        
        mysql -u root -p 
        
        - 创建一个数据库zabbixdb和数据库用户zabbix
        
        create database zabbix character set utf8;
        
        grant all privileges on zabbix.* to 'zabbix'@'localhost' identified by 'zabbix';
        
        flush privileges;
        
    5. 数据库导入zabbix template
      
      	- 使用数据库用户zabbix登录数据库
        
        mysql -u zabbix -p && use zabbix
        
       - 导入数据库模板
       	source /usr/share/doc/zabbix-server-mysql-2.4.7/create/schema.sql
        
        source /usr/share/doc/zabbix-server-mysql-2.4.7/create/images.sql
        
        source /usr/share/doc/zabbix-server-mysql-2.4.7/create/data.sql
        
    6. 配置zabbix server 
    	- 编辑文件 vim /etc/zabbix/zabbix_server.conf
        
      -  配置如下参数:
        DBName=zabbix
		   DBUser=zabbix
	     DBPassword=zabbix

    7. 配置 zabbix-agent
    
    	- 编辑文件  vim /etc/zabbix/zabbix_agentd.conf 
        
      -  配置文件zabbix server的ip
      	 Server=127.0.0.1
         
        ServerActive=127.0.0.1
        
        Hostname=127.0.0.1
        
     8. 修改php设置
     	修改php.ini为zabbix建议的设置，编辑文件/etc/php.ini
        
        设置参数如下:
        max_execution_time = 600

			max_input_time = 600

		 	memory_limit = 256

			Mpost_max_size = 32M

			upload_max_filesize = 16M

			date.timezone = Asia/Shanghai
            
      9. 修改firewall和selinux设置
      	开放zabbix端口10050和10051
        
        firewall-cmd --permanent --add-port=10050/tcp 
        
        firewall-cmd --permanent --add-port=10051/tcp
        
        systemctl restart firewalld
        
        setsebool -P httpd_can_connect_zabbix=1(如果使用selinux，需让apache和zabbix通过)
        
      10. 允许zabbix web console 对特定IP段可用
      	 编辑文件vim /etc/httpd/conf.d/zabbix.conf添加允许访问 zabbix web interface的ip段. 如果设置 ‘Allow from All’, 这可以允许全部可以访问
         
        Alias /zabbix /usr/share/zabbix 
        
        <Directory "/usr/share/zabbix"> 
        
        Options FollowSymLinks 
        
        AllowOverride None 
        
        Require all granted 
        
        	<IfModule mod_php5.c> 
            
            php_value max_execution_time 300 
            
            php_value memory_limit 128M php_value 
            
            post_max_size 16M 
            
            php_value upload_max_filesize 2M 
            
            php_value max_input_time 300 
            
            php_value date.timezone Asia/Shanghai 
            
            </IfModule>
            
        </Directory>
        
        <Directory "/usr/share/zabbix/conf"> 
        
        	Require all denied 
            
        </Directory> 
        
        <Directory "/usr/share/zabbix/include"> 
        
        	Require all denied 
            
        </Directory>
        
      11. 启动zabbix-server 和zabbix-agent。重启httpd,，并设置zabbix-server和zabbix-agent开机自动启动
         systemctl start zabbix-server
         
         systemctl start zabbix-agent

         systemctl restart httpd
         
         systemctl restart mariadb
         
         systemctl enable zabbix-server
         
         systemctl enable zabbix-agent ---- (可选)
         
      12. 通过控制台访问zabbix
      	浏览器访问http://ip-address/zabbix,默认账号和密码是admin : zabbix
      		
      
      
      
       	
      
      