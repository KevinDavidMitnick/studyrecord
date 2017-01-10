## recevice.py 和消息队列交互管理虚拟机
#!/usr/bin/env python
#coding=utf8
import pika
import signal
import simplejson
import os
import sys
import fcntl

connection = pika.BlockingConnection(pika.ConnectionParameters('127.0.0.1'))
channel = connection.channel()
#定义routings key,VmCreate,VmStart,VmStop,VmReboot,VmDelete
routings=["VmCreate","VmStart","VmStop","VmReboot","VmDelete"]

#定义消息回调函数
def VmCreate(vm):
    vmname = vm['vmname']
    cmd = "vboxmanage  clonevm trunkey-x86 --snapshot 0 --mode machine --options link --name %s  --register" % vmname
    os.system(cmd)
    print "VmCreate:%s" % vmname

def VmStart(vm):
    vmname = vm['vmname']
    cmd = "vboxmanage startvm %s --type headless" % vmname
    os.system(cmd)
    vmport = vm.get("vmport")
    if vmport:
        cmd = "vboxmanage controlvm  %s  vrdeport %s" % (vmname,vmport)
        os.system(cmd)
    print "VmStart:%s" % vmname

def VmStop(vm):
    vmname = vm['vmname']
    cmd = "vboxmanage  controlvm %s poweroff" % vmname
    os.system(cmd)
    print "VmStop:%s" % vmname

def VmReboot(vm):
    vmname = vm['vmname']
    cmd = "vboxmanage  controlvm %s reset" % vmname
    os.system(cmd)
    print "VmReboot:%s" % vmname

def VmDelete(vm):
    vmname = vm['vmname']
    cmd = "vboxmanage unregistervm %s --delete" % vmname
    os.system(cmd)
    print "VmDelete:%s" %vmname

#消息处理函数
def message_handle(ch, method, properties, body):
    vmins = simplejson.loads(body)
    func = vmins.get('func')
    if func:
        eval(func)(vmins['vm'])
    ch.basic_ack(delivery_tag = method.delivery_tag)

#定义进程文件
def writePid():
    pid = str(os.getpid())
    f = file('/var/run/vboxagent.pid','w')
    fcntl.flock(f,fcntl.LOCK_EX)
    f.write(pid)
    fcntl.flock(f,fcntl.LOCK_UN)
    f.close()


def start():
    print 'start recevie'
    #写入进程号
    writePid()
    global connection,channel,routings
    if connection.is_closed:
        connection = pika.BlockingConnection(pika.ConnectionParameters('127.0.0.1'))
        channel = connection.channel()

    #定义交换机，设置类型为direct
    channel.exchange_declare(exchange='VmInstance', type='direct',durable=True)

    for routing in routings:
        #生成临时队列，并绑定到交换机上，设置路由键
        result = channel.queue_declare(queue=routing,durable=True)
        queue_name = result.method.queue
        channel.queue_bind(exchange='VmInstance',queue=queue_name,routing_key=routing)
        channel.basic_qos(prefetch_count=1)
        channel.basic_consume(message_handle, queue=queue_name)

    channel.start_consuming()

def stop():
    print 'stop receive'
    #删除进程文件
    os.remove("/var/run/vboxagent.pid") 
    global connection,channel
    #删除交换机
    channel.exchange_delete("VmInstance")
    #删除队列
    for routing in routings:
        channel.queue_delete(routing) 
    #关闭连接
    connection.close() 

def restart():
    print 'restart receive'
    stop()
    start()

#定义信号函数处理键盘事件
def handler(signal_num,frame):
    stop()
    sys.exit(signal_num)

def main():
    #定义信号函数
    signal.signal(signal.SIGINT,handler)
    signal.signal(signal.SIGTERM,handler)
    #执行参数回调
    func = sys.argv[1]
    eval(func)()

if __name__ == '__main__':
    main()


    
## vboxagent.py python守护进程示例
	
#!/bin/sh

# the following is chkconfig init header
#
# vboxagent:   virtualbox agent daemon
#
# chkconfig: 345 97 03
# description:  This is a daemon instructed by zstack management server \
#               to perform kvm related operations\
#               See http://zstack.org
#
# processname: vboxagent
# pidfile: /var/run/vboxagent.pid
#

pidfile='/var/run/vboxagent.pid'
check_status() {
    if [ ! -f $pidfile ]; then
        echo "vbox agent is stopped"
        exit 1
    else
        pid=`cat $pidfile`
        ps -p $pid > /dev/null
        if [ $? -eq 0 ]; then
            echo "vbox agent is running, pid is $pid"
            exit 0
        else
            echo "vbox agent is stopped, but pidfile at $pidfile is not cleaned. It may be caused by vbox agent crashed at last time, manually cleaning it would be ok"
            exit 1
        fi
    fi
}

function get_status(){
        [ ! -f $pidfile ] && echo 0 && return
    pid=`cat $pidfile`
    ps -p $pid > /dev/null
    if [ $? -eq 0 ]; then
            echo 1
        else
                echo 0
        fi

}

if [ $# -eq 0 ]; then
    echo "usage: $0
[start|stop|restart|status]"
    exit 1
fi

#获取当前进程运行状态
status=`get_status`

if [ "$@" = "status" ]; then
    check_status
else
        if [ "$status" = "1" ];then
        pid=`cat $pidfile`
            kill -15 $pid
            echo "kill -15 $pid"
        fi
    python receive.py $@ &> /dev/null &
fi

if [ $? -eq 0 ]; then
    echo "$@ vbox agent .... SUCCESS"
    exit 0
else
    echo "$@ vbox agent .... FAILED"
    exit 1
fi
