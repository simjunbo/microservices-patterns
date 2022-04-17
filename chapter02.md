# 분해 전략
- 애플리케이션을 기능에 따라 어떻게 여러 서비스로 분해할 것이냐

## 2.1 마이크로서비스 아키텍처란 무엇인가?
- 서비스란 무엇이고, 그 크기는 어느 정도가 적당할까?
- 아키텍처가 중요한 이유는 소프트웨어의 품질 속성, 즉 '~성'으로 끝나는 지표가 아키텍처에 의해 결정되기 때문
- 마이크로서비스 아키텍처는 관리성, 테스트성, 배포성이 높은 애플리케이션을 구축하는 아킥테서 스타일 임


### 2.1.1 소프트웨어 아키텍처의 정의와 중요성
- 아키텍처란 무엇이고 왜 그렇게 중요한 것일까?

#### 소프트웨어 아키텍처의 정의
- 핵심은 애플리케이션 아키텍처가 여러 파트(엘리먼트)로의 분해와 이런 파트 간의 관계(연관성) 라는 것
- 분해가 중요한 이유
  - 업무와 지식을 분리함. 덕분에 전문 지식을 보유한 사람들이 함께 생산적으로 애플리케이션 작업을 할 수 있음
  - 소프트웨어 엘리먼트가 어떻게 상호 작용하는지 밝힘

#### 소프트웨어 아키텍처의 4+1 뷰 모델
- 4+1 모델은 소프트웨어 아키텍처를 바라보는 상이한 4가지 뷰를 정의
  - ![스크린샷 2022-04-10 오후 8 03 29](https://user-images.githubusercontent.com/7076334/162615030-656cb881-5d1d-49a0-bb25-fa9c01e54d25.png)

- 4뷰 외에도 뷰를 구동시키는 시나리오(+1)
  - 각 시나리오는 특정 뷰 내에서 얼마나 다양한 아키텍처 요소가 협동하여 요청을 처리하는지 기술

#### 아키텍처의 중요성
- 애플리케이션 요건
  - 1) 애플리케이션이 해야 할 일을 정의한 기능 요건
    - 유스 케이스나 사용자 스토리 포맷으로 기술하는데, 이 기능 요건과 아키텍처는 거의 무관
  - 2) 이른바 '~성'으로 끝나는 서비스 품질 요건
    - 아키텍처는 이 요건을 충족시킬 수 있게 설계해야 함 (중요)

### 2.1.2 아키텍처 스타일 개요
- 특정 아키텍처 스타일은 엘리먼트(컴포넌트)와 관계(커넥터)의 한정된 팔레트(사용 가능한 범위)를 제공하며, 이를 토대로 애플리케이션 아키텍처의 뷰를 정의할 수 있다.
- 애플리케이션은 대부분 아키텍처 스타일을 조합해서 사용

#### 계층화 아키텍처 스타일
- 소프트웨어 엘리먼트를 계층별로 구성하는전형적인 아키텍처 스타일
- 계층마다 명확히 정의된 역할을 분담하며, 계층 간 디펜던시는 아키텍처로 제한

- 3계층 아키텍처 (논리 뷰 적용 사례)
  - 표현(프레젠테이션) 계층 : 사용자 인터페이스 또는 외부 API가 구현된 계층
  - 비즈니스 로직 계층 : 비즈니스 로직이 구현된 계층
  - 영속화(퍼시스턴스) 계층 : DB 상호 작용 로직이 구현된 계층

- 계층화 아키텍처의 흠
  - 표현 계층이 하나뿐이다 (호출 시스템이 하나 뿐일까?)
  - 영속화 계층이 하나 뿐이다 (상호 작용하는 DB가 하나 뿐일까?)
  - 비즈니스 로직 계층을 영속화 계층에 의존하는 형태로 정의한다. (이런 디펜던시 때문에 DB 없이 비즈니스 로직을 테스트 하는 것은 불가능)

