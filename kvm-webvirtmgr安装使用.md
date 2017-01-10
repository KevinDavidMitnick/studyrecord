生产环境的KVM宿主机越来越多，需要对宿主机的状态进行调控。这里用webvirtmgr进行管理。图形化的WEB，让人能更方便的查看kvm 宿主机的情况和操作
（1）配置解析主机名
修改所以节点的/etc/hosts文件，是所以节点之间能够通过dns解析主机名。
（2）客户端安装
1）安装KVM和Libvirt组件（略）。
2）Libvirtd服务监听配置
修改/etc/sysconfig/libvirtd文件，去掉下面一行的注释，使Libvirt服务处于监听状态：
vim /etc/sysconfig/libvirtd
LIBVIRTD_ARGS="--listen"
3）配置Libvirt服务
配置Libvirt服务，允许通过tcp方式通讯，修改/etc/libvirt/libvirtd.conf：
#允许tcp监听
listen_tcp = 1
#开放tcp端口
tcp_port = "16509"
#监听地址修改为0.0.0.0
listen_addr = "0.0.0.0"//如果不行，改为服务器本机地址，例如“192.168.0.52”
#配置tcp通过sasl认证
auth_tcp = sasl
启动服务：
service libvirtd start
4）创建libvirt管理用户(用于TCP连接)
saslpasswd2 -a libvirt virtadmin
可以使用sasldblistuser2命令查看创建了那些用户：
sasldblistusers2 -f /etc/libvirt/passwd.db
virtadmin@webvirt: userPassword
如果需要禁止用户，使用如下命令：
saslpasswd2 -a libvirt -d virtadmin


使用virsh -c qemu+tcp://IP_address/system nodeinfo
来测试TCP连接性
（3）服务器端安装
 1 安装支持的软件源
yum -y install http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
2 安装相关软件
yum -y install git python-pip libvirt-python libxml2-python python-websockify supervisor nginx
3 从git-hub中下载相关的webvirtmgr代码
cd /usr/local/src/
git clone git://github.com/retspen/webvirtmgr.git
4 安装webvirtmgr
cd webvirtmgr/
pip install -r requirements.txt
5 安装数据库
yum install python-sqlite2
6 对django进行环境配置
 ./manage.py syncdb
You just installed Django's auth system, which means you don't have any superusers defined.
Would you like to create one now? (yes/no): yes
Username (leave blank to use 'root'): admin
Email address: 2733176200@qq.com
Password:*********
Password (again):*********
 
 ./manage.py collectstatic #生成配置文件
./manage.py createsuperuser #添加管理员账号
 
7 拷贝web到 相关目录
cd ..
mkdir -pv /var/www
cp -Rv webvirtmgr /var/www/webvirtmgr
 
8 设置ssh
ssh-keygen
ssh-copy-id 192.168.2.32
ssh 192.168.2.32 -L localhost:8000:localhost:8000 -L localhost:6080:localhost:6080
 
9 编辑nginx配置文件
vim /etc/nginx/conf.d/webvirtmgr.conf 添加下面内容到文件中
server {
    listen 80 default_server;
 
    server_name $hostname;
    #access_log /var/log/nginx/webvirtmgr_access_log;
 
    location /static/ {
        root /var/www/webvirtmgr/webvirtmgr; # or /srv instead of /var
        expires max;
    }
 
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Forwarded-Proto $remote_addr;
        proxy_connect_timeout 600;
        proxy_read_timeout 600;
        proxy_send_timeout 600;
        client_max_body_size 1024M; # Set higher depending on your needs
    }
}
 
 
mv /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.bak
 
10 启动nginx
/etc/init.d/nginx restart
 
11 修改防火墙规则
/usr/sbin/setsebool httpd_can_network_connect true
 
 
12 设置 supervisor 
chown -R nginx:nginx /var/www/webvirtmgr
vim /etc/supervisord.conf #在文件末尾添加
[program:webvirtmgr]
command=/usr/bin/python /var/www/webvirtmgr/manage.py run_gunicorn -c /var/www/webvirtmgr/conf/gunicorn.conf.py
directory=/var/www/webvirtmgr
autostart=true
autorestart=true
logfile=/var/log/supervisor/webvirtmgr.log
log_stderr=true
user=nginx
 
[program:webvirtmgr-console]
command=/usr/bin/python /var/www/webvirtmgr/console/webvirtmgr-console
directory=/var/www/webvirtmgr
autostart=true
autorestart=true
stdout_logfile=/var/log/supervisor/webvirtmgr-console.log
redirect_stderr=true
user=nginx
 
 
修改/var/www/webvirtmgr/conf/gunicorn.conf.py 
bind = "0:8000"
 
13 设置开机启动
chkconfig supervisord on
vim /etc/rc.local
/usr/sbin/setsebool httpd_can_network_connect true
 
 
 
14 启动进程
/etc/init.d/supervisord restart
 
15查看进程
netstat -lnpt 即可以看到6080和8000已经启动
 
16 web访问
http://192.168.0.52/login/

如果403错误，根据nginx日志，添加防火墙规则：
sudo iptables -I INPUT -m state --state NEW -m tcp -p tcp --dport 16509 -j ACCEPT

同时，重新设置chown nginx:nginx /var/www/webxxxxx，


wget -O - http://retspen.github.io/libvirt-bootstrap.sh | sudo sh 添加bootstrap

