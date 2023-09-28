---
title: "데이터 중심의 애플리케이션 설계 - 스트림 프로세싱의 사용 사례"
toc: true
categories:
  - tech
tags:
  - book
  - design system
  - tech
  - backend
---

스트림 처리는 다양한 용도로 사용되며, 그 중 모니터링이 가장 대표적인 예다.

- 사기 탐지: 시스템은 신용카드 사용 패턴에 예기치 않은 변화가 있는지 모니터링하여 사기가 의심되는 경우 카드를 차단한다.
- 거래 시스템: 금융 거래 시스템은 시장의 가격 변화를 분석하고 미리 정의된 규칙에 따라 거래를 실행한다.
- 제조 시스템: 제조 공장에서는 기계 상태를 모니터링하여 오작동을 신속하게 파악한다.
- 군사 및 정보: 방어 시스템은 잠재적인 공격자의 활동을 추적하고 공격 징후가 있을 경우 경보를 발령하며, 이 때 고급 패턴 매칭 및 상관관계가 필요한 경우가 많다.

## 복잡한 이벤트 처리(CEP)

복잡한 이벤트 처리(CEP)는 이벤트 스트림을 분석하기 위해 1990년대에 개발된 접근 방식이다.
CEP는 특정 이벤트 패턴을 식별해야 하는 애플리케이션에 적합하다.

CEP의 핵심 사항은 다음과 같다:

- 패턴 매칭: 정규식이 문자열에서 패턴 매칭을 가능하게 하는 것처럼, CEP를 사용하면 규칙을 지정하여 스트림 내에서 특정 패턴의 이벤트를 검색할 수 있다.
- 쿼리 언어: CEP 시스템은 일반적으로 SQL 또는 그래픽 인터페이스와 같은 높은 수준의 선언적 쿼리 언어를 사용하여 탐지할 이벤트 패턴을 설명한다.
- 처리 엔진: 쿼리는 입력 스트림을 소비하고 일치하는 패턴을 찾기 위해 내부 상태 머신을 유지하는 처리 엔진에 제출된다. 일치하는 패턴이 발견되면 엔진은 감지된 패턴의 세부 정보가 포함된 복잡한 이벤트를 전송한다.
- 역할 반전: 데이터를 저장하고 쿼리를 일시적인 것으로 처리하는 기존 데이터베이스와 달리, CEP 엔진은 쿼리를 장기적으로 저장하고 이벤트가 지속적으로 통과하면서 쿼리 정의 패턴과 일치하는 것을 검색한다.

일부 CEP 구현에는 Esper, IBM InfoSphere Streams, Apama, TIBCO StreamBase, SQLstream 등이 있다.
Samza와 같은 분산 스트림 프로세서도 스트림의 선언적 쿼리에 대한 SQL을 지원하고 있다.

## 스트림 분석

스트림 분석은 스트림 처리의 또 다른 대표적인 애플리케이션 사용 사례다.
스트림 분석은 이벤트 스트림에 대한 집계 및 통계 메트릭에 중점을 둔다.

스트림 분석은 주로 특정 이벤트 시퀀스가 아닌 *많은 수의 이벤트*에 대한 통계 집계 및 계산을 다룬다.
스트림 분석에는 시간 간격별 이벤트 비율 측정, 롤링 평균 계산, 추세 감지 또는 알림을 위해 현재 통계와 이전 간격을 비교하는 등의 작업이 포함된다.

분석은 *윈도우*라고 하는 고정된 시간 간격에 걸쳐 통계를 계산하여 변동을 완화하는 동시에 트래픽 패턴에 대한 시의적절한 인사이트를 제공한다.

스트림 분석 시스템은 여러 가지 알고리즘이 동원된다.

- 집합 멤버십을 위한 블룸 필터
- 카디널리티 추정을 위한 하이퍼로그
- 백분위수 추정 알고리즘

이러한 알고리즘은 대략적인 결과를 생성하지만 정확한 알고리즘보다 훨씬 적은 메모리를 소비한다.

스트림 처리 자체는 본질적으로 근사치가 아니다.
최적화를 위해 확률론적 알고리즘이 사용된다.
Apache Storm, Spark Streaming, Flink, Concord, Samza, Kafka Streams와 같은 많은 오픈 소스 분산 스트림 처리 프레임워크는 분석을 처리하도록 설계되어 있다. Google Cloud Dataflow 및 Azure Stream Analytics와 같은 호스팅 서비스도 스트림 분석을 지원한다.