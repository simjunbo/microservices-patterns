# 트랜잭션 관리 : 사가
- 마이크로서비스 아키텍처에서도 단일 서비스 내부의 트랜잭션은 ACID가 보장하지만, 여러 서비스의 데이터를 업데이트하는 트랜잭션은 구현하기 까다로움
- 분산 트랜잭션 기법은 요즘 잘 안맞음
- 사가를 써야 하는데 ACD(원자성, 일관성, 지속성)만 지원하고 격리가 안되서 영향을 줄일 수 있는 설계 기법을 적용해야 됨
  - 코레오그래피 : 중앙 제어 장치 없이 참여자가 각자 서로 이벤트를 교환하는 방법
  - 오케스트레이션 : 중앙 제어 장치가 참여자가 할 일을 지시하는 방법

## 4.1 마이크로서비스 아키텍처에서의 트랜잭션 관리
- 스프링은 @Transactional만 붙이면 개발자가 간편하게 트랜잭셔널 비즈니스 로직 작성 가능
- 단일 DB를 접근하는 모놀리식 애플리케이션의 트랜잭션 관리는 어렵지 않지만 다중 DB 사용하는 모놀리식이나 MSA는 트랜잭션 관리가 어려움

### 4.1.1 분산 트랜잭션의 필요성
- <img width="638" alt="스크린샷 2022-05-01 오후 2 22 29" src="https://user-images.githubusercontent.com/7076334/166133415-20017fcc-6a58-44e3-b270-35998f4676ad.png"> 

  - 소비자 : 주문 가능한 소비자인지 확인
  - 주방 서비스 : 주문 내역 확인
  - 회계 서비스 : 신용 카드 승인
  - 주문 : 정상적인 경우 주문 생성
    - MSA에서는 일관성을 유지할 수 있는 수단을 강구해야 됨 

### 4.1.2 분산 트랜잭션(2PC)의 문제점
- 스프링에서는 ChainedTransactionManager, JtaTransactionManager 제공
- XA(분산 트랜잭션 관리 표준)는 2단계 커밋을 이용하여 전체 트랜잭션에 대해 커밋 아니면 롤백 보장
  - SQL DB 및 메시지 브로커(일부) 적용 가능

- 문제점
  - NoSQL(MongoDB, 카산드라)과 현대 메시지 브로커(RabbitMQ, 카프카)는 분산 트랜잭션 지원하지 않음
  - 동기 IPC 형태라 가용성이 떨어짐 (메시지 브로커 얘기 하는듯)

### 4.1.3 데이터 일관성 유지 : 사가 패턴
- 사가는 비동기 메시징을 이용하여 편성한 일련의 로컬 트랜잭션 (ACID 형태로 서비스별 데이터를 업데이트)
  - 비동기 메시징은 하나 이상의 참여자가 일시 불능 상태인 경우에도 사가 전체 단계를 실행시킬 수 있다.

- 사가와 ACID 트랜잭션 차이
  - 1) ACID 트랜잭션에 있는 격리성(I)이 사가는 없음
  - 2) 사가는 로컬 트랜잭션마다 변경분을 커밋하므로 보상 트랜잭션을 걸어 롤백을 해야 함 

- 예제 (6개의 로컬 트랜잭션)
  - ![image](https://user-images.githubusercontent.com/7076334/166135421-3d532809-fbc6-41ad-b36d-11cf836b4ce7.png)
    - 1) 주문 서비스: 주문을 APPROVAL_PENDING 상태로 생성합니다.
    - 2) 소비자 서비스: 주문 가능한 소비자인지 확인합니다.
    - 3) 주방 서비스: 주문 내역을 확인하고 티켓을 CREATE_PENDING 상태로 생성합니다.
    - 4) 회계 서비스: 소비자 신용카드를 승인합니다.
    - 5) 주방 서비스: 티켓 상태를 AWAITING_ACCEPTANCE로 변경합니다.
    - 6) 주문 서비스: 주문 상태를 APPROVED로 변경합니다.
      - 로컬 트랜잭션이 완료되면 메시지를 발행하여 다음 사가 단계 트리거
      - 도중에 에러가 발생되면 어떻게 롤백 시킬 수 있을까? 