#### 육각형 아키텍처 스타일
- 육각형 아키텍처는 논리 뷰를 비즈니스 로직 중심으로 구성하는 계층화 아키텍처 스타일의 대안
- ![스크린샷 2022-04-10 오후 8 39 45](https://user-images.githubusercontent.com/7076334/162616289-d524e104-f383-42b0-9d11-6cdf5040862b.png)
  - 육각형 아키텍처의 가장 큰 장점은 비즈니스 로직에 있던 표현/데이터 접근 로직이 어댑터와 분리되었기 때문에 비즈니스 로직이 표현/데이터 접근 로직 어디에도 의존하지 않는다는 점
  - 이렇게 분리하면 비즈니스 로직만 따로 테스트하기 쉽고, 현대 애플리케이션 아키텍처를 좀 더 정확하게 반영할 수 있음

### 2.1.3 마이크로서비스 아키텍처는 일종의 아키텍처 스타일이다
- 모놀리식
  - 모놀리식 아키텍처는 구현 뷰를 단일 컴포넌트(하나의 실행 파일이나 WAR 파일)로 구성한 아키텍처 스타일
  - 육각형 아키텍처 방식으로 구성한 논리 뷰를 가질 수 있음

- 마이크로서비스 (76P)
  - 일종의 아키텍터 스타일로 구현 뷰를 다수의 컴포넌트(여러 실행 파일이나 WAR 파일)로 구성
  - 여기서 컴포넌트는 곧 서비스고, 각 서비스 자체 논리 뷰 아키텍처를 갖고 있음
  - 커넥터는 이런 서비스가 서로 협동할 수 있게 해주는 통신 프로토콜 
  - 마이크로서비스 아키텍처의 핵심 제약 조건은 서비스를 느슨하게 결합 (이 제약을 이해하려면 서비스의 뜻을 정확히 알아야함)

#### 서비스란 무엇인가?
- 서비스는 어떤 기능이 구현되어 단독 배포가 가능한 소프트웨어 컴포넌트
- 서비스 작업은 크게 커맨트(명령/CUD)와 쿼리(조회/R)로 나눔
- 마이크로서비스 아키텍처는 API를 우회하여 서비스에 접근하는 코드를 작성할 수 없어서 모듈성이 보장됨
- 마이크로서비스는 자체 아키텍처를 갖고 있기 때문에 기술 스택을 독자적으로 구축할 수 있지만, 대부분 육각형 아키텍처 형태를 취함

#### 느슨한 결합
- 서비스는 구현 코드를 감싼 API를 통해서만 상호 작용하므로 클라이언트에 영향을 끼치지 않고 서비스 내부 구현 코드를 바꿀 수 있음
- 느슨하게 결합된 서비스는 유지보수성, 테스트성을 높이고 애플리케이션 개발 시간을 단축하는 효과가 있음
- 서비스의 영속적 데이터는 반드시 프라이빗하게 유지 (DB 스키마 변경 시, 독립적으로 할 수 있음)
- DB를 공유하지 않기 떄문에 여러 서비스끼리 데이터를 쿼리하고 일관성을 유지하는 일이 더 복잡해지는 단점은 있음

#### 공유 라이브러리의 역할
- 마이크로서비스에서 공유 라이브러리를 사용하고픈 유횩에 빠지기 쉬운데, 서비스 코드 중복을 줄이는 것은 좋지만 의도치 않은 서비스간 결합도가 발생할 수 있음
- 공용 라이브러리 요건이 변경되면 관련 서비스를 일히 다시 빌드 배포
  - ex) DspId
- 거의 바뀌지 않는 기능은 라이브러리에 담아 쓴느것이 좋다.
  - ex) Amount?

