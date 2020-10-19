# 도메인모델링
- [DDD전술적설계](https://engineering-skcc.github.io/microservice%20modeling/BackEnd-modeling-domainModeling/)
- [Aggregate와 도메인이벤트](https://engineering-skcc.github.io/microservice%20modeling/BackEnd-modeling-domainModeling/)

## 1.도메인모델 SAMPLE(도서대여시스템) 
* 마이크로서비스별로 작성
* 헥사고널 + DDD구조를 선택한 서비스
* 형식은 자유 (PPT, UML도구 ,화이트보드갭쳐,협업도구,구현후리버스)

### 1.1 대여(Rental)서비스의 도메인모델
![image](https://user-images.githubusercontent.com/15258916/96394365-bfaaba80-11fc-11eb-8e6c-7908c9b69d98.png)
- 도메인 모델에서는 비지니스 개념을 표현한다. 비지니스 개념은 객체로 표현되고 도메인 주도 설계의 전술적 설계 기법인 어그리게잇, 엔티티, VO, 표준타입 패턴을 적용한다.
- 위 그림은 그렇게 정의된 대여 서비스의 도메인 모델이다. 대여와 반납의 책임을 가지고 있는 어그리게잇이며 루트 엔티티인 대여카드(Rental) , Rental과 일대다 관계인 엔티티 유형의 대여도서(RentedItem), 엔티티 연체도서(OverdueItem), 엔티티 반납도서(RetrurnItem)로 구성된다. 대여도서(OverdueItem) 와 반납도서(RetrurnItem)은 대여도서(RentedItem)과 마찬가지로 Rental과 일대다관계이다.
- Rental의 개념은 대여카드이다. 모든 사용자는 대여를 위한 대여카드를 하나씩 보유한다. 대여카드는 대여, 반납, 연체 처리, 연체도서반납,연체해제처리의 책임을 가진다.
- 대여 시 빌린도서만큼 대여도서(RentedItem)가 생성되고, 연체되면 연체도서(OverdueItem) 로 이동하고, 반납 시 반납도서(RetrurnItem)로 최종 이동된다. 
- 개인 5권의 대여 한도가 체크되고 1권의 도서라도 연체 되면 대여할 수 없다. 이런 대여가능여부는 표준 타입인 대여가능여부(RentalStatus)에 의해 규정된다.

### 1.2 대여(Rental)서비스의 유스케이스 흐름
![image](https://user-images.githubusercontent.com/15258916/87246908-5728a080-c48b-11ea-90c3-86a10a3c27c1.png)

위의 그림은 도서대여와 도서반납의 유스케이스 흐름을 시퀀스 다이어그램 으로 작성한 것이다.
도서대여 및 도서반납의 비지니스 로직은 도메인 모델에 응집되어 있는 것을 확인할 수 있다. 
서비스는 그외 흐름제어 및 저장처리, 이벤트 메시지 처리를 담당한다. 

### 2.1 도서(Book)서비스의 도메인모델
<img src="https://github.com/CNAPS-MSA/CNAPS3/blob/master/img/bookDomainModel.png" width="70%">

### 2.1 카탈로그(Catalog)서비스의 도메인모델
<img src="https://github.com/CNAPS-MSA/CNAPS3/blob/master/img/catalogDomainModel.png" width="70%">