#### 사가는 보상 트랜잭션으로 변경분을 롤백한다.
- ACID 트랜잭션은 롤백 쉽게 가능. 하지만 사가는 단계마다 로컬 DB에 변경분을 커밋하므로 자동 롤백 불가능
  - 롤백에 대한 보상 트랜잭션을 미리 작성해야 함 


- ![image](https://user-images.githubusercontent.com/7076334/166135674-5d338f56-02f7-4166-a030-cf11ff4028ec.png)
  - 보상은 단계별 로컬 트랜잭션을 역순으로 보상 실행

- 주문 생성 사가의 보상 트랜잭션
  - <img width="629" alt="image" src="https://user-images.githubusercontent.com/7076334/166135776-1d2a0d33-167a-4b46-9bca-cce0ca192d04.png">

    - 읽기 전용 단계나, 항상 성공하는 단계는 보상 트랜잭션이 필요 없음 
    - 1~3 단계 : 보상 트랜잭션 (실패 가능성이 있는 단계 다음 / 응답 기준으로 생각함)
    - 4 단계 : 피봇 트랜잭션 (절대 실패하지 않는 단계 다음 / 최종 종착지로 이해함)
    - 5~6 단계 : 재시도 가능 트랜잭션 (항상 성공) 


## 4.2 사가 편성
- 사가는 단계를 편성하는 로직으로 구성됨
  - 코레오그래피 : 의사 결정과 순서화를 사가 참여자에게 맡김. 사가 참여자는 주로 이벤트 교환 방식으로 통신
  - 오케스트레이션 : 사가 편성 로직을 사가 오케스트레이터에 중앙화함. 사가 오케스트레이터는 사가 참여자에게 커맨드 메시지를 보내 수행할 작업을 지시

### 4.2.1 코레오그래피 사가
- 사가 참여자가 서로 이벤트를 구독해서 그에 따라 반응

#### 주문 생성 사가 구현 : 코레오그래피 스타일
- 정상 케이스 
  - ![image](https://user-images.githubusercontent.com/7076334/166149979-373210a6-c1f1-4a9e-a458-8f7d7cb2e33e.png)
    - 1.주문 서비스 : 주문을 APPROVAL_PENDING 상태로 생성 → 주문 생성 이벤트를 발행
    - 2.소비자 서비스 : 주문 생성 이벤트 수신 → 소비자가 주문을 할 수 있는지 확인 → 소비자 확인 이벤트를 발행
    - 3.주방 서비스 : 주문 생성 이벤트 수신 → 주문 내역 확인 → 티켓을 CREATE_PENDING 상태로 생성 → 티켓 생성됨 이벤트를 발행합니다.
    - 4.회계 서비스 : 주문 생성 이벤트 수신 → 신용카드 승인을 PENDING 상태로 생성합니다.
    - 5.회계 서비스 : 티켓 생성 및 소비자 확인 이벤트 수신 → 소비자 신용카드 과금 → 신용카드 승인됨 이벤트를 발행합니다.
    - 6.주방 서비스 : 신용카드 승인 이벤트 수신 → 티켓 상태를 AWAITING_ACCEPTANCE로 변경합니다.
    - 7.주문 서비스 : 신용카드 승인됨 이벤트 수신 → 주문 상태를 APPROVED로 변경 → 주문 승인됨 이벤트를 발행합니다.

- 실패 케이스
  - ![image](https://user-images.githubusercontent.com/7076334/166150438-833a4923-c758-4398-b31e-e8f09aff644f.png)
    - 1.주문 서비스: 주문을 APPROVAL_PENDING 상태로 생성 → 주문 생성 이벤트를 발행
    - 2.소비자 서비스: 주문 생성 이벤트 수신 → 소비자가 주문을 할 수 있는지 확인 → 소비자 확인 이벤트를 발행
    - 3.주방 서비스: 주문 생성 이벤트 수신 → 주문 내역 확인 → 티켓 상태를 CREATE_PENDING으로 생성 → 티켓 생성 이벤트를 발행
    - 4.회계 서비스: 주문 생성 이벤트 수신 → 신용카드 승인을 PENDING 상태로 생성
    - 5.회계 서비스: 티켓 생성 및 소비자 확인 이벤트 수신 → 소비자 신용카드 과금 → 신용카드 승인 실패 이벤트를 발행 (여기서 부터 다름)
    - 6.주방 서비스: 신용카드 승인 실패 이벤트 수신 → 티켓 상태를 REJECTED로 변경
    - 7.주문 서비스: 신용카드 승인 실패 이벤트 수신 → 주문 상태를 REJECTED로 변경
      - 5a, 5b 의 상태를 둘다 만족 시켜야 되는데 이벤트 수신 할때마다 체크 하면 되나? 

- 사가에서 발행/구독 방식으로 통신하면 어떤 문제점이 있을까?

#### 확실한 이벤트 기반 통신
- 코레오그래피 방식의 두 가지 통신 이슈 고려
  - 1) DB를 업데이트하는 작업과 이벤트를 발해앟는 작업은 원자적으로 처리. 앞장에서 본 트랜잭셔널 메시징을 사용해야 함
  - 2) 사가 참여자는 자신이 수신한 이벤트와 자신이 가진 데이터를 연관 지을 수 있어야 함
    - 데이터를 매핑할 수 있도록 다른 사가 참여자가 상관관계 ID가 포함된 이벤트를 발행 

#### 코레오그래피 사가의 장단점
- 장점
  - 단순함 : 비즈니스 객체를 생성, 수정, 삭제할 때 서비스가 이벤트를 발행
  - 느슨한 결합 : 참여자는 이벤트를 구독할 뿐 서로를 직접 알지 못함

- 단점
  - 이해하기 어렵다 : 여러 서비스에 구현 로직이 흩어져 있음(오케스트레이션 사가가 아니기 때문에). 사가의 동작이 어떻게 되는지 이해하기 어려운 편
  - 서비스 간 순환 의존성 : 참여자가 서로 이벤트를 구독하는 특성상, 순환 의존성 발생하기 쉬움(잠재적 설계 취약점)
  - 단단히 결합될 위험성 : 참여자는 자신에게 영향을 미치는 이벤트를 모두 구독해야 함.


### 4.2.2 오케스트레이션 사가
- 사가 오케스트레이터는 커맨드/비동기 응답 상호 작용을 하며 참여자와 통신

#### 주문 생성 사가 구현 : 오케스트레이션 스타일
- 정상 케이스
  - ![image](https://user-images.githubusercontent.com/7076334/166154601-cf36dbaf-9426-4399-9815-63181052fe70.png)
    - 1.사가 오케스트레이터가 소비자 확인 커맨드를 소비자 서비스에 전송합니다.
    - 2.소비자 서비스는 소비자 확인 메시지를 응답합니다.
    - 3.사가 오케스트레이터는 티켓 생성 커맨드를 주방 서비스에 전송합니다.
    - 4.주방 서비스는 티켓 생성 메시지를 응답합니다.
    - 5.사가 오케스트레이터는 신용카드 승인 메시지를 회계 서비스에 전송합니다.
    - 6.회계 서비스는 신용카드 승인됨 메시지를 응답합니다.
    - 7.사가 오케스트레이터는 티켓 승인 커맨드를 주방 서비스에 전송합니다.
    - 8.사가 오케스트레이터는 주문 승인 커맨드를 주문 서비스에 전송합니다.
      - 마지막은 직접 업데이트 해서 승인 처리해도 되지만, 일관성 차원에서 주문 서비스가 다른 참여자인 것처럼 취급 

#### 사가 오케스트레이터를 상태 기계로 모델링
- 상태 기계는 **상태**와 이벤트에 의해 트리거되는 **상태 전이**로 구성됨
- 전이가 발생할 때마다 액션이 일어나는데, 사가의 액션은 사가 참여자를 호출하는 작용
  - 상태 간 전이는 사가 참여자가 로컬 트랜잭션을 완료하는 시점에 트리거 되고, 로컬 트랜잭션의 상태와 결과에 따라 상태 전이를 어떻게 하고 어떤 액션을 취할지 결정

- 주문 생성 상태 기계 모델링 (설계, 구현, 테스트를 더 쉽게 진행 가능)
  - ![image](https://user-images.githubusercontent.com/7076334/166170485-74851403-ee97-4fc5-a214-974372ba490d.png)
    - 소비자 확인 : 초기 상태. 사가는 소비자 서비스가 주문 가능한 소비자인지 확인할 때까지 기다림
    - 티켓 생성 : 사가는 티켓 생성 커맨드에 대한 응답을 기다림
    - 신용카드 승인 : 회계 서비스가 소비자 신용카드를 승인할 때까지 기다림
    - 주문 승인됨 : 사가는 성공적으로 완료되었음을 나타내는 최종 상태
    - 주문 거부됨 : 참여자 중 하나가 주문을 거부했음을 나타내는 최종 상태
      - 결국 다양한 상태 전이를 거치며 결국 주문 승인됨, 주문 거부됨 두 상태 중 한쪽 발생

#### 사가 오케스트레이션과 트랜잭셔널 메시징
- 오케스트레이션 사가는 DB를 업데이트하는 서비스와 메시지를 발행하는 서비스가 단계마다 있음
  - ex)
  - 주문 서비스 (주문 및 주문 생성 하가 오케스트레이터를 생성) -> 우선순위 1번 사가 참여자 메시지 보냄 -> 1번 사가 참여자는 자신의 DB 업데이트(티켓 생성) 후 응담 메시지 보냄 -> 응답을 받고 주문 서비스는 사가 오케스트레이터 상태를 업데이트한 후 커맨드 메시지를 다음 사가 참여자에게 보냄
    - 트랜잭셔널 메시지(3장)를 사용해 DB 업데이트와 메시지 발행 작업을 원자적으로 처리 


#### 오케스트레이션 사가의 장단점
- 장점
  - 의존 관계 단순화 : 순환 의존 발생하지 않음. (오케스트레이터만 참여자를 호출함)
  - 낮은 결합도 : 각 서비스는 오케스트레이터가 호출하는 API를 구현할 뿐, 사가 참여자가 발행하는 이벤트는 몰라도 됨
    - 오케스트레이터를 통해서만 발행되니까로 이해했음 (수신, 응답 모두) 
  - 관심사를 더 분리하고 비즈니스 로직을 단순화 : 사가 편성 로직이 사가 오케스트레이터 한 곳에만 있으므로 도메인 객체는 더 단순해지고 자신이 참여한 사가에 대해서 알지 못함 (참여자는 그냥 응답 queue로 보내기만 하면 되니까)


- 단점
  - 똑똑한 오케스트레이터 하나가 깡통 서비스에 일일이 할 일을 지시하는 모양새가 될 수도 있음
    - (해결) 오케스트레이터가 순서화만 담당하고 여타 비즈니스 로직은 갖고 있지 않돌독 설계하면 됨 
  
## 4.3 비격리 문제 처리
- ACID의 격리성(I)은 동시에 실행 중인 여러 트랜잭션의 결과가 순서대로 실행된 결과와 동일함을 보장
  - 격리가 안되면 DB 용어로 비정상(anomaly)이 나타날 가능성이 있음
  - 실제도 성능 향상을 위해 격리 수준을 낮추는 경우가 흔하다. 

- 사가는 격리성이 빠져 있어서 두가지 문제를 야기 (트랜잭션의 isoloation 이랑 비슷한듯)
  - 1) 사가가 실행 중에 접근하는 데이터를 도중에 다른 사가가 바꿀 수 있음
  - 2) 사가가 업데이트를 하기 이전 데이터를 다른 사가가 읽을 수 있어 데이터 일관성이 깨질 수 있음