#### 서비스는 별로 중요하지 않다
- 서비스 크기보단든 작은 팀이 가장 짧은 시간에, 다른 팀과 협동하는 부분은 최소로 하여 개발 가능한 서비스를 설계
- 마이크로서비스 아키텍처는 작고, 느슨하게 결합된 서비스로 애플리케이션을 구성하기 때문에 **유지보수성, 테스트성, 배포성** 등 개발 단계의 품질 속성이 개선됨
- 조직 차원에서 소프트웨어를 더 빨리 개발할 수 있고, 주된 목표는 아니지만 애플리케이션 **확장성**도 향상됨


## 2.2 마이크로서비스 아키텍처 정의
- 애플리케이션 아키텍처 정의 3단계 프로세스
  - ![스크린샷 2022-04-10 오후 9 49 34](https://user-images.githubusercontent.com/7076334/162618985-374c9300-c5ed-41fe-931a-cc1e9675c91e.png)
    - 1단계는 애플리케이션 요건을 핵심 요청으로 추출하는 것 (사용자의 요청 처리)
      - 요청등 최대한 추상화
    - 2단계는 어떻게 여러 서비스로 분해할지 결정
      - 여러 전략 선택 가능 (DDD, 비즈니스 능력에 따라 서비스 정의)
      - 최종 결과는 기술 개념이 아닌 비즈니스 개념 중심으로 이루어진 서비스들
    - 3단계는 서비스별로 API를 정의하는 일
      - 1단계 식별된 시스템 작업을 각 서비스에 배정
      - 여러 서비스가 협동하는 방식을 결정 (IPC, gRPC)

- 분해 과정 장애물
  - 1) 네트워크 지연 (서비스 간 왕복이 잦은 경우)
  - 2) 서비스 간 동시 통신으로 인해 가용성이 떨어지는 문제 (완비형 서비스)
  - 3) 여러 서비스에 걸쳐 데이터 일관성을 지키는 요건 (SAGA)
  - 4) 애플리케이션 도처에 숨어 있는 만능 클래스 (DDD)

### 2.2.1 시스템 작업 식별 (82P)
- 애플리케이션 아키텍처 정의하는 첫 단추는 시스템 작업을 정의
- 2단계 프로세스로 시스템 작업 정의
  - 1단계는 시스템 작업을 기술하기 위해 필요한 보케블러리를 제공하는 핵심 클래스로 구성된 **고수준의 도메인 모델** 생성
  - 2단계는 시스템 작업 식별 후 그 동작을 도메인 모델 관점에서 기술

- 도메인 모델은 주로 사용자 스토리의 명사에서 도출
- 시스템 작업은 주로 동사에서 도출하며, 각 하나 이상의 도메인 객체와 그들 간의 관계로 기술

#### 고수준 도메인 모델 생성
- 추상적인 고수준의 애플리케이션 도메인 모델을 대략 그려 본다.
- 도메인 모델은 스토리에 포함된 명사를 분석하고 도메인 전문가와 상담하는 등 표준 기법을 활용하여 생성
  - ex) 주문하기 스토리 (83P)
    - 시나리오 포함된 명사 Consumer(소비자), Order(주문), Restaurant(음식점), CreditCard(신용카드)등 클래스 필요
  - ex) 주문접수 스토리 (83P)
    - 시나리오 포함된 명사 Courier(배달원), Delivery(배달) 등 클래스 필요

