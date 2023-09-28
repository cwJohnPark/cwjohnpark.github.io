---
title: "데이터 중심의 애플리케이션 설계 - 맵 리듀스와 분산 파일 시스템 "
toc: true
categories:
  - tech
tags:
  - book
  - design system
  - tech
  - backend
---

맵 리듀스는 유닉스 도구와 비슷하다. 차이점은 수천 대의 머신에 분산되어 있다는 점이다.
맵 리듀스는 유닉스 도구와 마찬가지로 상당히 무차별 대입 방식이지만 놀라울 정도로 효과적인 도구다.
단일 맵리듀스 작업은 단일 유닉스 프로세스와 유사하다. 맵리듀스는 유닉스 프로세스와 같이 하나 이상의 입력을 받아서 하나 이상의 출력을 생성한다.
맵리듀스 작업을 실행해도 일반적으로 입력은 수정되지 않으며, 출력을 생성하는 것 외에 다른 사이드 이펙트가 없다.

HDFS는 NAS(네트워크 연결 스토리지) 및 SAN(스토리지 영역 네트워크) 아키텍처의 공유 디스크 접근 방식과 달리 무공유 원칙을 기반으로 한다.

- 공유 디스크 스토리지는 중앙 집중식 스토리지 어플라이언스에 의해 구현되며, 종종 맞춤형 하드웨어와 파이버 채널과 같은 특수 네트워크 인프라스트럭처를 사용한다.
- HDFS의 경우, 아무것도 공유하지 않는 방식은 특별한 하드웨어가 필요하지 않으며 기존 데이터센터 네트워크로 연결된 컴퓨터만 있으면 된다.

HDFS는 각 머신에서 실행되는 데몬 프로세스로 구성되어 다른 노드가 해당 머신에 저장된 파일에 액세스할 수 있는 네트워크 서비스를 노출한다.
네임노드라는 중앙 서버는 어떤 파일 블록이 어떤 머신에 저장되어 있는지 추적한다.
HDFS는 개념적으로 데몬을 실행하는 모든 머신의 디스크 공간을 사용할 수 있는 하나의 큰 파일시스템을 생성한다.

머신 및 디스크 장애를 견디기 위해 파일 블록은 여러 머신에 복제된다.
복제는 단순히 여러 머신에 동일한 데이터의 복사본을 여러 개 저장하는 것을 의미할 수 있다.
또는, 전체 복제보다 낮은 스토리지 오버헤드로 손실된 데이터를 복구할 수 있는 삭제 코딩 체계를 의미할 수도 있다.

HDFS의 복제 기법은 동일한 시스템에 연결된 여러 디스크에 중복성을 제공하는 RAID와 유사하다.
하지만, HDFS의 분산 파일 시스템에서는 특별한 하드웨어 없이 기존 데이터센터 네트워크를 통해 파일 액세스 및 복제가 수행된다는 차이가 있다.
가장 큰 규모의 HDFS 배포는 수만 대의 머신에서 실행되며, 스토리지 용량은 수백 페타바이트에 달한다.

상용 하드웨어와 오픈 소스 소프트웨어를 사용하는 HDFS의 데이터 저장 및 액세스 비용이 전용 스토리지 어플라이언스의 동급 용량에 비해 훨씬 저렴하기 때문에 이러한 대규모 구축이 가능해졌다.