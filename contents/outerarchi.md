# 외부아키텍처정의
- 외부아키텍처 변화흐름
![ms3.0](https://github.com/CNAPS-MSA/CNAPS3/blob/master/img/MSA3.0.png)  

- [내외부아키텍처란?](https://engineering-skcc.github.io/microservice%20%EA%B0%9C%EB%85%90/modern-relactive/) 
    - [인프라 패턴](https://engineering-skcc.github.io/microservice%20outer%20achitecture/outer-architecture-1/)
    - [클라우드플랫폼의 이해](https://engineering-skcc.github.io/microservice%20outer%20achitecture/outer-architecture-2/)
- 플랫폼패턴
    - [다양한 서비스 등록, 탐색 위한 Service registry , Service discovery 패턴](https://engineering-skcc.github.io/microservice%20outer%20achitecture/outer-arch-registry/)
    - [서비스 단일 진입을 위한 API 게이트웨이(gateway) 패턴 & BFF(Backend For Frontend) 패턴](https://engineering-skcc.github.io/microservice%20outer%20achitecture/outer-arch-api-gw/)
    - [외부 구성 저장소 패턴](https://engineering-skcc.github.io/microservice%20outer%20achitecture/outer-arch-config/)
    - [마이크로서비스 인증/인가 패턴](https://engineering-skcc.github.io/microservice%20outer%20achitecture/outer-arch-Auth/)
    - [장애 및 실패 처리를 위한 서킷 브레이크 패턴](https://engineering-skcc.github.io/microservice%20outer%20achitecture/outer-arch-Circuit-breaker/)
    - [모니터링과 추적 패턴](https://engineering-skcc.github.io/microservice%20outer%20achitecture/outer-arch-monitoring/) 
    - [중앙화된 로그 집계 패턴](https://engineering-skcc.github.io/microservice%20outer%20achitecture/outer-arch-log/) 
    - [서비스 매시(Service Mesh) 패턴](https://engineering-skcc.github.io/microservice%20outer%20achitecture/outer-arch-Service-Mesh/)
- 마이크로서비스관계 패턴
    - [UI패턴](https://engineering-skcc.github.io/microservice%20outer%20achitecture/inner-architecture-1/)
    - [통신패턴과 EDA](https://engineering-skcc.github.io/microservice%20outer%20achitecture/inner-architecture-conn/)
    - [분산트랜젝션이슈-SAGA](https://engineering-skcc.github.io/microservice%20outer%20achitecture/inner-architecture-saga/)
    - [CQRS](https://engineering-skcc.github.io/microservice%20outer%20achitecture/inner-architecture-cqrs/)
    - [이벤트소싱](https://engineering-skcc.github.io/microservice%20outer%20achitecture/inner-architecture-Event-Sourcing/)
## 1.유형
### Public 클라우드서비스
VM 및 자체컨테이너기반 솔루션
- [AWS](https://aws.amazon.com/ko/?nc2=h_lg)
- [Azure](https://azure.microsoft.com/ko-kr/)
- [GCP](https://cloud.google.com/)

### K8s기반
관리형 쿠버네티스
- [AWS EKS](https://aws.amazon.com/ko/eks/)
- [Azure AKS](https://docs.microsoft.com/ko-kr/azure/aks/)
- [Google GKE](https://cloud.google.com/kubernetes-engine?hl=ko)
- [CloudZ](https://www.cloudz.co.kr/product/cloudZcp)
- 통신부분 

### spring cloud + netflix OSS기반
어플리케이션을 통한 MSA아키텍처 구성
- [spring cloud](https://spring.io/projects/spring-cloud)

## 사례

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

## 백엔드 마이크로서비스 구현
### 백엔드서비스 구현 전, Configuration 설정
- [벡엔드서비스 구현전에 각 service를 gateway의 end-point에 등록해야 한다.](/contents/endpointadd.md)


### 쿠버네티스 기반 최종 도서대여시스템 구성도
![도서대여시스템](https://github.com/CNAPS-MSA/CNAPS3/blob/master/img/ac.jpg)  
 
