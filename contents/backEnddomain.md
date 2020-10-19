# BackEnd개발 
# 사례:도서대여시스템
## 구현 할 DEV 아키텍처 개념도 
![image](https://github.com/CNAPS-MSA/CNAPS3/blob/master/img/dev.jpg)

- 최종운영은 쿠버네티스에 배포하겠지만 로컬에서 개발을 원할하게 진행하기 위한 환경을 잡았다.
- 개발환경은 H2 DB를 사용하였고, Spring Cloud의 G/W 및 Discovery 패턴을 적용하였다.
- Front와 API G/W 및 사용자 서비스가 하나의 서버로 구축되었으며, 서비스 Discovery처리를 위한 Register서비스가 있다.
- 백 엔드 서비스는 대여,서적,카탈로그,게시판,사용자로 5개이나 사용자서비스가 프론트엔드 서비스,G/W 와 같이 있다.  
- 프론트와 백 엔드의 기본통신방법은 REST API이며 서비스 간 동기 통신은 Feign를 사용하며 비동기 통신 메커니즘을 카프카로 지원한다.
- 도서검색기능과 인기서적기능의 원활한 사용을 위해 카탈로그서비스와 도서서비스를 분할하는 CQRS패턴을 적용했으며 카탈로그 서비스는 저장소로 읽기에 최적화된 Mongo DB를 사용한다.
- 서비스간의 내부구조가 다를 수 있음을 보여주기 위해 대여,도서,카탈로그의 내부 구조는 도메인 모델 중심의 DDD 구조이며, 게시판의 내부구조는 Simple CRUD 구조를 채택했다.


## 외부아키텍처 구현
- MSA개발환경을 쉽게 구축해 주는 도구인 Jhipster를 사용하였다.
- Jhipster 콘솔창의 질의응답을 통해 스프링 클라우드,스프링부트기반의 마이크로서비스 개발환경을 쉽게 구축해 준다.
  - [Jhipster를 활용한 MSA 외부아키텍처 구성(게이트웨이,레지스터)](/contents/jhipster_guide.md)

## 내부아키텍처 구현
- 백엔드 서비스의 구조를 정의하고 서비스를 생성해 본다.
  - [Jhipster사용하여 백엔드 서비스 프로젝트 구조 생성](/contents/jhipster_guide2.md)
- Jhipster가 생성한 구조는 **헥사고널 아키텍처 + DDD** 의 기본 사상을 만족하나 DTO의 위치 및 몇가지 수정이 필요해 보인다. 아래와 같이 좀더 바람직하도록 수정하였다.
  - [Package Refactoring](/contents/jhipster_package_ref.md)
- 헥사고널+트랜젝션스크립트 패키지구조는  게시판 서비스 구현을 통해 추후 살펴보자.

## 백엔드 마이크로서비스 구현
### 백엔드서비스 구현 전, Configuration 설정
- [벡엔드서비스 구현전에 각 service를 gateway의 end-point에 등록해야 한다.](/contents/endpointadd.md)

### 대여(Rental)서비스 
- 내부구조 : 헥사고널 + DDD 구조
- 저장소처리 : OR매퍼인 Sring DATA 
- 주요기능
  - 사내도서시스템의 핵심서비스로 도서대출/반납/연체/대출금지 해제처리를 수행한다.
- [내부 Business Logic 구현하기 - 1:도서대출/반납 기능](/contents/jhipster_businesslogic.md)
- [타서비스 동기호출처리 : Feign Client 연결하기](/contents/jhipster_feign.md) 
- [타서비스 비동기호출처리 : Kafka를 통한 EDA구현](/contents/jhipster_kafka.md)
- [내부 Business Logic 구현하기 - 2:도서 연체/연체된 도서 반납기능](/contents/OverdueBook.md)
- [내부 Business Logic 구현하기 - 3:도서 대여불가 해제처리 기능](/contents/releaseOverdue.md)
  
### Book 서비스
- 내부구조 : 헥사고널 + DDD 구조
- 저장소처리 : OR매퍼인 Sring DATA 
- 주요기능
  - 도서관리를 위한 서비스로 도서입고처리 및 운영자에게 기본관리기능을 제공한다. 
- [내부 Business Logic 구현하기 - 1: 도서 관리 기능](/contents/book_businesslogic.md)
 
### User 서비스
- 내부구조 : 헥사고널 + DDD 구조
- 저장소처리 : OR매퍼인 Sring DATA 
- 주요기능
  - 사용자관리, 포인트관리기능을 제공한다.
 - [내부 Business Logic 구현하기 - 1: 사용자 관리 기능 ](/contents/user_businesslogic.md)
 - [내부 Business Logic 구현하기 - 2: 포인트 관리 기능](/contents/user_point.md)

    
### Catalog 서비스
- 내부구조 : 헥사고널 + DDD 구조 + NoSQL DB
- 외부패턴 : CQRS패턴 적용
- 저장소처리 : OR매퍼인 Sring DATA 
- 주요기능
  - 도서검색 기능 최적화를 위해 CQRS패턴을 적용한 서비스로 도서목록 및 검색기능, 인기도서목록 기능 제공  
- [내부 Business Logic 구현하기 - 1. Catalog 서비스관리 기능](/contents/catalog_businesslogic.md)

   
### 게시판 서비스 
- 헥사고널 + Simple CRUD 구조 
- 저장소처리 : SQL매퍼인 MyBatis 
- 주요기능 
  - 공지사항, 자유게시판, 댓글기능
  - [내부 Business Logic 구현하기 - 1. 게시판 기능(작업중)]

### 예외 처리(Exception) 구현
- 서비스 내부에서 발생할 수 있는 에러를 처리할 수 있도록 예외처리를 구현한다.
  - [서비스 내부 에러 관리하기](https://engineering-skcc.github.io/msa/jhipster-exception/)
- Feign Client 사용 시 발생할 수 있는 외부 서비스의 에러를 처리할 수 있도록 예외처리를 구현한다.
  - [서비스 외부 에러 관리하기 - Feign Exception](https://engineering-skcc.github.io/msa/jhipster-feign/)
  
## 테스트 시나리오
1. 사용자 2명 등록한다. USER1,USER2
2. USER2에게 운영자 권한을 준다.
3. 운영자가 3권의 도서정보를 등록한뒤 입고처리한다.(2권은 대출가능,1권의 대여중)
4. USER으로 로그인한다.
5. USER1이 도서정보를 검색한다. 
6. 대출가능한 도서를 2권 대출한다.
    - 대여중인 도서는 대출할 수 없다.
7. 대출한 도서중 1권을 반납한다.
8. 2주가 지난도서는 연체된다.(1권을 연체처리한다.)
9. 연체도서를 반납한다.
10. USER1이 다시 도서를 대출하려고 하나 시스템은 대출불가 메시지를 보낸다.
11. USER1은 연체일자를 확인하고 포인트로 가감하여 연체를 해제한다.
12. USER1은 다시 대출가능상태가 되고 대출을 수행한다.

### 동작 확인하기
- 작성한 테스트 시나리오 대로 백엔드가 제대로 동작하는지 확인해본다.
  - [Local에서 동작시키기](/contents/backend_localtest.md)