- 위 과정을 몇 차례 거듭하면 핵심 클래스로 구성된 도메인 모델 완성
  - ![스크린샷 2022-04-10 오후 10 23 58](https://user-images.githubusercontent.com/7076334/162620325-397e1de7-dfa6-4a46-bf58-854ddcbac6c0.png)
    - 각 클래스의 의미는 84P

- 시나리오가 있어야 한다(시스템 작업 정의)

#### 시스템 작업 정의
- 애플리케이션이 어떤 요청을 처리할지 식별하는 단계
- 시스템 작업은 크게 두 종류
  - 커맨드(명령) : 데이터 생성, 수정, 삭제
  - 쿼리(조회) : 데이터 읽기

- 커맨드 식별하기 위해 스토리/시나리오에 포함된 동사를 먼저 분석
  - 주문하기 스토리
  - ![스크린샷 2022-04-10 오후 10 36 30](https://user-images.githubusercontent.com/7076334/162620851-dccebfc4-b12e-4cdb-aa89-eda86fb017a8.png)
    - 커맨드는 매개변수, 반환값, 동작 방식의 명세를 도메인 모델 클래스로 정의
    - 명세는 선행 조건, 후행 조건으로 구성 (선행은 주문하기 시나리오 전제(given), 후행 조건은 결과(then)를 나타냄)

  - 데이터를 가져오는 쿼리도 중요하지만 아키텍처와 연관된 시스템 작업은 대부분 커맨드


- UI에 제공하는 쿼리 제공
  - 소비자가 주문하는 과정에서 필요한 쿼리 도출
  - ex)
    - findAvailableRestaurants(deliveryAddress, deliveryTime) : 주어니 장소/시간으로 배달 가능한 음식점 목록 조회
    - findRestaurantMenu(id) : 메뉴 항목 등 음식점 정보를 조회

- 고수준 도메인 모델과 시스템 작업을 보면 애플리케이션이 무슨 일을 하는지 알 수 있기 때문에 아키텍처를 정의하는 데 대단히 유용함
- 시스템 작업을 정의한 후에는 애플리케이션 서비스를 식별

### 2.2.2 서비스 정의: 비즈니스 능력 패턴별 분해
- 비즈니스 능력에 따라 서비스를 정의

#### 비즈니스 능력은 곧 조직이 하는 일이다
- 비즈니스 능력을 보면 그 조직의 비즈니스가 무엇인지 알 수 있음
- ex) 수표 예금 : 비즈니스 능력은 거의 불변이지만, 처리하는 방법은 상당히 달라짐

#### 비즈니스 능력 식별
- 한 조직의 비즈니스 능력은 조직의 목표, 구조, 비즈니스 프로세스를 분석하여 식별
- 비즈니스 능력은 보통 특정 비즈니스 객체에 집중하며, 여러 개의 하위 능력으로 분해할 수 있음
  - ex) 클래임이라는 비즈니스 객체는 클레임 정보 관리, 클레임 검토, 클레임 지불 관리 등의 하위 능력이 있을 것

- FTGO 비즈니스 능력 도출 (88P)
  - 흥미로운 것은 음식점에 관련된 세 가지 능력(음식점 정보 관리, 음식점 주문 관리, 음식점 회계)이 저마다 독특한 음식점 작업을 나타내고 있음

#### 비즈니스 능력을 여러 서비스로
- 비즈니스 능력을 식별한 후 능력에 따라 또는 연관된 능력 그룹에 따라 서비스를 정의
- FTGO의 비즈니스 능력을 애플리케이션 서비스에 매핑 (주관적 판단)
  - ![스크린샷 2022-04-10 오후 11 26 48](https://user-images.githubusercontent.com/7076334/162624124-401103c6-8c3a-432e-95f4-f0f01c4259ff.png)
    - 공급자 관리 능력의 두 하위 능력은 각각 두 서비스로 매핑. 음식점과 배달원은 전혀 다른 성격의 공급자 이기 때문
    - 주문 접수 및 이행 능력은 서비스마다 상이한 프로세스 단계를 담당하도록 세 서비스로 매핑. 배달원 가용성 관리와 배달 관리 능력은 연관이 잇으니 하나의 서비스로 묶음
    - 회계 능력은 유형별 회계가 대동소이하기 때문에 자체 서비스에 매핑

  - 이렇게 서비스를 거의 변하지 않는 비즈니스 능력에 띠라 구성하면 비교적 안정적인 아키텍처를 구축할 수 있음

### 2.2.3 서비스 정의: 하위 도메인 패턴별 분해 (DDD)
- 객체 지향 도메인 모델 중심의 복잡한 소프트웨어 애플리케이션을 구축하는 방법
  - DDD 하위 도메인별로 서비스를 정의 
- DDD에는 마이크로서비스 아키텍처에 적용하면 정말 유요한 하위 도메인(sub-domain)과 경계 컨텍스트(bounded context) 개념이 있다.

- 기존에는 비즈니스를 포괄하는 단일 통합 모델을 만듬
  - ex) 소비자, 주문 등의 비즈니스 엔티티를 따로 정의
  - 하나의 모델에 대해 조직 내 여러 부서의 합의를 이끌어 내기가 정말 어려운 단점
  - DDD는 범위가 분명한 도메인 모델을 여러개 정의하여 기존 방식의 문제점을 해결 

