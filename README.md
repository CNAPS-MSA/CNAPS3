# 시냅스 방법론 3.0
> 시냅스 방법론 3.0 의 공정 및 주요 산출물, 샘플코드를 공유한다.



# SW Moderization 공정도




# 네이티브 공정도

![공정도](https://github.com/cnaps/main/blob/master/img/%EA%B3%B5%EC%A0%95%EB%8F%84.png)  

# 프렉티스 별 주요활동 & 산출물 
|Phase|Practice | Step | Output |
|------|------|---|---|
|Sprint0|외부아키텍처정의|- 인프라정의<br>- 플랫폼정의<br>- 백엔드서비스정의<br>- 통신방법정의<br>- 배포정책정의|인프라구성도<br>아키텍처구성도<br>배포구성도|
||내부아키텍처정의|- 프론트엔드기술정의<br>- 서비스내부구조정의<br>- 비지니스로직구조설계<br>- 데이터매핑구조설계<br>|서비스별패키지구조<br>기타아키텍처문서|
||구현환경정의|- 개발환경정의<br>- CI/CD환경구성<br>-  테스트환경정의<br>- 운영환경정의|클라우드 개발/테스트/운영환경<br>CI/CD환경|
||마이크로서비스도출|- 서브시스템식별<br>- 바운디드컨텍스트식별<br>- 마이크로서비스도출|서비스맵|
||서비스스펙(SPEC)정의|- 서비스별KeyConcept정의|서비스별KeyConcept<br>인터페이스정의서|
||테스트계획수립|- 테스트수행대상정의<br>- 테스트수행절차,방법,도구정의|테스트수행계획서|
||데이터이행계획수립|- 데이터이행대상정의<br>- 데이터이행방법정의|데이터이행계획서|
|SprintN#|마이크로서비스모델링|- 도메인모델링<br>- 데이터모델링<br>- API정의|도메인모델<br>데이터모델<br>API설계서|
||백엔드구현|- 백엔드코드구현<br>- 저장소구현 <br>- API테스트수행|백엔드구현소스|
||UI설계|-UI레이아웃정의<br>-UI속성및이벤트정의|UI설계서|
||프론트엔드구현|- 프론드엔드코드구현 -UI단위테스트수행|프론트엔드구현소스|
||지속적통합|- 파이프라인설계<br>- 빌드잡구현<br>- 빌드수행|파이프라인(빌드)<br>빌드결과|
||지속적배포|- 파이프라인설계<br>- 배포잡구현<br>- 배포수행|파이프라인(배포)<br>배포된서비스|
||데이터이행프로그램설계|- 이행절차설계<br>- 데이터클렌징<br>- 신구매핑정의|데이터매핑정의서|
||데이터이행리허설|- 테스트데이터준비<br>- 데이터이행테스트수행<br>- 이행절차보완|이행리허설결과|
|Test&Release|통합테스트|- 통합테스트환경준비<br>- 통합테스트수행<br>- 결과정리및결함수정|식별결함|
||성능테스트|- 성능테스트계획수립<br>- 환경준비 <br>- 성능테스트수행<br>- 결과정리및조치 |성능테스트수행결과서|
||데이터이행|- 기초데이터이행<br>- 본데이터이행||
||릴리즈|- 릴리즈수행|운영환경|


# 핵심프로세스 설명 
## 설계/개발 공정도
![설계/개발공정도](https://github.com/CNAPS-MSA/CNAPS3/blob/master/img/agileP.png)  

## 0.아키텍처정의 
- 아키텍처 구성도
  - 외부 아키텍처
    - 배포 구성도 
    - 쿠버네티스 기반 
  - 내부 아키텍처 
    - 헥사고널 & 클린아키텍처 이해하기
    ![백엔드아키텍처](https://github.com/CNAPS-MSA/CNAPS3/blob/master/img/BackEndA.png)  


## 1.설계
- 마이크로서비스 도출하기
  - 컨텍스트맵
    - DDD 전략적설계란?
    - 마이크로서비스 도출을 가장 쉽게 할 수 있는 방법 이벤트스토밍
    ex>  ![bc](/img/bc.png)  

    
- 마이크로서비스 설계하기 
  - 내부구조정의 
    - 헥사고널 아키텍처 적용 패키지구조
    ![패키지](/img/package.png)  

    - 단순 CRUD구조와 도메인모델구조 선택가이드
  - API설계서
    - API설계란?
     ![api](/img/apiD.png)  

  - 도메인모델
    - DDD 전술적설계란?
    ex>  
    <img src="/img/class.png" width="40%">
    ![domain](/img/class.png){ width="300" height="300"} 

  - 데이터모델
    ex>  ![data](/img/data.png){: width="60%" height="60%ㄴ"}

  

## 2.개발
- 내부영역개발
  - 도메인 구현
  - 레파지토리 구현
  - 서비스 구현
  - 도메인이벤트 구현
- 외부영역개발
  - 컨트롤러 구현
- EDA구현
  - 도메인이벤트 어댑터 구현
    - 카프카 설치
- CQRS구현
    
## 3.통합 및 배포
- 도커라이징
  - DockerFile
- 컨테이너라이징
  - 쿠버네티스 아키텍처의 이해
  - 실습
- 배포파이프라인구축
  - 애저
  - AWS

## 컨텐츠 및 교육교재
  - [MSA개념 및 주요패턴](https://engineering-skcc.github.io/tags/microservice/)
    - [마이크로서비스 개념](https://engineering-skcc.github.io/categories/#microservice-%EA%B0%9C%EB%85%90)
    - [마이크로서비스 Outer 아키텍처](https://engineering-skcc.github.io/categories/#microservice-outer-achitecture)
    - [마이크로서비스 Inner 아키텍처](https://engineering-skcc.github.io/categories/#microservice-inner-achitecture)
    - 교육교재 
  - 마이크로서비스 모델링
    - 도메인주도설계
      - 전략적설계
      - 전술적설계
    - API설계
    - 이벤트스토밍
    - 교육교재
  - Workflowy (실습 스크립트 LookUp): https://workflowy.com/#/217cf1148297
   


