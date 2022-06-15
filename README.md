

[TOC]

------



# 1. Cache 솔루션 개요

> 과제 대상 시스템 개념, 구성 등 시스템 이해를 위한 기술
>
> 기존 시스템이 있는 경우 기존 시스템 분석 내용 기술


## 1.2 Cache 솔루션 선정 사유

IT 서비스를 구성하는 기본적인 Component 중 널리 사용가능하며 범용적인 역할을 하는 Cache 솔루션을 과제로 선정한 사유는 다음 2가지 이다.

### 1.2.1 GDC[^1] 확대에 따른 CI 부서의 통폐합 및 역할 변경

[^1]: Global Delivery Center

 2022년, 김경환 프로가 속한 CI 부서가 통폐합, 사무실 이전 계획 및 담당업무, 역할 변경이 계획되어 있다. 따라서 개인 과제는 이러한  변화에 대한 영향이 가장 적은 주제로 선택하도록한다.

### 1.2.2 오픈소스 라이센스 변경 이슈와  프로테스트웨서 (Protestware)

2022년, IT 산업에서는 2 가지의 오픈소스 이슈가 있다. 

첫 번째는 오픈소스 라이센스 변경 이슈 이다. 변경에 따라 사용하는 오픈소스를 더 이상 사용하기 불가하여 대체 솔루션으로 변경해야 하는 상황이다.  최근 사례로는 Object Storage 인 minIO 가 있다. 라이센스 변경에 따라 minIO를 사용하는 솔루션은 소스코드를 공개해야하는 의무가 발생하였다.

두 번째는 프로테스트웨어 (Protestware) 이다. 프로테스트웨어는 개발자가 스스로 자신의 코드를 훼손하는 사보타주 행위이다. 개발자가 자신의 소프트웨어를 수정할 권리가 있지만 이 같은 행동은 오픈소스 생태계의 신뢰를 손상한다.

2022년 1월부터 3월까지 3건의 프로테스트웨어가 있다.

2022년 1월, 개발자에 의한 NPM 라이브러리 Faker.js, Colors.js 에 무한 루프 삽입 한 사건이다.

2022년 2월, 특정 테라폼 모듈이 라이센스 항목을 변경하여, 러시아의 침공을 인정하고 "푸틴은 멍청이(Putin khuylo!)" 라는 내용에 동의해야만 사용가능하게 만든 사건이다.

2022년 3월, npm 리포지토리에 호스팅된 자바스크립트 구성요소, node-ipc 개발자가 러시아의 우크라이나 침공을 비판하기 위해 사용자의 컴퓨터에 임의로 파일을 추가하거나 삭제하는 코드를 배포하는 사건이다.  

위와 같은 오픈소스 이슈 때문에, IT컨설팅, 시스템 통합(SI) 과 IT유지보수 그리고 CSP 사업을 하는 S 社 는 오픈소스 기반 솔루션 중 Cache 솔루션을 선택하여 내제화하여 물류시스템에 적용하고 CSP 사업에 관리형, 완전 관리형 서비스로 제공하기로 결정하였다.



## 1.3 Project Overview

### 1.3.1 Stakeholder

Cache 솔루션의 Stakeholder는 다음과 같다.

