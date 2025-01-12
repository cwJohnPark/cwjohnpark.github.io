---
title: "데이터 중심의 애플리케이션 설계 - 분산 시스템은 다수에 의해 진실이 규정된다."
toc: true
categories:
  - tech
tags:
  - book
  - design system
  - tech
  - backend
---

단일 노드에서 구동되는 프로그램과 분산 시스템의 차이점은 메시지를 주고받기 위해서 네트워크를 사용해야 한다는 점이다.
네트워크 통신은 부분 실패 문제, 시각 문제, 그리고 정지 프로세스 문제를 유발한다.

우리는 어떻게 시스템에서 무엇이 참이고 거짓을 판단 할 수 있을까? 분산 시스템의 동작에 대한 가정(즉, 시스템 모델)을 정의하고 이러한 가정을 실현하는 방식으로 실제 시스템은 설계된다.

## 분산 시스템은 정족수 기반이다.

한 노드가 모든 메시지를 수신할 수 있지만 해당 노드에서 보내는 모든 발신 메시지는 삭제되거나 지연된다면, 다른 노드들은 해당 노드로부터 응답을 듣지 못했기 때문에 해당 노드가 죽었다고 선언한다. 가비지 컬렉션에 의해 일시 중지 되는 노드 역시 다른 노드에 의해 죽은 것으로 판단힐 것이다.

이 사례의 교훈은 노드가 상황에 대한 자신의 판단을 반드시 신뢰할 수 없다는 것이다.
분산화된 시스템은 단일 노드에만 의존할 수 없다. 대신, 대부분의 분산 알고리즘은 **쿼럼(quorums)**, 즉 노드 간의 투표에 의존한다.

노드 정족수가 다른 노드를 죽은 노드로 선언하면 해당 노드가 여전히 살아 있다고 보이더라도 죽은 것으로 간주해야 한다.
개별 노드는 쿼럼 결정을 준수하고 물러나야 한다.

가장 일반적인 쿼럼은 노드 절반 이상의 절대 다수를 채택하는 것이다.
과반수 정족수를 사용하면 개별 노드에 장애가 발생하더라도 시스템이 계속 작동할 수 있다.
예를 들어, 노드가 3개인 경우 한 번의 장애를 허용하고, 노드가 5개인 경우 두 번의 장애를 허용할 수 있다.

## 시스템은 단 하나의 무엇을 요구한다

- 스플릿 브레인 문제를 피하기 위해 노드 중 오직 하나만이 데이터베이스 파티션의 리더로 인정된다.
- 오직 하나의 트랜잭션만이 특정 객체에 대한 잠금을 소유하므로써, 동시에 쓰기 작업이 데이터를 망가지게 않게 한다.
- 오직 한명의 사용자만이 특정 이름을 등록하여, 사용자를 식별할 수 있다.

단일 시스템에서는 위 사항을 어렵지 않게 해결할 수 있다. 하지만, 분산 시스템에서 위 사항을 지키려면 주의가 필요하다.
노드가 자신이 파티션의 리더, 잠금의 소유자, 사용자 이름을 성공적으로 가져온 사용자의 요청 처리자라고 믿더라도 반드시 노드의 정족수가 동의한다는 의미는 아니다.

HBase에는 잘못된 잠금 구현으로 인한 데이터 손상 버그 문제가 있었다.
여러 클라이언트가 파일에 쓰기를 시도하면 파일이 손상될 수 있으므로, HBase는 스토리지 서비스의 파일에 한 번에 한 클라이언트만 액세스를 허용한다.
즉, 클라이언트가 파일에 액세스하기 전에 잠금을 요구하여 이를 구현하려고 했다.

이러한 알고리즘을 사용한 HBase에는 어떠한 문제가 있었을까? 잠금을 보유한 클라이언트가 너무 오래 일시 중지되면 해당 잠금이 만료된다.
다른 클라이언트가 동일한 파일에 대한 임대를 획득하여 파일에 쓰기 작업을 시작할 수 있다.
일시 중지된 클라이언트는 다시 돌아왔을 때 여전히 유효한 임대가 있다고 잘못 믿고 파일에 쓰기를 진행한다.
결과적으로 클라이언트의 쓰기가 충돌하여 파일이 손상된다.

## 토큰 펜싱으로 착각 중인 노드를 방지하기

파일 스토리지와 같이 특정 리소스에 대한 액세스를 보호하기 위해 잠금을 사용하는 경우, 자신이 "선택된 노드"라고 착각한 노드가 나머지 시스템을 방해할 수 없도록 해야 한다. 이러한 문제를 회피하기 위한 방법 중 하나가 **펜싱(fencing)**이다.

잠금 서버가 잠금을 부여할 때마다 펜싱 토큰도 반환한다고 가정해 보자.
펜싱 토큰은 잠금이 부여될 때마다 증가하는 숫자다.
클라이언트가 스토리지 서비스에 쓰기 요청을 보낼 때마다 현재 펜싱 토큰을 포함하도록 요구하는 방식이다.

예를 들어, 클라이언트 1은 토큰 33으로 잠금을 획득하지만, 이후 긴 일시 정지에 들어가 잠금이 만료된다고 가정하자. 클라이언트 2는 토큰 34로 잠금을 획득한 후 스토리지 서비스에 34 토큰을 포함한 쓰기 요청을 보낸다. 나중에 클라이언트 1이 다시 살아나서 토큰 값 33을 포함한 쓰기 요청을 스토리지 서비스에 보낸다면, 스토리지 서버는 이미 더 높은 토큰 번호(34)로 쓰기를 처리한 것을 기억하고 있기 때문에 토큰 33으로 요청을 거부한다.

토큰 펜싱을 사용하는 가장 유명한 예는 ZooKeeper다. Zookeeper는 트랜잭션 ID _zxid_ 또는 노드 버전 *cversion*을 펜싱 토큰으로 사용한다.

서버 측에서 토큰을 확인하는 것은 보기보다 합리적인 방법이다. 클라이언트는 서비스를 운영하는 사람들의 우선순위와 매우 다른 우선순위를 가진 사람들에 의해 운영되는 경우가 많기 때문에 서비스가 클라이언트는 언제든 틀릴 수 있음을 가정해야 한다.
