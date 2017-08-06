title: Linux下启动Java守护进程方法
author: Gang Chen
tags:
  - Java
  - Linux
categories:
  - Linux
date: 2017-08-06 20:53:00
---
```bash
#!/bin/sh

DESC="Java Service"
NAME=java-service
PIDFILE=/tmp/$NAME.pid
PATH_TO_JAR=~/java-service.jar
PATH_TO_JAVA=/usr/local/jdk1.8.0_131/bin/java
SERVICE_CONFIG='-Dserver.port=8080 -DLOG_PATH=/var/log'
COMMAND="$PATH_TO_JAVA -- $SERVICE_CONFIG -jar $PATH_TO_JAR"


d_start() {
    start-stop-daemon --start --quiet --background --make-pidfile --pidfile $PIDFILE  --exec $COMMAND
}

d_stop() {
    start-stop-daemon --stop --quiet --pidfile $PIDFILE
    if [ -e $PIDFILE ]
        then rm $PIDFILE
    fi
}

case $1 in
    start)
    echo -n "Starting $DESC: $NAME"
    d_start
    echo "."
    ;;
    stop)
    echo -n "Stopping $DESC: $NAME"
    d_stop
    echo "."
    ;;
    restart)
    echo -n "Restarting $DESC: $NAME"
    d_stop
    sleep 1
    d_start
    echo "."
    ;;
    *)
    echo "usage: $NAME {start|stop|restart}"
    exit 1
    ;;
esac

exit 0
```