### 4.3.1 비정상 개요
- 비격리로 인한 비정상 종류
  - 소실된 업데이트 : 한 사가의 변경분을 다른 사가가 미처 못 읽고 덮어 씀
  - 더티 읽기 : 사가 업데이트를 하지 않은 변경분을 다른 트랜잭션이나 사가가 읽습니다.
  - 퍼지/반복 불가능한 읽기 : 한 사가의 상이한 두 단계가 같은 데이터를 읽어도 결과가 달라지는 현상. (다른 사가가 그 사이 업데이트를 했기 떄문)

#### 소실된 업데이트
- 한 사가의 변경분을 다른 사가가 덮어 쓸때
  - 1) 주문 생성 사가 첫번째 단계에서 주문 생성
  - 2) 사가 실행 중 주문 취소 사가가 주문을 취소
  - 3) 주문 생성 사가 마지막 단계에서 주문을 승인
    - 고객은 자신이 주문 취소한 음식을 배달 받게 됨 

#### 더티 읽기
- 한 사가가 업데이트 중인 데이터를 다른 사가가 읽을 때

- FTGO 애플리케이션의 주문 취소 사가
  - 소비자 서비스 : 신용 잔고를 늘림 (취소 분 만큼 늘리는걸로 이해함)
  - 주문 서비스 : 주문을 취소 상태로 변경
  - 배달 서비스 : 배달을 취소

