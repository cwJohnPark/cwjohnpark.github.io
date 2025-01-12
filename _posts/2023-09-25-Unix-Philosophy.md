---
title: "데이터 중심의 애플리케이션 설계 - Unix의 철학"
toc: true
categories:
  - tech
tags:
  - book
  - design system
  - tech
  - backend
---

유닉스에서 간단한 스크립트를 사용하여 로그 파일을 아주 쉽게 분석할 수 있는 이유는 유닉스의 철학 덕분이다.
유닉스 파이프의 발명가인 더그 맥일로이는 1964년에 처음으로 파이프를 개발했다. 그는 파이프를 배관에 비유하였다.
프로그램을 파이프로 연결한다는 아이디어는 현재 유닉스 개발자와 사용자들 사이에서 널리 알려진 일련의 설계 원칙인 유닉스 철학의 일부가 되었다.
이 철학은 1978년에 다음과 같이 설명되었다.

1. 각 프로그램이 한 가지 일만 잘하도록 하라. 새로운 작업을 수행하려면 새로운 '기능'을 추가하여 기존 프로그램을 복잡하게 만들지 말고 새로 구축하라.
2. 모든 프로그램의 출력이 아직 알려지지 않은 다른 프로그램의 입력이 될 것으로 예상하라. 불필요한 정보로 출력을 복잡하게 만들지 마라. 엄격한 열 또는 이진 입력 형식을 피하라. 대화형 입력을 고집하지 마라.
3. 소프트웨어, 심지어 운영 체제까지 몇 주 이내에 시험해 볼 수 있도록 설계 및 구축하라. 잘 하지 못하는 부분은 과감히 버리고 다시 만드는 것을 주저하지 마라.
4. 프로그래밍 작업을 가볍게 하기 위해 숙련되지 않은 도움보다는 도구를 우선적으로 사용하라. 심지어 도구를 빌드하기 위해 우회해야 하거나, 도구를 다 사용한 후 일부를 버려야할 경우에도 말이다.

이러한 접근 방식은 오늘날의 애자일 및 데브옵스 운동과 놀랍도록 유사하다.
예를 들어, "자동화", "신속한 프로토타이핑", "점진적 반복", "실험이 쉬움", "대규모 프로젝트를 관리 가능한 덩어리로 나누기"가 있다.

정렬 도구는 한 가지 일을 잘하는 프로그램의 좋은 예로 들 수 있다.
대부분의 프로그래밍 언어가 표준 라이브러리에서 제공하는 것보다 더 나은 정렬 구현이다.
디스크에 저장하지 않고 여러 스레드를 사용하지 않는 것이 유리한 경우에도 마찬가지다.
하지만 정렬은 그 자체로는 거의 유용하지 않다. `uniq`와 같은 다른 Unix 도구와 함께 사용할 때만 강력하다.
bash와 같은 유닉스 셸을 사용하면 이러한 작은 프로그램을 놀라울 정도로 강력한 데이터 처리 작업으로 쉽게 구성할 수 있다.
이러한 프로그램 중 상당수는 서로 다른 그룹에 의해 작성되었지만 유연한 방식으로 함께 결합할 수 있다.

유닉스에서 이러한 구성성을 가능하게 하는 것은 다음의 철학 덕분이다.

- 일관된 인터페이스 (A uniform interface)
- 로직과 그 사용의 분리 (Separation of logic and wiring)
- 투명성과 실험 (Transparency and experimentation)
