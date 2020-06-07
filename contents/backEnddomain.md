# BackEnd개발

# 구현 할 아키텍처 
![image](https://github.com/CNAPS-MSA/CNAPS3/blob/master/img/dev.jpg)

개발환경에서 구현할 아키텍처이다.

최종은 쿠버네티스에 배포하겠지만 로컬에게 개발을 원할하게 진행하기 위해 환경을 잡았다.

주요 차이점은 H2DB를 사용하였고, Spring 클라우드의 G/W 및 Discovery 패턴을 적용하였다.

Front와 API G/W 및 사용자 서비스가 하나의 서비로 구축되었으며, Discovery처리를 위한 Register서비스가 있다.

BackEnd서비스는 대여,서적,카탈로그,게시판,사용자로 5개이나 사용자서비스가 Front와 같이 있으므로 4개로 표현되었다.

프론트와 백엔드의 기본통신방법은 REST API이며 서비스간 동기통신은 Feign를 사용하며 비동기 통신 메커니즘을 카프카로 지원한다.

도서검색기능과 인기서적기능의 원활한 사용을 위해 카탈로그서비스와 도서서비스를 분할하는 CQRS패턴을 적용했으며 

카탈로그 서비스는 저장소로 읽기에 최적화된 Mongo DB를 사용한다.

서비스간의 내부구조가 다를 수 있음을 보여주기 위해 대여,서벅,카탈로그의 내부 구조는 도메인 모델 중심의 DDD 구조이며, 게시판의 내부구조는 SimpleCRUD 구조를 채택했다.

## 외부아키텍처

## 내부아키텍처
### 헥사고널+DDD 적용 패키지구조


### 헥사고널+트랜젝션스크립트 패키지구조
추가예정



## 구현기능

### Rental 서비스 
- 헥사고널 + DDD 구조
- 구현기능
  - 도서대출
    1. Book서비스의 도서정보가져와서 도서대출처리
    2. Book서비스 '대출됨'처리 (Book 서비스와 연계 구현)
    3. User서비스에 포인트 부여 (User 서비스와 연계 구현)
    4. Category서비스에 베스트셀러 카운트(Category서비스와 연계 구현)
  - 반납처리 
    1. 도서반납처리
    2. Book서비스 '반납됨'처리 (Book 서비스와 연계 구현)
    3. User서비스에 포인트 부여 (User 서비스와 연계 구현)
  - 연체처리
    1. 연체처리
    2. 대여불가처리 

### Book 서비스
- 헥사고널 + DDD 구조
- 구현기능
  - 도서입고
    1. 도서입고처리,입고 시 Catagory서비스에서 검색되도록 정보전송
    2. Book서비스 '대출됨/반납됨' 기능구현
  - 도서예약
    1. 대출됨 상태에 있는 도서 예약처리

### User 서비스
- 헥사고널 + DDD 구조
- 구현기능
  - 사용자관리
    1. 사용자 역할관리
  - 포인트관리
    1. 포인트 부여,감소
  - 대출금지헤제처리
    1. 연체일만큼 포인트로 감소
    2. Rental서비스 '대출불가처리' 해제

### Catagory 서비스
- 헥사고널 + DDD 구조 + NoSQL DB
- 구현기능
  - 도서목록 및 검색기능
    1. 도서입고 시 Catgory생성
  - 베스트셀러 기능   
    1. 대출기록으로 인기도서 목록 생성 
  
### 게시판 서비스 
- 헥사고널 + Simple CRUD 구조 
- 구현기능 
  - 공지사항
  - 자유게시판 
  - 댓글기능 


### 프로젝트 구조 생성하기
- [Jhipster사용하여 프로젝트 구조생성](/contents/jhipster_guide.md)
  - 도메인 구현
  - 레파지토리 구현
  
- [Package Refactoring](/contents/jhipster_package_ref.md)

### 내부영역개발
- [Business Logic개발](/contents/jhipster_businesslogic.md)
  - 서비스 구현

### 외부영역개발
- [Feign Client 연결하기](/contents/jhipster_feign.md)
  - 컨트롤러 구현
- [EDA구현](/contents/jhipster_kafka.md)
  


  
