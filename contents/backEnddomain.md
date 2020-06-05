# BackEnd개발(헥사고널+DDD)

# 내부구조정의
## 1.마이크로서비스내부 패키지구조
### 헥사고널+DDD 적용 패키지구조
![패키지](/img/package.png)  

### 헥사고널+트랜젝션스크립트 패키지구조
추가예정

### 구현기능
- 도서 대출하기
  - 동기(도서정보가져오기),비동기처리(도서재고 가감,포인트부여,베스트셀러)
- 반납하기 
  - 비동기처리(도서재고 증가,포인트부여)

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
  


  
