---
title: "Prometheus의 Pushgateway Redhat 6에서 구동하기"
date: 2020-02-02 10:05:00 -0800
category: [ LINUX ]
tags: [ Linux, Prometheus, Pushgateway ]
---

현재 EAI 시스템을 각 대륙별로 운영하고 있다.
각 나라별로 업무 시간이 다르다 보니 업무시간 외 장애로 연락이 오는 경우가 종종 발생한다.
이를 조금 보완하고자 각 서버의 상태를 일괄로 모아서 모니터링 할 수 있는 시스템 구축이 필요하였다.

서버가 해외에 존재하여 네트워크 속도가 빠르지 않기 때문에 최대한 각 지역의 서버에서 데이터 수집은 간단히 수행을 하고
모니터링 서버에서 조금더 많은 일을 했으면 하였다.
여기에 가장 적합한 것이 Prometheus라고 판단이 되어 이를 적용하게 되었다.

![Prometheus Architecture](../../assets/images/linux/architecture.png)

아키텍처 이미지를 보면 수집 대상 서버에 Pushgateway나 여러 Exporter를 설치하여 데이터 수집을 하게 되어있다.
서비스의 상태를 수집을 해야 되어 Pushgateway를 설치 후 이를 통해 데이터 집계를 하려 하였다.
다만 문제가 현재 사용중인 서버들이 Redhat 6 버전이여서 Pushgateway 데몬을 바로 돌릴 수는 없었다.
아래와 같이 스크립트를 먼저 작성하여 Pushgateway가 작동 할 수 있도록 하였다.

```sh
#!/bin/sh
#
# pushgateway   This shell script takes care of starting and stopping
#               pushgateway.
#
# chkconfig: - 80 30
# description: pushgateway
# processname: pushgateway
# config: /etc/pushgateway.conf
# pidfile: /var/run/pushgateway/pushgateway.pid

### BEGIN INIT INFO
# Provides: pushgateway ftpserver
# Required-Start: $local_fs $network $named $remote_fs
# Required-Stop: $local_fs $network $named $remote_fs
# Default-Stop: 0 1 6
### END INIT INFO

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Source ProFTPD configuration.
#PROFTPD_OPTIONS=""
if [ -f /etc/sysconfig/pushgateway ]; then
        . /etc/sysconfig/pushgateway
fi

# Check that networking is up.
[ ${NETWORKING} = "no" ] && exit 1

[ -x /usr/local/bin/pushgateway ] || exit 5

RETVAL=0

prog="pushgateway"

start() {
        echo -n $"Starting $prog: "
        #daemon pushgateway  2>/dev/null
        nohup /usr/local/bin/pushgateway &
        RETVAL=$?
        echo
        [ $RETVAL -eq 0 ] && touch /var/lock/subsys/pushgateway
}

stop() {
        echo -n $"Shutting down $prog: "
        killproc pushgateway
        RETVAL=$?
        echo
        [ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/pushgateway
}

# See how we were called.
case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  status)
        status pushgateway
        RETVAL=$?
        ;;
  restart)
        stop
        start
        ;;
  try-restart|condrestart)
        if [ -f /var/lock/subsys/pushgateway ]; then
          stop
          start
        fi
        ;;
  reload|force-reload)
        echo -n $"Re-reading $prog configuration: "
        killproc pushgateway -HUP
        RETVAL=$?
        echo
        ;;
  *)
        echo "Usage: $prog {start|stop|restart|try-restart|reload|status}"
        exit 2
esac

exit $RETVAL
```

Pushgateway 스크립트 실행을 하여 정상적으로 작동하는 확인을 위해 아래와 같이 테스트 데이터를 생성하여 정상적으로 조회가 되는지 확인 가능하다.

```
echo "cpu_utilization 20.25" | curl --data-binary @- http://localhost:9091/metrics/job/my_custom_metrics/provider/abc

curl -L http://localhost:9091/metrics
```