- DDD는 도메인을 구성하는 각 하위 도메인 마다 도메인 모델을 따로 정의
  - ![스크린샷 2022-04-10 오후 11 43 52](https://user-images.githubusercontent.com/7076334/162624906-b7e683e2-137d-4fad-9fab-7770d86d71f9.png)
    - 하위 도메인은 비즈니스 능력과 같은 방법으로 식별 하므로, 비즈니스 능력과 유사한 하위 도메인이 도출
    - 도메인 모델의 범위를 DDD 용어로는 경계 컨텍스트라고 함 (마이크로서비스에서 각 서비스들 간의 경계)

- DDD의 하위 도메인, 경계 컨텍스트 개념은 마이크로서비스 아키텍처의 서비스와 잘 맞고, 마이크로서비스 아키텍처의 서비스 자율팀 개념은 도메인 모델을 개별 팀이 소유/개발한다는 DDD 사고 방식과 어울림

### 2.2.4 분해 지침
- 그 외에 로버트 마틴이 주장한 객체 지향 설계에 근거한 두 가지 원칙
  - 클래스의 책임을 정의하는 단일 책임 원칙(SRP)
  - 클래스를 패키지로 구성하는 공동 폐쇄 원칙(CCP)

#### 단일 책임 원칙
- 클래스는 오직 하나의 변경 사유를 가져야 함
  - 클래스가 독립적으로 변경 가능한 책임을 여럿 짊어지고 있다면 안정적일 수 없다.

- 마이크로서비스 아키텍처에 이 원칙을 적용하면 하나의 책임만 가진 작고 응집된 서비스를 정의할 수 있음
  - ex) FTGO 새로운 아키텍처에서는 주문 접수부터 주문 준비, 배달에 이르기까지 소비자가 주문한 음식이 배달되는 과정 하나하나 모두 개별 서비스가 맡아서 처리

#### 공통 폐쇄 원칙
- 패키지의 클래스들은 동일한 유형의 변경에 대해 닫혀있어야 한다. 패키지에 영향을 주는 변경은 그 패키지에 속한 모든 클래스에 영향을 끼친다.

- 동일한 비즈니스 규칙도 상이한 측면을 구현한 클래스가 여럿 있을 수 있음
  - 이 규칙이 변경되도 개발자는 가급적 소수의 패키지에 있는 코드만 고치면 될 수 있게 만들자 (유지보수성이 현저히 향상)

- CCP를 적용해서 마이크로서비스 아키텍처를 구축하면 동일한 사유로 변경되는 컴포넌트를 모두 같은 서비스로 묶을 수 있음
  - 요건 변경 시, 수정/배포할 서비스 개수 줄어듬

- SRP, CCP, 비즈니스 능력/하위 도메인(DDD)에 따른 분해는 애플리케이션을 서비스로 분해하는 훌륭한 길잡이임

### 2.2.5 서비스 분해의 장애물

#### 네트,워크 지연
- 서비스를 여러 개로 나누면 서비스 간 왕복 횟수가 급증
- 한 차례 왕복으로 여러 객체를 한 번에 가져오는 배치 API를 구현하거나, 값비싸 IPC를 언어 수준의 메서드나 함수 호출로 대체

