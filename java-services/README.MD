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
Restart=always
RestartSec=1
ExecStart=/usr/bin/python /opt/baz/cleanup.py
StandardError=journal

```


### Define script

/etc/systemd/system/my-script.sh
for java
```bash

#!/bin/bash

#set -x

USAGE="Usage: bash deploy.sh {start|stop|restart|status}"
HOME_DIR=~/tmp2/
SERVICE_NAME="xxxx-api"
EXEC_PROGRAM="xxxx-api.jar"
PARAMS_COUNT=1


# 操作: start, stop, restart, status
SERVICE_OPERATE=${1}
DATE=`date '+%Y-%m-%d %H:%M:%S'`
# 找出对应进程PID
PID_CMD="ps -ef | grep ${EXEC_PROGRAM} | grep -v grep | awk '{print \$2}'"


# 进入相应目录
cd $HOME_DIR


status() {
    # 检查看进程是否启动
    echo $DATE " try to check ${SERVICE_NAME} server status"
    pid=`eval ${PID_CMD}`
    if [ -z "${pid}" ]; then
        echo $DATE "${SERVICE_NAME} server is not running"
    else
        echo $DATE "${SERVICE_NAME} server is running, pid: ${pid}"
    fi

    return ${pid}
}

start() {
    # 检查进程是否存在，如果存在则直接退出
    echo $DATE "try to start ${SERVICE_NAME} server"
    pid=`eval ${PID_CMD}`
    if [[ ! -z "${pid}" ]]; then
        echo $DATE "${SERVICE_NAME} server is running, pid ${pid}, exit..."
        return 1
    fi

    # 启动程序
    cmd="nohup java -jar ${EXEC_PROGRAM} > /dev/null 2>&1 &"
    echo $DATE "try to start ${SERVICE_NAME}server with cmd: ${cmd}"
    eval ${cmd}
    eval=$?
    sleep 5
    # 判断返回值
    if [[ ${eval} -eq 0 ]]; then
        echo $DATE  "success to start ${SERVICE_NAME}server "
        status
    else
        echo $DATE "fail to start ${SERVICE_NAME} server"
    fi
}

stop() {
    # 检查进程是否存在，如果不存在则直接退出
    echo $DATE "try to stop ${SERVICE_NAME} server"
    pid=`eval ${PID_CMD}`
    if [[ -z "${pid}" ]]; then
        echo $DATE "${SERVICE_NAME} server is not running, exit..."
        return 1
    fi

    # 结束程序
    cmd="kill -9 ${pid}"
    echo $DATE "try to stop ${SERVICE_NAME} server with cmd: ${cmd}"
    eval ${cmd}
    eval=$?
    sleep 2
    # 判断返回值
    if [[ ${eval} -eq 0 ]]; then
        echo $DATE "success to stop ${SERVICE_NAME} server"
        # 再次检查，看进程是否启动
        status
    else
        echo $DATE "fail to stop ${SERVICE_NAME} server"
    fi
}


case "${SERVICE_OPERATE}" in
  start)
    if [[ $# != ${PARAMS_COUNT} ]] ; then  # 判断参数个数
        echo "USAGE: $0 ${USAGE}"
        exit 1;
    fi
    start
    ;;
  stop)
    stop
    ;;
  restart)
    if [[ $# != ${PARAMS_COUNT} ]] ; then  # 判断参数个数
        echo "USAGE: $0 ${USAGE}"
        exit 1;
    fi
    stop
    start
    ;;
  status)
    status
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