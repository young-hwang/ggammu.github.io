---
title: "Prometheus의 Pushgateway Redhat 6에서 구동하기"
date: 2020-02-03 16:05:00 -0600
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