#### 동기 IPC로 인한 가용성 저하
- 가용성을 떨어뜨리지 않고 서비스 간 통신을 할 수 있을까?
- 비동기 메시징으로 강한 결합도를 제거하고 높이는 방법도 있음

#### 여러 서비스에 걸쳐 데이터 일관성 유지
- 과거에는 커밋 방식의 단계(2PC) 분산 트랜잭션을 많이 사용
- 요즘은 사가라는 다른 방식으로 트랜잭션을 관리
  - 기존 ACID 트랜잭션보다는 복잡하지만 다양한 상황에서도 잘 동작
  - 한 가지 단점은 최종 일관성을 보장 (결국 언젠각는 데이터가 동기화되어 일관성이 맞추어지는것), 원자적 업데이트의 걸림돌 

#### 일관된 데이터 뷰 확보
- 모놀리식은 ACID 트랜잭션 때문에 일관성 보장
- 마이크로서비스는 각 서비스의 DB가 일관적이라 해도 전역 범위에 일관된 데이터 뷰는 확보할 수 없음 (실제로 거의 문제 안됨)

#### 만능 클래스는 분해의 걸림돌
- 애플리케이션 곳곳에 사용되는 만능 클래스는 그 존재만으로도 분해의 걸림돌이다.
  - 이런 클래스는 애플리케이션의 여러 측면에 관한 비즈니스 로직이 있는데, 굉장히 많은 필드가 다수의 컬럼을 가진 DB 테이블에 매핑된 경우가 많음

