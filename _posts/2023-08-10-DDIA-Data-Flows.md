---
title: "데이터 중심의 애플리케이션 설계 - 데이터흐름의 3가지 방식"
toc: true
categories:
  - tech
tags:
  - book
  - design system
  - tech
  - backend
  - encoding
---

데이터가 한 프로세스에서 다른 프로세스로 이동하는 방식은 매우 다양하다.

- 데이터베이스를 통해
- 서비스 호출을 통해(REST 및 RPC)
- 비동기 메시지 전달을 통해

## 데이터베이스를 통한 데이터 흐름

데이터베이스에 쓰는 프로세스는 데이터를 인코딩하고, 데이터베이스에서 읽는 프로세스는 데이터를 디코딩gksek.

여러 개의 서로 다른 프로세스가 동시에 데이터베이스에 액세스하는 것이 일반적이다.
애플리케이션이 변경되는 환경에서는 데이터베이스에 액세스하는 일부 프로세스가 최신 코드를 실행하고 일부 프로세스는 이전 코드를 실행할 가능성이 높다. 예를 들어, 새 버전이 롤링 업그레이드로 배포되고 있기 때문에 일부 인스턴스는 업데이트되었지만 다른 인스턴스는 아직 업데이트되지 않은 경우가 있다.  
데이터베이스의 값이 최신 버전의 코드에 의해 작성된 후 여전히 실행 중인 이전 버전의 코드에서 읽혀질 수 있다. 따라서 데이터베이스에도 순방향 호환성이 필요하다.

레코드 스키마에 필드를 추가하고 최신 코드가 새 필드에 대한 값을 데이터베이스에 쓴다면, 그 후 새 필드에 대해 아직 알지 못하는 이전 버전의 코드가 레코드를 읽고 업데이트한 다음 다시 쓴다.  
이 상황에서 바람직한 동작은 일반적으로 새 필드를 해석할 수 없더라도 이전 코드가 새 필드를 그대로 유지하는 것이다.
앞서 설명한 인코딩 형식은 이러한 알 수 없는 필드 보존을 지원하지만, 애플리케이션 수준에서 주의를 기울여야 하는 경우도 있다. 예를 들어, 데이터베이스 값을 애플리케이션에서 모델 객체로 디코딩한 후 나중에 해당 모델 객체를 다시 인코딩하는 경우, 해당 변환 과정에서 알 수 없는 필드가 손실될 수 있다.

## 서로 다른 시간에 기록된 서로 다른 값

데이터베이스에서는 일반적으로 모든 값을 언제든지 업데이트할 수 있다.  
즉, 단일 데이터베이스 내에 5밀리초 전에 작성된 값과 5년 전에 작성된 값이 있을 수 있습니다.  
애플리케이션의 새 버전을 배포할 때, 몇 분 안에 이전 버전을 새 버전으로 완전히 교체할 수 있다.

데이터를 새로운 스키마로 재작성(마이그레이션)하는 것은 가능하지만, 대규모 데이터 집합에서는 비용이 많이 들기 때문에 대부분의 데이터베이스에서는 가능하면 이 작업을 피한다. 대부분의 관계형 데이터베이스는 기존 데이터를 다시 작성하지 않고도 기본값이 null인 새 열을 추가하는 등의 간단한 스키마 변경을 허용한다. 이전 행을 읽을 때 데이터베이스는 디스크의 인코딩된 데이터에서 누락된 열에 대해 null을 채운다.

스키마 진화는 기본 스토리지에 다양한 과거 버전의 스키마로 인코딩된 레코드가 포함되어 있더라도 전체 데이터베이스가 단일 스키마로 인코딩된 것처럼 보이도록 한다.

### 아카이브 스토리지