- 주문 취소 사가와 주문 생성 하가의 실행이 서로 겹칠 때 (배달 취소하기 늦어서 주문 취소 사가가 롤백되는 경우, 정상 주문 들어간 경우)
  - 1) 주문 취소 사가 : 신용 잔고를 늘림
  - 2) 주문 생성 사가 : 신용 잔고를 줄임 (다른 주문을 한거겠지 ?!)
  - 3) 주문 취소 사가(보상) : 신용 잔고를 줄이는 보상 트랜잭션 가동  (금액이 -가 될 수도 있다.)
    - 현재 한도 : 500
      - 1) 취소(500) / 현재 한도 : 1,000
      - 2) 주문(1,000) / 현재 한도 : 0
      - 3) 취소 보상 (500) / 현재 한도 : -500 

### 4.3.2 비격리 대책
- 개발자는 비격리로 인한 비정상을 방지하고 비즈니스에 미치는 영향을 최소화하는 방향으로 사가를 작성할 의무가 있음
  - 시맨틱 락 : 애플리케이션 수준의 락
  - 교환적 업데이트 : 업데이트 작업은 어떤 순서로 실행해도 되게금 설계
  - 비관적 관점 : 사가 단계 순서를 재조정하여 비즈니스 리스크를 최소화
  - 값 다시 읽기 : 데이터를 덮어 쓸 때 그 전에 변경된 내용은 없는지 값을 다시 읽고 확인하여 더티 쓰기 방지
  - 버전 파일 : 순서를 재조정할 수 있게 업데이트 기록 (애플리케이션 수준 낙관적 락?)
  - 값에 의한 : 요청별 비즈니스 위험성을 기준으로 동시성 메커니즘을 동적 선택

