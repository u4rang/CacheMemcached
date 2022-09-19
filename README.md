[TOC]

------

# 개정 이력

|    날짜    |             내용             | 작성자 |   비고    |
| :--------: | :--------------------------: | :----: | :-------: |
| 2022.03.02 |           최초작성           | 김삼성 | 주제 선정 |
| 2022.03.15 |       시스템 개요 작성       | 김삼성 |           |
| 2022.03.31 |      요구사항 정의 작성      | 김삼성 |   기능    |
| 2022.04.10 |   요구사항 정의 추가 작성    | 김삼성 |  비기능   |
| 2022.04.29 |       1차 피드백 반영        | 김삼성 |           |
| 2022.05.31 | 아키텍처 설계 문제 분석 작성 | 김삼성 |           |
| 2022.07.11 |       2차 피드백 반영        | 김삼성 |           |
| 2022.07.31 |   아키텍처 상세 설계 작성    | 김삼성 |           |
| 2022.08.26 |       3차 피드백 반영        | 김삼성 |           |
| 2022.09.02 |       4차 피드백 반영        | 김삼성 |           |
|            |                              |        |           |
|            |                              |        |           |



-----------



# 용어 사전

|             용어             | 설명                                                         |   비고   |
| :--------------------------: | ------------------------------------------------------------ | :------: |
|      고객사 물류시스템       | 고객사에서 연구개발 물품 및 위탁생산하는 의약품에 대한 배송상태를 조회하는 시스템 |   업무   |
|           SDSCache           | 자체개발하는 Golang 기반의 Cache솔루션                       |  솔루션  |
|   LRU(Least Recently Used)   | - Cache 항목 추가를 위한 공간을 확보하기 위해 Cache에서 필요 없는 항목을 eviction 알고리즘<br />- 가장 오랜된 항목들을 evict 하는 알고리즘 | 알고리즘 |
|    RTT(Round Trime Time)     | - 요청이 경과된 시간으로, Client로 응답이 올 때까지 Client에서 서버로 가는 요청 경관 시간 포함 |          |
| SLA(Service Level Agreement) | - 사용자에 대한 수용할 수 있는 웹 서비스 요청 응답 시간을 유지하기 위해 허용되는 최대 RTT<br />- 응답시간이 SLA를 넘어서는 경우 웹 서비스 요청을 생성하는 데 수용할 수 없는 응답시간이 나타남 |          |
|          Heartbeat           | - 정상 작동을 나태나거나 시스템의 일부를 동기화하는 주기 신호 |          |
|                              |                                                              |          |
|                              |                                                              |          |
|                              |                                                              |          |
|                              |                                                              |          |
|                              |                                                              |          |
|                              |                                                              |          |
|                              |                                                              |          |
|                              |                                                              |          |
|                              |                                                              |          |
|                              |                                                              |          |
|                              |                                                              |          |
|                              |                                                              |          |
|                              |                                                              |          |
|                              |                                                              |          |
|                              |                                                              |          |



------------

# Executive Message

고객사 물류시스템의 문제점 해결을 위해 다음 2 개의 사항에 집중한다.

- 고객사 물류시스템의 아키텍처 변경
- 고객사 물류시스템에서 요구하는 Cache 기능에 대해 자체 Cache  설계 및 개발, 적용



