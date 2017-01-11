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
    
        
 ## 修改配置文件
 	
	[program:bandwidth]
	command=python26 /usr/local/bin/bandwidth.sh  ;需要执行的命令wd)
	user =root  ;(default  is  current  user , required  if  root)
	autostart=true  ;start at supervisord start (default: true)
	autorestart=true  ;whether/when to restart (default: unexpected)
	startsecs=3  ;number of secs prog must stay running ( def . 1)
	stderr_logfile=/tmp/bandwidth_err.log  ;redirect proc stderr to stdout (default false) 错误输出重定向
	stdout_logfile=/tmp/bandwidth.log  ;stdout log path, NONE  for  none; default AUTO, log输出
	(更多配置说明请参考：http://supervisord.org/configuration.html)

 ## 运行命令
 	
    supervisord -c /etc/supervisord.conf //启动supervisor
    supervisorctl                //打开命令行
    supervisorctl status           //查看状态
    如果修改了配置文件/etc/supervisord.conf,需要执行supervisorctl reload来重新加载配置文件
    
  ## 管理自启动脚本(也可以在github上下载)
#!/bin/bash
#
# supervisord   This scripts turns supervisord on
#
# Author:       Mike McGrath <mmcgrath@redhat.com> (based off yumupdatesd)
#
# chkconfig: - 95 04
#
# description:  supervisor is a process control utility.  It has a web based
#               xmlrpc interface as well as a few other nifty features.
# processname:  supervisord
# config: /etc/supervisord.conf
# pidfile: /var/run/supervisord.pid
#

# source function library
. /etc/rc.d/init.d/functions

RETVAL=0
OPTIONS="-c /etc/supervisord.conf"

start() {
echo -n $"Starting supervisord: "
daemon supervisord $OPTIONS
RETVAL=$?
echo
[ $RETVAL -eq 0 ] && touch /var/lock/subsys/supervisord
}

stop() {
echo -n $"Stopping supervisord: "
killproc supervisord
echo
[ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/supervisord
}

restart() {
stop
start
}

case "$1" in
  start)
start
;;
  stop) 
stop
;;
  restart|force-reload|reload)
restart
;;
  condrestart)
[ -f /var/lock/subsys/supervisord ] && restart
;;
  status)
status supervisord
RETVAL=$?
;;
  *)
echo $"Usage: $0 {start|stop|status|restart|reload|force-reload|condrestart}"
exit 1
esac

exit $RETVAL