#### 사가의 구조
- 사가는 다음 세 가지 트랜잭션으로 구성됨
  - 보상 가능 트랜잭션 : 보상 트랜잭션으로 롤백 가능한 트랜잭션 (1~3)
    -  verifyConsumerDetails는 읽기 전용이라 따로 보상 트랜잭션 필요 없음
  - 피봇 트랜잭션 : 사가의 진행/중단 지점. 최종 보상 가능 트랜잭션 또는 최초 재시도 가능 트랜잭션 (4)
  - 재시도 가능 트랜잭션 : 피봇 트랜잭션 직후의 트랜잭션. 반드시 성공 (5~6)
    - <img width="629" alt="image" src="https://user-images.githubusercontent.com/7076334/166135776-1d2a0d33-167a-4b46-9bca-cce0ca192d04.png">

#### 대책 : 시맨틱 락
- 보상 가능 트랜잭션이 생성/수정하는 레코드에 무조건 플래그를 세팅하는 대책 (커밋 전이라 변경될 수 있다는 표시)
  - 다른 트랜잭션이 레코드에 접근하지 못하게 락을 걸어 놓거나, 경고처리 
  - 재시도 가능 트랜잭션(사가 완료) or 보상 트랜잭션(사가 롤백)에 의해 해제 

- Order 예제
  - PENDING 상태로 시맨틱 락 구현
    - APPROVED or REJECTED 가 최종 상태
    - cancelOrder() 를 통해 주문 취소하려면?
      - 1) 아직 최종 상태 전이기 때문에 cancelOrder() 실패 처리 (재시도 로직까지 고려해야되서 클라 복잡해짐)
      - 2) 락이 해제될 때까지 cancelOrder()를 블로킹 (애플리케이션에서 락을 관리해야됨)

#### 대책 : 교환적 업데이트
- 어떤 순서로도 실행 가능하게 설계하면 소실된 업데이트 문제를 방지
  - ex) debit(), credit() 복식부기!!! 

- 보상 트랜잭션이 계좌 사가를 롤백시켜야 하는 상황
  - 인출 -> 입금 (롤백)
  - 입금 -> 인출 (롤백)


#### 대책 : 비관적 관점
- 더티 읽기로 인한 비즈니스 리스크를 최소화하기 위해 사가 단계의 순서를 재조정

- 앞에서 얘기한 FTGO 애플리케이션의 주문 취소 사가
  - 소비자 서비스 : 신용 잔고를 늘림 (취소 분 만큼 늘리는걸로 이해함)
  - 주문 서비스 : 주문을 취소 상태로 변경
  - 배달 서비스 : 배달을 취소