![Cache_ExecutiveMessage drawio](https://user-images.githubusercontent.com/26420767/186798459-c7fcc9fd-3fff-4204-91a9-dcde3af9daeb.png)



------------------

# 1. 시스템 개요

> 과제 대상 시스템 개념, 구성 등 시스템 이해를 위한 기술
>
> 기존 시스템이 있는 경우 기존 시스템 분석 내용 기술
>



## 1.1 프로젝트 개요

### 1.1.1 프로젝트 배경

고객사 물류 시스템에서 발생하는 문제점을 해결 하기 위해 업무 분석 및 개선사항 도출을 통해 물류 업무 프로세스 변경과 고객사 물류 시스템 아키텍처의 변경 그리고 성능 개선을 위한 Cache 활용을 배경으로 한다. 

그리고 당사는 CSP사업자이다. 개발한 Cache솔루션을 Cache상품 카테고리에 포함한다. 서비스의 다양성 확보 및 직접 관리하는 Cache솔루션으로 Opensource 기반 솔루션에 대비 빠른 조치를 기대한다. 



#### 1.1.1.1 고객사 물류 시스템의 외부 API 반복 호출로 인한 성능 저하 이슈

고객사 물류 시스템 사용자가 상품 배송 상태를 조회 시, 물류 시스템의 전체 기능에 대한 응답속도가 느려지는 성능 저하 이슈가 있다.

성능 개선을 위해 적극적인 Cache 활용과 분석결과에 따라 물류 시스템에 대한 아키텍처의 재설계가 필요하다. 



#### 1.1.1.2 Cache솔루션에서 제공하는 기능의 부족 

오픈소스 기반 Cache인 Redis와 Memcached, Ignite 에서 제공하는 기능은 고객사 물류 시스템 성능 개선을 위해 만족하지 못한다. 

따라서 고객사 물류 시스템에서 Cache솔루션에 요구하는 기능을 도출하고 자체Cache솔루션을 개발한다.



#### 1.1.1.3 Opensource 라이센스 변경 이슈와  프로테스트웨어 (Protestware)

2022년, IT 산업에서는 2 가지의 Opensource 이슈가 있다. 

첫 번째는 Opensource 라이센스 변경 이슈 이다. 변경에 따라 사용하는 Opensource 더 이상 사용하기 불가하여 대체 솔루션으로 변경해야 하는 상황이다.  최근 사례로는 Object Storage 인 minIO 가 있다. 라이센스 변경에 따라 minIO를 사용하는 솔루션은 소스코드를 공개 해야 하는 의무가 발생하였다.

두 번째는 프로테스트웨어 이다. 프로테스트웨어는 개발자가 스스로 자신의 코드를 훼손하는 사보타주 행위이다. 개발자가 자신의 소프트웨어를 수정할 권리가 있지만 이 같은 행동은 Opensource 생태계의 신뢰를 손상한다.

2022년 1월부터 3월까지 3건의 프로테스트웨어가 있다.

|    일시    | 사건                                                         |
| :--------: | ------------------------------------------------------------ |
| 2022년 1월 | 개발자에 의한 NPM 라이브러리 Faker.js, Colors.js 에 무한 루프 삽입 한 사건 |
| 2022년 2월 | 특정 테라폼 모듈이 라이센스 항목을 변경하여, 러시아의 침공을 인정하고 "푸틴은 멍청이(Putin khuylo!)" 라는 내용에 동의해야만 사용가능하게 만든 사건 |
| 2022년 3월 | npm 리포지토리에 호스팅된 자바스크립트 구성요소, node-ipc 개발자가 러시아의 우크라이나 침공을 비판하기 위해 사용자의 컴퓨터에 임의로 파일을 추가하거나 삭제하는 코드를 배포하는 사건 |

위와 같은 Opensource 이슈 때문에, IT컨설팅, 시스템 통합(SI) 과 IT유지보수 그리고 CSP 사업을 하는 당 사는 Opensource 기반 솔루션 중 SDSCache을 선택하여 내제화(內製化)하여 고객사에 적용하고, CSP 사업에 관리형(Managed), 완전 관리형(Fully Managed) 서비스로 제공하기로 결정하였다.



#### 1.1.1.4 CSP에서 필수적인 Cache 서비스

Global 선도 

[^CSP]: Cloud Service Provider

 사업자의 경우 Opensource 기반 관리형 또는 완전 관리형 서비스 Cache를 상품으로 제공한다.

| CSP 社 |       서비스 명        | 특징                      |
| :----: | :--------------------: | ------------------------- |
|  AWS   | ElasticCache Memcached | Opensource Memcached 기반 |
|  AWS   |   ElasticCache Redis   | Opensource Redis 기반     |
|  GCP   | Memorystore Memcached  | Opensource Memcached 기반 |
|  GCP   |   Memorystore Redis    | Opensource Redis 기반     |
| Azure  | Azure Cache for Redis  | Opensource Redis 기반     |
|  SDS   |     SCP for Redis      | Opensource Redis 기반     |

따라서 당사는 CSP 사업자로 Opensource 기반 Cache 서비스 외 내제화된 Cache 서비스를 상품 카테고리에 추가하여 경쟁력 확보가 필요하다



### 1.1.2 프로젝트 범위

본 프로젝트의 범위는 다음과 같다.

- 고객사 물류 시스템의 Pain Point 분석 및 개선 아키텍처 제시
- 목표 성능 및 안정성, 그리고 기본 기능 제공 하는 Cache 아키텍처 제시 및 솔루션 확보
- BMT 및 PoC 수행을 통한 기존 Opensource 대비 성능 및 안정성, 기능성 비교



### 1.1.3 Busincess Context

금번 프로젝트의 Business Context는 다음과 같다.

![BusinessContext](https://user-images.githubusercontent.com/26420767/190298551-01578682-94c4-4979-a895-b6efe687af41.png)



SDSCache의 주요 Stakeholder와 주요 Concern는 다음과 같다.

![주요관심사](https://user-images.githubusercontent.com/26420767/190299433-f5c98d6e-17c9-41a3-b9f9-0f1366691a5d.png)



### 1.1.4 Business Goal

SDSCache에 대하여 Stakeholder들이 기대하는 Business 목표이다.

|                         Stakeholder                          |  ID   | Statement                                              | Importance(상, 중, 하) |
| :----------------------------------------------------------: | :---: | ------------------------------------------------------ | :--------------------: |
|                    고객사물류시스템개발자                    | BG-01 | SDSCache 안정적인 적용                                 |           상           |
|                    고객사물류시스템운영자                    | BG-02 | 고객사 물류 시스템의 안정적인 운영                     |           상           |
| 고객사물류시스템개발자,<br />고객사물류시스템운영자,<br />CSP시스템운영자 | BG-03 | SDSCache로 오픈소스 이슈 Hedge                         |           상           |
|                    고객사물류시스템사용자                    | BG-04 | SDSCache 도입으로 안정적인 서비스 기대                 |           상           |
|                       CSP시스템개발자                        | BG-11 | 관리형, 완전관리형 SDSCache 개발                       |           중           |
|                       CSP시스템운영자                        | BG-12 | 안정적인 CSP 시스템 운영                               |           중           |
|                       CSP시스템운영자                        | BG-13 | 다양한 Cache 상품 제공                                 |           상           |
|                       CSP시스템사용자                        | BG-14 | 관리형, 완전관리형 SDSCache 기반 관리 포인트 감소 기대 |           중           |



## 1.2 AS-IS 분석

### 1.2.1 고객사 물류 시스템

#### 1.2.1.1 고객사 물류 시스템의 Software Stack

|     항목     |  Software  | Version |    비고    |
| :----------: | :--------: | :-----: | :--------: |
|     Java     |  OpenJDK   |   1.8   | Opensource |
|  Framework   | SpringBoot |   2.5   | Opensource |
|  Framework   |    Lego    |         | S사 솔루션 |
| UI Framework |   Vue.js   |   2.X   | Opensource |
|      UI      |   UI Dev   |         | S사 솔루션 |
|   Database   | PostgreSQL |   10    | Opensource |



#### 1.2.1.2 고객사 물류 시스템의 Runtime View

다음은 고객사 물류 시스템의 Runtime View(Component Diagram) 이다. 

![RUNTIME_VIEW_AS-IS_물류시스템](https://user-images.githubusercontent.com/26420767/182987494-547db422-e569-4af8-802c-f44fcb3dcb37.png)

고객사 물류시스템에서 사용중인 기존 Cache Ignite로 사용하는 기능은 인증 및 권한, 세션 그리고 메뉴이다. 
고객사 물류 업무는 시스템 구축 시 요구사항 중 업무 데이터는 DB에  저장하고 DB에서 직접 조회하는 항목이 있다. 
따라서, 업무에서는 Cache를 사용하지 않고 매번 데이터를 DB에서 조회한다. 시간이 지나면서 일부 물류 업무의 조회 기능, 배송상태 조회에 대해 느리다는 의견의 SR이 있다.
또한 DB에서 Exclusive Lock의 발생으로 Warning 메시지가 업무시간 중 빈번하게 고객사 물류 시스템 운영자에게 전송된다.



#### 1.2.1.3 고객사 물류 시스템의 Module View

다음은 공통 시스템을 제외한 고객사 물류 시스템(myLogi)의 Module View 이다. 
1.2.1.2에서 언급한 문제점들이 고객사 물류 시스템 중 Delivery Module에서 발생한다. 

![MODULE_VIEW_AS-IS_물류시스템](https://user-images.githubusercontent.com/26420767/182988462-4bc61589-cecf-4bd6-8368-65e27be31c06.png)



#### 1.2.1.4 고객사 물류 시스템의 성능 이슈 분석

고객사 물류 시스템의 배송 상태를 확인하는 기능은 다수의 사용자에 의해 조회된다. 고객사 물류 시스템은 사용자에 의해 조회가 발생하면 매 번 상품의 배송 상태를 배송사가 제공하는 API를 호출하여 정보를 제공한다. 그리고 상품의 배송 정보를 저장하는 테이블에 갱신한다.

사용자가 배송 상태를 자주 조회하는 업무시간, 즉 오전 10시부터 오후 4시까지 다음과 같이 2가지 문제가 발생한다.

| 항목 | 문제                                                         | 비고 |
| :--: | ------------------------------------------------------------ | ---- |
| 문1. | "고객사 물류 시스템 사용자로 부터 배송 상태를 조회하는데, 조회가 느리다." 라는 SR이 1주일에 2 ~3회 건 정도  접수된다. |      |
| 문2. | 고객사 물류 시스템 운영담당자는 ARMS 로부터 **DML Row Lock** 에 대한 Warning 메시지를 1일 5회 이상 전달 받는다. |      |



다음은 문2. 에서 발생하는 Warning 메세지이다.

![LocK메시지](https://user-images.githubusercontent.com/26420767/183354035-383b1162-c41c-4818-bda9-69a8ada7917c.png)



다음은 Warning 메시지 발생 원인 "RawExclusiveLock" 상세 내용이다.

![Lock메시지2](https://user-images.githubusercontent.com/26420767/183408959-1dcb15e0-4266-4cfe-91f5-d90644db18a2.png)



Warning 메시지가 발생하는 업무는 "사용자가 외부 API(Nuxt서비스)를 조회하여 배송상태를 조회" 이다. 
다음은 Warning 메시지가 발생하는 업무인 "배송상태조회"의 Sequence Diagram 이다.

![SEQ_AS-IS_배송상태조회](https://user-images.githubusercontent.com/26420767/190858293-fad0d5e8-c708-4077-9e12-485150bb9f1c.png)



응답 메시지의 데이터 타입 및 크기가 문제를 발생하는지 확인하기 위해 조사하였다. 
배송사에서 배송에 대한 상태정보를 조회하면 다음과 같은 배송상태 정보를 받는다.

| 항목        | 설명           | 비고                                                         |
| ----------- | -------------- | ------------------------------------------------------------ |
| 데이터 타입 | JSON           | - JSON은 복잡한 XML 대비 Parsing 하기 위한 컴퓨팅 파워가 덜 소모된다. |
| 데이터 크기 | 1500 Byte 이내 | - 예제의 크기는 962 Byte<br />- 2022년 1월부터 6월까지 사용된 배송번호에 따른 배송상태의 데이터 중 크기가 가장 큰 것은 1480 Byte |



다음은 배송사로부터 받는 JSON 기반의 응답의 샘플이다.

~~~json
{
    \"Header\": {
        \"MessageId\": \"736cfa3c-e125-4db0-891b-9e2b88fc9062\",
        \"Waybill\": \"630X14832166\",
        \"References\": {
            \"Shipper\": \"First test\",
            \"Consignee\": \"First test\",
            \"PatientId\": \"\",
            \"ChargeReference\":\"DO#2012D0021\",
            \"PurchaseOrderCode\":\"DS\"
        }
    },
    \"Delivered\": null,
    \"PickupOnBoard\":{\"EventCode\":19,\"EventDateTime\":\"2022-05-29T10:37:00+09:00\"},
    \"Booked\":{\"EventCode\":1,\"EventDateTime\":\"2022-05-28T01:12:13.12+00:00\"},
    \"TransportationEvents\":[{\"Event\":\"Transportation Arrived\",\"Location\":\"Marken Seoul\",\"EventCode\":39,\"EventDateTime\":\"2022-05-29T18:00:00+09:00\"},
                              {\"Event\":\"Transportation Departed\",\"Location\":\"Marken London\",\"EventCode\":26,\"EventDateTime\":\"2022-06-01T22:56:32.497+00:00\"}],
    \"VerificationEvents\": []
}
~~~



위 2 가지 문제에 대해 Sequence Diagram 에서 표현하면 다음과 같다.

![SEQ_AS-IS_배송상태조회_문제점분석](https://user-images.githubusercontent.com/26420767/190858317-4859afbc-66b9-4866-a585-e98a78f3e0a4.png)



| 항목 | 원인분석                                                     | 비고                                                         |
| :--: | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 문1. | 배송상태조회업무에서 Cache를 사용하지 않는다.                | 분석 설계단계에서 식약청 공시 조회 및 배송상태조회 기능에 대해<br /> Cache 기능 미사용을 업무 요건으로 정의하였다. |
| 문2. | 배송상태를 실시간 동기 방식으로 조회 한다. <br />외부 시스템인 Nuxt의 상태 또는 네트워크 상태 등 외부 요인으로 인하여 지연이 발생한다. |                                                              |
| 문3. | 여러 사용자가 동시에 같은 송장번호를 조회 한다.<br /> 송장번호의 배송 상태를  갱신하기 위한 DML Row Lock 이 발생하며 시스템 자원의 불필요한 이벤트 발생과 응답 속도 지연이 발생한다. | 배송상태에 대해 이력으로 관리하지 않는다.                    |



DML Row Lock 이벤트가 자주 발생하는 시스템의 경우, Transaction Lock 경합이 발생하여 전체 서비스가 느려지거나 성능 상의 문제를 야기한다. 그리고 대부분의 경우 비즈니스 로직 또는 Transcation 처리 로직이 잘못 구현된 경우가 많다.



#### 1.2.1.5 고객사 물류시스템에서 사용하는 Cache솔루션에 대한 요구 기능

고객사 물류시스템의 업무에서 사용하는 SDSCache에 대한 요구 기능은 다음과 같다.

|  분류  |                항목                 | 설명                                                         |
| :----: | :---------------------------------: | ------------------------------------------------------------ |
| 비기능 |         3 ms 이하 응답속도          | Cache의 GET, SET, DEL 동작에 대한 기대 응답속도              |
| 비기능 |           3,000 QPS 지원            | 초당 3,000 QPS(Queries Per Second) 성능                      |
| 비기능 |          Multi Thread 지원          | Single Thread 환경에서 시간이 오래 걸리는 동작으로 인한 Cache솔루션 전체 성능 저하 방지 |
|  기능  |     TTL(Time To Live) 기능 지원     | Cache에 대한 시간에 의한 만료정책                            |
|  기능  | Cache 만료정책 알고리즘의 변경 지원 | 운영 중 Cache솔루션의 메모리 사용이 100%가 되면 알고리즘에 의해 Cache를 메모리에서 삭제 |
|  기능  |        Value 길이 10 MB 지원        | Cache솔루션에서 데이터을 갖는 Value에 대해 최대 10 MB 길이 사용 |

다음은 요구 기능이 도출된 근거에 대한 설명이다.



##### 1.2.1.5.1 3 ms 이하 응답속도

"별첨 2. 비교대상 Redis와 Memcached에 대한 BMT" 에서  Redis와 Memcached의 평균 응답 속도를 기반으로 도출되었다.

|   항목    | 평균 응답 속도 | 비고 |
| :-------: | :------------: | :--: |
|   Redis   |      3 ms      |      |
| Memcached |      3 ms      |      |

두 Cache솔루션의 평균 응답 속도 중 작은 값을 기준으로 선택되었다.



##### 1.2.1.5.2 3,000 QPS 지원

자체 개발 Cache솔루션은 고객사 물류시스템에서만 아니라 고객사의 모든 시스템에서 Cache로 사용하게 된다. 

QPS는 Queries Per Second 로 조회에 대하여 1초당 성능이다. 따라서 시스템을 사용하는 고객사의 임직원 수와 보정치를 이용해서 계산한다.

| 항목 | 임직원 수<br />(2022.08) | 1 명 당 <br />1초 조회 수 | 1 년 당 임직원<br />증가율 | 시스템<br />기대년수 | 여유율 |
| :--: | :----------------------: | :-----------------------: | :------------------------: | :------------------: | :----: |
|  값  |          920 명          |          1.2 회           |            15 %            |         5 년         |  20%   |

계산식
$$
QPS = (임직원 수) \times (1년 당 임직원 증가율)^{시스템기대년수} \times (1초 당 조회 수)\times (여유율)
$$


##### 1.2.1.5.3 Multi Thread 지원

Single Thread를 지원하는 솔루션의 문제 중 시간이 오래 걸리는 동작으로 인한 전체 솔루션의 성능 저하이다. 따라서 Multi Thread를 지원하는 Cache솔루션으로 위와 같은 문제에 대한 해결을 기대한다.



##### 1.2.1.5.4 TTL 기능 지원

업무 요건 중 배송상태에 대한 유효한 기간을 갖는다. 따라서 Cache에 대한 기간 만료 정책을 지원한다. 



##### 1.2.1.5.5 Cache 만료정책 알고리즘의 변경 지원

운영 중 SDSCache의 메모리 사용이 100%가 되면 업무 요건을 분석한다. 그리고 업무 요건에 적합한 알고리즘으로 변경하도록 지원한다.



##### 1.2.1.5.6 Value 길이 10 MB 지원

고객사 물류시스템이 배송상태 정보를 Cache솔루션에 저장하는 Cache의 Value 최대 길이는 1480 Byte 이다. 
배송상태 정보 외 ERP 시스템과의 인터페이스에서 사용되는 데이터 타입이 XML인 Object는 최대 길이가 8 MB이다. 이 데이터를 Cache솔루션에 저장하여 성능 개선을 기대한다.  따라서 10 MB Value 길이를 지원한다. 



### 1.2.2 기존 SDSCache 분석

2021년부터 2022년까지 지난 1 년간 오픈소스 기반 Cache솔루션에 대한 관심도와 선진 CSP 社에서 제공하는 SDSCache을 조사한다. Cache의 기본적인 기능과 SDSCache 별 특화된 기능을  조사한다. 



#### 1.2.2.1 SDSCache의 관심도에 관한 Google Trend

널리 사용되는 Cache솔루션을 조사하여 다음과 같은 목록을 도출하였다.

- Apache Ignite
- Memcached
- Redis(REmote DIctionary Server in full)
- Couchbase Server
- Hazelcast IMDG

<img width="1164" alt="GoogleTrend_Cache" src="https://user-images.githubusercontent.com/26420767/186809626-c4ea9cd6-eece-47a0-8fa2-a0b4a9e19b84.png">

이 중 관심도가 높은 **Redis**와 **Memcached**를 선택한다.



#### 1.2.2.2 선진 CSP 社 Cache 서비스

다음은 선진 CSP 社에서 서비스하는 Cache 서비스이다.

| CSP 社 |       서비스 명        |          특징           |
| :----: | :--------------------: | :---------------------: |
|  AWS   | ElasticCache Memcached | 오픈소스 Memcached 기반 |
|  AWS   |   ElasticCache Redis   |   오픈소스 Redis 기반   |
|  GCP   | Memorystore Memcached  | 오픈소스 Memcached 기반 |
|  GCP   |   Memorystore Redis    |   오픈소스 Redis 기반   |
| Azure  | Azure Cache for Redis  |   오픈소스 Redis 기반   |



선진 CSP에서 사용하는 대표적인 Opensource SDSCache은 **Redis**와 **Memcached** 이다.



#### 1.2.2.3 비교 대상 Opensource Cache솔루션 선택

비교 대상 Opensource Cache솔루션으로  **Redis**와 **Memached**을 대상으로 선택한다. 

선택된 근거는 Google Trend에서 관심도가 높은 결과와 선진 CSP 社에서 주로 사용되는 오픈소스에 대한 조사결과에 대한 결과이다.



#### 1.2.2.4 Opensource SDSCache 대표 기능 분석

다음은 대표적인 기능에 대한 각 솔루션이 지원하는 여부이다.

|                          기능                          |  Memcached  |    Redis     |
| :----------------------------------------------------: | :---------: | :----------: |
|            DB 부하를 오프로드하는 단순 캐시            |     예      |      예      |
| 쓰기 및 스토리지를 위해 수평적으로 확장할 수 있는 기능 |     예      |      예      |
|                    다중 스레드 유형                    |     예      |    아니요    |
|          고급 데이터 유형[^고급 데이터 유형]           |   아니요    |      예      |
|                      데이터 저장                       | Only Memroy | Memory, Disk |
|             데이터 세트 정렬 및 순위 지정              |   아니요    |      예      |
|                Publish와 Subscribe 기능                |   아니요    |      예      |
|                      Replication                       |   아니요    |      예      |
|          자동 장애조치가 있는 다중 가용 영역           |   아니요    |    아니요    |
|                         지속성                         |   아니요    |    아니요    |

Memcached와 Redis의 **공통점은 In Memory와 Key-Value 방식**이다. SDSCache은 Memcached을 기본으로  Redis의 장점을 취하여 설계한다.

[^고급 데이터 유형]: string, set, sorted set, hashes, list



#### 1.2.2.5 Memcached 분석

##### 1.2.2.5.1 Memcached 개요

-  메모리 객체 캐싱 시스템
- 범용 분산형 메모리 캐시 시스템
- 주요 용도 : 데이터와 객체를 메모리에 캐시하여 DB사용을 줄여 서비스 속도를 빠르게 함
- "애플리케이션 -- memcached(고속) -- DB(저속)" 순으로 배치
- LRU 방식 : 신규 데이터 들어오면 예전 데이터를 사용빈도 낮은 것부터 삭제
-  Key 및 Value 길이 제한
   -  Key : 250 Bytes
   -  Value : 1 MB

-  기본포트 : 11211
-  라이선스 : BSD



##### 1.2.2.5.2 Memcached 명령어

|  명령어   |             설명             |          비고          |
| :-------: | :--------------------------: | :--------------------: |
|    get    |           값 읽기            | <code>get mykey</code> |
|    set    |          키 값 설정          |   신규 키이면 추가됨   |
|    add    |        새로운 키 추가        | 기존 키가 있으면 무효  |
|  replace  |      기존 키에 덮어쓰기      |                        |
|  append   | 기존 키에 데이터를 뒤에 추가 |                        |
|  prepend  | 기존 키에 데이터를 앞에 추가 |                        |
|   incr    |   숫자 값 지정한만큼 증가    |                        |
|   decr    |   숫자 값 지정한만큼 감소    |                        |
|  delete   |           키 삭제            |                        |
| flush_all |         모두 비우기          |                        |
|   stats   |          통계 보기           |                        |
|  version  |       솔루션 버전 출력       |                        |
| verbosity |        로그 레벨 증가        |                        |
|   quit    |        텔넷 세션 종료        |                        |



##### 1.2.2.5.3 Memcached HA(High Availability)

|   HA 항목   | 설명                                                         |
| :---------: | ------------------------------------------------------------ |
| Moxi: Proxy | - Client의 Consistency Hashing 을 통한 분산 지원을 기반<br />- Server 만 Client에 추가하여 분산환경 구성<br />- Moxi의 Proxy 역할로 분산처리 |



##### 1.2.2.5.4 Memcached 데이터 구조

|       항목        |                             설명                             |
| :---------------: | :----------------------------------------------------------: |
|    Hash Table     |                       Cache 항목 조회                        |
|     LRU[^LRU]     | Cache가 가득 찼을 때 Cache 항목 제거(evication) 순서 결정하는 알고리즘 |
| Cache 데이터 구조 |        Key, Data, Flage 및 Point 을 담고 있는 구조체         |
|  Slab Allocator   |               Cache 항목 데이터 메모리 관리자                |

[^LRU]: Least Recently Used



##### 1.2.2.5.5 Memcached Network

|     항목     | 옵션                   | 설명                                                         |
| :----------: | ---------------------- | ------------------------------------------------------------ |
|     TCP      | <code>-p</code>        | - 1.5.6 버전 이후 기본값                                     |
|     UDP      | <code>-U</code>        | - 1.5.6 버전 이후 <code>disabled</code> 기본값               |
| Unix Sockets | <code>-s <file></code> | - 단일 사용자가 Daemon에 엑세스 할 수 있도록 제한 또는 Network를 통해 Daemon을 노출하지 않을 때 사용<br />- 활성화 시 <code>TCP</code>와 <code>UDP</code> 는 <code>disabled</code> |



#### 1.2.2.6 Redis 분석

##### 1.2.2.6.1 Redis 개요

- 오픈소스 인메모리 키-값 저장소
- 데이터베이스, 캐시, 메시지 브로커 등으로 사용됨
- 선택적 영구성 키-값 저장소
- string, set, sorted set, hashes, list 자료구조 지원
- Shard(Master + Slave) 1, 2, 3 ... 확장 구조
- (MongoDB 대비) 읽기/쓰기 빠름
- 라이선스 : BSD
- 기본포트 : 6379
- key내의 구분자로는 콜론(:) 을 쓰는 것이 관례지만 다른 기호를 구분자로 써도 됨



##### 1.2.2.6.2 Redis 명령어

|   명령어    |             설명             |
| :---------: | :--------------------------: |
| CLIENT LIST |     클라이언트 목록 조회     |
|     DEL     |           키 삭제            |
|    DUMP     |         키의 값 덤프         |
|   EXISTS    |         키 존재 여부         |
|  FLUSHALL   |         모두 비우기          |
|     GET     |         키의 값 조회         |
|    INFO     |       레디스 정보 조회       |
|     KEY     | 패턴을 만족하는 키의 값 조회 |
|    KEYS     |        키의 목록 조회        |
|     SET     |         키 - 값 설정         |



##### 1.2.2.6.3 Redis HA(High Availability)

|                     HA 종류                     | 설명                                                         |
| :---------------------------------------------: | ------------------------------------------------------------ |
|         Standalone: <br />No HA, Master         | - Redis 서버 1대로 구성, Master Node<br />- 서버 다운 시 AOF 또는 RDB 파일을 이용해서 재시작 |
|      이중화: <br />Half HA, Master - Slave      | - Redis 서버 2대로 구성, Master - Slave<br />- Slave는 Master 의 데이터를 실시간으로 전달받아 보관<br />- Master 다운 시 Slave 를 이용해서 서비스를 계속 할 수 있지만, 수동으로 Slave를 Master로 변경 필요<br />- 애플리케이션이 새로운 Master 로 접속하도록 변경 필요 |
| 이중화 + Sentinel: <br />HA, 무중단 서비스 가능 | - Master - Slave 구성에 Sentinel 을 추가해서 각 서버를 감시<br />- Sentinel은 Master 서버를 감시하고 있다가 다운되면 Slave를 Master로 승격<br />- Redis Client은 새로운 Master로 접속해서 서비스를 계속<br />- 다운되었던 Master가 다시 시작하면 Sentinel이 Slave로 전환<br />- 일반적으로 Sentinel은 3대로 구성 |
|   Redis Cluster: <br />HA, 무중단 서비스 가능   | - 샤딩: 클러스터는 샤딩(sharding) 방법을 제공하는 것이다. 클러스터 Master가 3대이면 데이터가 3대에 나누어 저장<br />           예를 들어, 100개의 데이터가 있다면 1번 마스터에 33개, 2번 Master에 다른 33개, 3번 마스터에 나머지 34개가 저장되는 방식<br />- Hash 함수: 데이터를 나누는 방식은 키에 hash 함수를 적용해서 값을 추출하고, 이 값을 각 Master 서버에 할당<br />                     예를 들어, 1~100까지 나오는 hash 함수가 있고, 클러스터 Master 서버가 3대이면 1번 서버에 1~33까지, 2번 서버에 34~66까지, 3번 서버에 67~100까지 할당.  <br />                     클러스터 구성 시에 각 Master 서버에 할당<br />- 16384 슬롯(slot): Redis에서 hash 값의 개수는 16384(0~16383)<br />- Redis Client: Client는 서버와 동일한 hash 함수를 가지고 있으며, Master 서버에 접속해서 각 서버에 할당된 슬롯 정보 보유.  키가 입력되면 hash 함수를 적용해서 어느 Master에 저장할지 판단해서 해당 Master에 저장<br />데이터 서버 + Sentinel: 각 Master 서버는 데이터의 처리와 Sentinel의 역할을 같이 수행<br />                                    예를 들어, 1번 마스터 서버가 다운되면 나머지 살아있는 마스터들 중에서 리더를 선출해서 리더가 1번 마스터의 슬레이브를 마스터로 승격<br />- 최소 3대 구성: Master 서버는 최소 3대로 구성하고 각각은 Slave를 가질 수 있음<br />                        Master를 관리하는 Master(master of master)는 없음<br />- 샤딩(Sharding): 대량의 데이터를 처리하기 위해 여러 개의 데이터베이스에 분할하는 기술<br />                           즉, DBMS안에서 데이터를 나누는 것이 아니고 DBMS 밖에서 데이터를 나누는 방식<br />                           레디스 클러스터는 샤딩방식 |



#### 1.2.2.7 Opensource SDSCache 문제점 분석

다음은 Opensource SDSCache Memcached와 Redis 에 대한 문제점 분석이다.

##### 1.2.2.7.1 Memcached의 문제점

|                  항목                   | 설명                                                         | 비고   |
| :-------------------------------------: | ------------------------------------------------------------ | ------ |
| Collection 자료구조에<br /> 대한 미지원 | - 문자열과 숫자에 대한 데이터 타입만 지원                    | 기능성 |
|           Replication 미지원            | - Cache에 대하여 Disk에 보관하지 않고 모든 것을 Memory에서 관리하여 Replication 미지원<br />- 3rd party Library "**repcached**[^repcached]" 을 통해 부분적 지원 | 안전성 |

[^repcached]: http://repcached.lab.klab.org/



##### 1.2.2.7.2 Redis의 문제점

|     항목      | 설명                                                         | 비고 |
| :-----------: | ------------------------------------------------------------ | ---- |
| Signle Thread | - 시간이 오래 걸리는 요청(A)이 들어오면 나머지 요청은 A가 종료될 때까지 대기하여 전체적인 성능을 하락하는 원인이 됨 | 성능 |



### 1.2.3 Opensource Cache Client 분석 

#### 1.2.3.1 Memcached

|    Client    | 지원 언어 | 비동기 | Thread safe | 비고 |
| :----------: | :-------: | :----: | :---------: | :--: |
| spymemcached |   Java    | 미지원 | Thread safe |      |



#### 1.2.3.2 Redis

| Client  | 지원 언어 | 비동기 |  Thread safe  |                 비고                  |
| :-----: | :-------: | :----: | :-----------: | :-----------------------------------: |
|  Jedis  |   Java    | 미지원 | Thread unsafe |                                       |
| Lettuce |   Java    |  지원  |  Thread safe  | Strping Boot 2.0 부터 기본 클라이언트 |
| Hiredis |     C     |  지원  |       -       |            Cluster 미지원             |



### 1.2.4 요구사항과 비교 대상 Opensource Cache솔루션 분석

다음은 Cache솔루션에 대한 요구사항과 Redis, Memcached 에서 지원하는 비교표 이다.

|   분류    |                항목                 |            Redis             |        Memcached         |
| :-------: | :---------------------------------: | :--------------------------: | :----------------------: |
|  비기능   |         3 ms 이하 응답속도          |             만족             |           만족           |
|  비기능   |           3,000 QPS 지원            |             만족             |           만족           |
|  비기능   |          Multi Thread 지원          |             미흡             |           만족           |
|   기능    |     TTL(Time To Live) 기능 지원     |             만족             |           만족           |
|   기능    | Cache 만료정책 알고리즘의 변경 지원 | 만족<br />(LRU, LFU, Random) | 미흡(LRU알고리즘만 지원) |
|   기능    |        Value 길이 10 MB 지원        |             만족             |      미흡(1MB 지원)      |
| 종합 의견 |                                     |             미흡             |           미흡           |

Redis와 Memcached는 요구사항을 만족하지 못한다. 따라서 요구사항을 만족하는 자체개발 Cache를 설계 및 개발, 배포하도록 결정한다.



### 1.2.5 SCP Cache 상품 문제점 분석

#### 1.2.5.1 상품 카테고리 내 Cache 부재

타 CSP와는 달리 상품 카테고리에 Cache 상품이 없다.

![상품01](https://user-images.githubusercontent.com/26420767/185856734-cbc78a5f-97ca-46da-98b0-a9e216681c68.png)



#### 1.2.5.2 Community 버전의 Redis 만 지원

DB Service 상품에 Redis 만 지원한다. 하지만 지원하는 Redis 는 Community 버전으로 이중화 구성 시, Sentinel 을 기반으로 하는 Active-Standby 구조이다. 

![상품02](https://user-images.githubusercontent.com/26420767/185857537-0f15209b-a6dc-4225-8093-f81c1cc22f70.png)



#### 1.2.5.3 SCP의 Cache 상품의 문제점

|            항목             | 설명                                                         |   비고   |
| :-------------------------: | ------------------------------------------------------------ | :------: |
| 상품 내 Cache 카테고리 부재 | - IT에서 성능 향상을 위해 사용되는 Cache에 대한 미지원       | 기본기능 |
|  Cache 상품의 다양성 미흡   | - 타 SCP 대비 Cache 상품에 대한 목록 및 기능, 구성에 대한 지원 미흡 |  다양성  |



참고사이트 : Samsung Cloud Platform 상품 페이지[^SCP]

[^SCP]: https://cloud.samsungsds.com/serviceportal/product.html



## 1.3 고객사 물류시스템의 사용자 요구사항

### 1.3.1 고객사 물류시스템의 응답시간

#### 1.3.1.1 Seow의 응답시간 분류

인터넷 서비스 이용자의 응답시간에 따른 심리를 분석한 Seow(2008)는 응답시간을 다음과 같이 분류하였다.

|   Category    |  Respone Time  | Guidelines for software design                               |
| :-----------: | :------------: | ------------------------------------------------------------ |
| Instantaneous |   0.2 Second   | Response time for input button: 5~100ms<br/> Time for displaying menu: 200ms |
|   Immediate   | 0.5 ~ 1 Second | Time for turning page<br/> Expected time for page up or down |
|  Continuous   |  2 ~ 3 Second  | Need alert information of system periodically<br/> Must give feedback if response time is over 5 second |
|    Captive    | 7 ~ 10 Second  | Users leave the web pages                                    |



#### 1.3.1.2 고객사 물류 시스템 사용자의 기대 응답시간

시스템 사용자 중 최근 6개월 (2022.01 ~ 2022.06) 간 상위 10명(시스템 사용율 80%)을 선정하여 시스템에 대하여 Seow의 응답시간 분

류[^Seow의 응답시간 분류]와 기대 응답시간에 대해 조사하였다.  

다음은 사용자의 Seow의 응답시간 분류와 기대 응답시간에 대해 조사에 대한 결과이다.

|      부서      |  성명  | Seow의 응답시간 분류 | 기대 응답시간 |         비고          |
| :------------: | :----: | :------------------: | :-----------: | :-------------------: |
|  세포주2그룹   | 한규민 |      Immediate       |     1 초      |                       |
|    SCM그룹     | 오소영 |      Continuous      |     2 초      |                       |
|    분석그룹    | 정어진 |    Instantaneous     |    0.2 초     | 최대한 빠른 응답 필요 |
|  세포주1그룹   | 설가인 |      Continuous      |     3 초      |                       |
|  세포주1그룹   | 김지나 |      Immediate       |     1 초      |                       |
|  공정개발그룹  | 김은솔 |      Continuous      |     2 초      |                       |
|    SCM그룹     | 이우주 |      Continuous      |     1 초      |                       |
|    SCM그룹     | 김영현 |      Immediate       |     1 초      |                       |
|    분석그룹    | 김성진 |      Continuous      |     3 초      |                       |
| 바이오분석그룹 | 이보람 |      Immediate       |     1 초      |                       |

조사 결과를 바탕으로 PI 담당자가 정의한 사용자의 기대 응답시간은 **Immediate 분류에 해당하는 1 초** 이다.



[^Seow의 응답시간 분류]: S. C. Seow, Designing and engineering time. Addison-Wesley, Boston, 2008.



### 1.3.2 고객사 물류시스템의 배송상태조회의 실시간 필요에 대한 인터뷰

시스템 사용자 중 최근 6개월 (2022.01 ~ 2022.06) 간 상위 10명(시스템 사용율 80%)을 선정하여 배송상태조회의 실시간 필요성에 대한 인터뷰를 진행하였다. 

|                    부서                    |  성명  | 업무시간 중 <br />배송상태조회 주기 | 배송상태 최신화<br />허용 시간 | 비고                                                         |
| :----------------------------------------: | :----: | :---------------------------------: | :----------------------------: | :----------------------------------------------------------- |
|                세포주2그룹                 | 한규민 |            4시간 당 1회             |             4 시간             |                                                              |
|                  SCM그룹                   | 오소영 |            2시간 당 1회             |             1 시간             |                                                              |
|                  분석그룹                  | 정어진 |            1시간 당 1회             |             1 시간             | - 에러 메시지 팝업 또는 브라우저가 멈추는 현상 <br />- 3번 중 1번 이상, 높은 빈도로 발생 |
|                세포주1그룹                 | 설가인 |            2시간 당 1회             |             2 시간             |                                                              |
|                세포주1그룹                 | 김지나 |            1시간 당 1회             |             1 시간             |                                                              |
|                공정개발그룹                | 김은솔 |          1시간 30분 당 1회          |             1 시간             |                                                              |
|                  SCM그룹                   | 이우주 |            1시간 당 1회             |             1 시간             |                                                              |
|                  SCM그룹                   | 김영현 |             2시간당 1회             |             2 시간             |                                                              |
|                  분석그룹                  | 김성진 |             1시간당 1회             |             1 시간             |                                                              |
|               바이오분석그룹               | 이보람 |             2시간당 1회             |             2 시간             |                                                              |
| 배송상태 최신화에 대한 허용 가능 최소 시간 |        |                                     |             1시간              |                                                              |

시스템 사용자가 배송상태에 대한 허용 가능한 시간은 1시간에서 4시간까지 범위가 다양하다.   

**따라서 가장 짧은 1시간을 배송상태 최신화하는 주기로 선정한다.** 



## 1.4 SDSCache RoadMap 및 프로젝트 범위

### 1.4.1 SDSCache RoadMap

SDSCache은 우선 내제화한다. 내제화의 단계는 크게 Standalone 과 Cluster 단계로 구분한다. Standalone 에서는 Cache 의 기본적인 기능인 읽기, 쓰기, 삭제을 구현한다. Cluster 단계 에서는 여러 개의 노드로 구성하여 가용성 및 안정성을 확보하도록 한다. 가용성 부분에서는 Active-Standby와 Active-Active 에 대하여 단계적으로 확보한다. 그리고 CSP 사업에서 활용하도록 관리형 서비스 단계와 완전 관리형 서비스 단계로 한다.

![image-20220410234147561](https://raw.githubusercontent.com/u4rang/save-image-repo/main/img/image-20220410234147561.png)



### 1.4.2 프로젝트 범위

|        분류        |                        과업                         | 범위 여부 |
| :----------------: | :-------------------------------------------------: | :-------: |
| 고객사 물류 시스템 | 배송 상태 조회 기능 문제 분석 및 대안 아키텍처 설계 |   포함    |
|                    |  배송 상태 조회 기능 대안 아키텍처 적용 및 테스트   |   포함    |
|                    |            대안 아키텍처에 SDSCache 활용            |   포함    |
|      SDSCache      |          Cache 기본 기능 및 아키텍처 설계           |   포함    |
|                    |           Cache Replication 아키텍처 설계           |   포함    |
|                    |           Cache 기본 기능 개발 및 테스트            |   포함    |
|                    |       Cache Client 기본 기능 및 아키텍처 설계       |   포함    |
|                    |        Cache Client 기본 기능 개발 및 테스트        |   포함    |
|                    |        Cache Replication 기능 개발 및 테스트        |  미포함   |



# 2. 요구사항 정의

## 2.1 System Overview

System Context는 다음과 같이 아키텍처 재설계 영역과 설계 및 구현, 테스트 영역으로 나뉜다.
아키텍처 재설계 영역은 고객사 물류 시스템이 대상이다. 설계 및 구현, 테스트 영역은 SDSCache와 접속 및 인터페이스를 제공하는 SDSCacheClient이 대상이다.  

![SystemContext](https://user-images.githubusercontent.com/26420767/190321135-7a41f8c1-78dc-41f8-87d5-4ede668b2ac5.png)

본 프로젝트에서 개발하는 Cache솔루션을 이하 문서에서는 SDSCache로 명칭한다.



### 2.1.1 External Entity

|   구분   | 설명                                                         |
| :------: | ------------------------------------------------------------ |
|  myLogi  | - 상품에 대한 배송 계획 수립 및 배송사에 최신 배송 상태를 조회하여 배송 상태 정보를 제공한다.<br />- 기대사항 : 사용자가 배송 계획 수립 및 배송 상태 확인을 1,000 ms 이하의 응답속도로 제공해야 한다. |
| SDSCache | - 보통의 SDSCache이 제공하는 GET, SET, DEL, TOUCH, TTL 기능을 고객사 물류 시스템에 제공한다. <br />- 기대사항 : SDSCache은 고객사 물류 시스템이 요청하는 기능을 3ms 이내 정확하게 제공해야 한다. |



### 2.1.2 External Interface

|   분류   |                             구분                             | 설명                                                         |
| :------: | :----------------------------------------------------------: | ------------------------------------------------------------ |
| SDSCache |  SET KEY FLAG EXPIRATION_SECOND VALUE_LENGTH/r/n<br />VALUE  | - 역할 : KEY와 VALUE 저장<br />- User Interface : TCP / UDP / Unix Socket<br />- 특성<br />  - 부하 상황에서도 10 ms 이내로 전송한다.<br />  - 기존 Cache 를 대체하는 표준화된 인터페이스를 제공한다. |
| SDSCache |                           GET KEY                            | - 역할 : KEY에 따른 VALUE 조회<br />- User Interface : TCP / UDP / Unix Socket<br />- 특성<br />  - 부하 상황에서도 10 ms 이내로 전송한다.<br />  - 기존 Cache 를 대체하는 표준화된 인터페이스를 제공한다. |
| SDSCache |                           DEL KEY                            | - 역할 : KEY에 따른 VALUE 삭제<br />- User Interface : TCP / UDP / Unix Socket<br />- 특성<br />  - 부하 상황에서도 10 ms 이내로 전송한다.<br />  - 기존 Cache 를 대체하는 표준화된 인터페이스를 제공한다. |
| SDSCache | APPEND KEY FLAG EXPIRATION_SECOND ALUE_LENGTH/r/n<br />VALUE | - 역할 : 기존 KEY의 VALUE 뒤에 새 VALUE 추가<br />- User Interface : TCP / UDP / Unix Socket<br />- 특성<br />  - 부하 상황에서도 10 ms 이내로 전송한다.<br />  - 기존 Cache 를 대체하는 표준화된 인터페이스를 제공한다. |
| SDSCache | PREPEND KEY FLAG EXPIRATION_SECOND ALUE_LENGTH/r/n<br />VALUE | - 역할 : 기존 KEY의 VALUE 앞에 새 VALUE 추가<br />- User Interface : TCP / UDP / Unix Socket<br />- 특성<br />  - 부하 상황에서도 10 ms 이내로 전송한다.<br />  - 기존 Cache 를 대체하는 표준화된 인터페이스를 제공한다. |
| SDSCache |                     INCR KEY [INCR_NUM]                      | - 역할 : 양의 정수인 Value 에 대해 INCR_NUM 을 더하기<br />- User Interface : TCP / UDP / Unix Socket<br />- 특성<br />  - 부하 상황에서도 10 ms 이내로 전송한다.<br />  - 기존 Cache 를 대체하는 표준화된 인터페이스를 제공한다. |
| SDSCache |                     DECR KEY [DECR_NUM]                      | - 역할 : 양의 정수인 Value 에 대해 INCR_NUM 을 빼기<br />- User Interface : TCP / UDP / Unix Socket<br />- 특성<br />  - 부하 상황에서도 10 ms 이내로 전송한다.<br />  - 기존 Cache 를 대체하는 표준화된 인터페이스를 제공한다. |
| SDSCache |                          FLUSH_ALL                           | - 역할 : SDSCache의 유효한 모든 Cache를 삭제한다.<br />- User Interface : TCP / UDP / Unix Socket<br />- 특성<br />  - 부하 상황에서도 10 ms 이내로 전송한다.<br />  - 기존 Cache 를 대체하는 표준화된 인터페이스를 제공한다. |
| SDSCache |                 TOUCH KEY EXPIRATION_SECOND                  | - 역할 : KEY에 TTL(Time To Live)을 설정<br />- User Interface : TCP / UDP / Unix Socket<br />- 특성<br />  - 지정한 시간 이 후 Cache를 자동 삭제한다. |
| SDSCache |                           TTL KEY                            | - 역할 : 남은 TTL을 초 단위로 확인<br />- User Interface : TCP / UDP / Unix Socket<br />- 특성<br />  - Cache의 유효시간을 확인한다. |
|  myLogi  |                   RETRIEVE DELIVERY STATUS                   | - 역할 : 송장번호에 따른 배송상태 조회<br />- User Interface : REST API<br />- 특성<br />  - 외부 배송사의 API 규약에 따라 호출한다.<br />  - 외부 시스템의 환경에 따라 고객사 물류 시스템의 서비스 품질에 영향을 미친다. |



### 2.1.3 System Feature

System Feature에 대해 고객사 물류 시스템과 SDSCache로 나누어 정의한다.



#### 2.1.3.1 고객사 물류 시스템에 대한 System Feature

고객사 물류 시스템은 운영 중인 시스템으로 기존 아키텍처에서 Pain Point를 해결하기 위한 재설계에 대한 System Feature를 정의한다.

|  ID   |                            Title                             |                             설명                             | 중요도 |               Biz. Goal ID                |
| :---: | :----------------------------------------------------------: | :----------------------------------------------------------: | :----: | :---------------------------------------: |
| SF-01 |           일부 조회성 업무에 대한 Cache 기능 활용            | 업무 규칙에 따라 일정 시간동안 SDSCache에 저장된 Cache를<br /> 제공하여 빠른 응답속도를 보장한다. |   상   | BG-01,<br />BG-02,<br />BG-03,<br />BG-04 |
| SF-02 | Batch로 외부 데이터 가져와서 <br />Online에서 저장된 데이터 제공 | 업무 규칙에 따라 일정 시간동안 유효성을 가진 데이터에 대해 <br />Batch로 외부 REST API 호출하여 SDSCache, Database에 저장한다.<br />Online에서는 SDSCache와 Database에 저장된 데이터로 서비스한다. |   상   |             BG-02,<br />BG-04             |



#### 2.1.3.2 SDSCache에 대한 System Feature

|  ID   |                Title                |                             설명                             | 중요도 |               Biz. Goal ID                |
| :---: | :---------------------------------: | :----------------------------------------------------------: | :----: | :---------------------------------------: |
| SF-11 | 대량의 KEY, VALUE 조회 및 저장 기능 | 시스템이 요청하는 대량의 KEY, VALUE 조회 및 저장을 안정적으로 서비스 한다. |   상   |       BG-03,<br />BG-11,<br />BG-13       |
| SF-12 |        정확한 명령 수행 기능        | 시스템이 요청하는 명령들에 대해 유실 없이 100% 서비스 한다.  |   상   |       BG-03,<br />BG-11,<br />BG-13       |
| SF-13 |      표준화된 인터페이스 제공       | 기존 Cache를 효율적으로 대체 할 수 있도록 표준화된 인터페이스를 제공한다. |   상   |       BG-03,<br />BG-11,<br />BG-13       |
| SF-14 |          유연한 확장 기능           |     Cache 부하 상황에 따른 유연한 확장 기능을 제공한다.      |   중   | BG-02,<br />BG-03,<br />BG-12,<br />BG-14 |
| SF-15 |           장애 전파 차단            | Cluster 구성 시, 한 개 Node에 장애가 발생하여도 서비스에 영향이 없다. |   중   | BG-02,<br />BG-03,<br />BG-12,<br />BG-14 |



## 2.2 기능 요구사항

> 과제 수행 기간 내 구현 및 인터페이스 가능한 범위 도출
>
> 전체 범위와 구현 대상 범위를 구분

### 2.2.1 고객사 물류 시스템에 대한 기능 요구사항

![UseCaseDiagram_고객사물류시스템](https://user-images.githubusercontent.com/26420767/190331408-2ed95923-faf2-48c9-a216-3ff239edb30b.png)



다음은 고객사 물류 시스템에 대한 Usecase 이다.

|  ID   |       Title       |                             설명                             | 중요도 | 난이도 | System Feature ID |
| :---: | :---------------: | :----------------------------------------------------------: | :----: | :----: | :---------------: |
| UC-01 | 배송상태조회한다  | 고객사 물류 시스템 사용자 김삼성 프로가 상품의 송장번호로 배송상태를 조회한다. |   상   |   하   | SF-01,<br />SF-02 |
| UC-02 | 배송상태갱신한다. | 고객사 물류 시스템 Batch가 상품의 송장번호로 배송상태를 조회한다. |   상   |   상   | SF-01,<br />SF-02 |
| UC-03 | 배송상태저장한다. |    상품의 배송상태정보를 Database와 SDSCache에 저장한다.     |   상   |   중   | SF-01,<br />SF-02 |



### 2.2.1.1 UC-01 배송상태조회한다

|  UC-01   | 배송상태조회한다                                             |
| :------: | ------------------------------------------------------------ |
|   설명   | 고객사 물류 시스템 사용자 김삼성 프로가 상품의 송장번호로 배송상태를 조회한다. |
|  행위자  | 고객사 물류 시스템 사용자                                    |
| 선행조건 | 상품의 송장번호가 유효하다.                                  |
| 후행조건 | -                                                            |
| 기존동작 | 1. 고객사 물류 시스템 사용자 김삼성 프로가 상품의 배송상태정보를 조회한다.<br />2. 고객사 물류 시스템은 DB에 저장된 상품의 최신의 조회 일시를 가진 배송상태정보를 제공한다. |
| 추가동작 | -                                                            |



### 2.2.1.2 UC-02 배송상태갱신한다

|  UC-02   | 배송상태갱신한다                                             |
| :------: | ------------------------------------------------------------ |
|   설명   | 고객사 물류 시스템 Batch가 상품의 송장번호로 배송상태를 조회한다. |
|  행위자  | 고객사 물류 시스템  Batch                                    |
| 선행조건 | 상품의 송장번호가 유효하다.                                  |
| 후행조건 | -                                                            |
| 기존동작 | 1. 고객사 물류 시스템 Batch가 상품의 송장정보를 조회한다..<br />2. 고객사 물류 시스템 Batch는 상품의 배송사에서 제공하는 API에 송장번호로 배송상태를 조회한다. |
| 추가동작 | -                                                            |



### 2.2.1.3 UC-03 배송상태저장한다

|  UC-03   | 배송상태저장한다                                             |
| :------: | ------------------------------------------------------------ |
|   설명   | 상품의 배송상태정보를 Database와 SDSCache에 저장한다.        |
|  행위자  | 고객사 물류 시스템 Batch                                     |
| 선행조건 | UC-03 이 정상적으로 동작한다.                                |
| 후행조건 | -                                                            |
| 기존동작 | 1. 고객사 물류 시스템 Batch 은 조회한 상품의 배송상태정보와 조회 일시에 대해 Database에 저장한다.<br />2. 고객사 물류 시스템 Batch 은 조회한 상품의 배송상태정보에 대해 SDSCache에 저장한다. |
| 추가동작 | -                                                            |





### 2.2.2 SDSCache에 대한 기능 요구사항

![UseCaseDiagram_Cache](https://user-images.githubusercontent.com/26420767/190337585-3f699401-8548-4138-85d4-61a516b2a220.png)

다음은 Cache의 기본 기능이며 주요 UseCase이다.

- Cache추가한다
- Cache삭제한다
- Cache조회한다



다음은 **"Cache Mode적용한다"**에 대한 상세 UseCase 이다.

![UseCaseDiagram_Cache_CacheMode](https://user-images.githubusercontent.com/26420767/190337834-46a74534-8a94-4c16-a14e-8c9f60ceedfa.png)



다음은  SDSCache에 대한 UseCase 목록이다.

|  ID   |          Title          |                             설명                             | 중요도 | 난이도 |       System Feature ID       |
| :---: | :---------------------: | :----------------------------------------------------------: | :----: | :----: | :---------------------------: |
| UC-11 |      Cache추가한다      | 외부 REST API와 Database 에서 조회한 데이터를 SDSCache에 저장한다 |   상   |   상   | SF-11,<br />SF-12,<br />SF-13 |
| UC-12 |      Cache삭제한다      | SDSCache에 저장된 Cache를 SDSCache에서 더이상 사용하지 않도록 관리하다 |   상   |   상   |       SF-12.<br />SF-13       |
| UC-13 |      Cache조회한다      |              SDSCache에 저장된 Cache를 조회한다              |   상   |   상   | SF-11,<br />SF-12,<br />SF-13 |
| UC-14 |       TTL설정한다       |  Cache에 TTL(Time to Live )을 설정하여 유효시간을 관리한다   |   상   |   상   |       SF-12,<br />SF-13       |
| UC-15 |      Cache제거한다      |              SDSCache에 저장된 Cache를 삭제한다              |   상   |   상   |       SF-12,<br />SF-13       |
| UC-16 |    CacheMode적용한다    |       Local, Partition, Replication 중 Mode를 적용한다       |   중   |   상   |             SF-15             |
| UC-17 |    Cache모니터링한다    |       SDSCache에 대하여 Cache HitRatio를 모니터링한다        |   중   |   상   |             SF-12             |
| UC-18 |  Cache알고리즘적용한다  |    SDSCache의 HitRatio를 높이기 위한 알고리즘을 선택한다.    |   중   |   상   |             SF-13             |
| UC-19 |  Cache알고리즘추가한다  |    SDSCache의 HitRatio를 높이기 위한 알고리즘을 추가한다.    |   중   |   상   |             SF-13             |
| UC-1A |    LocalMode설정한다    | 데이터가 읽기 전용 또는 특정 만료 빈도로 주기적으로 새로 고쳐질 수 있는 시나리오에 적합한 Local Mode를 설정한다. |   중   |   상   |             SF-14             |
| UC-1B |  PartitionMode설정한다  | 대규모 데이터 세트로 작업하고 업데이트가 빈번한 시나리오에 적합한 Partitioned Mode를 설정한다. |   중   |   상   |             SF-14             |
| UC-1C | ReplicationMode설정한다 | 데이터 세트가 작고 업데이트가 자주 발생하지 않는 시나리오에 적합한 Replicated Mode를 설정한다. |   중   |   상   |             SF-14             |
| UC-1D |    Cache노드관리한다    | SDSCache가 Cluster 로 구성된 경우, 노드를 추가하거나 삭제한다. |   중   |   중   |             SF-14             |

Cache의 Cluster Mode에 대한 상세 설명은 별첨 "2. Cache의 Cluster 유형"을 참고한다.



### 2.2.2.1 UC-11 Cache추가한다

|  UC-11   | Cache추가한다                                                |
| :------: | ------------------------------------------------------------ |
|   설명   | 외부 REST API와 Database 에서 조회한 데이터를 SDSCache에 저장한다 |
|  행위자  | 고객사 물류 시스템, CSP 시스템 사용자                        |
| 선행조건 | 조회하는 데이터 A의 Key가 SDSCache에 미존재                  |
| 후행조건 | -                                                            |
| 기존동작 | 1. 고객사 물류 시스템이 Database 또는 API에서 데이터 A를 조회한다<br />2. 고객사 물류 시스템이 받은 데이터 A를 SDSCache에 추가한다 |
| 추가동작 | -                                                            |



### 2.2.2.2 UC-12 Cache삭제한다

|  UC-12   | Cache삭제한다                                                |
| :------: | ------------------------------------------------------------ |
|   설명   | SDSCache에 저장된 Cache를 SDSCache에서 더이상 사용하지 않도록 관리하다 |
|  행위자  | 고객사 물류 시스템, CSP 시스템 사용자                        |
| 선행조건 | 조회하는 데이터 A의 Key가 SDSCache에 존재                    |
| 후행조건 | 데이터 A가 SDSCache에 유효하지 않은 상태 또는 삭제           |
| 기존동작 | 1. 고객사물류시스템 또는 CSP시스템사용자이 SDSCache의 데이터 A를 삭제 요청한다 |
| 추가동작 | TTL 값 변경의 경우, UC-07 "TTL설정한다" 로 이동한다<br />데이터 무효화의 경우, UC-08 "Cache제거한다"로 이동한다 |



### 2.2.2.3 UC-13 Cache조회한다

|  UC-13   | Cache조회한다                                                |
| :------: | ------------------------------------------------------------ |
|   설명   | SDSCache에 저장된 Cache를 조회한다                           |
|  행위자  | 고객사 물류 시스템, CSP 시스템 사용자                        |
| 선행조건 | 조회하는 데이터 A가 SDSCache에 존재                          |
| 후행조건 | -                                                            |
| 기존동작 | 1. 고객사 물류 시스템이 데이터 A를 SDSCache에 조회한다<br />2. SDSCache은 데이터 A가 있으면 고객사 물류 시스템에 제공한다 |
| 추가동작 | -                                                            |



### 2.2.2.4 UC-14 TTL설정한다

|  UC-14   | TTL설정한다                                                  |
| :------: | ------------------------------------------------------------ |
|   설명   | Cache에 TTL(Time to Live )을 설정하여 유효시간을 관리한다    |
|  행위자  | 고객사 물류 시스템, CSP 시스템사용자                         |
| 선행조건 | UC-12 "Cache삭제한다"                                        |
| 후행조건 | -                                                            |
| 기존동작 | 1. 고객사 물류 시스템 또는 CSP시스템사용자는 Cache의 데이터 A에 대해 삭제 지시한다<br />2. SDSCache은 Cache의 데이터 A에 대한 TTL을 변경하여 유효시간을 변경한다 <br />`expected expire time = set expire time, 단위 second` |
| 추가동작 | 1. 데이터 A가 만료되어도 Cache솔루션에서 제거되지 않는다<br />2. 누군가 만료된 데이터 A에 엑세스 하려고 하면 SDSCache은 데이터 A를 검사한다<br />3. 데이터 A가 만료되었는지 확인 후 메모리에서 제거한다 |



### 2.2.2.5 UC-15 Cache제거한다

|  UC-15   | Cache제거한다                                                |
| :------: | ------------------------------------------------------------ |
|   설명   | SDSCache에 저장된 Cache를 삭제한다                           |
|  행위자  | 고객사 물류 시스템, CSP 시스템 사용자                        |
| 선행조건 | UC-12 "Cache삭제한다"                                        |
| 후행조건 | -                                                            |
| 기존동작 | 1. 고객사물류시스템 또는 CSP시스템사용자가 SDSCache 내 Cache의 데이터 A를 무효화를 지시한다<br />2. SDSCache은 데이터 A에 대해 메모리에서 삭제한다 |
| 추가동작 | 1. 데이터 A가 만료되어도 SDSCache에서 제거되지 않는다<br />2. 만료된 데이터 A에 엑세스 하려고 하면 SDSCache은 데이터 A에 대한 유효성 검사한다<br />3. 데이터 A가 만료되었는지 확인 후 메모리에서 제거한다 |



### 2.2.2.6 UC-16 CacheMode적용한다

|  UC-16   | CacheMode적용한다                                            |
| :------: | ------------------------------------------------------------ |
|   설명   | Local, Partition, Replication 중 Mode를 적용한다             |
|  행위자  | 고객사 물류 시스템 개발자, CSP시스템개발자                   |
| 선행조건 | SDSCache솔루션이 Cluster로 구성된다.                         |
| 후행조건 | -                                                            |
| 기존동작 | 1. 고객사 물류 시스템 개발자 또는 CSP 시스템 개발자가 SDSCache의 Cache Mode를 변경 지시한다 |
| 추가동작 | -                                                            |



### 2.2.2.7 UC-17 Cache모니터링한다

|  UC-17   | Cache모니터링한다                                            |
| :------: | ------------------------------------------------------------ |
|   설명   | SDSCache에 대하여 Cache HitRatio를 모니터링한다              |
|  행위자  | 고객사 물류 시스템 운영자, CSP 시스템 운영자                 |
| 선행조건 | -                                                            |
| 후행조건 | -                                                            |
| 기존동작 | 1. 고객사 물류 시스템 운영자 또는 CSP 시스템 운영자가 SDSCache의 Hit Ratio을 요청한다<br />2. SDSCache는 다음 식을 바탕으로 Hit Ratio 정보를 제공한다.<br />`hit ratio = Total number of cache hits / (Total Number of cache hits + Number of cache misses)` |
| 추가동작 | -                                                            |



### 2.2.2.8 UC-18 Cache알고리즘적용한다

|  UC-18   | Cache알고리즘적용한다                                        |
| :------: | ------------------------------------------------------------ |
|   설명   | SDSCache의 HitRatio 를 높이기 위한 알고리즘을 선택한다       |
|  행위자  | 고객사 물류 시스템 운영자, CSP 시스템 운영자                 |
| 선행조건 | Hit Ratio 가 기존보다 좋은 Cache 알고리즘 A 존재             |
| 후행조건 | -                                                            |
| 기존동작 | 1. 고객사 물류 시스템 운영자 또는 CSP 시스템 운영자는 SDSCache에 새로운 Cache 알고리즘 A을 적용 요청한다 |
| 추가동작 | -                                                            |



### 2.2.2.9 UC-19 Cache알고리즘추가한다

|  UC-19   | Cache알고리즘추가한다                                        |
| :------: | ------------------------------------------------------------ |
|   설명   | SDSCache의 HitRatio를 높이기 위한 알고리즘을 추가한다        |
|  행위자  | CSP 시스템 개발자                                            |
| 선행조건 | - UC-18 "Cache알고리즘적용한다"<br />- Hit Ratio 가 기존보다 좋은 Cache 알고리즘 A 존재하며 SDSCache에는 추가되지 않은 상태 |
| 후행조건 | -                                                            |
| 기존동작 | 1. CSP 시스템 개발자는 Cache 알고리즘A 을 템플릿에 맞추어 SDSCache에 추가 지시한다<br />2. SDSCache은 Cache 알고리즘A 을 알고리즘 목록 중 하나로 등록한다 |
| 추가동작 | -                                                            |



### 2.2.2.10 UC-1A LocalMode설정한다

|  UC-1A   | LocalMode설정한다                                            |
| :------: | ------------------------------------------------------------ |
|   설명   | 데이터가 읽기 전용 또는 특정 만료 빈도로 주기적으로 새로 고쳐질 수 있는 시나리오에 적합한 Local Mode를 설정한다 |
|  행위자  | 고객사 물류 시스템 개발자, CSP 시스템 개발자                 |
| 선행조건 | UC-16 CacheMode적용한다                                      |
| 후행조건 | -                                                            |
| 기존동작 | 1. SDSCache은 다른 노드에 캐시 데이터를 배포하지 않는다      |
| 추가동작 | -                                                            |



### 2.2.2.11 UC-1B PartitionMode설정한다

|  UC-1B   | PartitionMode설정한다                                        |
| :------: | ------------------------------------------------------------ |
|   설명   | 대규모 데이터 세트로 작업하고 업데이트가 빈번한 시나리오에 적합한 Partitioned Mode를 설정한다 |
|  행위자  | 고객사 물류 시스템 개발자, CSP 시스템 개발자                 |
| 선행조건 | UC-16 CacheMode적용한다<br />SDSCache는 Cluster 로 구성된다  |
| 후행조건 | -                                                            |
| 기존동작 | 1. 지정된 파티션 개수 만큼 노드에 데이터의 사본을 복사한다   |
| 추가동작 | -                                                            |



### 2.2.2.12 UC-1C ReplicationMode설정한다

|  UC-1C   | ReplicationMode설정한다                                      |
| :------: | ------------------------------------------------------------ |
|   설명   | 데이터 세트가 작고 업데이트가 자주 발생하지 않는 시나리오에 적합한 Replicated Mode를 설정한다 |
|  행위자  | 고객사 물류 시스템 개발자, CSP 시스템 개발자                 |
| 선행조건 | UC-16 CacheMode적용한다<br />SDSCache가 Cluster로 구성된다   |
| 후행조건 | -                                                            |
| 기존동작 | 1. 모든 노드에 데이터의 사본을 복사한다                      |
| 추가동작 | -                                                            |



### 2.2.2.13 UC-1D Cache노드관리한다

|  UC-1D   | Cache노드관리한다                                            |
| :------: | ------------------------------------------------------------ |
|   설명   | SDSCache가 Cluster 로 구성된 경우, 노드를 추가하거나 삭제한다 |
|  행위자  | 고객사 물류 시스템 운영자, CSP 시스템 운영자                 |
| 선행조건 | SDSCache가 Cluster로 구성된다                                |
| 후행조건 | -                                                            |
| 기존동작 | 1. 고객사 물류 시스템 운영자 또는 CSP 시스템 운영자가 SDSCache에 노드 추가 및 삭제를 요청한다<br />2. SDSCache은 노드의 추가 삭제 작업을 수행한다 |
| 추가동작 | -                                                            |



## 2.3 품질 요구사항

> 아키텍처 품질속성별 명확하고 정량적인 달성목표 작성



### 2.3.1 고객사 물류 시스템에 대한 품질 요구사항

고객사 물류시스템에 대한 품질 요구사항은 다음과 같다.

|   ID   | 품질속성 |                  품질속성 상세화                  | 품질속성 시나리오                                            | 중요도 | 난이도 | System Feature ID |
| :----: | :------: | :-----------------------------------------------: | ------------------------------------------------------------ | :----: | :----: | :---------------: |
| QAS-01 |   성능   |           고객사 물류 시스템의 응답속도           | 고객사 물류 시스템 사용자 김삼성 프로가 3,600 초 이내 조회한 운송장번호 A 를 다시 조회하면 <br/>SDSCache에 저장된 데이터를 기반으로 1,000 ms 이내 운송장번호에 대한 배송상태정보를 확인한다 |   상   |   상   |       SF-01       |
| QAS-02 |  안정성  |           고객사 물류 시스템의 응답보장           | 고객사 물류 시스템사용자 김삼성 프로가 Cache에 없는 운송장번호 A의 배송상태를 조회하면, <br/>운송장번호 A의 배송상태는 Cache에 저장되고 김삼성 프로에게 배송상태를 제공한다<br/>1초 후, 이삼성 프로가 운송장번호 A 를 조회하면 SDSCache에 저장된 <br/>운송장번호 A Cache를 이용하여 1,000 ms 이내 배송상태정보를 제공한다 |   상   |   상   | SF-01,<br />SF-02 |
| QAS-03 |  기능성  | 고객사 물류 시스템의 요청 기능에 대한 동작 정확성 | 고객사 물류시스템 사용자 100 명이 동시에 각각의 배송상태정보를 조회 시,<br />같은 운송장번호에 대해서는 SDSCache의 Cache를 기반으로 응답한다 <br/>배송상태조회Batch는 1시간 단위로 배송상태정보를 최신화하여 Database와 SDSCache에 저장한다 |   상   |   중   |       SF-02       |



#### 2.3.1.1 QAS-01 고객사 물류 시스템의 응답속도

|  QAS-01  | 고객사 물류시스템의 응답속도                                 |
| :------: | ------------------------------------------------------------ |
| 품질속성 | 성능                                                         |
|   설명   | 고객사 물류 시스템 사용자 김삼성 프로가 3,600 초 이내 조회한 운송장번호 A 를 다시 조회하면 <br/>SDSCache에 저장된 데이터를 기반으로 1,000 ms 이내 운송장번호에 대한 배송상태정보를 확인한다 |
|   자극   | 고객사 물류 시스템 사용자 김삼성 프로                        |
|   환경   | 고객사 물류 시스템이 SDSCache과 함께 정상적으로 서비스하는 중 |
|   반응   | SDSCache가 Cache를 고객사 물류 시스템에 전송한다.            |
|   측정   | `[요청 처리 속도] = [고객사물류시스템WEB서버의 Access Log에 Response기록한 시각] - [고객사물류시스템WEB서버의 Access Log에 Request기록한 시각]` |



#### 2.3.1.2 QAS-02 고객사 물류 시스템의 응답보장

|  QAS-02  | 고객사 물류시스템의 응답보장                                 |
| :------: | ------------------------------------------------------------ |
| 품질속성 | 신뢰성                                                       |
|   설명   | 고객사 물류 시스템사용자 김삼성 프로가 Cache에 없는 운송장번호 A의 배송상태를 조회하면, <br/>운송장번호 A의 배송상태는 Cache에 저장되고 김삼성 프로에게 배송상태를 제공한다<br/>1초 후, 이삼성 프로가 운송장번호 A 를 조회하면 SDSCache에 저장된 <br/>운송장번호 A Cache를 이용하여 1,000 ms 이내 배송상태정보를 제공한다 |
|   자극   | 고객사 물류 시스템사 용자 김삼성 프로                        |
|   환경   | 고객사 물류 시스템이 SDSCache과 함께 정상적으로 서비스하는 중 |
|   반응   | 고객사 물류 시스템이 조회한 운송장번호A의 배송상태정보를 고객사 물류 시스템 사용자 김삼성 프로에게 전달하고, <br />SDSCache에 Key는 운송장번호A, Value는 배송상태정보로 Cache를 저장하도록 한다. |
|   측정   | `[트래픽 처리 속도] < 1,000 ms`                              |
|   제약   | -                                                            |



#### 2.3.1.3 QAS-03 고객사 물류 시스템의 요청 기능에 대한 동작 정확성

|  QAS-03  | 고객사 물류시스템의 요청 기능에 대한 동작 정확성             |
| :------: | ------------------------------------------------------------ |
| 품질속성 | 정확성                                                       |
|   설명   | 고객사 물류시스템 사용자 100 명이 동시에 각각의 배송상태정보를 조회 시,<br />같은 운송장번호에 대해서는 SDSCache의 Cache를 기반으로 응답한다 <br/>배송상태조회Batch는 1시간 단위로 배송상태정보를 최신화하여 Database와 SDSCache에 저장한다 |
|   자극   | 고객사 물류 시스템 사용자                                    |
|   환경   | 고객사 물류 시스템이 SDSCache과 함께 정상적으로 서비스하는 중 |
|   반응   | 물류시스템은 운송장번호와 배송상태 정보에 대해 1차적으로 SDSCache을 확인하고, 없으면 Database와 API를 조회한다 <br />조회된 데이터는 물류시스템 사용자에게 전달하고, SDSCache에 전달하여 저장하도록 지시한다 |
|   측정   | `[기대한결과응답횟수] / [전체요청회수] * 100`                |
|   제약   | -                                                            |



### 2.3.2 SDSCache에 대한 품질 요구사항

SDSCache에 대한 품질 요구사항은 다음과 같다.

|   ID   |  품질속성  |           품질속성 상세화           | 품질속성 시나리오                                            | 중요도 | 난이도 | System Feature ID |
| :----: | :--------: | :---------------------------------: | ------------------------------------------------------------ | :----: | :----: | :---------------: |
| QAS-11 |    성능    |          SDSCache 응답속도          | 고객사 물류 시스템 사용자 김삼성 프로가 3,600 초 이내 조회한 운송장번호 A 를 다시 조회하면 <br/>SDSCache에 저장된 데이터를 기반으로 1,000 ms 이내 운송장번호 A의 최신 배송상태정보를 확인한다 |   상   |   상   | SF-11,<br />SF-12 |
| QAS-12 |    성능    | SDSCache의 데이터 저장 및 응답 속도 | 고객사 물류 시스템 사용자 김삼성 프로가 SDSCache에 없는 운송장번호 A의 배송상태정보를 조회하면, <br/>Database의 운송장번호 A에 대한 배송상태정보를 조회하여 SDSCache에 저장되고 김삼성 프로에게 배송상태를 제공한다<br/>1초 후, 이삼성 프로가 운송장번호 A 를 조회하면 SDSCache에 저장된 <br/>운송장번호 A의 배송상태정보를 이용하여 1,000 ms 이내 배송상태를 제공한다 |   상   |   상   |       SF-12       |
| QAS-13 |   내구성   |         SDSCache의 응답보장         | 1,000[^1,000명] 명의 고객사 물류 시스템 사용자가 동시에 운송장번호와 배송상태를 조회하면 동일한 운송장번호와 <br/>그 배송상태는 SDSCache에 저장된 데이터를 기반으로 응답하며 <br/>고객사 물류 시스템 사용자는 1,000 명의 요청에 따른 고객사 물류 시스템의 부하에 대해 인지하지 못한다 |   상   |   중   | SF-11,<br />SF-14 |
| QAS-14 | 유지보수성 |  SDSCache의 알고리즘  변경 용이성   | Cache 알고리즘에 대한 정책이 변경되면<br />SDSCache는 정책 변경 즉시, 새로운 알고리즘 정책으로 Cache 를 관리한다 |   상   |   중   | SF-12,<br />SF-13 |
| QAS-15 |   정확성   |      SDSCache의 데이터 정확성       | SDSCache은 같은 Key에 대해서 같은 Value을 유효 TTL 이내 100 % 제공한다 |   중   |   상   |       SF-12       |
| QAS-16 |    보안    |       SDSCache의 구간 암호화        | 고객사물류시스템과 SDSCache 간의 통신은 TLS 기반으로 암호화하여 통신한다 |   중   |   중   | SF-12,<br />SF-13 |
| QAS-17 | 유지보수성 |     SDSCache의 유지보수 용이성      | SDSCache에 새로운 Cache 알고리즘(LRU)을 추가하면 <br/>2개월 이내 고급개발자 4MM로 개발한다 |   중   |   중   |       SF-13       |
| QAS-18 |   안정성   |          SDSCache의 정확성          | SDSCache은 유효한 Key A 인 Cache를 고객사 물류 시스템에게 100 % 제공한다 |   중   |   중   | SF-11,<br />SF-12 |
| QAS-19 |   확장성   |          SDSCache의 가용성          | Infra가 확보되어 있는 환경에서 SDSCache 담당자가 Cache 노드를 증설하는 경우 시스템 중단 없이 60분 이내에 처리한다 |   중   |   중   |       SF-14       |
| QAS-1A |   내구성   |    노드 장애 시 Cache 유실 방지     | SDSCache이 여러 개의 노드로 구성하고, 임의의 한 노드에서 장애가 발생하더라도 Cache는 유실되지 않는다 |   중   |   중   |       SF-15       |

응답속도 1,000 ms 에 대한 기준은 "별첨 3. 고객사 물류시스템의 응답시간 선정" 에 따른 결과이다.

[^1,000명]: 2022년 08월, 고객사의 재직 중 임직원 수는 920명이다. 



#### 2.3.2.1 QAS-11 SDSCache의 응답속도

|  QAS-11  | SDSCache의 응답속도                                          |
| :------: | ------------------------------------------------------------ |
| 품질속성 | 성능                                                         |
|   설명   | 고객사 물류 시스템 사용자 김삼성 프로가 3,600 초 이내 조회한 운송장번호 A 를 다시 조회하면 <br/>SDSCache에 저장된 데이터를 기반으로 1,000 ms 이내 운송장번호 A의 최신 배송상태정보를 확인한다 |
|   자극   | 고객사 물류 시스템 사용자 김삼성 프로                        |
|   환경   | 고객사 물류 시스템이 SDSCache와 함께 정상적으로 서비스하는 중 |
|   반응   | SDSCache이 Cached 데이터를 고객사 물류 시스템에 전송한다     |
|   측정   | `[요청 처리 속도] = [고객사물류시스템WEB서버의 Access Log에 Response기록한 시각] - [고객사물류시스템WEB서버의 Access Log에 Request기록한 시각]` |
|   제약   | `[트래픽 처리 속도] < 1,000 ms`                              |



#### 2.3.2.2 QAS-12 SDSCache의 데이터 저장 및 응답 속도

|  QAS-12  | SDSCache의 데이터 저장 및 응답 속도                          |
| :------: | ------------------------------------------------------------ |
| 품질속성 | 성능                                                         |
|   설명   | 고객사 물류 시스템 사용자 김삼성 프로가 SDSCache에 없는 운송장번호 A의 배송상태정보를 조회하면, <br/>Database의 운송장번호 A에 대한 배송상태정보를 조회하여 SDSCache에 저장되고 김삼성 프로에게 배송상태를 제공한다<br/>1초 후, 이삼성 프로가 운송장번호 A 를 조회하면 SDSCache에 저장된 <br/>운송장번호 A의 배송상태정보를 이용하여 1,000 ms 이내 배송상태를 제공한다 |
|   자극   | 고객사 물류 시스템 사용자 김삼성 프로                        |
|   환경   | 고객사 물류 시스템이 SDSCache과 함께 정상적으로 서비스하는 중 |
|   반응   | 고객사 물류 시스템이 조회한 운송장번호A의 배송상태정보를 고객사 물류 시스템 사용자 김삼성 프로에게 전달하고, <br />SDSCache에 Key는 운송장번호A, Value는 배송상태정보를 저장하도록 한다 |
|   측정   | [트래픽 처리 속도] < 1,000 ms                                |
|   제약   | 운송장번호 A가 등록되고 최초 Batch가 실행되기 전까지 최대 1시간 배송상태정보가 없다. |



#### 2.3.2.3 QAS-13 SDSCache의 응답보장

|  QAS-13  | SDSCache의 응답보장                                          |
| :------: | ------------------------------------------------------------ |
| 품질속성 | 내구성                                                       |
|   설명   | 1,000 명의 고객사 물류 시스템 사용자가 동시에 운송장번호와 배송상태를 조회하면 동일한 운송장번호와 <br/>그 배송상태는 SDSCache에 저장된 데이터를 기반으로 응답하며 <br/>고객사 물류 시스템 사용자는 1,000 명의 요청에 따른 고객사 물류 시스템의 부하에 대해 인지하지 못한다 |
|   자극   | 1,000 명의 고객사 물류 시스템 사용자                         |
|   환경   | 고객사 물류 시스템이 SDSCache과 함께 정상적으로 서비스하는 중 |
|   반응   | 고객사 물류 시스템은 운송장번호 A와 배송상태정보에 대해 1차적으로 SDSCache에 확인하고, 없으면 Database를 조회한다 <br />조회된 데이터는 고객사 물류 시스템 사용자에게 전달하고, SDSCache에 전달하여 저장하도록 지시한다 |
|   측정   | `[요청 처리 속도] = [고객사물류시스템WEB서버의 AccessLog에 Response기록한 시각] - [물류시스템WEB서버의 Access Log에 Request기록한 시각]`<br />`hit ratio = Total number of cache hits / (Total Number of cache hits + Number of cache misses` |
|   제약   | -                                                            |



#### 2.3.2.4 QAS-14 SDSCache의 알고리즘 변경 용이

|  QAS-14  | SDSCache의 알고리즘 변경 용이                                |
| :------: | ------------------------------------------------------------ |
| 품질속성 | 유지보수성                                                   |
|   설명   | Cache 알고리즘에 대한 정책이 변경되면<br />SDSCache은 정책 변경 즉시, 새로운 알고리즘 정책으로 Cache 를 관리한다. |
|   자극   | 업무의 특성에 따라 적용된 알고리즘 A보다 Hit Ratio 가 좋은 Cache 알고리즘 B 존재 |
|   환경   | 고객사 물류 시스템이 SDSCache과 함께 정상적으로 서비스하는 중 |
|   반응   | SDSCache은 Cache 알고리즘 B로 변경한다. 변경 즉시, 변경된 알고리즘 B로 동작하다 |
|   측정   | Cache 알고리즘이 변경되면 변경된 알고리즘 B으로 동작한다.    |



#### 2.3.2.5 QAS-15 SDSCache의 데이터 정확성

|  QAS-15  | SDSCache의 데이터 정확성                                     |
| :------: | ------------------------------------------------------------ |
| 품질속성 | 정확성                                                       |
|   설명   | SDSCache은 같은 Key 에 대해서는 같은 값을 유효 TTL 이내 100 % 제공한다. |
|   자극   | 고객사 물류 시스템 사용자 김삼성 프로                        |
|   환경   | 고객사 물류 시스템이 SDSCache과 함께 정상적으로 서비스하는 중 |
|   반응   | 같은 Key 을 가진 데이터에 대해 일관된 결과를 제공한다.       |
|   측정   | `hit ratio = Total number of cache hits / (Total Number of cache hits + Number of cache misses)` |



#### 2.3.2.6 QAS-16 SDSCache의 구간 암호화

|  QAS-16  | SDSCache의 구간 암호화                                       |
| :------: | ------------------------------------------------------------ |
| 품질속성 | 보안                                                         |
|   설명   | 고객사 물류 시스템과 SDSCache 간의 통신은 TLS1.3 기반으로 암호화하여 통신한다. |
|   자극   | 고객사 정보보호 부서의 통신구간 암호화 권장                  |
|   환경   | 고객사 물류 시스템이 SDSCache과 함께 정상적으로 서비스하는 중 |
|   반응   | TLS 기반으로 고객사 물류 시스템과 SDSCache은 네트워크 통신한다. |
|   측정   | TLS1.3 기반 통신구간 암호화                                  |



#### 2.3.2.7 QAS-17 SDSCache의 유지보수 용이성

|  QAS-17  | SDSCache의 유지보수 용이성                                   |
| :------: | ------------------------------------------------------------ |
| 품질속성 | 유지보수성                                                   |
|   설명   | SDSCache에 새로운 Cache 알고리즘(LRU)을 추가하면 <br/>2개월 이내 고급개발자 4MM 로 개발한다. |
|   자극   | 새로운 Cache 알고리즘(LRU) 추가                              |
|   환경   | 고객사 물류 시스템이 SDSCache과 함께 정상적으로 서비스하는 중 |
|   반응   | SDSCache은 Cache 알고리즘(LRU) 를 추가한다. 추가 시, 他 알고리즘에 영향을 미치지 않는다. |
|   측정   | 새로운 Cache 알고리즘 추가 후 기존 Cache 알고리즘 동작 이상 없음 확인 |



#### 2.3.2.8 QAS-18 SDSCache의 정확성

|  QAS-18  | SDSCache의 정확성                                            |
| :------: | ------------------------------------------------------------ |
| 품질속성 | 안정성                                                       |
|   설명   | SDSCache은 유효한 Key A 인 Cache를 고객사 물류 시스템에게 100 % 제공한다. |
|   자극   | 고객사 물류 시스템 사용자 김삼성 프로                        |
|   환경   | 고객사 물류 시스템이 SDSCache과 함께 정상적으로 서비스하는 중 |
|   반응   | SDSCache에 저장된 Key A에 대해 유효 TTL 동안 Cache를 제공한다. |
|   측정   | `hit ratio = Total number of cache hits / (Total Number of cache hits + Number of cache misses)` |



#### 2.3.2.9 QAS-19 SDSCache의 가용성

|  QAS-19  | SDSCache의 가용성                                            |
| :------: | ------------------------------------------------------------ |
| 품질속성 | 가용성                                                       |
|   설명   | Infra가 확보되어 있는 환경에서 SDSCache 담당자가 Cache 노드를 증설하는 경우 시스템 중단 없이 60분 이내에 처리한다 |
|   자극   | SDSCache의 노드 증설 시도                                    |
|   대상   | SDSCache 클러스터                                            |
|   환경   | 고객사 물류 시스템이 SDSCache과 함께 정상적으로 서비스하는 중 |
|   반응   | 신규 Cache 노드가 SDSCache 클러스터에 포함                   |
|   측정   | 노드 추가 결정 시점부터 60분 이내 신규 Cache 노드가 추가되어 서비스한다. |



#### 2.3.2.10 QAS-1A 노드 장애 시 Cache 유실 방지

|  QAS-1A  | 노드 장애 시 Cache 유실 방지                                 |
| :------: | ------------------------------------------------------------ |
| 품질속성 | 신뢰성                                                       |
|   설명   | SDSCache이 여러 개의 노드로 구성하고, 임의의 한 노드에서 장애가 발생하더라도 Cache는 유실되지 않는다 |
|   자극   | SDSCache 클러스터 중 임의의 한 개 노드에 장애                |
|   대상   | SDSCache 클러스터                                            |
|   환경   | 고객사 물류 시스템이 SDSCache과 함께 정상적으로 서비스하는 중 |
|   반응   | 임의의 한 개 노드에 장애가 발생, 나머지 SDSCache 클러스터에서 정상적으로 서비스 |
|   측정   | 장애 발생 전 후, 유효 Cache 의 개수와 내용이  100% 일치      |



## 2.4 아키텍처 제약사항

> 품질속성에 영향을 미치는 아키텍처 제약 사항 기술

아키텍처 제약사항은 다음 2 가지이다.

|  ID   | 설명                                                         |
| :---: | ------------------------------------------------------------ |
| CR-01 | OS는 Linux 중 Ubuntu LTS 20.04 버전으로 한정한다.            |
| CR-02 | SDSCache에 대한 개발 언어는 Golang으로 구현한다.             |
| CR-03 | SDSCache Client에 대한 개발 언어는 Java으로 구현한다.<br />고객사 물류 시스템이 Java 언어 기반이다. |

CR-02 에서 개발 언어 Golang이 선택된 내용은 "별첨. 4. SDSCache 개발 언어" 를 참고한다.

# 3. 아키텍처 설계문제 분석

## 3.1 아키텍처 드라이버

> 설계 핵심요구사항 선정, 선정사유 설명





### 3.1.1 고객사 물류 시스템에 대한 아키텍처 드라이버

|  ID   |               품질속성<br />상세화               | 품질 요구사항                                                | 중요도 | 난이도 |     QAS<br />ID     |
| :---: | :----------------------------------------------: | ------------------------------------------------------------ | :----: | :----: | :-----------------: |
| AD-01 |           고객사 물류시스템의 응답속도           | 고객사 물류 시스템 사용자 김삼성 프로가 3,600 초 이내 조회한 운송장번호 A 를 다시 조회하면 <br />SDSCache에 저장된 Cache를 기반으로 1,000 ms 이내 운송장번호 A에 대한 배송상태정보를 확인한다 |   상   |   상   |       QAS-01        |
| AD-02 |           고객사 물류시스템의 응답보장           | 고객사 물류시스템사용자 김삼성 프로가 Cache에 없는 운송장번호 A의 배송상태정보를 조회하면, <br/>운송장번호 A의 배송상태는 Cache에 저장되고 김삼성 프로에게 배송상태를 제공한다.<br/>3초 후, 이삼성 프로가 운송장번호 A 를 조회하면 Cache에 저장된 <br/>운송장번호 A의 배송상태를 이용하여 1,000 ms 이내 배송상태를 제공한다 |   상   |   상   |       QAS-02        |
| AD-03 | AD-03. 고객사 물류 시스템의 Database 안정성 확보 | 고객사 물류시스템 사용자 100 명이 동시에 각각의 배송상태정보를 조회 시,<br />같은 운송장번호에 대해서는 SDSCache의 Cache를 기반으로 응답한다 <br/>배송상태조회Batch는 1시간 단위로 배송상태정보를 최신화하여 Database와 SDSCache에 저장한다 |   상   |   상   | QAS-02,<br />QAS-03 |



### 3.1.2 SDSCache에 대한 아키텍처 드라이버

|  ID   |      품질속성<br />상세화       | 품질 요구사항                                                | 중요도 | 난이도 |     QAS<br />ID     |
| :---: | :-----------------------------: | ------------------------------------------------------------ | :----: | :----: | :-----------------: |
| AD-11 |       SDSCache의 응답속도       | SDSCache은 50 Byte 크기 Key와 1,000 Byte 크기 Value 의 Cache에 대해 <br />평균 3 ms 이내 응답한다 |   상   |   상   | QAS-11,<br />QAS-12 |
| AD-12 |       SDSCache의 응답보장       | SDSCache은  50 Byte 크기 Key와 1,000 Byte 크기 Value 의 Cache 에 대해 <br />1시간 동안 1,000 TPS 에 대한 응답을 보장한다. |   상   |   중   |       QAS-13        |
| AD-13 | SDSCache의 알고리즘 변경 용이성 | SDSCache는 운영 중 Cache를 유지하는 기간을 관리하는 알고리즘이 변경되면<br />정책 변경 즉시, 새로운 알고리즘 정책으로 Cache 를 관리한다 |   상   |   중   |       QAS-14        |
| AD-14 |  노드 장애 시 Cache 유실 방지   | SDSCache이 여러 개의 노드로 구성하고, 임의의 한 노드에서 <br />장애가 발생하더라도 Cache는 유실되지 않는다 |   상   |   중   | QAS-19<br />QAS-1A  |
| AD-15 |    SDSCache의 데이터 정확성     | 사용자가 SDSCache에 Key로 "empid_26751, Value로 "김삼성프로" <br />Cache의 SET을 요청하여 성공으로 응답 받은 경우, <br />사용자가 SDSCache에 Key "empid_26751" 인 Cache의 GET을 요청하는 경우,<br /> Value "김삼성프로" 로  100% 동일하게 응답한다 |   중   |   상   | QAS-15,<br />QAS-17 |
| AD-16 |     SDSCache의 구간 암호화      | SDSCache과 CacheClient 간의 통신 중 TCP 기반으로 인 경우 <br />TLS 1.3 규격 기반으로 암호화하여 통신한다 |   중   |   중   |       QAS-16        |
| AD-17 |   SDSCache의 유지보수 용이성    | SDSCache에 새로운 Cache 알고리즘(LRU, Least Recent Used)이 추가하면 <br />2개월 이내 Golang 언어에 대한 경력 5년 이상의 개발자로 4MM로 개발한다 |   중   |   중   |       QAS-17        |
| AD-18 |        SDSCache의 가용성        | Infra가 확보되어 있는 환경에서 SDSCache 담당자가 Cache 노드를 <br />증설하는 경우 시스템 중단 없이 60분 이내에 처리한다 |   중   |   중   |       QAS-18        |



## 3.2 아키텍처 문제분석 및 설계전략

> 아키텍처 문제분석과 그에 대응할 설계전략 수립
>
> 아키텍처 스타일 및 패턴 활용 계획
>
> 뷰/절차 활용 계획

### 3.2.1 아키텍처 설계 문제 분석표

#### 3.2.1.1 고객사 물류 시스템에 대한 설계 문제 분석표

|               설계문제               |                아키텍처 드라이버                 |                      아키텍처 설계 전략                      | 이유<br />(이득/비용/위험요인 고려사항 포함)                 |
| :----------------------------------: | :----------------------------------------------: | :----------------------------------------------------------: | ------------------------------------------------------------ |
|  ISSUE01. 고객사 물류 시스템의 성능  |       AD-01. 고객사 물류 시스템의 응답속도       | AS-01. 근 실시간(Near Realtime)으로 배송상태조회에 대한 업무 프로세스 변경 | 사용자의 업무 분석으로 근 실시간, 1시간 이내 배송상태조회가 필요하다. |
|                                      |                                                  | AS-02. 조회가 많은 업무 특성을 고려한 Cache솔루션과 Data Store의 배치 | Cache솔루션과 Data Store에 대한 효율적인 배치가 필요하다.    |
| ISSUE02. 고객사 물류 시스템의 안정성 |       AD-02. 고객사 물류 시스템의 응답보장       | AS-03. 트래킹이 필요한 배송상태에 대하여 Cache솔루션에서 보유하여 제공 | 1,000ms 이내 배송상태정보 요청에 대하여 응답해야 한다.       |
|                                      | AD-03. 고객사 물류 시스템의 Database 안정성 확보 | AS-04. 배송상태정보에 관한 조회 업무는 Online 시스템과 변경 업무는 Batch 시스템으로 각각 분리 | 동일 Raw에 대하여 여러 Transaction 에서 수정이 발생하여 DB 경합 이벤트가 발생하는 원인을 제거한다. |



#### 3.2.1.2 SDSCache에 대한 설계 문제 분석표

|                설계문제                |           아키텍처 드라이버           |                      아키텍처 설계 전략                      | 이유<br />(이득/비용/위험요인 고려사항 포함)                 |
| :------------------------------------: | :-----------------------------------: | :----------------------------------------------------------: | ------------------------------------------------------------ |
|        ISSUE11. SDSCache의 성능        |      AD-11. SDSCache의 응답속도       |              AS-11. Cache 를 보관하는 자료 구조              | Cache에서 저장하는 Value는 문자열 뿐만 아니라, List, Set, Sorted Set 이 필요하다. |
|       ISSUE12. SDSCache의 정확성       |      AD-12. SDSCache의 응답보장       |              AS-12. 요청에 대한 응답 코드 정의               | 응답에 대하여 명확한 정의를 하여 요청에 대한 결과를 정확하게 전달해야 한다. 따라서 표준화가 필요하다. |
|                                        |    AD-15.SDSCache의 데이터 정확성     | AS-13. Cache를 조회 할 때, Key의 데이터 타입은 문자열, 비교 방법은 동등성(Equality) | 요청의 명확성을 위해 Unique 한 값을 갖는 Key에 대한 정의가 필요하다. |
|                                        |                                       |                  AS-14. Cache 의 만료 정책                   | 기간에 대한 특징을 가진 Cache에 대하여 유효기간에 설정 및 방법에 대한 정의가 필요하다. |
|       ISSUE13. SDSCache의 신뢰성       |  AD-14. 노드 장애 시 Cache 유실 방지  |             AS-15. Node에 대한 Replication 구성              | 한 개의 노드에 장애가 발생하여도 지속적인 서비스 제공이 필요하다. |
|                                        |     AD-16. SDSCache의 구간 암호화     |             AS-16. TCP 통신 시, TLS 기반 암호화              | 정보보호부서의 네트워크 보안 요청이 있으면,<br />Cache솔루션과 Cache Client 간 네트워크 구간에 대한 암호화가 필요하다. |
| ISSUE14. Cache솔루션의 노드 확장용이성 |       AD-17. SDSCache의 가용성        |             AS-17. SDSCache에 대한 클러스터 구성             | Cache의 분산 및 가용성, 성능을 만족하기 위해 클러스터 구성이 필요하다. |
|                                        |                                       |               AS-18. Proxy를 활용한 Cache 분산               | Cache를 분산하여 저장하여 Cache 클러스터를 효율적으로 사용하도록 고려해야 한다. |
| ISSUE15. Cache솔루션의 기능 변경용이성 | AD-13. SDSCache의  알고리즘 변경 용이 |   AS-19. Strategy Pattern을 이용한 알고리즘 캡슐화 및 교환   | 알고리즘을 환경에 따라 영향도 없이 바꾸는 기능이 필요하다.   |
|                                        |   AD-16. SDSCache의 유지보수 용이성   |     AS-1A. TDD 방법론으로 추가 및 변경에 대한 실패 방지      | 요구사항 변경에 따른 Cache솔루션의 변경에 대한 지지대가 필요하다. |



다음은 위 설계 문제 분석표에 따른 아키텍처 설계 전략 이다. 

아키텍처 설계 전략은 설계 문제, 아키텍처 드라이버, 설계 문제를 해결하는 설계 근거 그리고 위험요인, 고려사항 순으로 필요에 따라 구성 한다. 



### 3.2.2 아키텍처 설계 전략

#### 3.2.2.1 고객사 물류 시스템에 대한 아키텍처 설계 전략

##### 3.2.2.1.1 AS-01. 근 실시간(Near Realtime)으로 배송상태조회에 대한 업무 프로세스 변경

- 설계 문제
  - ISSUE01. 고객사 물류시스템의 성능
- 아키텍처 드라이버
  - AD-01. 고객사 물류시스템의 응답속도
- 설계 문제를 해결하는 설계 근거

"1.3.2. 고객사 물류시스템의 배송상태조회의 실시간 필요에 대한 인터뷰" 에서 나온 배송시간 최신화에 대한 허용가능 최대 시간 1시간을 바탕으로 업무 프로세스를 근 실시간(Near Realtime)으로 변경한다.

1시간 간격으로 배송상태를 업데이트 하는 방법으로는 배송상태를 조회하여 갱신하는 Batch 프로그램으로 배송상태를 조회하여 DB와 Cache솔루션에 저장한다. 사용자는 1차적으로 Cache에 저장된 배송상태를 조회하고 Cache에 배송상태가 없으면 DB에 저장된 배송상태를 조회한다. 

개선된 업무 프로세스에 대한 Sequence Diagram은 다음과 같다.

![SEQ_TOBE_배송상태조회](https://user-images.githubusercontent.com/26420767/190858807-573154d1-4dac-4107-a081-d81dddfafc8c.png)

- 위험 요인
  - Batch 프로그램에 대한 모니터링이 필요하다.

- 고려 사항
  - 새 송장번호가 추가되면 최초 1회는 즉시 Batch 프로그램에 의해 Nuxt서비스에 조회 후 배송상태정보 최신화를 고려한다.




##### 3.2.2.1.2 AS-02. 조회가 많은 업무 특성을 고려한 Cache솔루션과 Data Store의 배치

- 설계 문제
  - ISSUE01. 고객사 물류시스템의 성능
- 아키텍처 드라이버
  - AD-01. 고객사 물류시스템의 응답속도
- 설계 문제를 해결하는 설계 근거

사용자가 배송상태를 조회할 때 Cache솔루션과 Data Store의 배치와 배치 프로그램이 외부 API를 조회하여 배송상태를 Cache솔루션과 Data Store에 갱신하는 것에 대하여 전략을 각각 가져간다. 

Cache솔루션과 Data Store의 배치전략은 다음과 같이 읽기 2 개, 쓰기 3 개가 있다.

a. [읽기] Cache Aside 패턴

![_Cache_배치전략_CacheAside패턴 drawio](https://user-images.githubusercontent.com/26420767/186612163-c3d1bad5-71c2-4172-9970-d4c4408e655f.png)

b. [읽기] Read Through 패턴

![_Cache_배치전략_ReadThrough패턴 drawio](https://user-images.githubusercontent.com/26420767/186613693-8ef4dd94-c120-43b6-8823-fb4d758465ca.png)

c. [쓰기] Write Selective 패턴

![_Cache_배치전략__Write_Selective패턴 drawio](https://user-images.githubusercontent.com/26420767/186616567-488413ac-3013-479b-be49-c40ad649de99.png)

d. [쓰기] Write Back 패턴

![_Cache_배치전략_WriteBack패턴의 복사본 drawio](https://user-images.githubusercontent.com/26420767/186612975-1cbed571-1864-4be2-9171-f16180d7bd3f.png)

e. [쓰기] Write Through 패턴

![_Cache_배치전략__WriteThrough패턴 drawio](https://user-images.githubusercontent.com/26420767/186615045-6a4e397e-8968-4d5f-b16a-12c6079ea4fb.png)

위 패턴을 비교표로 정리하면 다음과 같다.

읽기를 위한 패턴 비교표

|   항목   | Cache Aside 패턴                                             | Read Through 패턴                  |
| :------: | ------------------------------------------------------------ | ---------------------------------- |
|   특징   | - Cache 장애 대비 구성<br />- 정합성 문제 발생 가능<br />- 반복적인 조회에 적합 | - Cache에 저장하는 주체가 DB<br /> |
| 선택여부 | 선택                                                         | 미선택                             |



쓰기를 위한 패턴 비교표

|   항목   | Write Selective 패턴                                         | Write Back 패턴                                              | Write Through 패턴                                           |
| :------: | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
|   툭징   | - DB에 저장 후 선택적으로 Cache 저장<br />- Cache에서 사용되는 데이터만 저장되어 리소스 절약<br />- Cache에 저장되는 데이터는 사용되는 데이터로 Hit Ratio 상승 | - Cache가 Queue의 역할<br />- Cache 장애 시 데이터 유실<br />- 정합성 확보<br />- 불필요한 리소스 저장 | - Cache에 저장 후 Cache가 DB에 반영<br />- 항상 동기화 되는 장점<br />- Cache에 사용되지 않는 데이터까지 저장되어 리소스 낭비와 Write 작업 부하 발생<br />- TTL을 사용하여 사용되지 않는 데이터를 삭제 |
| 선택여부 | 선택                                                         | 미선택                                                       | 미선택                                                       |

고객사 물류시스템의 배송상태에 대한 Cache와 Data Store의 배치 전략은 읽기를 위한 패턴은 "Cache Aside 패턴"이 쓰기를 위한 패턴은 "Write Selective 패턴" 을 선택한다.



##### 3.2.2.1.3 AS-03. 트래킹이 필요한 배송상태에 대하여 SDSCache에서 제공 

- 설계 문제
  - ISSUE02. 고객사 물류시스템의 성능
- 아키텍처 드라이버
  - AD-02. 고객사 물류시스템의 응답보장
- 설계 문제를 해결하는 설계 근거

다음은 개선된 배송상태조회에 대한 Sequenece Diagram 이다.

![SEQ_TOBE_배송상태조회_2](https://user-images.githubusercontent.com/26420767/190858822-4715050a-ecda-4340-b795-13213989582f.png)

|         구분         | 설명                                                         |
| :------------------: | ------------------------------------------------------------ |
|        Batch         | 1시간 간격으로 배송상태를 업데이트 하는 방법으로는 배송상태를 조회하여 갱신하는 Batch 프로그램으로 <br />배송상태를 조회하여 Database와 SDSCache에 저장한다 |
|       Online 1       | 고객사 물류시스템에서는 송장에 대한 배송상태를 1차로 SDSCache에서 조회한다 |
| Online 2<br />(선택) | SDSCache에 배송상태정보가 없을 경우 DB에서 배송상태정보를 조회하고 SDSCache에 배송상태정보를 저장하여 <br />동일한 송장번호에 대해 SDSCache를 사용하도록 한다. |

- 위험 요인
  - 송장번호에 대한 배송상태정보가 Cache솔루션과 DB에 없는 경우, 사용자에게 배송상태정보를 제공 불가하다.
- 고려 사항
  - 새 송장번호가 추가되면 최초 1회는 즉시 배송상태를 조회하는 것에 대하여 고려한다.



##### 3.2.2.1.4 AS-04. 배송상태정보에 관한 조회 업무는 Online 시스템과 변경 업무는 Batch 시스템으로 각각 분리

- 설계 문제
  - ISSUE02. 고객사 물류 시스템의 안정성
- 아키텍처 드라이버
  - AD-03. AD-03. 고객사 물류 시스템의 Database 안정성 확보
- 설계 문제를 해결하는 설계 근거

"1.2.1.4 고객사 물류 시스템의 성능 이슈 분석" 에서 문제점 2가지 중 "문2. 고객사 물류 시스템 운영담당자는 ARMS 로부터 **DML Row Lock** 에 대한 Warning 메시지를 1일 5회 이상 전달 받는다." 의 DML Row Lock 의 원인이다. 

![Cache_dml_row_lcok_설명 drawio](https://user-images.githubusercontent.com/26420767/190860228-bea5ebd0-dfd9-45ac-a9ff-80c2cfc5cd7f.png)

| 단계 | 설명                                                         | 비고                                    |
| ---- | ------------------------------------------------------------ | --------------------------------------- |
| 1    | Transaction 1. 에서 A Row 에 대한 배송상태정보 변경을 위해 row lock을 획득한다 | Tranaction 1. row lock 획득             |
| 2    | Transaction 2. 에서도 A Row 에 대한 배송상태정보 변경을 위해 row lock을 획득을 시도하지만<br />이미 Transaction 1. 에서 row lock 을 소유하고 있기 때문에 Transaction 2.에서는 row lock waiting 상태가 발생한다. | DML Row Lock에 대한 Warning 메시지 발생 |
| 3    | Transaction 1, 2 외 다수의 Transaction에서 A Row에 대한 배송상태정보 변경을 위해 row lock waiting 상태가 발생한다. |                                         |



"1.3.2 고객사 물류시스템의 배송상태조회의 실시간 필요에 의한 인터뷰"를 기반으로 다수의 Transaction에서 변경 작업을 하지 않고, 한 개의 Transaction에서 일정 주기로 변경을 하고 나머지 Transaction 에서는 조회 작업을 하도록 한다.

|      구분       | 설명                                                         | 비고                |
| :-------------: | ------------------------------------------------------------ | ------------------- |
| Batch 프로그램  | Nuxt 사에서 배송상태정보를 조회하여 Database 및 SDSCache에 저장한다. | 한 개의 Transaction |
| Online 프로그램 | Database 및 SDSCache에서 배송상태정보를 조회한다.            | 다수의 Transaction  |



#### 3.2.2.2 SDSCache에 대한 아키텍처 설계 전략

##### 3.2.2.2.1 AS-11. Cache 를 보관하는 자료 구조

- 설계 문제
  - ISSUE11. Cache솔루션의 성능
- 아키텍처 드라이버
  - AD-11. SDSCache의 응답속도
- 설계 문제를 해결하는 설계 근거

Cache를 저장하는 자료 구조에 대하여,  Big-O 표기법으로 Array, Slice, List, Map의 추가, 삭제, 읽기 동작에 성능 비교 자료 이다.

|   동작   |    Array, Slice     |        List         |        Map        |
| :------: | :-----------------: | :-----------------: | :---------------: |
|   추가   |        O(N)         |        O(1)         |       O(1)        |
|   삭제   |        O(N)         |        O(1)         |       O(1)        |
|   읽기   | O(1) - Index의 접근 | O(N) - Index로 접근 | O(N) - Key로 접근 |
| 선택여부 |       미선택        |       미선택        |       선택        |

Map은  Key와 Value의 쌍으로만 동작하는 특징을 갖는다.  그리고 Cache 는 Map과 마찬가지로 Key 와 Value 로 구성되는 특징을 갖는다. 동일한 Key를 가진 Cache를 추가하는 경우, Map 자료구조의 특징에 의해 최근에 추가되는 Cache가 남게 된다. 따라서 동일한 Key를 갖는 Cache를 처리하는 기능을 Map 자료구조의 특징으로 만족 시킨다.

Array, Slice 그리고 List에 비해  Map은 상대적으로 Memory를 적게 소모한다. Array, Slice, List 는 구성요소의 선 후 자료구조에 대해 연결하기 위하여 메모리 주소를 갖는 변수가 별도로 필요하다.

따라서 Cache를 저장하는 자료구조는 Map을 선택한다.

- 위험 요인
  - Map은 Index를 사용해서 접근할 수 없고 입력한 순서가 보장되지 않는다. 

- 고려 사항
  - 입력한 순서에 대한 유효TTL 처리를 위한 별도 처리가 필요하다.
  - 읽기 동작 시, Key 가 존재하지 않는 경우에 대하여 응답에 대한 정의가 필요하다.




##### 3.2.2.2.2 AS-12. 요청에 대한 응답 코드 정의

- 설계 문제
  - ISSUE11. Cache솔루션의 성능
- 아키텍처 드라이버
  - AD-12. SDSCache의 응답보장
- 설계 문제를 해결하는 설계 근거

Cache Client의 요청에 대하여 정확한 커뮤니케이션을 위해 응답에 결과 이 외 응답코드를 함께 전달한다. 

응답코드를 사용하면서 얻는 장점은 다음과 같다. 

- 응답코드로 인해 응답이 성공인지 실패인지 또한, 구체적으로 어떤 성공인지 어떤 실패인지를 자세하게 알 수 있다.
- Cache Client에서 응답 코드에 따른 처리 로직을 구현 할 수 있다.

- 고려 사항
  - 타 Cache, Memcached와 Redis의 응답코드를 참고한다.

다음은 Memcached와 Redis에서 사용하는 응답코드이다.

[TODO. 여기 작성할 것]



##### 3.2.2.2.3 AS-13. Cache를 조회 할 때, Key의 데이터 타입은 문자열, 비교 방법은 동등성(Equality)

- 설계 문제
  - ISSUE02. Cache솔루션의 정확성
  - ISSUE12. 고객사 물류시스템의 정확성
- 아키텍처 드라이버
  - AD-04.SDSCache의 데이터 정확성
  - AD-12.고객사 물류시스템의 요청 기능에 대한 동작 정확성
- 설계 문제를 해결하는 설계 근거

SDSCache에 저장된 데이터를 조회하기 위해서는 Key 기반으로 조회한다. 

사용자가 기대하는 데이터를 조회하기 위해서는 Key에 대해서 명확하게 구별하는 특징을 갖아야 한다. 

 Key의 데이터 타입을 위한 비교표는 다음과 같다.

|   항목   | string (문자열)                                              | int (숫자형)                                                 | rune                                                     | byte           |
| :------: | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------------- | -------------- |
|   특징   | - 문자열을 저장하기 위한 자료형<br />- 정수 및 문자 등 다양한 표현<br />- 동등성(Equality) 연산자 지원 | - 정수를 저장하기 위한 자료형<br />- 32비트 시스템에서는 int32, 64비트 시스템에서는 int64 | - 유니코드 저장을 위한 자료형<br />- 크기는 int32와 동일 | - 8비트 자료형 |
|   예시   | "Hello World", "123", "A"                                    | 123                                                          | 0xAC00                                                   | 'A'            |
| 선택여부 | 선택                                                         | 미선택                                                       | 미선택                                                   | 미선택         |



데이터 타입에 따른 출력은 다음과 같다.

```go
var str1 = "Hello World"
var str2 = "123"
var str3 = "A"

var x = 123
var y int64 = 123

var korGa rune = '가'
var korGa2 rune = 0xAC00
var korGa3 rune = 44032

var engA byte = 'A'
var engA2 byte = 65

fmt.Println(str1, str2, str3)
// 출력: Hello World 123 A

fmt.Printf("%s %s %s", str1, str2, str3)
// 출력: Hello World 123 A

fmt.Println(x, y)
// 출력: 123 123

fmt.Printf("%d %d", x, y)
// 출력: 123 123

fmt.Println(korGa, korGa2, korGa3, engA, engA2)
// 출력: 44032 44032 44032 65 65

fmt.Printf("%c %c %c %c %c", korGa, korGa2, korGa3, engA, engA2)
// 출력: 가 가 가 A A
```



Cache솔루션에 저장된 데이터를 정확하게 조회하기 위해 Key의 데이터 타입은 "string(문자열)" 을 선택한다.

- 고려 사항
  - Key는 영문자(대소문자)와 숫자로만 구성하여 사용하도록 한다. 
    한글 그리고 중국어는 Character Type에 따라 영향을 받는다.



##### 3.2.2.2.4 AS-14. Cache의 만료 정책

- 설계 문제
  - ISSUE12. Cache솔루션의 정확성
- 아키텍처 드라이버
  - AD-15.SDSCache의 데이터 정확성
- 설계 문제를 해결하는 설계 근거

Cache솔루션은 Database 또는 Object Storage(이하 DB) 를 대신하여 제한된 메모리에 자주 사용되는 데이터를 두고 Client로부터의 요청에 빠른 응답을 위한 목적으로 사용된다. Cache솔루션에서 보관하는 데이터는 임시적인 성격을 갖는다. 영구적으로 보관되는 Database 또는 Object Storage에서 데이터 A가 변경되면 Cache솔루션에서 보관하는 데이터 A의 Cache는 즉시 가치를 잃게 된다.

Cache의 유효성에 대해 다음과 같은 방법으로 보장한다.

1. DB에서 데이터 A가 변경되면 Cache솔루션에 변경된 데이터 A로 다시 SET 한다.
2. Cache솔루션에서 Cache를 보관하는 자료구조에 TTL(Time To Live) 변수를 두고 Cache에 대한 만료 정책을 사용한다. TTL 변수의 값이 0으로 되면 해당 Cache는 더이상 유효하지 않다.

- 위험 요인
  - Cache Stampede 현상이 발생할 가능성이 있다. TTL이 0으로 되면 이 Cache를 보고 있는 많은 애플리케이션들이 동시에 Cache를 갱신하기 위하여 DB로 요청을 한다. 따라서 DB에 Deplicate Read 현상과 읽어온 값을 쓰기 위해 Cache솔루션에 Duplicate Write 현상이 발생한다. 
    전체 시스템의 처리량이 느려질 뿐만 아니라 불필요한 요청의 폭주로 장애 발생 가능성이 있다.

**Cache Stampede 현상**

![Cache_CacheStampede현상 drawio](https://user-images.githubusercontent.com/26420767/186102727-babe2800-349c-44df-8123-55b7084f67b6.png)

- 고려 사항
  - Cache Stampede 현상을 해결하기 위해 PER(Probablistic Early Recomputation) 알고리즘 도입을 고려한다. PER 알고리즘은 Cache의 TTL이 실제로 만료되기 전에 일정 확률로 Cache를 갱신하는 방법이다. DB에서 Key가 완전히 만료되기 전에 데이터를 먼저 읽어오게 함으로써 Cache Stampede 현상을 막는다.


**Probablistic Early Recomputation 알고리즘**

```go
def fetch_aot(key, expiry_gap_ms):
    ttl_ms = cache.pttl(key)	// pttl은 millisecond 단위
    
    if tttl_mx - (random() * expiry_gap_ms) > 0:
        return cache.get(key)
        
    return nil
    
// Usage
fetch_aot('foo', 20000)
```



##### 3.2.2.2.5 AS-16. TCP 통신 시, TLS 기반 암호화 

- 설계 문제
  - ISSUE13. Cache솔루션의 신뢰성
- 아키텍처 드라이버
  - AD-16. SDSCache의 구간 암호화
- 설계 문제를 해결하는 설계 근거

Unix Socket 통신을 제외한 TCP, UDP 통신은 네트워크를 기반으로 한다. 
네트워크를 통한 커뮤니케이션은 다음과 같은 위험이 있다.

|                   항목                   |                             내용                             |         비고         |
| :--------------------------------------: | :----------------------------------------------------------: | :------------------: |
|                   도청                   | 통신 중 패킷이 부정한 방법으로 복사되어서 개인 정보를 도둑 맞는 것 |        암호화        |
|                내용 변경                 | 통신 중 패킷을 도둑 맞아서 정보가 부정한 방법으로 변경되는 것 |  암호화 및 전자서명  |
|               부정 액세스                |          다른 사람의 컴퓨터에 허가 없이 침입하는 것          |    인증 및 방화벽    |
| Dos 공격<br />(Denial of Service Attack) | 서버 등에 모두 처리 할 수 없는 양의 패킷을 보내서 기능을 마비시키는 것 |    방화벽 및 WAF     |
|          컴퓨터 바이러스의 침입          |   컴퓨터에 위해를 가하는 목적으로 만들어진 프로그램의 침입   | WAF 및 백신 프로그램 |

 

위험으로부터 보호하는 기술은 다음과 같다. 

|   항목   | 내용                                                         |     비고      |
| :------: | :----------------------------------------------------------- | :-----------: |
|  암호화  | - 데이터를 어떤 규칙을 근거로 가공하고, 제 3자가 쉽게 읽을 수 없도록 하는 것 | TLS, SSL, SSH |
| 전자서명 | - 데이터가 변경되지 않았는지를 판단하는 장치<br />- 데이터를 특수한 방법으로 수치화하고, 이것을 암호화한 것<br />  1. 송신 측) 데이터를 특수한 방법으로 수치화<br />  2. 송신 측) 산출한 수치1을 암호화 <br />  3. 수신 측) 암호화된 수치를 복호화<br />  4. 수신측) 송신측과 같은 방법으로 데이터를 수치화하고, 1에서 얻은 수치와 동일하면 데이터가 변경되지 않았다고 판단 |               |
|   인증   | - 정보 A가 A 사용자에게 유일하게 속하는 사실을 확인하고 이를 증명하는 행위 |               |
|  방화벽  | - 패킷을 제어하는 기능을 갖고 있는 소프트웨어나 하드웨어     |               |

위험으로부터 보호하는 기술 중 Cache솔루션이 제공함으로써 신뢰성을 높이는 기술, 통신구간에 대한 암호화를 선택한다.

통신구간에 대한 암호화는 다음과 같은 방법이 있다.

|                항목                 | 내용                                                       | 비고               | 선택여부 |
| :---------------------------------: | ---------------------------------------------------------- | ------------------ | :------: |
| VPN<br />(Virtual Private Network)  | - 공중망의 회선을 사설망처럼 이용하는 가상 사설망          | SSL VPN, IPSec VPN |  미선택  |
|               전용선                | - 송수신 시스템 간에 송수신 시스템만을 위한 개별 전송 회선 |                    |  미선택  |
| TLS<br />(Transport Layer Security) | - 모든 종류의 네트워크 트래픽에 대한 암호화 지원           | TLS 1.3 최신 버전  |   선택   |

통신구간에 대한 암호화 방법 중 SDSCache의 기능을 통해 제공 가능한 TLS 를 선택한다.

TLS에 대한 상세자료는 별첨 6. TLS를 참고한다. 

- 고려 사항
  - 정보보호부서의 통신구간 암호화 요청을 확인하고,TLS기반 통신구간 암호화 적용한다.




##### 3.2.2.2.6 AS-15, AS-17, AS-18

- 설계 문제
  - ISSUE13. SDSCache의 신뢰성
  - ISSUE14. SDSCache의 노드 확장용이성
- 아키텍처 드라이버
  - AD-14. 노드 장애 시 Cache 유실 방지
  - AD-18. SDSCache의 가용성
- 설계 전략
  - AS-15. Node에 대한 Replication 구성
  - AS-17. SDSCache에 대한 클러스터 구성
  - AS-18. Proxy를 활용한 Cache 분산
  
- 사전 조건
  - SDSCache은 클러스터로 구성된다. 

- 설계 문제를 해결하는 설계 근거

클러스터는 다음 목적에 의해 시스템 구성에 사용된다.

| 항목                | 내용                                                         | 비고       |
| ------------------- | ------------------------------------------------------------ | ---------- |
| 고가용성<br />(HA)  | - 시스템의 가용성을 높이기 위한 방법 중 하나이다. <br />- 하나의 Node에 장애가 발생하면 다른 노드가 서비스를 이어받아(FailOver) 계속해서 서비스 되도록 한다. | 장애극복   |
| 고계산용<br />(HPC) | - 고성능의 계산 능력을 제공하기 위하여 사용된다. <br />- HPC 클러스터를 구성하는 모든 Node는 네트워크에 연결되어 상호간에 통신이 가능하므로 다수의 프로세서가 협동적으로 문제를 풀 수 있는 환경을 제공한다. | 과학계산용 |
| 부하분산<br />(LVS) | - 대규모 서비스를 제공하기 위한 목적으로 사용되며 주로 웹서비스 등에 활용가치가 높다.<br />- 동일한 서비스를 제공하는 여러 개의 Node를 네트워크에 연결하여 Load Balancer(L4, L7) 으로 분산하여 서비스 한다. | 웹서비스   |

"AD-14. 노드 장애 시 Cache 유실 방지"를 위한 설계 전략으로 고가용성(HA) 클러스터 - "AS-15. Node에 대한 Replication 구성"을 선택한다.

Replication은 Master(Active)-Slave(Standby)로 구성한다.
Master Node에 장애(Fault)가 발생하면 Slave Node가 Master로 승격하고 Active 모드로 변경한다.   

![Cache_Replication drawio](https://user-images.githubusercontent.com/26420767/186146149-1d09aec4-fbf8-4218-9c5f-ca03c8cd1b93.png)

Master Node의 장애를 탐지하기 위해 Slave Node는 Master Node로 Heartbeat(Master와 Slave 간 주기적인 메시지 교환)을 주기적으로 요청한다.

장애 탐지를 위한 항목은 다음과 같다.

|        항목         | 내용                                                         | 비고 |
| :-----------------: | :----------------------------------------------------------- | :--: |
| Heartbeat 요청 주기 | - 일정시간 간격으로 대상 Node에 Heartbeat를 요청하여 상태를 확인한다. |      |
| Hearbeat 실패 횟수  | - Heartbeat가 실패하여도 수용 가능한 횟수이다.<br />- 실패 횟수를 초과하면 대상 Node에 장애가 발생한 것으로 판단한다. |      |

**장애 발생 최대 시간**은 다음과 같은 수식으로 표현한다.
$$
장애발생최대시간 = (Heartbeat요청주기 * Heartbeat실패횟수) + 장애극복시간
$$



"AD-18. SDSCache의 가용성"를 위한 설계전략으로 부하분산(LVS) 클러스터 - "AS-08. SDSCache의 가용성 구성"와 "AS-09. Proxy를 활용한 Cache 분산"을 선택한다.

| 항목                             | 내용                                                         | 비고 |
| -------------------------------- | ------------------------------------------------------------ | ---- |
| AS-17. SDSCache의 가용성 구성    | - Cache클러스터를 구성하는 Node를 2개 이상 배치하고 각 Node는 서로 다른 Cache를 가지고 서비스한다. |      |
| AS-18. Proxy를 활용한 Cache 분산 | - Proxy는 Cache의 Key를 이용한 Function으로 각 Node가 서비스하는 Cache에 대해 라우팅 서비스를 제공한다. <br />- 함수는 Cache의 Key를 Hash화 하여 언제나 동일한 값을 반환하도록 설계한다. |      |

![Cache_LVS_Step1 drawio](https://user-images.githubusercontent.com/26420767/186281839-353a24d9-8e5a-4f07-8174-e9ff3f07f3c3.png)



Proxy가 SPOF(Single Point Of Failure) 이므로 이중화를 통해 가용성을 확보한다. 그리고 VIP(Virtual IP)를 Client에 제공하여 한 노드의 Proxy에 장애가 발생하여도 VIP가 Take Over 하여 여분의 Proxy로 지속적인 서비스를 제공하도록 한다. Proxy 서버에서 분산을 위해 사용하는 Function은 공유하여 Proxy #1 또는 Proxy #2 에서 Key A를 Function에 Parameter로 입력하면 동일하게 Node #2로 Cache를 요청하도록 한다. 

![Cache_LVS_Step2 drawio](https://user-images.githubusercontent.com/26420767/186283166-52fd81ec-9a4c-4a04-a20e-f6df06fd7992.png)



위 다이어그램에 "AS-07. Node에 대한 Replication 구성" 을 적용하면 다음과 같은 아키텍처를 갖는다.

![Cache_LVS_Step3 drawio](https://user-images.githubusercontent.com/26420767/186284116-4efc6116-b4a1-4ed8-b4c8-7adbd33d566c.png)

 

- 고려 사항
  - Proxy 에서 Cache클러스터에 속한 모든 Node의 정보를 관리 한다.
  - Proxy 에서 Node의 장애 극복에 대한 코디네이션한다.
    - 장애 극복에 대한 처리와 Client 요청에 대한 분산 처리에 대한 컨트럴타워 역할을 수행한다.



##### 3.2.2.2.7 AS-19. Strategy Pattern을 이용한 알고리즘 캡슐화 및 교환

- 설계 문제
  - ISSUE15. SDSCache의 기능 변경용이성
- 아키텍처 드라이버
  - AD-13. SDSCache의  알고리즘 변경 용이
- 설계 문제를 해결하는 설계 근거

SDSCache에서 업무 특성에 따라 Cache의 만료정책을 관리하는 알고리즘을 변경한다. 변경이 되는 알고리즘은 Cache의 만료정책에 대해서만 영향을 미칠 뿐 Cache솔루션의 다른 영역에 영향을 미치지 않는다. 즉, 알고리즘을 정의하고 각각을 캡슐화하여 교환하여 사용하도록 설계한다. 
위와 같은 요구사항을 만족하는 디자인 패턴은 Strategy Pattern이다. 

|         항목          | 내용                                                         | 비고 |
| :-------------------: | ------------------------------------------------------------ | ---- |
| Strategy Pattern 정의 | 바뀌는 부분을 찾아내고 바뀌지 않는 부분은 분리 시킨다. <br />바뀌는 부분은 따로 뽑아서 캡슐화 시킨다.<br />1. 알고리즘군을 정의한다.<br />2. 각각을 캡슐화 한다.<br />3. 교환하여 사용 가능하도록 구성한다. |      |
| Strategy Pattern 장점 | - 독립성, Client가 사용하는 알고리즘을 변경 없이 새로운 알고리즘으로 교체한다. |      |



다음은 Stragtegy Pattern에 대한 Class Diagram 이다.

![Cache_StrategyPattern drawio](https://user-images.githubusercontent.com/26420767/186308967-afee0d95-b8b3-4bb2-8e8a-c712be022191.png)



Golang언어 기반 Cache 만료정책 알고리즘에 대해 Strategy Pattern을 적용하면 다음과 같다.

![Cache_SDSCache_StrategyPattern drawio](https://user-images.githubusercontent.com/26420767/186326617-e8549cb8-3c41-4018-8168-3f8861969048.png)



EvictionAlgorithm.go 

```go
package goStrategyPattern

type EvictionAlgorithm interface {
	evict(c *SDSCache)
}

```

Fifo.go

```go
package goStrategyPattern

import (
	"fmt"
)

type Fifo struct {
}

func (l *Fifo) evict(c *SDSCache) {
	fmt.Println("Evicting by FIFO algorithm")
}

```

Lru.go

```go
package goStrategyPattern

import (
	"fmt"
)

type Lru struct {
}

func (l *Lru) evict(c *SDSCache) {
	fmt.Println("Evicting by LRU algorithm : ")
}

```

Lfu.go

```go
package goStrategyPattern

import (
	"fmt"
)

type Lfu struct {
}

func (l *Lfu) evict(c *SDSCache) {
	fmt.Println("Evicting by LFU algorithm")
}

```

SDSCache.go

```go
package goStrategyPattern

type SDSCache struct {
	storage           map[string]string
	evictionAlgorithm EvictionAlgorithm
	capacity          int
	maxCapacity       int
}

func InitCache(e EvictionAlgorithm) *SDSCache {
	storage := make(map[string]string)
	return &SDSCache{
		storage:           storage,
		evictionAlgorithm: e,
		capacity:          0,
		maxCapacity:       2,
	}
}

func (c *SDSCache) SetEvictionAlgo(e EvictionAlgorithm) {
	c.evictionAlgorithm = e
}

func (c *SDSCache) Set(key, value string) {
	if c.capacity == c.maxCapacity {
		c.Evict()
	}
	c.capacity++
	c.storage[key] = value
}

func (c *SDSCache) Get(key string) {
	delete(c.storage, key)
}

func (c *SDSCache) Evict() {
	c.evictionAlgorithm.evict(c)
	c.capacity--
}

```

- 고려 사항
  - SDSCache에서 제공할 알고리즘이 1개 또는 2개 인경우는 분기문으로 처리하는 것이 간편하다. 따라서 알고리즘의 갯수에 따라 적용여부를 판단한다.




##### 3.2.2.2.8 AS-1A. TDD 방법론으로 추가 및 변경에 대한 실패 방지

- 설계 문제
  - ISSUE15. Cache솔루션의 기능 변경용이성
- 아키텍처 드라이버
  - AD-17. SDSCache의 유지보수 용이성
- 설계 문제를 해결하는 설계 근거

EPC/바이오 IT팀 CI 담당자들이 뽑은 시스템을 유지보수하는데 어려움 중 다음 우선순위가 높은 다음 2가지 이다.

- 요구사항 변경에 따라 Code의 추가 및 변경에 따른 기존 Code에 대한 영향 및 실패에 대한 어려움
- Document 현행화에 대한 어려움

EPC/바이오 IT팀에서 담당하는 많은 시스템들이 대부분 다음과 같은 프로세스로 구축되고 유지보수 되고 있다.

![Cache_기존프로세스 drawio](https://user-images.githubusercontent.com/26420767/186337605-e33b1a9d-7db5-421c-a171-455f044dafe9.png)

"요구사항 분석" 단계에서 시작하여 단계별로 산출물(Document 및 코드) 에 대한 현행화가 같이 진행된다. 하지만 단계를 진행하다가 이전 단계로 다시 이동해야 하는 경우도 발생하는 경우가 발생하여 유지보수 담당자들은 어려움을 느끼고 있다. 또한 테스트를 진행하면서 기존 Code에 대한 기능에 대한 영향 없음을 매번 확인하고 영향이 있는 경우에도 이전 단계로 이동하게 되는 어려움이 발생한다.

다음은 TDD 방법론 프로세스이다. 이전 단계로 돌아가는 경우는 기존 프로세스 비교하여 현저히 적다.

![Cache_TDD프로세스 drawio](https://user-images.githubusercontent.com/26420767/186341885-1d7aad79-1303-4756-92e3-a8e4e7f471b2.png)

다음은 TDD 방법론의 장점과 단점이다.

| 장점                                                         | 단점                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| - 작업과 동시에 테스트를 진행함으로써 실시간으로 오류 사항을 파악하여 시스템 결함을 방지 | - 기존의 개발프로세스에 테스트케이스 설계까지 추가되어 코드 생산 비용이 높아짐 |
| - 짧은 개발 주기를 통해 고객의 요구사항을 빠르게 수용하거나 피드백을 줄 수 있고 현재 진행 사항을 쉽게 파악 | - 어떻게 테스트를 할 것이며, 프로젝트 성격에 따른 테스트 프레임워크 선택 등 여러 부분에 대한 고려 필요 |
| - 자동화 도구를 이용해 TDD의 테스트 케이스를 단위 테스트로 사용 가능<br />Junit(Java), CppUnit(C/C++), Golang(자체지원) |                                                              |
| - Documentation 작성을 최소화하고 간단 명료하게 코드를 작성하여 시스템을 파악 |                                                              |

- 위험 요인
  - 고객사 물류시스템은 TDD 방법론으로 개발되지 않아 기존 코드에 대한 테스트 코드가 작성되어 있지 않다.

- 고려 사항
  - 당사는 ITSM으로 시스템을 관리 중이다. ITSM에서는 다음 단계로 진행하기 위해서는 산출물 등 근거자료가 필수이다. 따라서 고객사 물류시스템의 경우는 TDD 방법론을 적용하기는 어렵다. 따라서 ITSM에 등록되지 않은 시스템인 SDSCache에 대해서만 TDD방법론을 적용한다. 


따라서, SDSCache은 TDD 방법론으로 진행하여 구축 완료 후 유지보수 기간 담당자가 변경되어도 변경에 대한 실패방지를 지원한다.



## 3.3 Software Stack

### 3.3.1 고객사 물류 시스템의 Software Stack

| 항목         | Software   | Version | 비고       |
| ------------ | ---------- | ------- | ---------- |
| Java         | OpenJDK    | 1.8     | Opensource |
| Framework    | SpringBoot | 2.5     | Opensource |
| Framework    | Lego       |         | S사 솔루션 |
| UI Framework | Vue.js     | 2.X     | Opensource |
| UI           | UI Dev     |         | S사 솔루션 |
| Database     | PostgreSQL | 10      | Opensource |



### 3.3.2 SDSCache의 Software Stack

| 항목       | Software                     | Version | 비고                                     |
| ---------- | ---------------------------- | ------- | ---------------------------------------- |
| Golang     | Golang                       | 1.18    |                                          |
| 라이브러리 | github.com/benbjohnson/clock | v1.3.0  | TTL 계산을 위한 Clock 라이브러리         |
| 라이브러리 | github.com/rs/zerolog        | v1.27.0 | JSON 형식 Output 지원하는 Log 라이브러리 |



### 3.3.3 SDSCache Client의 Software Stack

| 항목       | Software | Version | 비고                                 |
| ---------- | -------- | ------- | ------------------------------------ |
| Java       | OpenJDK  | 1.8     | Opensource                           |
| 라이브러리 | Maven    | v1.3.0  | Build Tool                           |
| 라이브러리 | junit    | 4.11    | 단위 테스트를 위한 테스트 라이브러리 |



# 4. 아키텍처 상세설계

## 4.1 개념 아키텍처

> 전체 아키텍처를 조망할 수 있는 개념도, 뷰 설명
>
> 큰 모듈 및 기능 간의 크기, 위치, 관계가 한 눈에 파악이 되도록 작성
>
> 설계전략이 어디에 적용되는지 이해하기 쉽게 설명

시스템은 기능 요구사항 및 품질 요구사항에 대해 만족하도록 설계 되었다. "4.1 개념 아키텍처" 에서는 전체 아키텍처를 조망하는 아키텍처 개념도, 시스템 구성도, 설계 전략 및 적용 범위 그리고 구현 및 검증 대상에 대해 작성하였다.



### 4.1.1 아키텍처 개념도

시스템의 아키텍처 개념도는 다음과 같다. 
고객사 물류 시스템은 배송상태 업데이트를 위한 배치 컴포넌트를 별도로 분리한다. SDSCache는 별도의 서버에 실행하도록 구성한다.

![Cache_아키텍처개념도 drawio](https://user-images.githubusercontent.com/26420767/191025653-4e48e194-eb68-4bb0-8aad-e15081164b9c.png)



### 4.1.2 시스템 구성도

시스템 구성도는 다음과 같이 Layered Architecture Style로 나타낸다. Client Layer 에서 발생하는 사용자의 요청을 처리하기 위해 Web Layer가 있다. Web Layer는 On-premis에서는 다른 Service Layer나 Cache Layer와 서브넷은 다르지만 동일 네트워크에 놓을 수 있다. 하지만 Cloud에서는  Client Layer와 가깝게 위치하는 CDN에도 놓울 수 있다. 
다음은 Service Layer이다. 사용자의 요청에 대하여 실제로 처리하는 영역이다. AS-IS와 다른 점은 두 가지 이다. 
첫 번째 Batch Layer가 추가되어 Service Layer 내 Delivery Module 역할을 분담한다. 즉, 외부 서비스인 Nuxt 사의 API를 호출하여 배송상태에 대한 갱신하는 역할이다. 
두번째 Cache Layer가 추가되었다. Batch Layer에서 갱신한 배송상태정보에 대하여 Service Layer가 요청하면 제공하는 역할이다.

![Cache_시스템구성도 drawio](https://user-images.githubusercontent.com/26420767/191025787-9df414d5-bd28-416f-9f70-36fb5cf18c90.png)

Cache Layer는 크게 7 가지로 구성된다. 자세한 설명은 다음 표를 참고한다. 

|      Module      |                      역할                       |                  비고                  |
| :--------------: | :---------------------------------------------: | :------------------------------------: |
|   cache Module   | - Cache 구조체<br />- Object 구조체<br />- Main |                                        |
|  server Module   |                  - Connection                   |        - Port Listen and Close         |
|  handler Module  |             - 처리할 명령어를 전달              | - Key의 정합성 체크<br />- Error 처리  |
|  command Module  |               - 주요 명령어 처리                | - TTL 기능<br />- 무효화 알고리즘 선택 |
|   util Module    |       - 공통 기능<br />- Key 유효성 검사        |                                        |
| constants Module |       - 명령어 정의<br />- 응답 코드 정의       |                                        |
| Eviction Module  |                - Cache Eviction                 |     - Cache에 대한 유효성 알고리즘     |

- 리시버(Receiver)를 통해 어떤 구조체(struct)의 메소드인지 명시



### 4.1.3 설계전략 및 적용 범위

"3.2 아키텍처 문제분석 및 설계전략"에서 선택된 설계전략을 시스템 구성도에서 Layer 및 Module에서 적용하여 구현한다. 다음 그림에서는 각각의 설계전략이 시스템 구성도의 어느 부분에 적용되었는지 나타낸다. 일부 설계 전략은 여러 Layer 및 Module에 걸쳐서 나타난다. 이런 경우는 그 중 주요 요소에 배치하였다. 각각의 상세 내용은 실행 아키텍처에 상세하게 기술한다.

![Cache_시스템구성도_설계전략및적용범위 drawio](https://user-images.githubusercontent.com/26420767/191029062-069970cc-1cfb-4eb1-b63f-89c8dde324f0.png)



### 4.1.4 구현 및 검증 대상

이번 설계는 아키텍처 구현 및 검증을 포함하고 있다. 실제 구현 및 검증하는 범위는 시스템 구성도에서 다음 부분 이다. 
고객사 물류시스템의 Service Layer와 Batch Layer 그리고 Cache Layer이다. 고객사 물류시스템의 Service Layer는 Batch Layer와의 기능을 분리 및 수정이 발생한다. 신규 신규 영역은 Batch Layer와 Cache Layer 이다. 

![Cache_시스템구성도_구현및검증대상 drawio](https://user-images.githubusercontent.com/26420767/191029136-5f8f5d3c-4194-4f83-9076-a71dd2b5c18d.png)



## 4.2 실행 아키텍처

> 모듈이 실제 작동하는 구조를 기술함
>
> 큰 모듈내 세부 모듈간 크기, 위치, 관계를 상세히 작성
>
> 사용하는 SW Stack을 명확히 작성
>
> SW가 동작하는 HW 환경은 검증/구현 대상에 대해 명확히 작성

다음은 "실행 아키텍처 목차 별 설계 전략" 에 관한 표 이다.

|       목차        |                   상세 내용                   |                          세부 목차                           |             관련 기능 요구사항 및 아키텍처 전략              |
| :---------------: | :-------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 4.2.1 시나리오 뷰 |       각 Use-Case의 상세 시나리오 기술        |                           SDSCache                           | UC-01. Cache추가한다.<br />UC-02. Cache삭제한다.<br />UC-03. Cache조회한다.<br />UC-04.TTL설정한다.<br />UC-05. Cache제거한다.<br />UC-07. Cache알고리즘적용한다.<br />UC-08.  Cache알고리즘추가한다. |
|                   |                                               |                      고객사 물류 시스템                      |                   UC-14. 배송상태조회한다.                   |
|   4.2.2 모듈 뷰   | 시스템의 모듈 및 코드 단위를 통한 서비스 설계 |                      Repository Module                       |              AS-01. Cache를 보관하는 자료 구조               |
|                   |                                               |                       Eviction Module                        | AS-05. Cache 의 만료 정책<br />AS-10. Strategy Pattern을 이용한 알고리즘 캡슐화 및 교환 |
|                   |                                               |                        Server Module                         |             AS-06. TCP 통신 시, TLS 기반 암호화              |
|                   |                                               |                         Util Module                          | AS-04. Cache를 조회 할 때, Key의 데이터 타입은 문자열, 비교 방법은 동등성(Equality) |
|                   |                                               |                        Handler Module                        |              AS-03. 요청에 대한 응답 코드 정의               |
|                   |                                               |                        Client Module                         | AS-02. 요청에 대하여 동기화 및 비동기화 응답<br />  AS-09. Proxy를 활용한  Cache 분산 |
|   4.2.3 실행 뷰   |           각 서비스의 실행 아키텍처           |                  Online과 Batch 서비스 분리                  | AS-12. 근 실시간(Near Realtime)으로 배송상태조회에 대한 업무 프로세스 변경 |
|                   |                                               |                   Cache솔루션에서 자료제공                   | AS-14. 트래킹이 필요한 배송상태에 대하여 Cache솔루션에서 보유하여 제공 |
|                   |                                               | Online 서비스에서 데이터 조회 시 Cache 먼저 조회 후, DB 조회 | AS-13. 조회가 많은 업무 특성을 고려한 Cache솔루션과 Data Store의 배치 |
|   4.2.4 배치 뷰   |            시스템의 서버 배치 환경            |                                                              |                                                              |
|                   |                                               |                                                              |                                                              |
|                   |                                               |                                                              |                                                              |



### 4.2.1 시나리오 뷰



### 4.2.2 모듈 뷰



### 4.2.3 실행 뷰



### 4.2.4 배치 뷰



## 4.3 설계문제 분석 및 평가

> 요구사항과 설계결과 Align 및 누락여부 확인
>
> 시나리오 기반(ATAM 등), 의사결정 기반, 전문가 경험기반 분석 및 평가 등 활용



### 4.3.1 아키텍처 설계 결정표

| 설계전략 | 적용점 | 상세 적용점 | 해결방법 |
| -------- | ------ | ----------- | -------- |
|          |        |             |          |
|          |        |             |          |
|          |        |             |          |
|          |        |             |          |
|          |        |             |          |
|          |        |             |          |
|          |        |             |          |
|          |        |             |          |
|          |        |             |          |



### 4.3.2 요구사항 관점의 분석

| 분석관점 |      |      |      | 분석결과 | 상세 적용점 |
| -------- | ---- | ---- | ---- | -------- | ----------- |
|          |      |      |      |          |             |
|          |      |      |      |          |             |
|          |      |      |      |          |             |
|          |      |      |      |          |             |
|          |      |      |      |          |             |
|          |      |      |      |          |             |
|          |      |      |      |          |             |
|          |      |      |      |          |             |
|          |      |      |      |          |             |



### 4.3.3 민감점 분석

| 분석관점 |      | 분석결과 |
| -------- | ---- | -------- |
|          |      |          |
|          |      |          |
|          |      |          |
|          |      |          |
|          |      |          |
|          |      |          |
|          |      |          |
|          |      |          |
|          |      |          |



### 4.3.4 위험 요인

| ID   | 위험요인/전략 | 확률 | 영향 | 완화전략 |
| ---- | ------------- | ---- | ---- | -------- |
|      |               |      |      |          |
|      |               |      |      |          |
|      |               |      |      |          |
|      |               |      |      |          |
|      |               |      |      |          |
|      |               |      |      |          |
|      |               |      |      |          |
|      |               |      |      |          |
|      |               |      |      |          |



### 4.3.5 Traceability Summary

| ID   | 품질 요구사항 | 품질속성<br />시나리오 | 아키텍처<br />드라이버 | 설계문제 | 설계전략 | S/W Stack | 문서목차 | 아키텍처<br />검증<br />시나리오 |
| ---- | ------------- | ---------------------- | ---------------------- | -------- | -------- | --------- | -------- | -------------------------------- |
|      |               |                        |                        |          |          |           |          |                                  |
|      |               |                        |                        |          |          |           |          |                                  |
|      |               |                        |                        |          |          |           |          |                                  |
|      |               |                        |                        |          |          |           |          |                                  |
|      |               |                        |                        |          |          |           |          |                                  |
|      |               |                        |                        |          |          |           |          |                                  |
|      |               |                        |                        |          |          |           |          |                                  |
|      |               |                        |                        |          |          |           |          |                                  |
|      |               |                        |                        |          |          |           |          |                                  |



# 5. 아키텍처 구현 및 검증

## 5.1 아키텍처 검증 시나리오

> 아키텍처 요구사항 및 목표 달성을 확인할 수 있는 검증 시나리오 작성
>
> 시뮬레이션 기반 검증(PoC/BMT 등)



## 5.2 아키텍처 구현

> 검증 시나리오가 작동하는 것을 확인할 수 있는 아키텍처 구현



## 5.3 현장 활용도

> 현장 활용도 점검



# 별첨

## 1. Cache 란

### 1.1 Cache 대상

- Database 또는 API에 조회 시, 비용이 높으며 많은 시간이 소요되는 우선순위가 높은 데이터
- 비교적 정적이고 자주 엑세스하는 데이터
- 운송장번호 처럼 일정기간 동안 변화가 없는 데이터



### 1.2 Cache 이점

- 애플리케이션 속도 향상
- 비용이 높고 많은 시간이 소요되는 Database 및 API의 부담 완화
- 응답 지연 시간 감소



### 1.3 Cache의 구조

Cache의 구조에는 Look aside cache와 Write back 이 있다.

#### 1.3.1 Look aside cache

![LookAsideCache.drawio](https://raw.githubusercontent.com/u4rang/save-image-repo/main/img/LookAsideCache.drawio.png)



#### 1.3.2 Write back 

![WriteBack.drawio](https://raw.githubusercontent.com/u4rang/save-image-repo/main/img/WriteBack.drawio.png)



### 1.4 Cache의 유형

Cache의 유형은 크게 클라이언트 측과 서버 측으로 나눈다.

![image-20220410235249569](https://raw.githubusercontent.com/u4rang/save-image-repo/main/img/image-20220410235249569.png)

클라이언트 측은 웹 브라우저가 있다.

서버 측은 웹 서버와 역방향 프록시 캐시가 있다.

과제에서 다루는 Cache의 유형은 웹 서버이다.



## 2. Cache의 Cluster 유형

Cache의 Cluster 유형은 다음과 같이 3가지 유형이 있다.

|      항목       | Local                                                        | Replication                                                  | Partition                                                    |
| :-------------: | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
|      특징       | - 다른 노드에 데이터 배포 안됨                               | - 모든 노드에 데이터 사본 보유<br />                         | - 확장성이 뛰어난 분산 캐시 모드<br />- 클러스터 리스스를 최대한 활용 가능 |
|    유의사항     | - 단일 노드에 장애 발생 시 데이터 손실 위험 존재             | - 노드가 추가될 수록 변경 시 더 많은 트래픽이 발생하여 성능 및 확장성 제한 | - 클러스터의 단일 노드 장애 발생 시 데이터 손실 위험 존재    |
| 적합한 시나리오 | - 데이터가 읽기 전용 또는 특정 만료 빈도로 주기적으로 새로 고쳐질 수 있는 시나리오에 적합 | - 많이 변경되지 않은 비교적 작은 데이터 세트가 있는 시나리오에 적합 | - 대규모 데이터 세트로 작업하고 업데이트가 빈번한 시나리오에 적합 |



### 2.1 Local Mode

![Cache_Cluster_LocalMode drawio](https://user-images.githubusercontent.com/26420767/190362394-7538c5b0-321b-4595-b97e-81165749e0c0.png)



### 2.2 Replication Mode

![Cache_Cluster_ReplicationMode drawio](https://user-images.githubusercontent.com/26420767/190362649-9211d2d6-1459-4452-90c0-947def2c0e05.png)



### 2.3 Partition Mode

![Cache_Cluster_PartitionMode drawio](https://user-images.githubusercontent.com/26420767/190362553-afa0be13-f906-48b1-ba69-6b58815a773b.png)



## 3. 비교대상 Redis와 Memcached에 대한 BMT

### 3.1 BMT 환경



### 3.2 Redis



### 3.3 Memcached 



## 4. CSP의 관리형(Managed), 완전관리형(Fully Managed) 서비스

### 4.1 완전관리형, Fully Managed

사용자가 RDS, DynamoDB, ElastiCache, Redshift와 같은 DB 서비스를 이용하는 것으로 내부적으로 서버/OS가 있지만 사용자에게는 드러나지 않으며, DB 솔루션 설치 또한 필요 없이 바로 DB를 사용할 수 있다.

사용자는 설정만으로 또는 설정하지 않아도 쉽게 백업, 가용성에 대한 부분을 보장 받을 수 있다.

Redshift에 경우, 지속적 증분 백업을 다른 리전으로 복제하여 스트리밍 복구를 지원하여 빠르게 백업데이터를 빠르게 사용할 수 있다.

또한, 디스크 결함/ 노드 결함/ 네트워크 결함 등의 내결함으로 발생하는 이벤트에 대해 AZ/리전 레벨수준에서 대비하여 사용자에게 가용성을 제공한다.



### 4.2 관리형, Managed

사용자가 AWS EC2에 DB를 직접 설치하여 운영하는 경우로 AWS에서 서버/OS 를 모두 제공하지만, DB 솔루션 설치 및 운영에 대한 부분은 제공하지 않는다.

DB 설치, 백업, 가용성에 대한 부분을 사용자가 직접 관리 해야 한다. 



AWS Summit Seoul 2017 - 클라우드 기반 AWS 데이터베이스 선택 옵션 참고





## 5. SDSCache 개발 언어

### 5.1 개발 언어 후보 개요

개발 언어는 2022.08 기준, 가장 최신 버전으로 선택하여 BMT를 수행한다. 

개발 언어에 대한 요구사항은 다음과 같다.

- 객체 지향 언어
- 다양한 플랫폼 지원 (Linux, Windows, OS X)
- Cloud 환경과 Container 환경에 대한 지원
- Pointer에 대한 지원
  - Cache 데이터 구조에 대한 빠른 지원 및 메모리 주소를 통한 제어



### 5.2 개발 언어 BMT

### 5.2.1 개발 언어 항목별 비교

|           항목            | 설명                                                         | 가중치 | 점수 | Java | C++  | golang | Python |
| :-----------------------: | ------------------------------------------------------------ | :----: | :--: | :--: | :--: | :----: | :----: |
| 병행성<br />(Concurrency) |                                                              |        |      |      |      |        |        |
|    멀티코어 환경 지원     |                                                              |        |      |      |      |        |        |
|      멀티쓰레딩 지원      |                                                              |        |      |      |      |        |        |
|         정적언어          |                                                              |        |      |      |      |        |        |
|          강타입           |                                                              |        |      |      |      |        |        |
|        컴파일 언어        |                                                              |        |      |      |      |        |        |
|       Pointer 지원        | 데이터 구조에 대한 세밀한 지원                               |        |      |      |      |        |        |
|           성능            |                                                              |        |      |      |      |        |        |
|  Garbage Collection 지원  |                                                              |        |      |      |      |        |        |
|    다양한 플랫폼 지원     |                                                              |        |      |      |      |        |        |
|      Cloud 환경 지원      |                                                              |        |      |      |      |        |        |
|    Container 환경 지원    |                                                              |        |      |      |      |        |        |
|         기술지원          | 국내에 활성화된 커뮤니티 또는 <br />레퍼런스가 있는 기술지원 업체 |        |      |      |      |        |        |
|        릴리즈 주기        | 메이저 버전 릴리즈 주기 평균 1년                             |        |      |      |      |        |        |
|                           |                                                              |        | 점수 |      |      |        |        |



### 5.2.2 코드 복잡도와 Concurrency 관점 언어별 특징

![코드 복잡도와 Concurrency 관점 언어별 특징](https://user-images.githubusercontent.com/26420767/183367593-7e2a1d1d-3333-438e-90f3-639539315d63.png)



### 5.2.3 Code Readability와 Efficiency 관점 언어별 특징

![Code Readability와 Efficiency 관점 언어별 특징](https://user-images.githubusercontent.com/26420767/183367535-5046f718-da47-415e-84ba-71c00f753e4c.png)



### 5.3 golang 언어 개요

#### 5.3.1 golang 정의

- 명료한 문법을 가지고 있는 컴파일 언어
  - 2009년 Google 이 제작
  - 컴파일 속도가 빨라 인터프리터 언어처럼 사용 가능
  - 다양한 플랫폼을 지원
  - 문서화가 된 공통 라이브러리 지원
- 라이선스
  - BSD 3-clause



#### 5.3.2 golang 특징

| 분류   | 특징                                             | 설명                                                         |
| ------ | ------------------------------------------------ | ------------------------------------------------------------ |
| 언어적 | Fast Performance                                 | - 컴파일 언어로 빠른 컴파일 속도 제공                        |
|        | Garbage Colletion                                | - 가상 머신 없이 가비지 컬랙션 제공<br />- 메모리 효율성     |
|        | Concurrency & Scalability                        | - 언어차원에서 동시성과 병렬성(멀티코어 환경) 지원<br />- 정적타입, 강 타입포인터 연산이 존재하지 않는 포인터 지원클래스, 상속, Assertions, 오버로딩 미지원 |
|        | 높은 생산성, 통일성                              |                                                              |
|        | Cross Platform 지원                              | - Windows<br />- Linux<br />- OS X<br />- 모바일에서도 지원 예정<br />  - Android<br />  - iOS <br />  - https://github.com/golang/go/wiki/Mobile |
|        | Error Checks                                     |                                                              |
| 문법적 | No Type Inheritance                              |                                                              |
|        | No Method or Operator Overloading                |                                                              |
|        | No support for Pointer Arithmetic                |                                                              |
|        | No support for Assertions                        |                                                              |
|        | No Exceptions - instead use an error return type |                                                              |
|        | Dependency Management                            |                                                              |



#### 5.3.3 Goroutine 비동기 매커니즘

Erlang에서 영향 받은 메카니즘으로 각각 병렬로 동작하며 메시지 채널을 통해 값을 주고 받는 방식으로 이벤트 처리 및 병렬 프로그래밍에 유용하다.

- Unbuffered Channels

<img width="556" alt="UnbufferedChannels" src="https://user-images.githubusercontent.com/26420767/183370129-6c99bf7d-7537-45b8-af53-c3b53ea7a256.png">

- Buffered Channels

![BufferedChannels](https://user-images.githubusercontent.com/26420767/183370351-10c7ab43-5b1b-4274-9665-3abe8f322a22.png)



#### 5.3.4 Native Binary 지원

다른 머신 플랫폼을 타켓으로 할 경우, 플랫폼에 맞도록 배포를 해야 한다.

- VM Based Languages

![VMBasedLanguages](https://user-images.githubusercontent.com/26420767/183370690-62c06462-a8d5-4f3d-8039-3d2fa6c24335.png)

- golang

![Golang](https://user-images.githubusercontent.com/26420767/183370767-7afa8840-55d3-40f7-b5d3-ae6e637f2c19.png)



#### 5.3.5 golang 버전에 따른 출시일 및 특징

| 버전 | 출시일(Release Date) | 특징                                                         |
| ---- | -------------------- | ------------------------------------------------------------ |
| 1.19 | 2022-08-02           | - 메모리 모델을 C++/Java/Rust 등에서 사용하는 모델과 맞춤<br />  - Atomic 값을 사용하기 쉽게 atomic.Int64 및 atomic.Pointer[T] 같은 타입 추가<br />- LoongArch 64 지원(Linux 5.19 이상)<br />- RISC-V 10% 이상 성능 개선(함수 인자 및 결과를 레지스터를 이용하여 전달 지원)<br />- 런타임에 Soft Memory Limit 지원 |
| 1.18 | 2022-03-15           | - Generics 지원<br />- AMD64 아키텍처의 최소 대상 버전을 선택하는 GOAMD64 환경 변수 지원 |
| 1.17 | 2021-08-16           | - unsafe.Pointer 안전 규칙을 준수하는 코드 작성 간소화 지원<br />- 슬라이스에서 배열 포인터로의 변환 |
| 1.16 | 2021-02-16           | - module 기본 옵션 채택                                      |
| 1.15 | 2020-08-11           |                                                              |
| 1.14 | 2020-02-25           |                                                              |
| 1.13 | 2019-09-03           |                                                              |
| 1.12 | 2019-02-25           |                                                              |
| 1.11 | 2018-08-24           | - 모듈이라는 패키지 관리 기능이 추가                         |
| 1.10 | 2018-02-16           |                                                              |
| 1.9  | 2017-08-24           |                                                              |
| 1.8  | 2017-02-16           | - 32비트 MIPS 명령어 지원<br />- 컴파일러 프론트엔드 추가<br />- 가비지 컬렉션 개선<br />- Cgo의 오버헤드 개선 |
| 1.7  | 2016-08-15           | - 컴파일 속도의 개선, 실행 퍼포먼스 향상<br />- /x/net/context 패키지의 기본 패키지화 |
| 1.6  | 2016-02-17           | - HTTP/2가 기본으로 지원<br />- 템플릿 문법의 개선           |
| 1.5  | 2015-08-19           | - Go 컴파일러가 Go로 작성                                    |
| 1.4  | 2014-12-10           |                                                              |
| 1.3  | 2014-06-18           |                                                              |
| 1.2  | 2013-12-01           |                                                              |
| 1.1  | 2013-05-13           |                                                              |



#### 5.3.6 golang 사용사례

- 솔루션 영역
  - CockroachDB 개발에 사용 
  - Youtube 개발에 사용
  - Docker 개발에 사용
  - Kubernetes(컨테이넌 관리소프트웨어) 사용
  - Terraform 에 사용
- 서비스 영역
  - 국내
    - 카카오엔터프라이즈
    - 당근마켓
    - 왓차
    - 버즈빌
  - 해외
    - 우버
    - 넷플릭스
    - BBC



### 5.3.7 golang 자료구조 성능 비교

다음은 Big-O 표기법으로 Array, Slice, List, Map의 추가, 삭제, 읽기 동작에 성능 비교 자료 이다.

| 동작 |    Array, Slice     |        List         |        Map        |
| :--: | :-----------------: | :-----------------: | :---------------: |
| 추가 |        O(N)         |        O(1)         |       O(1)        |
| 삭제 |        O(N)         |        O(1)         |       O(1)        |
| 읽기 | O(1) - Index의 접근 | O(N) - Index로 접근 | O(N) - Key로 접근 |

Map은  Key와 Value의 쌍으로만 동작하기 때문에 Index를 사용해서 접근할 수 없고 입력한 순서가 보장되지 않는다. Array, Slice 그리고 List에 비해 상대적으로 Memory를 적게 소모한다.

[^1,000명]: 



## 5. HTTP 상태코드

참고자료 : https://developer.mozilla.org/ko/docs/Web/HTTP/Status



## 6. TLS