백업 용도나 데이터 웨어하우스로 로드하기 위해 때때로 데이터베이스의 스냅샷을 찍을 수도 있다. 이 경우, 소스 데이터베이스의 원본 인코딩에 여러 시대의 스키마 버전이 혼합되어 있더라도 데이터 덤프는 일반적으로 최신 스키마를 사용하여 인코딩된다.
데이터 덤프는 한 번에 작성되고 그 이후에는 변경할 수 없으므로, Avro 객체 컨테이너 파일과 같은 형식이 적합하다. 또한 Parquet과 같은 분석 친화적인 열 중심 형식으로 데이터를 인코딩하는 것도 좋은 방법이다.

## 서비스를 통한 데이터 흐름: REST 및 RPC

네트워크를 통해 통신해야 하는 프로세스가 있는 경우, 그 통신을 배열하는 몇 가지 다른 방법이 있다. 가장 일반적인 배열은 클라이언트와 서버의 두 가지 역할을 갖는 것이다.  
서버는 네트워크를 통해 API를 노출하고, 클라이언트는 서버에 연결하여 해당 API에 요청할 수 있다.  
이 때, **서버가 노출하는 API를 서비스**라고 한다.

웹은 클라이언트(웹 브라우저)가 웹 서버에 요청하여 HTML, CSS, JavaScript, 이미지 등을 다운로드하는 GET 요청을 하고, 서버에 데이터를 제출하는 POST 요청을 하는 방식으로 작동한다. API는 표준화된 프로토콜 세트와 데이터 형식(HTTP, URL, SSL/TLS, HTML 등)으로 구성된다. 웹 브라우저, 웹 서버, 웹사이트 작성자는 대부분 이러한 표준에 따르기 때문에 어떤 웹 브라우저를 사용해도 모든 웹사이트에 액세스할 수 있다.

웹 브라우저만이 유일한 클라이언트 유형은 아니다. 예를 들어, 모바일 기기나 데스크톱 컴퓨터에서 실행되는 네이티브 앱도 서버에 네트워크 요청을 할 수 있으며, 웹 브라우저 내부에서 실행되는 클라이언트 측 JavaScript 애플리케이션은 XMLHttpRequest를 사용하여 HTTP 클라이언트가 될 수 있다(이 기술을 Ajax 라고 함). 이 경우 서버의 응답은 일반적으로 사람에게 표시하기 위한 HTML이 아니라 클라이언트 측 애플리케이션 코드에서 추가 처리에 편리한 인코딩(예: JSON)으로 된 데이터다. 전송 프로토콜로 HTTP를 사용할 수 있지만, 그 위에 구현되는 API는 애플리케이션별로 다르므로 클라이언트와 서버가 해당 API의 세부 사항에 대해 합의해야 한다.

서버는 그 자체로 다른 서비스의 클라이언트가 될 수 있다(예: 일반적인 웹 앱 서버는 데이터베이스의 클라이언트 역할을 함).
이 접근 방식은 대규모 애플리케이션을 기능 영역별로 더 작은 서비스로 분해하여 한 서비스가 다른 서비스의 일부 기능이나 데이터를 필요로 할 때 다른 서비스에 요청하는 데 자주 사용된다.  
이러한 애플리케이션 구축 방식은 전통적으로 **서비스 지향 아키텍처(SOA)**라고 불렸으며, 최근에는 **마이크로서비스 아키텍처**로 세분화되었다.

서비스는 일반적으로 클라이언트가 데이터를 제출하고 쿼리할 수 있다는 점에서 데이터베이스와 유사하다. 그러나 데이터베이스는 쿼리 언어를 사용하여 임의의 쿼리를 허용하는 반면, 서비스는 서비스의 비즈니스 로직(애플리케이션 코드)에 의해 미리 결정된 입력과 출력만 허용하는 애플리케이션별 API를 노출한다. 이러한 제한은 어느 정도의 캡슐화를 제공한다. 서비스는 클라이언트가 할 수 있는 일과 할 수 없는 일에 대해 세분화된 제한을 가할 수 있다.

<u>서비스 지향/마이크로서비스 아키텍처의 핵심 설계 목표는 서비스를 독립적으로 배포하고 발전시켜서 애플리케이션을 쉽게 변경하고 유지 관리할 수 있도록 하는 것이다.</u>

