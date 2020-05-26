# 시냅스 방법론 3.0
> 시냅스 방법론 3.0 의 공정 및 주요 산출물, 샘플코드를 공유한다.



# SW Moderization 공정도




# 네이티브 공정도

![공정도](https://github.com/cnaps/main/blob/master/img/%EA%B3%B5%EC%A0%95%EB%8F%84.png)  

# 프렉티스 별 주요활동 & 산출물 

<details>
<summary>산출물목록</summary>
<div markdown="1">

|Phase|Practice | Step | Output |
|------|------|---|---|
|Sprint0|**외부아키텍처정의**|- 인프라정의<br>- 플랫폼정의<br>- 백엔드서비스정의<br>- 통신방법정의<br>- 배포정책정의|인프라구성도<br>아키텍처구성도<br>배포구성도|
||**내부아키텍처정의**|- 프론트엔드기술정의<br>- 서비스내부구조정의<br>- 비지니스로직구조설계<br>- 데이터매핑구조설계<br>|서비스별패키지구조<br>기타아키텍처문서|
||**구현환경정의**|- 개발환경정의<br>- CI/CD환경구성<br>-  테스트환경정의<br>- 운영환경정의|클라우드 개발/테스트/운영환경<br>CI/CD환경|
||**마이크로서비스도출**|- 서브시스템식별<br>- 바운디드컨텍스트식별<br>- 마이크로서비스도출|서비스맵|
||**서비스스펙(SPEC)정의**|- 서비스별KeyConcept정의|서비스별KeyConcept<br>인터페이스정의서|
||테스트계획수립|- 테스트수행대상정의<br>- 테스트수행절차,방법,도구정의|테스트수행계획서|
||데이터이행계획수립|- 데이터이행대상정의<br>- 데이터이행방법정의|데이터이행계획서|
|SprintN#|**마이크로서비스모델링**|- 도메인모델링<br>- 데이터모델링<br>- API정의|도메인모델<br>데이터모델<br>API설계서|
||**백엔드구현**|- 백엔드코드구현<br>- 저장소구현 <br>- API테스트수행|백엔드구현소스|
||**UI설계**|-UI레이아웃정의<br>-UI속성및이벤트정의|UI설계서|
||**프론트엔드구현**|- 프론드엔드코드구현 <br>- UI단위테스트수행|프론트엔드구현소스|
||**지속적통합**|- 파이프라인설계<br>- 빌드잡구현<br>- 빌드수행|파이프라인(빌드)<br>빌드결과|
||**지속적배포**|- 파이프라인설계<br>- 배포잡구현<br>- 배포수행|파이프라인(배포)<br>배포된서비스|
||데이터이행프로그램설계|- 이행절차설계<br>- 데이터클렌징<br>- 신구매핑정의|데이터매핑정의서|
||데이터이행리허설|- 테스트데이터준비<br>- 데이터이행테스트수행<br>- 이행절차보완|이행리허설결과|
|Test&Release|통합테스트|- 통합테스트환경준비<br>- 통합테스트수행<br>- 결과정리및결함수정|식별결함|
||성능테스트|- 성능테스트계획수립<br>- 환경준비 <br>- 성능테스트수행<br>- 결과정리및조치 |성능테스트수행결과서|
||데이터이행|- 기초데이터이행<br>- 본데이터이행||
||릴리즈|- 릴리즈수행|운영환경|

</div>
</details>

# Cloud Native 핵심프로세스 설명 
## Cloud Native 개발공정도
![설계/개발공정도](https://github.com/CNAPS-MSA/CNAPS3/blob/master/img/agileP.png)  
- [핵심프로세스설명:애자일기반 마이크로서비스 개발프로세스](https://engineering-skcc.github.io/agile/microservice-agile/)

## 0.아키텍처정의 
- [외부아키텍처정의](/contents/outerarchi.md) 
- [내부아키텍처정의](/contents/innerarchi.md)  
 

## 1.설계
- [마이크로서비스 도출하기](/contents/ddd.md) 
- BackEnd 설계하기 
  - [내부구조정의](/contents/mspackage.md) 
  - [API설계](/contents/API.md) 
  - [도메인모델링](/contents/domain.md) 
  - [데이터모델링](/contents/data.md) 
 - [FrontEnd 설계하기](https://engineering-skcc.github.io/microservice%20modeling/FrontEnd-modeling/)
## 2.개발
- [BackEnd개발(헥사고널&DDD)](/contents/backEnddomain.md) 
- [BackEnd개발(헥사고널&트랜젝션)](/contents/backEnddata.md) 
- FrontEnd개발
    
## 3.(CI/CD) 통합 및 배포
- [지속적통합](/contents/ci.md)
- [지속적배포](/contents/cd.md) 

## 4.SAMPLE
- [도서대여시스템](/contents/sample.md)
 - 도서대여시스템 사용하기
- [Jhipster Sample](/contents/jhipster_guide.md)

## 컨텐츠 및 교육교재
  - [MSA개념 및 주요패턴](https://engineering-skcc.github.io/tags/microservice/)
    - [마이크로서비스 개념](https://engineering-skcc.github.io/categories/#microservice-%EA%B0%9C%EB%85%90)
    - [마이크로서비스 Outer 아키텍처](https://engineering-skcc.github.io/categories/#microservice-outer-achitecture)
    - [마이크로서비스 Inner 아키텍처](https://engineering-skcc.github.io/categories/#microservice-inner-achitecture)
  - [마이크로서비스 모델링](https://engineering-skcc.github.io/categories/#microservice-modeling)
    - 교육교재 
  - 마이크로서비스 모델링 
    - 도메인주도설계
      - [전략적설계](https://engineering-skcc.github.io/microservice%20modeling/ddd-Srategic-design/)
      - [전술적설계](https://engineering-skcc.github.io/microservice%20modeling/BackEnd-modeling-domainModeling/)
    - [API설계](hhttps://engineering-skcc.github.io/microservice%20modeling/BackEnd-modeling-API/)
    - [이벤트스토밍](https://engineering-skcc.github.io/microservice%20modeling/Event-Storming/)
    - 교육교재
  - Workflowy (실습 스크립트 LookUp): https://workflowy.com/#/217cf1148297
   

