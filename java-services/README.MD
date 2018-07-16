# check service at regular intervals


## pattern

/etc/systemd/system/my-service.timer  ----->
 
/etc/systemd/system/my-service.service ---->

/etc/systemd/system/my-script


## demo
### Define the timer
/etc/systemd/system/my-service.timer
```bash
[Unit]
Description=Timer for my-service


[Timer]
# Define target unit (in case the name part is the same, can be omitted)
Unit=baz-cleanup.service

## Activate target unit based on monotonic timers 
# Time to wait after booting before we run first time
OnBootSec=2min

# Time between running each consecutive time
OnUnitActiveSec=1min


[Install]
WantedBy=multi-user.target

```

### Define the target service unit

/etc/systemd/system/my-service.service
```bash
[Unit]
Description=My Service Management


[Service]
Type=simple
ExecStart=/usr/bin/python /opt/baz/cleanup.py
StandardError=journal

```


### Define script

/etc/systemd/system/my-script.sh
for java
```bash

#!/bin/bash

USAGE="Usage: bash my-script.sh {start|stop|restart|status}"
HOME_DIR=/opt/my-service/

# 进入相应目录
cd $HOME_DIR

# 读参数
serviceName="my-service"
serviceProcessName="my-service.jar"
# 操作: start, stop, restart, status
serviceOperate=${1}
# 获取当前执行的脚本命令，用于监控时启动命令
curExecCmd="sh $0 $@"

# 找出对应进程
PID_CMD="ps -ef | grep ${serviceProcessName} | grep -v grep | awk '{print \$2}'"


checkService() {
    # 检查看进程是否启动
    echo `date -d "now" +"%Y-%m-%d %H:%M:%S"`" try to check ${serviceName} server status"
    pid=`eval ${PID_CMD}`
    if [ -z "${pid}" ]; then
        echo "${serviceName} server is not running"
    else
        echo "${serviceName} server is running, pid: ${pid}"
    fi
    
    return ${pid}
}

startService() {
    # 检查进程是否存在，如果存在则直接退出
    echo `date -d "now" +"%Y-%m-%d %H:%M:%S"`" try to start ${serviceName} server"
    pid=`eval ${PID_CMD}`
    if [[ ! -z "${pid}" ]]; then
        echo "${serviceName} server is running, pid ${pid}, exit..."
        return 1
    fi

    # 启动程序
    cmd="nohup java -jar ${serviceJarName} > /dev/null 2>&1 &"
    echo `date -d "now" +"%Y-%m-%d %H:%M:%S"`" try to start ${serviceName}server , cmd: ${cmd}"
    eval ${cmd}
    eval=$?
    sleep 5
    # 判断返回值
    if [[ ${eval} -eq 0 ]]; then
        echo `date -d "now" +"%Y-%m-%d %H:%M:%S"`" success to start ${serviceName}server "
        checkService
    else
        echo `date -d "now" +"%Y-%m-%d %H:%M:%S"`" fail to start ${serviceName} server"
    fi
}

stopService() {
    # 检查进程是否存在，如果不存在则直接退出
    echo `date -d "now" +"%Y-%m-%d %H:%M:%S"`" try to stop ${serviceName} server"
    pid=`eval ${PID_CMD}`
    if [[ -z "${pid}" ]]; then
        echo "${serviceName} server is not running, exit..."
        return 1
    fi
    
    # 结束程序
    cmd="kill -9 ${pid}"
    echo `date -d "now" +"%Y-%m-%d %H:%M:%S"`" try to stop ${serviceName} server, cmd: ${cmd}"
    eval ${cmd}
    eval=$?
    sleep 2
    # 判断返回值
    if [[ ${eval} -eq 0 ]]; then
        echo `date -d "now" +"%Y-%m-%d %H:%M:%S"`" success to stop ${serviceName} server"
        # 再次检查，看进程是否启动
        checkService
    else
        echo `date -d "now" +"%Y-%m-%d %H:%M:%S"`" fail to stop ${serviceName} server"
    fi
}


case "${serviceOperate}" in
  start)
    if [[ $# != 3 ]] ; then  # 判断参数个数
        echo "USAGE: $0 ${USAGE}"
        exit 1;
    fi
    startService
    ;;
  stop)
    stopService
    ;;
  restart)
    if [[ $# != 3 ]] ; then  # 判断参数个数
        echo "USAGE: $0 ${USAGE}"
        exit 1;
    fi
    stopService
    startService
    ;;
  status)
    checkService
    ;;
  *)
    echo ${USAGE}
    exit 1
esac

exit 0

```

for node
```bash


```


### Start/Enable the timer
Start the timer. Inspect the journal to verify that the perioric task is actually triggered.
```bash
systemctl daemon-reload && systemctl start baz-cleanup.timer
```




##references
[Systemd 定时器教程](http://www.ruanyifeng.com/blog/2018/03/systemd-timer.html)  
[README-setup-systemd-timer.md](https://gist.github.com/drmalex07/350238961f451d6946dd)  
[How to Use Systemd Timers](https://jason.the-graham.com/2013/03/06/how-to-use-systemd-timers/)  
[systemd official](https://www.freedesktop.org/software/systemd/man/systemd.service.html#)