예를 들어, 각 서비스는 한 팀이 소유해야 하며, 해당 팀은 다른 팀과 조율할 필요 없이 서비스의 새 버전을 자주 릴리스할 수 있어야 한다. 즉, 이전 버전과 새 버전의 서버와 클라이언트가 동시에 실행될 것으로 예상해야 하므로 서버와 클라이언트에서 사용하는 데이터 인코딩은 서비스 API 버전 간에 호환되어야 하며, 이 장에서 설명한 것과 정확히 일치해야 한다.

### 웹 서비스

웹 서비스란?

- HTTP를 서비스와 통신하기 위한 기본 프로토콜로 사용하는 애플리케이션

웹 서비스는 웹에서만 사용되는 것이 아니라 여러 다른 맥락에서 사용되기도 한다.

예를 들면 다음과 같다:

1. 사용자 디바이스에서 실행되는 클라이언트 애플리케이션(예: 모바일 디바이스의 네이티브 앱 또는 Ajax를 사용하는 JavaScript 웹 앱)이 HTTP를 통해 이루어짐
2. 서비스 지향/마이크로서비스 아키텍처의 일부로 동일한 데이터센터 내에 위치한 동일한 조직이 소유한 다른 서비스에 요청하는 서비스 (이러한 종류의소프트웨어를 미들웨어라고 부름)
3. 한 서비스가 인터넷을 통해 다른 조직의 백엔드 시스템 간의 데이터 교환 (신용카드 처리 시스템과 같은 공개 API 또는 사용자 데이터에 대한 공유 액세스를 위한 OAuth)

웹 서비스에는 두 가지 접근 방식이 널리 사용된다: REST와 SOAP 이다.  
이 두 가지 접근 방식은 철학적인 측면에서 거의 정반대이며, 종종 각각의 지지자들 사이에서 열띤 논쟁의 대상이 되기도 한다.  
REST는 프로토콜이 아니라 HTTP의 원칙을 기반으로 하는 설계 철학이다.

- 단순한 데이터 형식을 강조함
- 리소스 식별을 위해 URL을 사용하고 캐시 제어, 인증 및 콘텐츠 유형 협상을 위해 HTTP 기능을 사용함
- REST는 적어도 조직 간 서비스 통합의 맥락에서 SOAP에 비해 인기를 얻고 있으며, 종종 마이크로서비스와 연관됨
- REST의 원칙에 따라 설계된 API를 RESTful이라고 함

이와 대조적으로 SOAP의 특징은:

- 네트워크 API 요청을 위한 XML 기반 프로토콜이다.
- HTTP를 통해 사용되지만, HTTP와 독립적인 것을 목표로 하며 대부분의 HTTP 기능을 사용하지 않음
- HTTP 대신, 다양한 기능을 추가하는 광범위하고 복잡한 관련 표준(웹 서비스 프레임워크)이 함께 제공됨
- SOAP 웹 서비스의 API는 웹 서비스 설명 언어(WSDL)라는 XML 기반 언어를 사용함

> WSDL을 사용하면 클라이언트가 로컬 클래스 및 메서드 호출(XML 메시지로 인코딩되고 프레임워크에서 다시 디코딩됨)을 사용하여 원격 서비스에 액세스할 수 있도록 코드를 생성할 수 있다.

SOAP의 단점:

- 정적으로 유형화된 프로그래밍 언어에서는 유용하지만 동적으로 유형화된 언어에서는 덜 유용함
- WSDL은 사람이 읽을 수 있도록 설계되지 않았고 SOAP 메시지는 수동으로 구성하기에는 너무 복잡한 경우가 많기 때문에 SOAP 사용자는 도구 지원, 코드 생성 및 IDE에 크게 의존함
- SOAP과 서로 다른 공급업체의 구현 간에 상호 운용성이 떨어짐

이러한 모든 이유로 인해 SOAP는 여전히 많은 대기업에서 사용되고 있지만 대부분의 소규모 기업에서는 선호도가 떨어지고 있다.

## 원격 프로시저 호출(RPC)의 문제점