- 주문 취소 사가 재조정
  - 주문 서비스: 주문을 취소 상태로 변경
  - 배달 서비스: 배달을 취소
  - 회계 서비스: 신용 잔고를 늘림 
    - 신용 잔고는 재시도 가능 트랜잭션에서 증가하므로 더티 읽기 가능성이 사라짐 (주문 생성을 다시 못하는 시점으로 이해)

#### 대책 : 값 다시 읽기
- 사가가 레코드를 업데이트하기 전에 값을 다시 읽어 값이 변경되지 않았는지 확인
  - 값을 다시 읽을 때 변경이 있으면 사가를 중단하고 나중에 재시작 (낙관적 오프라인 락 패턴) 

#### 대책 : 버전 파일
- 레코드에 수행한 작업을 하나하나 기록하는 대책

- 순서가 안 맞는 요청을 정상 처리하려면, 작업이 도착하면 기록해 두었다가 정확한 순서대로 실행하면 됨
  - 주문 생성사가와 주문 취소 사가와 동시에 실행된다면?
  - 회계 서비스는 일단 승인 취소 요청을 기록하고 나중에 신용카드 승인 요청이 도착하면 이미 승인 취소 요청이 접수된 상태이니 승인 작업을 생략
    - 만약 승인 요청이 먼저 도착한다면 ?! 승인 취소를 무시해도 되나?

#### 대책 : 값에 의한
- 비즈니스 위험성을 기준으로 동시성 메커니즘을 선택
  - 위험성이 낮은 요청은 위에서 설명한 사가 대책을 사용
  - 위험성이 큰 요청은(돈이 오가는) 분산 트랜잭션을 사용
    - 결국 금액은 RDB를 사용하라는 얘기인가 ?!!

