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

## 1.샘플
- 배포 구성도 
- 쿠버네티스 기반
 