RPC 모델은 원격 네트워크 서비스에 대한 요청을 동일한 프로세스 내에서 프로그래밍 언어에서 함수나 메서드를 호출하는 것과 동일하게 보이게 한다. (이러한 추상화를 위치 투명성, location transparency이라고 한다). RPC는 처음에는 편리해 보이지만 이 접근 방식에는 근본적인 결함이 있다.

- 네트워크 문제로 인해 요청이나 응답이 손실되거나 원격 컴퓨터가 느리거나 사용할 수 없는 문제가 발생한다.
- 네트워크 요청에는 시간 초과로 인해 결과 없이 반환될 수 있다. 이 경우 원격 서비스에서 응답을 받지 못하면 요청이 전달되었는지 여부를 알 수 없다.
- 실패한 네트워크 요청을 다시 시도하면 요청은 실제로 통과하고 응답만 손실되는 경우가 발생할 수 있다.
- 네트워크 요청은 함수 호출보다 훨씬 느리고 지연 시간도 매우 가변적이어서 네트워크가 혼잡하거나 원격 서비스에 과부하가 걸리면 정확히 같은 작업을 수행하는 데 몇 초가 걸릴 수 있다.
- 로컬 함수를 호출할 때 로컬 메모리에 있는 객체에 대한 참조(포인터)를 전달할 수 있지만, 네트워크 요청을 할 때는 모든 매개변수를 네트워크를 통해 전송할 수 있는 바이트 시퀀스로 인코딩해야 한다.
- 클라이언트와 서비스는 서로 다른 프로그래밍 언어로 구현될 수 있으므로 RPC 프레임워크는 데이터 타입을 한 언어에서 다른 언어로 변환해야 한다.

이러한 모든 요소는 원격 서비스를 프로그래밍 언어에서 로컬 객체와 너무 비슷하게 만들려고 할 필요가 없다는 것을 의미하며, 근본적으로 다른 것이기 때문이다. REST의 매력 중 하나는 네트워크 프로토콜이라는 사실을 숨기려 하지 않는다는 것이다.

### RPC의 현재 방향

이러한 모든 문제에도 불구하고 RPC는 사라지지 않고 있다.  
다양한 RPC 프레임워크가 구축되어있다. 예를 들어 Thrift와 Avro에는 RPC 지원이 포함되어 있고, gRPC는 프로토콜 버퍼를 사용하는 RPC 구현이며, Finagle도 Thrift를 사용하고, Rest.li는 HTTP를 통해 JSON을 사용한다.  
이러한 프레임워크 중 일부는 서비스 검색, 즉 클라이언트가 특정 서비스를 찾을 수 있는 IP 주소와 포트 번호를 찾을 수 있도록 하는 서비스 검색 기능도 제공한다.  
바이너리 인코딩 형식의 사용자 지정 RPC 프로토콜은 REST를 통한 JSON과 같은 일반적인 프로토콜보다 더 나은 성능을 달성할 수 있다.

그러나 RESTful API는 다른 중요한 장점도 있다.

- 실험 및 디버깅에 적합하고(코드 생성이나 소프트웨어 설치 없이 웹 브라우저나 명령줄 도구 curl을 사용하여 간단히 요청할 수 있음),
- 모든 메인스트림 프로그래밍 언어 및 플랫폼에서 지원되며, 방대한 도구 생태계(서버, 캐시, 로드 밸런서, 프록시, 방화벽, 모니터링, 디버깅 도구, 테스트 도구 등)가 있다

이러한 이유로 REST는 공용 API의 주류가 되었다. RPC 프레임워크의 주요 초점은 일반적으로 동일한 데이터센터 내에 있는 동일한 조직이 소유한 서비스 간의 요청에 있다.

### RPC를 위한 데이터 인코딩 및 진화