![image-20220410233048266](https://raw.githubusercontent.com/u4rang/save-image-repo/main/img/image-20220410233048266.png)



### 1.3.2 Busincess Context

금번 과제의 Business Context는 다음과 같다.

![image-20220410233258258](https://raw.githubusercontent.com/u4rang/save-image-repo/main/img/image-20220410233258258.png)



### 1.3.3 Business Goal

Cache 솔루션이 제공됨에 따라 Stakeholder들이 기대하는 Business 목표이다.

| Stakeholder      | ID    | Statement                                                    | Importance(상, 중, 하) |
| ---------------- | ----- | ------------------------------------------------------------ | ---------------------- |
| 물류시스템개발자 | BG-01 | Cache 솔루션의 효율적인 대체                                 | 상                     |
| 물류시스템운영자 | BG-02 | 안정적인 운영과 Cache 솔루션 오픈소스 이슈 Hedge             | 상                     |
| 물류시스템사용자 | BG-03 | Cache 솔루션 도입으로 안정적인 서비스 기대                   | 상                     |
| CSP시스템개발자  | BG-04 | 관리형, 완전관리형 Cache 솔루션 개발                         | 중                     |
| CSP시스템운영자  | BG-05 | Cache 솔루션 오픈소스 이슈 Hedge와 Cache 솔루션에 대한 VOC 해결 | 중                     |
| CSP시스템사용자  | BG-06 | 관리형, 완전관리형 Cache 솔루션으로 관리 요소 감소 기대      | 중                     |



## 1.4 Cache 솔루션 RoadMap

Cache 솔루션은 우선 내제화한다. 내제화의 단계는 크게 Standalone 과 Cluster 단계로 구분한다. Standalone 에서는 Cache 의 기본적인 기능인 읽기, 쓰기, 삭제을 구현한다. Cluster 단계 에서는 여러 개의 노드로 구성하여 구축하는 것을 목표로 한다.

그리고 CSP 사업에서 활용하도록 관리형 서비스 단계와 완전 관리형 서비스 단계로 한다.

![image-20220410234147561](https://raw.githubusercontent.com/u4rang/save-image-repo/main/img/image-20220410234147561.png)





## 1.5 기존 Cache 솔루션 기능 분석

Cache 솔루션을 확보하기 위해서는 선도 CSP 社에서 제공하는 Cache 솔루션을 조사한다. 이를 기반으로 Cache의 기본적인 기능 외에 他 Cache 솔루션이 제공하는 기능을  조사한다. 

다음은 선도 CSP 社에서 서비스하는 Cache 솔루션 이다.

| CSP 社 | 서비스 명              | 특징                    |
| ------ | ---------------------- | ----------------------- |
| AWS    | ElasticCache Memcached | 오픈소스 Memcached 기반 |
| AWS    | ElasticCache Redis     | 오픈소스 Redis 기반     |
| GCP    | Memorystore Memcached  | 오픈소스 Memcached 기반 |
| GCP    | Memorystore Redis      | 오픈소스 Redis 기반     |
| Azure  | Azure Cache for Redis  | 오픈소스 Redis 기반     |

대표적인 오픈소스 Cache 솔루션은 Memcached와 Redis 가 대표적이다. 다음은 대표적인 기능에 대한 두 솔루션의 비교이다.

| 기능                                                   | Memcached | Redis  |
| ------------------------------------------------------ | --------- | ------ |
| DB 부하를 오프로드하는 단순 캐시                       | 예        | 예     |
| 쓰기 및 스토리지를 위해 수평적으로 확장할 수 있는 기능 | 예        | 예     |
| 다중 스레드 유형                                       | 예        | 아니요 |
| 고급 데이터 유형                                       | 아니요    | 아니요 |
| 데이터 세트 정렬 및 순위 지정                          | 아니요    | 아니요 |
| Publish와 Subscribe 기능                               | 아니요    | 아니요 |
| 자동 장애조치가 있는 다중 가용 영역                    | 아니요    | 아니요 |
| 지속성                                                 | 아니요    | 아니요 |

Cache 솔루션은 Memcached와 Redis의 장점만을 취하여  적용한다.



## 1.6 과제 목표

금번 과제를 통해 해결하고자 하는 목표는 다음과 같다.

- Cache 솔루션의 기본적인 기능인 읽기, 쓰기, 삭제 확보를 목표로 한다.
- 기본적인 기능을 기반으로 성능 및 안정성 확보를 목표로 한다.
- 오픈소스 Memcached 와 Redis 의 장점을 Cache 솔루션에 적용을 목표로 한다.



## 1.7 과제 범위

내제화 단계 중 Standalone 을 범위로 제한한다.

![image-20220411001829658](https://raw.githubusercontent.com/u4rang/save-image-repo/main/img/image-20220411001829658.png)



금번 과제에는 Cache 솔루션의 Cluster 구현은 제외한다. 



기 운영중인 서비스인 B社 물류시스템에 금번 과제에서 구현한 Cache 솔루션을 Standalone 으로 적용하여 테스트한다.



Cache 솔루션의 특징인 성능과 안정성을 부하분산기 Apache jMeter 를 이용하여 검증한다.

검증 대상은 물류시스템에 대해 Database 의 데이터 조회와 API 조회에 대하여 Cache 를 이용하는 시나리오로 한다.



# 2. 솔루션 요구사항 정의

## 2.1 System Overview

<System Context 그림 추가>

### 2.1.1 External Entity

| 구분 | 설명 | 기대사항 |
| ---- | ---- | -------- |
|      |      |          |
|      |      |          |
|      |      |          |
|      |      |          |



### 2.1.2 External Interface

| 구분 | 설명 |
| ---- | ---- |
|      |      |
|      |      |
|      |      |
|      |      |



## 2.2 기능 요구사항

> 과제 수행 기간 내 구현 및 인터페이스 가능한 범위 도출
>
> 전체 범위와 구현 대상 범위를 구분

![image-20220411033627670](https://raw.githubusercontent.com/u4rang/save-image-repo/main/img/image-20220411033627670.png)

다음은 Cache의 기본기능 이며 주요 UseCase 이다.

- Cache추가한다
- Cache삭제한다
- Cache조회한다



다음은 UseCase 목록이다.

| ID    | Title                   | 설명                                                         | 중요도 | 난이도 | System Feature ID |
| ----- | ----------------------- | ------------------------------------------------------------ | ------ | ------ | ----------------- |
| UC-01 | Cache추가한다           | Database, API 에서 조회한 데이터 A를 Cache에 저장한다.       | 상     | 상     |                   |
| UC-02 | Cache삭제한다           | Cache에 저장된 데이터 A를 Cache에서 더이상 사용하지 않도록 관리하다. | 상     | 상     |                   |
| UC-03 | Cache조회한다           | Cache에 저장된 데이터 A를 조회한다.                          | 상     | 상     |                   |
| UC-04 | TTL설정한다             | Cache의 데이터 A에 TTL(Time to Live )을 설정하여 유효시간을 관리한다. | 상     | 상     |                   |
| UC-05 | Cache제거한다           | Cache의 데이터 A를 무효화한다.                               | 상     | 상     |                   |
| UC-06 | CacheMode적용한다       | Local, Partitioned, Replicated 중 Mode 를 적용한다.          | 중     | 상     |                   |
| UC-07 | Cache모니터링한다       | Cache 솔루션에 대하여 서버 리소스에서부터 Cache 의 HitRatio 까지 모니터링한다. | 상     | 상     |                   |
| UC-08 | Cache알고리즘적용한다   | Cache 솔루션의 HitRatio 를 높이기 위한 알고리즘을 선택한다.  | 중     | 상     |                   |
| UC-09 | Cache알고리즘추가한다   | Cache 솔루션의 HitRatio 를 높이기 위한 알고리즘을 추가한다.  | 중     | 상     |                   |
| UC-10 | LocalMode설정한다       | 데이터가 읽기 전용 또는 특정 만료 빈도로 주기적으로 새로 고쳐질 수 있는 시나리오에 적합한 Local Mode를 설정한다. | 중     | 상     |                   |
| UC-11 | PartitionedMode설정한다 | 데이터 세트가 작고 업데이트가 자주 발생하지 않는 시나리오에 적합한 Partitioned Mode를 설정한다. | 중     | 상     |                   |
| UC-12 | ReplicatedMode설정한다  | 대규모 데이터 세트로 작업하고 업데이트가 빈번한 시나리오에 적합한 Replicated Mode를 설정한다. | 중     | 상     |                   |
| UC-13 | Cache노드관리한다       | Cache 솔루션이 Cluster 인 경우 노드를 추가하거나 삭제한다.   | 중     | 중     |                   |



### 2.2.1 UC-01 Cache추가한다

| UC-01    | Cache추가한다                                                |
| -------- | ------------------------------------------------------------ |
| 설명     | Database, API 에서 조회한 데이터 A를 Cache에 저장한다        |
| 행위자   | 물류시스템, CSP시스템사용자                                  |
| 선행조건 | 조회하는 데이터 A가 Cache 솔루션에 미존재                    |
| 후행조건 | -                                                            |
| 기존동작 | 1. 물류시스템이 Database 또는 API에서 데이터 A를 조회한다.<br />2. 물류시스템이 받은 데이터 A를 Cache 솔루션에 추가한다. |
| 추가동작 | -                                                            |



### 2.2.2 UC-02 Cache삭제한다

| UC-02    | Cache삭제한다                                                |
| -------- | ------------------------------------------------------------ |
| 설명     | Cache에 저장된 데이터 A를 Cache에서 더이상 사용하지 않도록 관리한다 |
| 행위자   | 물류시스템, CSP시스템사용자                                  |
| 선행조건 | 조회하는 데이터 A 가 Cache 솔루션에 존재                     |
| 후행조건 | 데이터 A가 Cache 솔루션에 유효하지 않은 상태 또는 삭제       |
| 기존동작 | 1. 물류시스템 또는 CSP시스템사용자이 Cache 솔루션의 데이터 A 를 삭제요청한다 |
| 추가동작 | TTL 값 변경의 경우, UC-04 "TTL설정한다" 로 이동한다<br />데이터 무효화의 경우, UC-05 "Cache제거한다"로 이동한다 |



### 2.2.3 UC-03 Cache조회한다

| UC-03    | Cache조회한다                                                |
| -------- | ------------------------------------------------------------ |
| 설명     | Cache에 저장된 데이터 A를 조회한다                           |
| 행위자   | 물류시스템, CSP시스템사용자                                  |
| 선행조건 | 조회하는 데이터 A 가 Cache 솔루션에 존재                     |
| 후행조건 | -                                                            |
| 기존동작 | 1. 물류시스템이 데이터 A를 Cache 솔루션에 조회한다.<br />2. Cahe 솔루션은 데이터 A가 있으면 물류시스템에 제공한다. |
| 추가동작 | -                                                            |



### 2.2.4 UC-04 TTL설정한다

| UC-04    | TTL설정한다                                                  |
| -------- | ------------------------------------------------------------ |
| 설명     | Cache의 데이터 A에 TTL(Time to Live )을 설정하여 유효시간을 관리한다 |
| 행위자   | 물류시스템, CSP시스템사용자                                  |
| 선행조건 | UC-02 "Cache삭제한다"                                        |
| 후행조건 | -                                                            |
| 기존동작 | 1. 물류시스템 또는 CSP시스템사용자는 Cache의 데이터 A에 대해 삭제 지시한다.<br />2. Cache 솔루션은 Cache의 데이터 A에 대한 TTL을 변경하여 유효시간을 변경한다. <br />`expected expire time = current time + set expire time, 단위 second` |
| 추가동작 | 1. 데이터 A 가 만료되어도 Cache솔루션에서 제거되지 않는다.<br />2. 누군가 만료된 데이터 A 에 엑세스 하려고 하면 Cache 솔루션은 데이터 A 를 검사한다.<br />3. 데이터 A 가 만료되었는지 확인 후 메모리에서 제거한다. |



### 2.2.5 UC-05 Cache제거한다

| UC-05    | Cache제거한다                                                |
| -------- | ------------------------------------------------------------ |
| 설명     | Cache의 데이터 A를 무효화한다.                               |
| 행위자   | 물류시스템, CSP시스템사용자                                  |
| 선행조건 | UC-02 "Cache삭제한다"                                        |
| 후행조건 | -                                                            |
| 기존동작 | 1. 물류시스템 또는 CSP시스템사용자가 Cache 솔루션에 Cache의 데이터 A를 무효화를 지시한다.<br />2. Cache 솔루션은 데이터 A에 대해 무효화 처리한다. |
| 추가동작 | 1. 데이터 A 가 만료되어도 Cache솔루션에서 제거되지 않는다.<br />2. 누군가 만료된 데이터 A 에 엑세스 하려고 하면 Cache 솔루션은 데이터 A 를 검사한다.<br />3. 데이터 A 가 만료되었는지 확인 후 메모리에서 제거한다. |



### 2.2.6 UC-06 CacheMode적용한다

| UC-06    | CacheMode적용한다                                            |
| -------- | ------------------------------------------------------------ |
| 설명     | Local, Partitioned, Replicated 중 Mode 를 적용한다           |
| 행위자   | 물류시스템개발자,CSP시스템개발자                             |
| 선행조건 | Cache솔루션이 Cluster 형태이다                               |
| 후행조건 | -                                                            |
| 기존동작 | 1. 물류시스템개발자 또는 CSP시스템개발자가 Cache 솔루션의 Cache Mode를 변경 지시한다. |
| 추가동작 | -                                                            |





### 2.2.7 UC-07 Cache모니터링한다

| UC-07    | Cache모니터링한다                                            |
| -------- | ------------------------------------------------------------ |
| 설명     | Cache 솔루션에 대하여 서버 리소스에서부터 Cache 의 HitRatio 까지 모니터링한다 |
| 행위자   | 물류시스템운영자,CSP시스템운영자                             |
| 선행조건 | -                                                            |
| 후행조건 | -                                                            |
| 기존동작 | 1. 물류시스템운영자 또는 CSP시스템운영자가 Cache 솔루션의 주요지표 Hit Ratio 을 요청한다.<br />2. Cache 솔루션은 다음의 식을 기반으로 Hit Ratio 정보를 제공한다.<br />`hit ratio = Total number of cache hits / (Total Number of cache hits + Number of cache misses)` |
| 추가동작 | -                                                            |



### 2.2.8 UC-08 Cache알고리즘적용한다

| UC-08    | Cache알고리즘적용한다                                        |
| -------- | ------------------------------------------------------------ |
| 설명     | Cache 솔루션의 HitRatio 를 높이기 위한 알고리즘을 선택한다   |
| 행위자   | CSP시스템개발자                                              |
| 선행조건 | Hit Ratio 가 기존보다 좋은 Cache 알고리즘 X 존재             |
| 후행조건 | -                                                            |
| 기존동작 | 1. CSP시스템개발자는 Cache 솔루션에 새로운 Cache 알고리즘 X을 적용요청한다. |
| 추가동작 | -                                                            |



### 2.2.9 UC-9 Cache알고리즘추가한다

| UC-9     | Cache알고리즘추가한다                                        |
| -------- | ------------------------------------------------------------ |
| 설명     | Cache 솔루션의 HitRatio 를 높이기 위한 알고리즘을 추가한다   |
| 행위자   | CSP시스템개발자                                              |
| 선행조건 | - UC-08 "Cache알고리즘적용한다"<br />- Hit Ratio 가 기존보다 좋은 Cache 알고리즘 X 존재하며 Cache 솔루션에는 추가되지 않은 상태 |
| 후행조건 | -                                                            |
| 기존동작 | 1. CSP시스템개발자는 Cache 알고리즘X 을 템플릿에 맞추어 Cache 솔루션에 추가 지시한다.<br />2. Cache 솔루션은 Cache 알고리즘X을 알고리즘 목록 중 하나로 등록한다. |
| 추가동작 | -                                                            |



![image-20220411030437412](C:\Users\u4rang\AppData\Roaming\Typora\typora-user-images\image-20220411030437412.png)

### 2.2.10 UC-10 LocalMode설정한다

| UC-10    | LocalMode설정한다                                            |
| -------- | ------------------------------------------------------------ |
| 설명     | 데이터가 읽기 전용 또는 특정 만료 빈도로 주기적으로 새로 고쳐질 수 있는 시나리오에 적합한 Local Mode를 설정한다. |
| 행위자   | 물류시스템개발자, CSP시스템개발자                            |
| 선행조건 | UC-06 CacheMode적용한다                                      |
| 후행조건 | -                                                            |
| 기존동작 | [작성예정]                                                   |
| 추가동작 | -                                                            |



### 2.2.11 UC-11 PartitionedMode설정한다

| UC-11    | PartitionedMode설정한다                                      |
| -------- | ------------------------------------------------------------ |
| 설명     | 데이터 세트가 작고 업데이트가 자주 발생하지 않는 시나리오에 적합한 Partitioned Mode를 설정한다 |
| 행위자   | 물류시스템개발자, CSP시스템개발자                            |
| 선행조건 | UC-06 CacheMode적용한다                                      |
| 후행조건 | -                                                            |
| 기존동작 | [작성예정]                                                   |
| 추가동작 | -                                                            |



### 2.2.12 UC-12 ReplicatedMode설정한다

| UC-12    | ReplicatedMode설정한다                                       |
| -------- | ------------------------------------------------------------ |
| 설명     | 대규모 데이터 세트로 작업하고 업데이트가 빈번한 시나리오에 적합한 Replicated Mode를 설정한다 |
| 행위자   | 물류시스템개발자, CSP시스템개발자                            |
| 선행조건 | UC-06 CacheMode적용한다                                      |
| 후행조건 | -                                                            |
| 기존동작 | [작성예정]                                                   |
| 추가동작 | -                                                            |



### 2.2.13 UC-13 Cache노드관리한다

| UC-13    | Cache노드관리한다                                            |
| -------- | ------------------------------------------------------------ |
| 설명     | Cache 솔루션이 Cluster 인 경우 노드를 추가하거나 삭제한다    |
| 행위자   | 물류시스템운영자, CSP시스템운영자                            |
| 선행조건 | Cache솔루션이 Cluster 형태이다                               |
| 후행조건 | -                                                            |
| 기존동작 | 1. 물류시스템운영자 또는 CSP시스템운영자가 Cache 솔루션에 노드 추가 및 삭제를 요청한다.<br />2. Cache 솔루션은 노드의 추가 삭제 작업을 수행한다. |
| 추가동작 | -                                                            |





## 2.2 품질 요구사항

> 아키텍처 품질속성별 명확하고 정량적인 달성목표 작성

| ID     | 품질속성   | 품질속성 상세화                         | 품질속성 시나리오                                            | 중요도 | 난이도 | System Feature ID |
| ------ | ---------- | --------------------------------------- | ------------------------------------------------------------ | ------ | ------ | ----------------- |
| QAS-01 | 성능       | Cache 솔루션의 응답속도                 | 물류시스템사용자 김경환 프로가 3,600 초 이내 조회한 운송장번호 A 를 다시 조회하면 <br/>Cache 솔루션에 저장된 데이터를 기반으로 3,000 ms 이내 운송장번호 A 를 확인한다. | 상     | 상     |                   |
| QAS-02 | 성능       | Cache 솔루션의 데이터 저장 및 응답 속도 | 물류시스템사용자 김경환 프로가 Cache에 없는 운송장번호 A의 배송상태를 조회하면, <br/>운송장번호 A의 배송상태는 Cache에 저장되고 김경환 프로에게 배송상태를 제공한다.<br/>3초 후, 김채이 프로가 운송장번호 A 를 조회하면 Cache에 저장된 <br/>운송장번호 A의 배송상태를 이용하여 3,000 ms 이내 배송상태를 제공한다.<br/> | 상     | 상     |                   |
| QAS-03 | 내구성     | Cache 솔루션의 응답보장                 | 100,000 명의 물류시스템사용자가 동시에 운송장번호와 배송상태를 조회하면 동일한 운송장번호와 <br/>그 배송상태는 Cache 솔루션에 저장된 데이터를 기반으로 응답하며 <br/>물류사용자는 100,000 명의 요청에 따른 물류시스템의 부하에 대해 인지하지 못한다.<br/> | 상     | 중     |                   |
| QAS-04 | 모듈성     | Cache 솔루션의 모듈화                   | Cache 알고리즘에 변경사항이 발생하면 변경의 범위는 해당 Cache 알고리즘에 한정되고 <br/>他 Cache 알고리즘은 변경에 영향을 받지 않는다. <br/> | 상     | 중     |                   |
| QAS-05 | 정확성     | Cache 솔루션의 데이터 정확성            | Cache 솔루션은 같은 키에 대해서는 같은 값을 유효 TTL 이내 99.999999999 % 제공한다. | 중     | 상     |                   |
| QAS-06 | 보안       | Cache 솔루션의 구간 암호화              | 물류시스템과 Cache 솔루션 간의 통신은 TLS 기반으로 암호화하여 통신한다. | 중     | 중     |                   |
| QAS-07 | 유지보수성 | Cache 솔루션의 유지보수 용이성          | Cache 솔루션에 새로운 Cache 알고리즘(LRU)을 추가하면 <br/>2개월 이내 고급개발자 4MM 로 개발한다.<br/> | 중     | 중     |                   |
| QAS-08 | 안정성     | Cache 솔루션의 정확성                   | Cache 솔루션은 배송상태의 키 A 인 캐시에 대해 배송상태 데이터를 물류시스템에게 99.999999999 % 제공한다. | 중     | 하     |                   |



### 2.2.1 QAS-01 Cache 솔루션의 응답속도

| QAS-01   | Cache 솔루션의 응답속도                                      |
| -------- | ------------------------------------------------------------ |
| 품질속성 | 성능                                                         |
| 설명     | 물류시스템사용자 김경환 프로가 3,600 초 이내 조회한 운송장번호 A 를 다시 조회하면 <br/>Cache 솔루션에 저장된 데이터를 기반으로 3,000 ms 이내 운송장번호 A 를 확인한다. |
| 자극     | 물류시스템사용자 김경환 프로                                 |
| 환경     | 물류시스템이 Cache 솔루션과 함께 정상적으로 서비스하는 중    |
| 반응     | Cache 솔루션이 Cached 데이터를 물류시스템에 전송한다.        |
| 측정     | [요청 처리 속도] = [물류시스템사용자 김경환 프로가 요청한 시각] - [물류시스템WEB서버의 Access Log에 Response 기록한 시각] |
| 제약     | [트래픽 처리 속도] < 3,000 ms                                |



### 2.2.2 QAS-02 Cache 솔루션의 데이터 저장 및 응답 속도

| QAS-02   | Cache 솔루션의 데이터 저장 및 응답 속도                      |
| -------- | ------------------------------------------------------------ |
| 품질속성 | 성능                                                         |
| 설명     | 물류시스템사용자 김경환 프로가 Cache에 없는 운송장번호 A의 배송상태를 조회하면, <br/>운송장번호 A의 배송상태는 Cache에 저장되고 김경환 프로에게 배송상태를 제공한다.<br/>3초 후, 김채이 프로가 운송장번호 A 를 조회하면 Cache에 저장된 <br/>운송장번호 A의 배송상태를 이용하여 3,000 ms 이내 배송상태를 제공한다.<br/> |
| 자극     | 물류시스템사용자 김경환 프로                                 |
| 환경     | 물류시스템이 Cache 솔루션과 함께 정상적으로 서비스하는 중    |
| 반응     | 물류시스템이 조회한 운송장번호A를 물류시스템사용자 김경환 프로에게 전달하고, Cache 솔루션에 운송장번호A를 저장하도록 한다. |
| 측정     | [트래픽 처리 속도] < 3,000 ms                                |
| 제약     | -                                                            |



### 2.2.3 QAS-03 Cache 솔루션의 응답보장

| QAS-03   | Cache 솔루션의 응답보장                                      |
| -------- | ------------------------------------------------------------ |
| 품질속성 | 내구성                                                       |
| 설명     | 100,000 명의 물류시스템사용자가 동시에 운송장번호와 배송상태를 조회하면 동일한 운송장번호와 <br/>그 배송상태는 Cache 솔루션에 저장된 데이터를 기반으로 응답하며 <br/>물류사용자는 100,000 명의 요청에 따른 물류시스템의 부하에 대해 인지하지 못한다.<br/> |
| 자극     | 100,000 명의 물류시스템사용자                                |
| 환경     | 물류시스템이 Cache 솔루션과 함께 정상적으로 서비스하는 중    |
| 반응     | 물류시스템은 운송장번호와 배송상태 정보에 대해 1차적으로 Cache 솔루션을 확인하고, 없으면 Database와 API를 조회한다. 조회된 데이터는 물류시스템 사용자에게 전달하고, Cache 솔루션에 전달하여 저장하도록 지시한다. |
| 측정     | `[요청 처리 속도] = [물류시스템WEB서버의 Access Log에 Request기록한 시각] - [물류시스템WEB서버의 Access Log에 Response 기록한 시각]<`br />`hit ratio = Total number of cache hits / (Total Number of cache hits + Number of cache misses` |
| 제약     | -                                                            |



### 2.2.4 QAS-04 Cache 솔루션의 모듈화

| QAS-04   | Cache 솔루션의 모듈화                                        |
| -------- | ------------------------------------------------------------ |
| 품질속성 | 모듈성                                                       |
| 설명     | Cache 알고리즘에 변경사항이 발생하면 변경의 범위는 해당 Cache 알고리즘에 한정되고 <br/>他 Cache 알고리즘은 변경에 영향을 받지 않는다. <br/> |
| 자극     | Hit Ratio 가 기존보다 좋은 Cache 알고리즘 X 존재             |
| 환경     | 물류시스템이 Cache 솔루션과 함께 정상적으로 서비스하는 중    |
| 반응     | Cache 솔루션은 Cache 알고리즘 X를 추가한다. 추가 시, 他 알고리즘에 영향을 미치지 않는다. |
| 측정     | Cache 알고리즘 X가 추가되어도 기존 알고리즘은 기존과 동일하게 동작한다. |



### 2.2.5 QAS-05 Cache 솔루션의 데이터 정확성

| QAS-05   | Cache 솔루션의 데이터 정확성                                 |
| -------- | ------------------------------------------------------------ |
| 품질속성 | 정확성                                                       |
| 설명     | Cache 솔루션은 같은 키에 대해서는 같은 값을 유효 TTL 이내 99.999999999 % 제공한다. |
| 자극     | 물류시스템사용자 김경환 프로                                 |
| 환경     | 물류시스템이 Cache 솔루션과 함께 정상적으로 서비스하는 중    |
| 반응     | 같은 키을 가진 데이터에 대해 일관된 결과를 제공한다.         |
| 측정     | `hit ratio = Total number of cache hits / (Total Number of cache hits + Number of cache misses` |



### 2.2.6 QAS-06 Cache 솔루션의 구간 암호화

| QAS-06   | Cache 솔루션의 구간 암호화                                   |
| -------- | ------------------------------------------------------------ |
| 품질속성 | 보안                                                         |
| 설명     | 물류시스템과 Cache 솔루션 간의 통신은 TLS 기반으로 암호화하여 통신한다. |
| 자극     | 보안부서의 통신구간 암호화 강제화                            |
| 환경     | 물류시스템이 Cache 솔루션과 함께 정상적으로 서비스하는 중    |
| 반응     | TLS 기반으로 물류시스템과 Cache 솔루션은 네트워크 통신한다.  |
| 측정     | TLS1.3 기반 통신구간 암호화                                  |



### 2.2.7 QAS-07 Cache 솔루션의 유지보수 용이성

| QAS-07   | Cache 솔루션의 유지보수 용이성                               |
| -------- | ------------------------------------------------------------ |
| 품질속성 | 유지보수성                                                   |
| 설명     | Cache 솔루션에 새로운 Cache 알고리즘(LRU)을 추가하면 <br/>2개월 이내 고급개발자 4MM 로 개발한다.<br/> |
| 자극     | 새로운 Cache 알고리즘(LRU) 추가                              |
| 환경     | 물류시스템이 Cache 솔루션과 함께 정상적으로 서비스하는 중    |
| 반응     | Cache 솔루션은 Cache 알고리즘(LRU) 를 추가한다. 추가 시, 他 알고리즘에 영향을 미치지 않는다. |
| 측정     |                                                              |



### 2.2.8 QAS-08 Cache 솔루션의 정확성

| QAS-08   | Cache 솔루션의 정확성                                        |
| -------- | ------------------------------------------------------------ |
| 품질속성 | 안정성                                                       |
| 설명     | Cache 솔루션은 배송상태의 키 A 인 캐시에 대해 배송상태 데이터를 물류시스템에게 99.999999999 % 제공한다. |
| 자극     | 물류시스템사용자 김경환 프로                                 |
| 환경     | 물류시스템이 Cache 솔루션과 함께 정상적으로 서비스하는 중    |
| 반응     | Cache에 저장된 배송상태의 키 A에 대해 유효 TTL 동안 데이터를 제공한다. |
| 측정     | `hit ratio = Total number of cache hits / (Total Number of cache hits + Number of cache misses` |



## 2.3 아키텍처 제약사항

> 품질속성에 영향을 미치는 아키텍처 제약 사항 기술

| ID    | 설명                                              |
| ----- | ------------------------------------------------- |
| CR-01 | OS는 Linux 중 Ubuntu LTS 20.04 버전으로 한정한다. |
| CR-02 | 언어는 Golang 으로 구현한다.                      |



# 3. 아키텍처 설계문제 분석

## 3.1 아키텍처 드라이버

> 설계 핵심요구사항 선정, 선정사유 설명



## 3.2 아키텍처 문제분석 및 설계전략

> 아키텍처 문제분석과 그에 대응할 설계전략 수립
>
> 아키텍처 스타일 및 패턴 활용 계획
>
> 뷰/절차 활용 계획



# 4. 아키텍처 상세설계

## 4.1 개념 아키텍처

> 전체 아키텍처를 조망할 수 있는 개념도, 뷰 설명
>
> 큰 모듈 및 기능 간의 크기, 위치, 관계가 한 눈에 파악이 되도록 작성
>
> 설계전략이 어디에 적용되는지 이해하기 쉽게 설명



## 4.2 실행 아키텍처

> 모듈이 실제 작동하는 구조를 기술함
>
> 큰 모듈내 세부 모듈간 크기, 위치, 관계를 상세히 작성
>
> 사용하는 SW Stack을 명확히 작성
>
> SW가 동작하는 HW 환경은 검증/구현 대상에 대해 명확히 작성



## 4.3 설계문제 분석 및 평가

> 요구사항과 설계결과 Align 및 누락여부 확인
>
> 시나리오 기반(ATAM 등), 의사결정 기반, 전문가 경험기반 분석 및 평가 등 활용



# 5. 아키텍처 구현 및 검증

## 5.1 아키텍처 검증 시나리오

> 아키텍처 요구사항 및 목표 달성을 확인할 수 있는 검증 시나리오 작성
>
> 시뮬레이션 기반 검증(PoC/BMT 등)



## 5.2 아키텍처 구현

> 검증 시나리오가 작동하는 것을 확인할 수 있는 아키텍처 구현



## 5.3 아키텍처 구현 결과 검증

> 아키텍처 요구사항 및 목표 달성요부 확인
>
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



### 1.3 Cache의 유형

Cache의 유형은 크게 클라이언트 측과 서버 측으로 나눈다.

![image-20220410235249569](https://raw.githubusercontent.com/u4rang/save-image-repo/main/img/image-20220410235249569.png)

클라이언트 측은 웹 브라우저가 있다.

서버 측은 웹 서버와 역방향 프록시 캐시가 있다.

과제에서 다루는 Cache의 유형은 웹 서버이다.
