## 介绍：
	zabbix 安装包是个二进制包，install.sh
    
   1. influxdb和collectd支持
   	- 49行加入如下：
      
		influxdb_repo_file=/etc/yum.repos.d/influxdb.repo
            
		zstack_ali_repo_file=/etc/yum.repos.d/zstack-aliyun-yum.repo
            
		PRODUCT_TITLE_FILE='./product_title_file'
            
		UPGRADE_LOCK=/tmp/zstack_upgrade.lock
            
    -  1273 行加入如下：
      install_influxdb_collectd(){
      echo_title "Install influxdb and collectd"
      echo ""
   	  show_spinner cs_install_influxdb_collectd
      }
      
    - 1280 行加入如下:
    	install_hwmonitor(){
        
    	echo_title "Install hwmonitor"
        
    	echo ""
        
    	show_spinner cs_install_hwmonitor
		}
        
     - 1288 行加入如下:
      install_keymanager(){
     
    	echo_title "Install keymanager"
        
    	echo ""
        
    	show_spinner cs_install_keymanager
		 }
     
     - 1666行加入如下:
       cs_install_influxdb_collectd(){
    echo_subtitle "Install influxdb and collectd-virt"
    yum install -y collectd-virt &> /dev/null
    #echo config to collectd.conf 
    cat > /etc/collectd.conf <<EOF
Interval 10
FQDNLookup false

LoadPlugin syslog
LoadPlugin aggregation
LoadPlugin cpu
LoadPlugin disk
LoadPlugin interface
LoadPlugin memory
LoadPlugin network
LoadPlugin virt

<Plugin aggregation>
	<Aggregation>
		#Host "unspecified"
		Plugin "cpu"
		#PluginInstance "unspecified"
		Type "cpu"
		#TypeInstance "unspecified"

		GroupBy "Host"
		GroupBy "TypeInstance"

		CalculateNum false
		CalculateSum false
		CalculateAverage true
		CalculateMinimum false
		CalculateMaximum false
		CalculateStddev false
	</Aggregation>
</Plugin>

<Plugin cpu>
  ReportByCpu true
  ReportByState true
  ValuesPercentage true
</Plugin>

<Plugin disk>
  Disk "/^sd/"
  Disk "/^hd/"
  Disk "/^vd/"
  IgnoreSelected false
</Plugin>

<Plugin "interface">
Interface "enp2s0"
Interface "enp3s0"
Interface "enp4s0"
Interface "enp5s0"
Interface "enp6s0"
Interface "enp7s0"
IgnoreSelected false
</Plugin>

<Plugin memory>
	ValuesAbsolute true
	ValuesPercentage false
</Plugin>

<Plugin virt>
	Connection "qemu:///system"
	RefreshInterval 10
	HostnameFormat name
    PluginInstanceFormat name
</Plugin>

<Plugin network>
	Server "localhost" "25826"
</Plugin>

Include "/etc/collectd.d"

EOF

    #yum install influxdb
    yum install -y influxdb  &> /dev/null
    #echo influxdb.conf config 
    cat >/etc/influxdb/influxdb.conf <<EOF
[meta]
  dir = "/var/lib/influxdb/meta"
[data]
  dir = "/var/lib/influxdb/data"
  wal-dir = "/var/lib/influxdb/wal"
 [[collectd]]
   enabled = true
   bind-address = ":25826"
   database = "collectd"
   typesdb = "/usr/share/collectd/types.db"
   batch-size = 5000
   batch-pending = 10
   batch-timeout = "10s"
   read-buffer = 0

EOF
  
    #create collectd database and restart service.
    systemctl restart influxdb   &>>$ZSTACK_INSTALL_LOG
    systemctl enable influxdb    &>>$ZSTACK_INSTALL_LOG
    influx -execute 'create database collectd' &>>$ZSTACK_INSTALL_LOG
    systemctl restart collectd   &>>$ZSTACK_INSTALL_LOG
    systemctl enable collectd    &>>$ZSTACK_INSTALL_LOG
    pass
}

	- 1777行加入如下:
    cs_install_hwmonitor(){
echo_subtitle "Begin Install hwmonitor "
    yum install -y ipmitool &> /dev/null
    #echo config to collectd.conf 
    cat > /etc/hwmonitor.conf <<EOF
hwmonitor_database_url=root:zstack.mysql.password@tcp(127.0.0.1:3306)/hwmonitor?charset=utf8
ipmi_username=ADMIN
ipmi_password=ADMIN
EOF

    #echo hwmonitor service 
    cat >/etc/systemd/system/hwmonitor.service <<EOF
[Unit]
Description=hwmonitor Service
After=zstack.service
Before=shutdown.target reboot.target halt.target

[Service]
Type=forking
User=root
ExecStart=/etc/init.d/hwmonitor-server start
Restart=on-abort
RemainAfterExit=Yes
TimeoutStartSec=3
TimeoutStopSec=3

[Install]
WantedBy=multi-user.target
EOF

    #echo hwmonitor script
    cat >/etc/init.d/hwmonitor-server <<EOF
#!/bin/bash
# chkconfig: 345 97 03
# description:  IPMI Monitor Server.
# Source function library.
BASE_PATH=/usr/local/zstack

start() {
     echo -n "Starting hwmonitor: "
     \${BASE_PATH}/hwmonitor --config-path /etc/hwmonitor.conf &
}

stop() {
   echo -n "Stopping hwmonitor: "
      killall hwmonitor
}

case "\$1" in
  start)
       start
   ;;
  stop)
       stop
   ;;
  restart)
       stop
       start
   ;;
  *)
       echo $"Usage: hwmonitor {start|stop|restart}"
   ;;
