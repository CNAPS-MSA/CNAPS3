# BackEnd개발

# 구현 아키텍처 

# 내부구조정의
## 1.마이크로서비스내부 패키지구조
### 헥사고널+DDD 적용 패키지구조
![패키지](/img/package.png)  

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
  


  
