# 시냅스(CNAPS)방법론 3.0 Quick Guide
> 시냅스 방법론 3.0 의 공정 및 주요 산출물, 샘플코드를 공유한다.
> Agile 기반의 Cloud / MSA 적용한 신규 시스템 구축 프로젝트를 위한 통합 관리/개발 방법론으로 프로젝트 수행을 위한 공정도와 공정 별 샘플과 산출물을 공유한다.

# Cloud / MSA 기반 마이크로서비스 개발 방법론

![image](https://user-images.githubusercontent.com/18652530/96830176-f0e1ef80-1475-11eb-8301-798117fa9913.png)

# 프렉티스별 주요활동 & 산출물 

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

# Cloud Native 핵심프로세스 
- Native 방법론의 핵심 아키텍팅/설계/개발 프로세스를 각 활동들을 설명하며 샘플을 제시한다.
- 각 활동의 개념등은 SK 주식회사 C&C 기술블로그와 연계하여 설명한다.
- 샘플은 '사내도서대여시스템'을 주제로 아키텍팅/설계/개발/배포등 활동별로 Seemless하게 제공한다.

## Cloud Native - DEV 개발공정도
CNAPS3.0의 개발공정만 별도로 표시한 공정도이다.
![설계/개발공정도](https://github.com/CNAPS-MSA/CNAPS3/blob/master/img/agileP.png)  
- [핵심프로세스설명:애자일기반 마이크로서비스 개발프로세스](https://engineering-skcc.github.io/agile/microservice-agile/)
 
## SPRINT#0 - 아키텍처정의& 마이크로서비스도출
- [Scrum Team 구성](https://engineering-skcc.github.io/agile-quickguide/Agile-QuickGuide03-스크럼팀구성/)
- [Scrum 환경 구성](https://engineering-skcc.github.io/agile-quickguide/Agile-QuickGuide08-스크럼환경/)
- [외부아키텍처정의](/contents/outerarchi.md) 
- [내부아키텍처정의](/contents/innerarchi.md)  
- [마이크로서비스 도출하기](/contents/ddd.md) 
- [Product Backlog 도출](https://engineering-skcc.github.io/agile-quickguide/Agile-QuickGuide04-제품백로그도출/)
- [Service Spec 정의](https://engineering-skcc.github.io/msa/ServiceSpec/)
- [일감 크기 산정](https://engineering-skcc.github.io/agile-quickguide/Agile-QuickGuide05-일감크기추정/)
- [Release Planning](https://engineering-skcc.github.io/agile-quickguide/Agile-QuickGuide06-릴리즈계획/)
- [품질/소통 Planning](https://engineering-skcc.github.io/agile-quickguide/Agile-QuickGuide07-소통&품질/)
- 테스트 계획 수립 
- 데이터 이행 계획 수립 

## SPRINT #1~N
- [Sprint Planning](https://engineering-skcc.github.io/agile-quickguide/Agile-QuickGuide11-스프린트계획/)
- [Daily Scrum Meeting](https://engineering-skcc.github.io/agile-quickguide/Agile-QuickGuide12-스크럼미팅/)
- BackEnd 설계하기 
  - [내부구조정의](/contents/mspackage.md) 
  - [API설계](/contents/API.md) 
  - [도메인모델링](/contents/domain.md) 
  - [데이터모델링](/contents/data.md) 
 - [FrontEnd 설계하기](https://engineering-skcc.github.io/microservice%20modeling/FrontEnd-modeling/)
- [BackEnd 마이크로서비스 개발](/contents/backEnddomain.md) 
- [FrontEnd 개발](/contents/jhipster-front1.md)
- [시연수행: 로컬 테스트](/contents/backend_localtest.md)
- [DevOps](/contents/devops.md)
- [지속적 통합과 지속적 배포 (CI/CD)](/contents/cicd.md) 
- [Sprint 리뷰](https://engineering-skcc.github.io/agile-quickguide/Agile-QuickGuide13-스프린트리뷰/)
- [Sprint 회고](https://engineering-skcc.github.io/agile-quickguide/Agile-QuickGuide14-스프린트회고/)
- [성과측정/분석](https://engineering-skcc.github.io/agile-quickguide/Agile-QuickGuide15-성과측정/)
- 데이터 이행 프로그램 설계
- 데이터 이행 리허설

## SPRINT FINAL
- [통합테스트](https://engineering-skcc.github.io/msa/마이크로서비스테스트/)
- [성능테스트](https://engineering-skcc.github.io/performancetest/Cloud-환경-성능부하테스트/)
- 데이터 이행

## 컨텐츠 및 교육교재
- [기술블로그 : MSA개념 및 주요패턴](https://engineering-skcc.github.io/tags/microservice/)
- [전사교육과정: 마이크로서비스 모델링:(준비중)]
  - 마이크로로시스템 개념 및 MSA 패턴이해:교육교재 1 
  - 이벤트스토밍을 활용한 마이크로서비스 설계: 교육교재 2
  - Workflowy (실습 스크립트 LookUp): https://workflowy.com/#/217cf1148297
   

