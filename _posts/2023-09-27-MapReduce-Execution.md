---
title: "데이터 중심의 애플리케이션 설계 - 맵리듀스의 작업 실행 방법"
toc: true
categories:
  - tech
tags:
  - book
  - design system
  - tech
  - backend
---

맵리듀스는 HDFS와 같은 분산 파일 시스템에서 대규모 데이터 세트를 처리하는 코드를 작성할 수 있는 프로그래밍 프레임워크다.

맵리듀스의 데이터 처리 패턴을 단계별로 설명하면 다음과 같다.

1. 입력 파일 세트를 읽고 레코드로 분할한다.
2. 매퍼 함수를 호출하여 각 입력 레코드에서 키와 값을 추출한다.
3. 모든 키-값 쌍을 키별로 정렬한다.
4. 리듀서 함수를 호출하여 정렬된 키-값 쌍을 반복한다. 동일한 키가 여러 번 있는 경우 정렬을 통해 목록에서 인접한 위치에 있으므로 메모리에 많은 상태를 유지하지 않고도 해당 값을 쉽게 결합할 수 있다.

이 네 단계를 하나의 맵리듀스 작업으로 수행할 수 있다.

- 2단계(맵)와 4단계(축소)는 사용자 정의 데이터 처리 코드를 작성한다.
- 1단계(파일을 레코드로 분할)는 입력 형식 파서가 처리한다.
- 3단계(정렬 단계)는 매퍼의 출력이 항상 리듀서에 전달되기 전에 정렬되기 때문에 작성할 필요가 없다.

맵리듀스 작업을 생성하려면 다음과 같이 동작하는 두 개의 콜백 함수, mapper 및 reducer를 구현해야 한다.

## 매퍼

매퍼는 모든 입력 레코드에 대해 한 번씩 호출되며, 입력 레코드에서 키와 값을 추출하는 작업을 수행한다.
각 입력에 대해 키-값 쌍을 얼마든지 생성할 수 있다.
한 입력 레코드에서 다음 레코드로 넘어갈 때 어떤 상태도 유지하지 않으므로 각 레코드는 독립적으로 처리된다.

## 리듀서

맵리듀스 프레임워크는 매퍼가 생성한 키-값 쌍을 가져와서 동일한 키에 속하는 모든 값을 수집하고 해당 값 컬렉션에 대한 반복기(Iterator)를 사용하여 리듀서를 호출한다.
리듀서는 출력 레코드(예를 들어, 동일한 URL의 발생 횟수)를 생성할 수 있다.
맵리듀스에서 두 번째 정렬 단계가 필요한 경우, 두 번째 맵리듀스 작업을 작성하고 첫 번째 작업의 출력을 두 번째 작업의 입력으로 사용하는 방식으로 구현하면 된다.

정리하자면, 매퍼의 역할은 데이터를 정렬에 적합한 형태로 만들어서 준비하는 것이다. 리듀서의 역할은 정렬이 완료된 데이터를 처리하는 것이다.