RPC 클라이언트와 서버를 독립적으로 변경하고 배포할 수 있는 것이 중요하다. 데이터베이스를 통한 데이터 흐름과 비교할 때, 서비스를 통한 데이터 흐름의 경우 모든 서버가 먼저 업데이트되고 모든 클라이언트가 두 번째로 업데이트된다고 가정하는 것이 합리적이다. 따라서 요청에 대해서는 역방향 호환성만 필요하고 응답에 대해서는 정방향 호환성만 있으면 된다.

서비스 호환성은 RPC가 조직 경계를 넘어 커뮤니케이션에 사용되는 경우가 많기 때문에 서비스 제공업체가 클라이언트를 제어할 수 없고 업그레이드를 강제할 수 없는 경우가 많다는 사실 때문에 더욱 어렵게 만든다. 따라서 호환성을 오랫동안, 어쩌면 무기한으로 유지해야 한다. 호환성을 깨는 변경이 필요한 경우 서비스 제공업체는 여러 버전의 서비스 API를 나란히 유지 관리해야 하는 경우가 많다.
API 버전 관리가 어떻게 작동해야 하는지에 대한 합의가 없다.

RESTful API의 경우, 일반적인 해결 방식은 **URL 또는 HTTP Accept 헤더에 버전 번호를 사용하는 것이다.** API 키를 사용하여 특정 클라이언트를 식별하는 서비스의 경우, 또 다른 옵션은 클라이언트가 요청한 API 버전을 서버에 저장하고 별도의 관리 인터페이스를 통해 이 버전 선택을 업데이트할 수 있도록 한다.

## 메시지 전달 데이터 흐름

RPC와 데이터베이스 사이에 있는 비동기 메시지 전달 시스템에 대해 간략히 살펴보자.  
클라이언트의 요청(일반적으로 메시지라고 함)이 짧은 지연 시간으로 다른 프로세스에 전달된다는 점에서 RPC와 유사하다.

메시지가 직접 네트워크 연결을 통해 전송되지 않고 메시지 브로커(메시지 큐 또는 메시지 지향 미들웨어라고도 함)라는 중개자를 통해 전송되며 메시지를 일시적으로 저장한다는 점에서 데이터베이스와 유사하다.

메시지 브로커를 사용하면 직접 RPC에 비해 몇 가지 장점이 있다:

- 수신자가 사용할 수 없거나 과부하가 걸린 경우 버퍼 역할을 할 수 있으므로 시스템 안정성이 향상된다.
- 메시지 브로커는 충돌이 발생한 프로세스에 메시지를 자동으로 재전송하여 메시지가 손실되는 것을 방지할 수 있다.
- 발신자가 수신자의 IP 주소와 포트 번호를 알 필요가 없으므로 가상 머신이 자주 왔다 갔다 하는 클라우드 배포에서 특히 유용하다.
- 하나의 메시지를 여러 수신자에게 보낼 수 있습니다.
- 발신자와 수신자를 논리적으로 분리합니다(발신자는 메시지를 게시하기만 하고 누가 메시지를 소비하는지는 신경 쓰지 않음).

그러나 메시지 전달 통신은 일반적으로 단방향이므로 발신자는 일반적으로 메시지에 대한 응답을 기대하지 않는다는 점이 RPC와의 차이점이다. 통신 패턴은 비동기식입니다.

### 메시지 브로커

과거에는 메시지 브로커의 환경은 TIBCO, IBM WebSphere, webMethods와 같은 회사의 상용 엔터프라이즈 소프트웨어가 지배적이었다. 최근에는 RabbitMQ, ActiveMQ, Hor- netQ, NATS, Apache Kafka와 같은 오픈 소스 구현이 인기를 얻고 있다.  
한 프로세스가 지정된 큐(또는 토픽)으로 메시지를 보내면 브로커가 해당 큐(또는 토픽)의 하나 이상의 소비자 또는 구독자에게 메시지가 전달되도록 한다. 동일한 토픽에 많은 생산자와 많은 소비자가 있을 수 있다.