- FTGO의 만능 클래스 Order
  - ![스크린샷 2022-04-11 오전 1 53 08](https://user-images.githubusercontent.com/7076334/162630546-b4adf9e7-2291-402b-b0bb-0dc9d6de0c4e.png)
    - 주문 처리, 음식점 주문 관리, 배달, 지불에 대항하는 필드/메서드가 Order 클래스에 몰려 있음
    - Order 클래스를 라이브러리로 묶고 Order DB를 중앙화해서 주문을 처리하는 모든 서비스가 이 라이브러리를 통해 DB에 접근하도록 만들면?
      - 마이크로서비스 아키텍처의 핵심 원칙에 위배 (바람직하지 못한 구조)
      - Order 스키마가 변경된다면 관련 팀원들 모두 코드를 수정
    - 주문 DB를 주문 서비스 안으로 캡슐화해서 다른 서비스가 주문 서비스를 통해서만 주문을 조회/수정 하게 만드는것
      - 주문 서비스는 비즈니스 로직이 거의 없는, 빈껍데게 도메인 모델을 가진 데이터 서비스로 전락 

- DDD를 적용하여 각 서비스를 자체 도메인 모델을 갖고 있는 개별 하위 도메인으로 취급
  - 주문과 조금이라도 연관된 서비스는 모두 각자 버전의 Order 클래스를 가진 도메인 모델을 따로 두는 것
  - 배달 서비스 도메인 모델
    - ![스크린샷 2022-04-11 오전 2 14 05](https://user-images.githubusercontent.com/7076334/162631335-b26c6b7d-3d29-42de-b44e-be3628a00054.png) 
    - Order 대신 Delivery라는 적절한 이름의 모델을 사용하며, 배달 상태, 픽업 주소/시간, 배달 주소/시간 등의 구조
  - 주방 서비스 도메인 모델
    - ![스크린샷 2022-04-11 오전 2 14 19](https://user-images.githubusercontent.com/7076334/162631341-9def4e47-b9f7-495c-a237-3aa12eb03a1c.png)
    - Order 대신 Ticket 사용. 상태, 배달 요청 시간, 준비 완료 시간 속성으로 구성되며, 음식점이 준비해야 할 음식을 나타내는 품목 리스트 참조
  - 주문 서비스 도메인 모델
    - ![스크린샷 2022-04-11 오전 2 14 29](https://user-images.githubusercontent.com/7076334/162631343-c571f544-f2f5-4c48-b3f9-f7b66bcdcff0.png)
    - 주문 서비스는 기존 필드/메서드는 원래 버전에 비하면 엄청 단순해짐

- 상이한 서비스의 상이한 객체 간 일관성을 유지 하는 방법은?
  - 이런 서비스 간 일관성은 이벤트 주도 메커니즘인 사가를 활용해서 유지
- 도메인 모델을 여러 개 두면 UX 구현에도 영향이 있음
  - 보통 API 게이트웨이로 처리


### 2.2.6 서비스 API 정의
- 서비스 후보 목록화 후, 각 서비스별 API(작업과 이벤트)를 정의할 차례
  - 서비스 API는 외부 클라이언트 또는 타 서비스가 호출하는 시스템 작업과 서비스간 협동을 지원하기 위해 타 서비스 호출 전용으로 만든 작업
  - 서비스 이벤트는 주로 타 서비스와 협동하기 위해 발행
- 서비스 API를 정의하려면 우선 각각의 시스템 작업을 서비스로 매핑한 후, 그 시스템 작업을 구현하려면 어느 서비스가 서로 협동해야 할지 파악해야 됨


#### 시스템 작업을 서비스로 배정 (99P)
- 제일 먼저 어느 서비스가 요청의 진입점인지 결정
- 간혹 매핑 관계가 분명하지 않을 때도 있음
  - ex) 배달원의 위치 업데이트 하는 기능 => 배달원 서비스? 배달 서비스? (위치가 필요한 주체)
  - 어떤 작업이 제공하는 정보가 필요한 서비스에 그 작업을 배정하는 것이 더 합리적

#### 서비스 간 협동 자원에 필요한 API 확정 (100P)
- 작업 대부분은 여러 서비스에 걸쳐 있다.
  - ex) 주문 생성하는 createOrder()
    - 소비자 서비스 : 소비자가 주문을 할 수 있는지 확인하고 소비자의 지불 정보를 획득
    - 음식점 서비스 : 주문 품목이 올바른지, 소비자가 요청한 배달 주소/시간에 맞추어 해당 음식 점이 준비 가능 한지 등
    - 주방 서비스 : 티켓을 생성
    - 회계 서비스 : 소비자 시용카드를 승인

- 마이크로서비스 아키텍처는 비동기 메시징이 중추적인 역할을 한다.

- 앞으로
  - REST 같은 동기 통신 메커니즘, 메시지 브로커를 이용한 비동기 메시징 등 구체적인 IPC 기술 사용
  - CQRS 패턴(자기 완비형) : 주문 서비스는 음식점 서비스 데이터의 레플리카를 갖게 되어 음식점 서비스를 호출해서 주문이 올바른지 확인할 필요 없음
  - 사가 패턴 : 여러 서비스에 흩어진 데이터를 확실하게 업데이트하고 자기 완비형 서비스를 구현하는 수단
  - API 게이트웨이 : 단순히 요청을 서비스로 넘기는 것 뿐만 아니라 API 조합 패턴을 이용해서 쿼리 작업도 수행

## 2.3 마치며
- 아키텍처는 애플리케이션 개발 속도에 직접 영향을 주는 갖가지 '~성'을 좌우. ex) 관리성, 테스트성, 배포성
- 마이크로서비스는 기술적 관심보다는 비즈니스 능력, 하위 도메인(DDD)등 비즈니스 관심사 위주로 구성
- 서비스를 분해하는 패턴은 크게 두가지
  - 비즈니스 능력에 따른 분해 : 비즈니스 아키텍처 기반
  - 하위 도메인에 따른 분해 : DDD 개념 기반
- DDD를 적용하고 서비스마다 도메인 모델을 따로 설계하면, 의존 관계가 뒤엉켜 분해를 가로 막는 만능 클래스를 제거 가능