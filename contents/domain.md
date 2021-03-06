# 도메인모델링
- [DDD전술적설계](https://engineering-skcc.github.io/microservice%20modeling/BackEnd-modeling-domainModeling/)
- [Aggregate와 도메인이벤트](https://engineering-skcc.github.io/microservice%20modeling/BackEnd-modeling-domainModeling/)

## 소프트웨어 구조를 그림으로 표현하기 
>프로젝트 현장에 많이 가서 개발된 시스템을 이해하기 위해 시스템의 설계 문서를 찾는 경우가 많다. 그런 문서로 아키텍처 정의서 , 어플리케이션 내부 구조를 표현한 문서들을 찾는데 매우 실망하는 경우가 많다. 대부분 그런 문서가 미약하거나 조악하다. 그래서 실제로 어플리케이션 소스코드를 직접 받아서 보는 경우가 많은데 이런 시스템의 내부 구조는 이해할 수 없을 정도로 뒤죽박죽인 경우가 많다.
소프트웨어는 유연해야 하고 그렇기 위해서는 어플리케이션 내부구조정의에  고민을 많이 해야 한다. 구현에만 급급하면 유연한 시스템을 만들 수 없고 최초 개발 시에는 빨리 개발했지만 유지보수 시 복잡하고 이해하기 힘들어서 더디게 유지보수 되는 시스템이 되고 만다. 

>무거운 설계 문서를 만들어야 한다는 것은 절대 아니다. 그렇지만 엔지니어 들은 자신이 개발한 시스템의 구조를 그림으로 표현해야 하는 능력이 쌓아야 한다. 본인 입장에서는 그림을 통해 자신이 구현할 시스템의 구조를 생각하고 고민하고 개선할 수 있다. 또 협력하는 사람들과는 같이 논의하여 협업을 도모 할 것이며 이후 시스템을 다른 사람이 유지보수한다면 쉽게 이해할 수 있을 것이다. 

>특히 서비스의 핵심인 도메인 모델을 이해하는 것은 중요하다 그리고 서비스의 연관관계를 표현하는 것도 중요하다. UML 표기법도 좋고 간단한 그림도 좋다. 그림의 형식은 중요하지 않다. 

# 사례:도서대여시스템
## 1.도메인모델 SAMPLE(도서대여시스템) 
* 마이크로서비스별로 작성
* 헥사고널 + DDD구조를 선택한 서비스
* 형식은 자유 (PPT, UML도구 ,화이트보드갭쳐,협업도구,구현후리버스)

### 1.1 대여(Rental)서비스의 도메인모델
![image](https://user-images.githubusercontent.com/15258916/96394365-bfaaba80-11fc-11eb-8e6c-7908c9b69d98.png)
-	도메인 모델에서는 비지니스 개념을 표현한다. 비지니스 개념은 객체로 표현되고 도메인 주도 설계의 전술적 설계 기법인 애그리게잇, 엔터티, VO, 표준 타입 패턴을 적용한다.
-	위 그림은 그렇게 정의된 대출 서비스의 도메인 모델이다. 대출과 반납의 책임을 가지고 있는 애그리게잇이며 루트 엔터티인 대출카드(Rental) , Rental과 일대다 관계인 엔터티 유형의 대출도서(RentedItem), 엔터티 연체도서(OverdueItem), 엔터티 반납도서(RetrurnItem)로 구성된다. 대출도서(OverdueItem) 와 반납도서(RetrurnItem)은 대출도서(RentedItem)과 마찬가지로 Rental과 일대다관계이다.
-	Rental의 개념은 대출카드이다. 모든 사용자는 대출을 위한 대출카드를 하나씩 보유한다. 대출카드는 대출, 반납, 연체 처리, 연체도서반납,연체해제처리의 책임을 가진다.
-	대출 시 빌린 도서만큼 대출도서(RentedItem)가 생성되고, 연체되면 연체도서(OverdueItem) 로 이동하고, 반납 시 반납도서(RetrurnItem)로 최종 이동된다. 
-	개인 5권의 대출 한도가 체크되고 1권의 도서라도 연체 되면 대출할 수 없다. 이런 대출가능여부는 표준 타입인 대출가능여부(RentalStatus)에 의해 규정된다.

### 1.2 대여(Rental)서비스의 유스케이스 흐름
![image](https://user-images.githubusercontent.com/15258916/87246908-5728a080-c48b-11ea-90c3-86a10a3c27c1.png)

위의 그림은 도서대여와 도서반납의 유스케이스 흐름을 시퀀스 다이어그램 으로 작성한 것이다.
도서대여 및 도서반납의 비지니스 로직은 도메인 모델에 응집되어 있는 것을 확인할 수 있다. 
서비스는 그외 흐름제어 및 저장처리, 이벤트 메시지 처리를 담당한다. 

### 2.1 사용자(User)서비스의 도메인모델
![image](https://user-images.githubusercontent.com/15258916/96394449-013b6580-11fd-11eb-9fb9-556c242fb5d9.png)
- 사용자 서비스의 도메인 모델은 사용자와 권한으로 구성된다. 사용자는 모두 어그리게잇이며 루트 엔터티이며 권한은 엔티티이다. 사용자와 권한은 다대다의 관계를 갖는다.
- 사용자의 권한은 'ROLE_ADMIN'인 관리자와 'ROLE_USER'인 일반 사용자로 구성되어있으며 사용자 생성 시 기본 권한인 'ROLE_USER'로 설정된다. 관리자 권한의 경우, 오직 관리자에 의해서만 부여 받을 수 있다.

### 2.2 도서(Book)서비스의 도메인모델
![image](https://user-images.githubusercontent.com/15258916/96394638-7c048080-11fd-11eb-9b2f-884d18369c5d.png)
- 모델은 도메인 엔터티인 대출 도서와 입고 도서로 구성된다. 도서공급자에 의해 도서가 입고되면 입고 도서 객체가 생성되고, 이 입고된 도서는 유형에 따라 분류되고 도서관이 지정되어 대출 가능한 상품인 대출 도서로 등록된다. 대출할 수 있게 정식 등록된 객체가 대출도서이다. 
- 도서는 이용가능을 판단하는 도서 상태를 가지고 있어 대출되면 대출 불가 상태가 되고 반납되면 대출가능상태가 된다. 입고 도서는 루트엔터티이며 애그리게잇이고 대출도서도 역시 루트엔터티이며 애그리게잇이다.


### 2.3 카탈로그(Catalog)서비스의 도메인모델
![image](https://user-images.githubusercontent.com/15258916/96394649-81fa6180-11fd-11eb-866b-55a6124aae69.png)
- 도서 카탈로그 엔터티이다. 조회/검색 용도이기 때문에 사용자가 도서 조회/검색 시 필요한 속성들만을 가지고 있다. 빠르고 쉽게 도서 정보를 확인할 수 있는 필수 정보인 도서명, 설명, 작가, 출판사 ,대여 여부, 대출횟수등으로 구성된다. 
- 도서카탈로그는 엔티티이며 어그리게잇이다.


