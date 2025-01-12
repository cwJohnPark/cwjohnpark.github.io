---
title: "데이터 중심의 애플리케이션 설계 - 실전에서의 네트워크 문제"
toc: true
categories:
  - tech
tags:
  - book
  - design system
  - tech
  - backend
---

중간 규모의 데이터센터는 월 평균 12개의 네트워크 오류를 겪는다.
EC2와 같은 퍼블릭 클라우드를 사용하든, 사설 관리형 데이터센터를 사용하든 네트워크 문제는 누구도 피해갈 수 없다.

예를 들어,

- 스위치의 소프트웨어를 업그레이드하고나서 네트워크 토폴로지가 재구성되어 패킷에 수 분 이상 지연될 수 있다.
- 바다 속 상어가 케이블을 물어 손상을 주는 경우

네트워크가 안정적이라면 네트워크에 문제가 발생하는 동안 사용자에게 오류 메시지를 표시하는 것이 올바른 접근 방식일 수 있다.
그러나, 소프트웨어가 네트워크 문제에 어떻게 반응하는지 파악하고 시스템이 문제로부터 복구할 수 있는지 확인해야 한다.
Chaos Monkey와 같은 툴을 이용하여 의도적으로 네트워크 문제를 유발하여 시스템의 응답을 테스트하는 것도 좋은 방법일 수 있다.

어떻게 오류를 감지할까? 대부분의 시스템이 오류가 발생한 노드를 자동으로 감지해야 한다.

- 로드밸런서는 죽은 노드에 요청을 보내는 것을 중단해야 한다.
- 단일 리더의 분산 애플리케이션 환경에서, 리더가 실패하면, 팔로워 중 하나가 리더로 승격해야 한다.

노드가 살아있는지를 파악하기 위해 명시적으로 피드백을 받아야할 수도 있다.

- 어떠한 프로세스도 목적지 포트를 리스닝하고 있지 않다면, 운영체제는 RST나 FIN 패킷을 응답하여 TCP 연결을 닫아야 한다.
- 노드 프로세스의 오류가 발생하지만, 운영체제는 구동중일 때, 다른 노드에게 알림으로써 타임아웃을 기다리는 시간을 최소화 할 수 있다.(Hbase가 대표적인 예다.)
- 네트워크 스위치의 관리형 인터페이스에 접근할 수 있다면, 하드웨어 레벨에서 링크 오류를 감지할 수 있다.
- 라우터가 IP 주소에 연결할 수 없다면, ICMP 패킷을 응답할 수 있다.