## 4.4 주문 서비스 및 주문 생성 사가 설계
- ![image](https://user-images.githubusercontent.com/7076334/166251928-f15c7e24-b4b6-42a1-93ce-64c500b41ba9.png)
  - CreateOrderSaga : OrderService, Order, 주문 생성 사가를 오케스트레이션
    - KitchenServiceProxy, OrderServiceProxy : 사가 오케스트레이터는 사가 참여자 프록시 클래스를 거쳐 참여자에게 커맨드 메시지 전달 
  - OrderService(참여자), Order, OrderRepository : 핵심 비즈니스 로직
  - OrderCommandHandlers : OrderService를 호출하여 커맨드 메시지를 처리하는 어댑터

### 4.4.1 OrderService 클래스
- ![image](https://user-images.githubusercontent.com/7076334/166253760-ea107576-498b-48f8-91a8-c96b619ef340.png)
- ![image](https://user-images.githubusercontent.com/7076334/166253554-e322237c-9f77-436e-b2e9-116edb281732.png)
  - OrderService는 주문 생성/관리를 담당하는 서비스 API 계층이 호출하는 도메인 서비스
  - Order를 생성/수정하고, OrderRepository를 통해 Order를 저장
  - SagaManager를 통해 CreateOrderSaga 생성
    - SagaManager는 이벤추에이트에서 기본 제공되는 오케스트레이터와 참여자를 작성하는 클래스


### 4.4.2 주문 생성 사가 구현
- CreateOrderSaga : CreateOrderSagaState로 커맨드 메시지를 생성하고, 사가 참여자 프록시 클래스가 지정한 메시지 채널을 통해 참여자에게 메시지를 전달
- CreateOrderSagaState : 사가의 저장 상태, 커맨드 메시지를 생성
- 사가 참여자 프록시 클래스 : 프록시 클래스마다 커맨드 채널, 커맨드 메시지 탙입, 반환형으로 구성된 사가 참여자의 메시징 API를 정의

#### CreateOrderSaga 오케스트레이터
- CreateOrderSaga는 상태 기계를 구현한 클래스
  - 예제 183P
    - CreateOrderSaga 클래스의 핵심은 사가 데피니션(DSL 를 이용하여 정의)
    - invokeParticipant : 포워드 트랜잭션을 정의한 메서드
    - onReply : 성공 시 수행
    - withCompensation : 보상 시 수행

#### CreateOrderSagaState 클래스
- CreateOrderSagaState는 사가 인스턴스의 상태를 나타낸 클래스
- OrderServicer가 이 클래스의 인스턴스를 생성하고, 이벤추에이트 트램 사가 프레임워크가 이 인스턴스를 DB에 저장
- CreateOrderSagaState는 사가 참여자에게 보낼 메시지를 만드는 일
  - 예제 185P
    - CreateOrderSag는 CreateOrderSagaState를 호출하여 커맨드 메시지를 생성하고, 생성된 메시지를 KitchenServiceProxy 같은 클래스의 끝점으로 전달

#### KitchenServiceProxy 클래스
- 주방 서비스의 커맨드 메시지 3개의 끝점을 정의
  - create : 티켓 생성
  - confirmCreate : 생성 확인
  - cancel : 티켓 취소
  - ```
    public final CommandEndpoint<CreateTicket> create = CommandEndpointBuilder
          .forCommand(CreateTicket.class)
          .withChannel(KitchenServiceChannels.kitchenServiceChannel)
          .withReply(CreateTicketReply.class)
          .build();
    ```
    - 커맨드 타입, 커맨드 메시지의 목적지 채널, 예상 응답 타입 지정


- 프록시 클래스 사용 시 이점
  - 1) 프록시 클래스는 타입이 정해진 끝점을 정의하므로 엉뚱한 메시지가 서비스에 전달될 일은 거의 없음
  - 2) 프록시 클래스는 잘 정의된 서비스 호출 API라서 코드를 이해하고 테스트하기 쉬움

#### 이벤추에이트 트램 사가 프레임워크
- 이벤추에이트 트램 사가는 사가 오케스트레이터 및 사가 참여자를 모두 작성할 수 있는 프레임워크
  - ![image](https://user-images.githubusercontent.com/7076334/166459031-8d46b4ca-56d0-4093-bb5c-1a834fa8816f.png)

- OrderService가 사가를 생성할 때 이벤트 순서
  - ![image](https://user-images.githubusercontent.com/7076334/166461094-145075fe-dd4a-4f12-ab19-eefb27cd4dc4.png)
    - 1) OrderService는 CreateOrderSagaState를 생성
    - 2) OrderService는 SagaManager를 호출하여 사가 인스턴스를 생성
    - 3) SagaManager는 사가 데피니션의 첫 번째 단계를 실행
    - 4) CreateOrderSagaState를 호출하여 커맨드 메시지를 생성
    - 5) SagaManager는 커맨드 메시지를 사가 참여자(소비자 서비스)에게 보냄
    - 6) SagaManager는 사가 인스턴스를 DB에 저장

- SagaManager가 사가 참여자의 응답 메시지를 수신할 때 발생하는 이벤트
  - ![image](https://user-images.githubusercontent.com/7076334/166461073-b36fe5f9-fe86-4dc1-b432-7998030b3fd6.png)
    - 1) 이벤추에이트 트램은 소비자 서비스의 응답을 SagaManager에 전달
    - 2) SagaManager는 DB에서 사가 인스턴스를 조회
    - 3) SagaManager는 그 다음 사가 데피니션 단계를 실행
    - 4) CreateOrderSagaState를 호출하여 커맨드 메시지를 생성
    - 5) SagaManager는 커맨드 메시지를 사가 참여자(주방 서비스)에게 보냠
    - 6) SagaManager는 업데이트 사가 인스턴스를 DB에 저장

### 4.4.3 OrderCommandHandlers 클래스
- ![image](https://user-images.githubusercontent.com/7076334/166462146-7ee65851-292d-4151-9281-14683f181e75.png)
  - OrderCommandHandlers : 사가가 전송한 커맨드 메시지를 담당할 핸들러 메서드 정의
  - SagaCommandDispatcher : 커맨드 메시지를 적절한 핸들러 메서드에 보내고 응답을 반환하는 클래스

### 4.4.4 OrderServiceConfiguration 클래스
- 다양한 스프링 빈 정의
  - CommandHandlers, CommandHnaldersDispatcher, Proxy, Saga, Manager, Service

## 느낀점
- 이렇게 또 이론만 늘었다...
- 설명과 그림이 다른 장에 위치해서 보기 힘들다

## 참고
- https://hanamon.kr/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98%EC%9D%98-acid-%EC%84%B1%EC%A7%88/

- https://supawer0728.github.io/2018/03/22/spring-multi-transaction/
