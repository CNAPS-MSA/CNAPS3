# 내부아키텍처정의
- [기존어플리케이션구조 문제점인식](https://engineering-skcc.github.io/microservice%20inner%20achitecture/inner-architecture-1/)
- [클린아키텍처와 헥사고널](https://engineering-skcc.github.io/microservice%20inner%20achitecture/inner-architecture-2/)
- [마이크로서비스 내부구조](https://engineering-skcc.github.io/microservice%20inner%20achitecture/inner-architecture-2/)
- [프론트엔드 패턴](https://engineering-skcc.github.io/microservice%20outer%20achitecture/inner-architecture-1/)

## 1.BackEnd Microservice Inner Architecture
### 헥사고널 + DDD적용
![백엔드아키텍처](https://github.com/CNAPS-MSA/CNAPS3/blob/master/img/BackEndA.png)  
### 헥사고널 + 트랜젝션스크립트
추가예정

## 2.FrontEnd Architecture
- 채널요건 고려 
  - 웹 또는 모바일, 비지니스 업무 환경 고려
- 프론트엔드 프레임웍 선택
  - 프론트엔드는 매우 다양한 선택지가 존재함
- 세부 기술정의 
  - 백엔드와의 통신 방법 정의
- 표준 UI 유형 정의

## 3.아키텍처 스파이크 수행
- PoC나 파일럿이라도 호칭할 수 있는 아키텍처 검증을 위한 활동
- 정의된 내외부,프론트,백엔드 아키텍처를 구현하여 검증
- 검증범위,목적,환경,시나리오(검증항목)등이 정의되어야 함

# 사례:도서대여시스템
## Jhipster를 활용한 마이크로서비스 내부아키텍처 구현
### 내부아키텍처 구현
- 백엔드 서비스의 구조를 정의하고 서비스를 생성해 본다.
  - [Jhipster사용하여 백엔드 서비스 프로젝트 구조 생성](/contents/jhipster_guide2.md)
- Jhipster가 생성한 구조는 **헥사고널 아키텍처 + DDD** 의 기본 사상을 만족하나 DTO의 위치 및 몇가지 수정이 필요해 보인다. 아래와 같이 좀더 바람직하도록 수정하였다.
  - [Package Refactoring](/contents/jhipster_package_ref.md)
- 헥사고널+트랜젝션스크립트 패키지구조는  게시판 서비스 구현을 통해 추후 살펴보자.