토픽은 단방향 데이터 플로우만 제공한다. 그러나 소비자는 메시지를 다른 토픽에 게시할 수도 있고, 원래 메시지를 보낸 사람이 사용하는 응답 대기열에 게시할 수도 있다(RPC와 유사한 요청/응답 데이터 흐름이 가능).
메시지 브로커는 일반적으로 특정 데이터 모델을 강제하지 않으며, 메시지는 일부 메타데이터가 포함된 바이트 시퀀스일 뿐이므로 어떤 인코딩 형식이든 사용할 수 있다. 인코딩이 이전 버전 및 이전 버전과 호환되는 경우 게시자와 소비자를 독립적으로 변경하고 원하는 순서대로 배포할 수 있는 유연성이 가장 뛰어나다.
소비자가 메시지를 다른 주제로 다시 게시하는 경우, 앞서 데이터베이스의 맥락에서 설명한 문제를 방지하기 위해 알 수 없는 필드를 보존하도록 주의를 기울여야 할 수 있다.

### 분산 액터 프레임워크

액터 모델은 단일 프로세스에서 동시성을 위한 프로그래밍 모델이다. 스레드와 스레드 관련 문제인 경쟁 조건, 잠금, 교착 상태를 직접 처리하는 대신 로직이 액터에 캡슐화됩니다. 각 액터는 일반적으로 하나의 클라이언트 또는 엔티티를 나타내며, 다른 액터와 공유되지 않는 일부 로컬 상태를 가질 수 있으며, 비동기 메시지를 주고받으며 다른 액터와 통신한다. 메시지 전달은 보장되지 않으며 특정 오류 시나리오에서는 메시지가 손실될 수 있다. 각 액터는 한 번에 하나의 메시지만 처리하므로 스레드에 대해 걱정할 필요가 없으며, 프레임워크에서 각 액터를 독립적으로 스케줄링할 수 있다.

분산 액터 프레임워크는 여러 노드에 걸쳐 애플리케이션을 확장하는 데 사용된다. 발신자와 수신자가 같은 노드에 있든 다른 노드에 있든 상관없이 동일한 메시지 전달 메커니즘이 사용된다. 서로 다른 노드에 있는 경우, 메시지는 바이트 시퀀스로 투명하게 인코딩되어 네트워크를 통해 전송되고 상대방에서 디코딩된다.

액터 모델은 단일 프로세스 내에서도 메시지가 손실될 수 있다고 이미 가정하기 때문에 위치 투명성은 RPC보다 액터 모델에서 더 잘 작동한다. 네트워크를 통한 지연 시간은 동일한 프로세스 내에서보다 높을 가능성이 높지만, 액터 모델을 사용하면 로컬과 원격 통신 간의 근본적인 불일치가 적다.

분산형 액터 프레임워크는 기본적으로 메시지 브로커와 액터 프로그래밍 모델을 단일 프레임워크에 통합한다. 그러나 액터 기반 애플리케이션의 롤링 업그레이드를 수행하려는 경우 새 버전을 실행하는 노드에서 이전 버전을 실행하는 노드로 또는 그 반대로 메시지를 전송할 수 있으므로 여전히 정방향 및 역방향 호환성에 대해 걱정해야 한다.

널리 사용되는 세 가지 분산 액터 프레임워크는 다음과 같이 메시지 인코딩을 처리한다:

- Akka는 기본적으로 Java의 기본 제공 직렬화를 사용하며, 이는 정방향 또는 역방향 호환성을 제공하지 않는다. 그러나 이를 프로톨 버퍼와 같은 것으로 대체할 수 있으므로 롤링 업그레이드를 수행할 수 있다.
- Orleans는 기본적으로 롤링 업그레이드 배포를 지원하지 않는 사용자 정의 데이터 인코딩 형식을 사용하므로, 애플리케이션의 새 버전을 배포하려면 새 클러스터를 설정하고 이전 클러스터에서 새 클러스터로 트래픽을 이동한 다음 이전 클러스터를 종료해야 한다.
- Erlang OTP에서는 고가용성을 위해 설계된 많은 기능이 있음에도 불구하고 레코드 스키마를 변경하는 것이 의외로 어렵고, 롤링 업그레이드가 가능하지만 신중하게 계획해야 한다.