esac
EOF

    #enable hwmonitor service.
    chmod a+x /usr/local/zstack/hwmonitor
    chmod a+x /etc/init.d/hwmonitor-server
    systemctl daemon-reload    &>>$ZSTACK_INSTALL_LOG
    systemctl enable hwmonitor    &>>$ZSTACK_INSTALL_LOG
    systemctl restart hwmonitor   &>>$ZSTACK_INSTALL_LOG
    pass
}

- 1854行加入:

	cs_install_keymanager(){
    echo_subtitle "Begin Install keymanager "
    #unzip keymanager package.
    cd $ZSTACK_INSTALL_ROOT
    unzip  keymanager.zip >>$ZSTACK_INSTALL_LOG 2>&1
    if [ $? -ne 0 ];then
       fail "failed to unzip keymanager package: $ZSTACK_INSTALL_ROOT/keymanager*.zip."
    fi
    chmod a+x keymanager/keymanager

    #echo keymanager service 
    cat >/etc/systemd/system/keymanager.service <<EOF
[Unit]
Description=hwmonitor Service
After=zstack.service
Before=shutdown.target reboot.target halt.target

[Service]
Type=forking
User=root
ExecStart=/etc/init.d/keymanager-server start
Restart=on-abort
RemainAfterExit=Yes
TimeoutStartSec=3
TimeoutStopSec=3

[Install]
WantedBy=multi-user.target
EOF

    #echo keymanager script
    cat >/etc/init.d/keymanager-server <<EOF
#!/bin/bash
# chkconfig: 345 97 03
# description:  keymanager Monitor Server.
# Source function library.
BASE_PATH=/usr/local/zstack

start() {
     echo -n "Starting keymanager: "
     \${BASE_PATH}/keymanager/keymanager  &
}

stop() {
   echo -n "Stopping keymanager: "
      killall keymanager
}

case "\$1" in
  start)
       start
   ;;
  stop)
       stop
   ;;
  restart)
       stop
       start
   ;;
  *)
       echo $"Usage: keymanager {start|stop|restart}"
   ;;
esac
EOF

    #enable keymanager service.
    chmod a+x /usr/local/zstack/keymanager/keymanager &>>$ZSTACK_INSTALL_LOG
    systemctl daemon-reload    &>>$ZSTACK_INSTALL_LOG
    systemctl enable  keymanager    &>>$ZSTACK_INSTALL_LOG
    systemctl restart keymanager   &>>$ZSTACK_INSTALL_LOG

    pass 
}

- 2590行加入如下：
   	install_influxdb_collectd
    
    install_hwmonitor
    
    install_keymanager
	
            
