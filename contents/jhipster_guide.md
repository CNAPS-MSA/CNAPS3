# Jhipster를 활용한 MSA 외부아키텍처 구성(게이트웨이,레지스터)

## Jhipster란 무엇인가?

![image](https://user-images.githubusercontent.com/18453570/81132833-b26d7c80-8f8a-11ea-8ce5-95b841fe6fae.png)

**Jhipster**란, modern 웹 애플리케이션과 마이크로서비스 아키텍처를 빠르게 적용, 개발, 배포할 수 있도록 도와주는 오픈소스 개발 플랫폼이다.
지원해주는 영역은 아래와 같다.

- Front-end 영역 지원 : Angular, React, Vue.js
- Back-end 영역 지원 : Spring Boot (Java & Kotlin 포함), Micronaut, Quarkus, Node.js, and .NET.
- Deployment 영역 지원 : Docker and Kubernetes for AWS, Azure, Cloud Foundry, Google Cloud Platform, Heroku, and OpenShift.

## Jhipster의 Goal

Jhipster 사용의 목적은 완전하고 현대적인 웹 애플리케이션과 마이크로서비스 아키텍처의 생성이며 다음과 같은 항목들을 통합하는 것이 목표이다.

- 광범위한 테스트를 커버할 수 있는 우수한 성능의 강력한 서버 스택
- 세련되고 현대적인 모바일 친화적 UI를 위한 Angular, React, Vue + Bootstrap를 갖춘 CSS
- Webpack 및 Maven 또는 Gradle을 사용하여 애플리케이션을 빌드하는 강력한 워크 플로우
- 클라우드에 빠르게 배포할 수있는 코드 기반 인프라
  
실제로 Jhipster는 설치가 간편하고, Directory 생성 후 몇가지 옵션을 선택하면 바로 실행가능한 웹 애플리케이션으로 만들어진다. 모놀리틱, 마이크로서비스, 게이트웨이를 선택할 수 있고 그 외에 Swagger나 Docker, kafka, JPA 등 현대적인 애플리케이션 개발에 필요한 환경설정과 라이브러리를 자동으로 설치해준다. 또한 기본적인 인증처리, Rest API를 이용한 통신 내용이 포함되어있다.
## Jhipster Microservice Architecture

![image](https://user-images.githubusercontent.com/18453570/81143306-806d1200-8fac-11ea-9c7f-3506f51fccd8.png)

- JHipster 레지스트리 : MSA의 필수요소, 다른 모든 구성요소를 서로 연결하고 서로 통신할 수 있게 함

- Microservice : 백엔드 코드가 들어있고, 실행 후 도메인에 대한 API를 노출. 여러 마이크로서비스로 구성 될 수 있으며 몇 개의 엔티티와 비즈니스 규칙이 포함된다.

- Gateway : 모든 프론트 엔드 코드를 가지고 있으며 전체 마이크로 서비스에서 생성한 API 를 사용한다.

- Back-End : src/ main / java 폴더에 존재한다.

- Front-End : src / main /webapp 폴더에 존재하고, Angular JS 모듈의 대부분을 포함한다.

-----------상세내용추가-----------

> [Jhipster Details](/contents/jhipster_details.md): 서비스 생성 시 선택한 옵션 및 Jhipster 코드 상세 Detail 설명을 확인할 수 있다.


# Jhipster 환경구축 (Mac & Windows 동일)

1. Java 11 설치 ->  [AdoptOpenJDK builds](https://adoptopenjdk.net/)
2. Node.js 설치 -> **반드시 LTS 64-bit version 설치** [Node.js website](https://nodejs.org/en/) 
3. JHipster 설치 -> 
```
npm install -g generator-jhipster
```

외에 Git, Docker은 애플리케이션 실행 및 사용환경에 따라 필요한 경우에 설치한다.

- git : https://git-scm.com/
      : 샘플에서는 코드 버전관리를 위해 사용하였으며, 코드 버전관리를 하고 싶다면 설치하는 것을 권장한다. 
- Docker : https://www.docker.com/products/docker-desktop
         : 카프카, mongoDB 등 애플리케이션 동작을 위한 필수 라이브러리 실행을 위해 사용하였다. 반드시 설치 완료 및 실행할 수 있도록 준비해두어야한다.

## MacOS

Mac을 사용하는 경우 brew를 사용하여 Jhipster를 설치할 수 있다. 

```
brew install jhipster
```

> 참조 : [Jhipster공식 설치가이드 링크](https://www.jhipster.tech/installation/)

# Jhipster로 개발 시작하기

Jhipster로 Microservice Application을 개발시 순서는 아래와 같다.

1. gateway 만들기
  1. Registry 만들기
2. Microservice 만들기
3. 생성한 Microservice에 Entity 생성하기
4. 생성된 Entity를 gateway가 인식할 수 있도록 gateway에 등록하기

따라서, Sample 프로젝트에서는 book, bookCatalog, rental 3개의 Microservie를 생성하고 gateway에 등록해 Simple CRUD게시판까지 연결해보자.

> 참고 : 개발 환경은 MacOS이며, java 11 사용

## Gateway 만들기

1. gateway 폴더 생성
2. gateway 폴더를 jhipster 프로젝트로 변경

![image](https://user-images.githubusercontent.com/18453570/81142695-42232300-8fab-11ea-88a2-50a2cf6d900b.png)

```
mkdir gateway
cd gateway
jhipster
```

3. 옵션 선택

![image](https://user-images.githubusercontent.com/18453570/81142883-c2e21f00-8fab-11ea-80eb-dee5b067068a.png)

---------------------- 옵션 설명 ------------------------

## Registry 실행 및 gateway 실행

gateway Directory 내에서 아래 Command를 실행시킨다.

```
docker-compose -f src/main/docker/jhipster-registry.yml up
```
아래 이미지처럼 실행되었다면 정상이다.
이미지에서 보이는 주소로 접속해 Registry가 정상적으로 작동하는지 확인해본다.

![image](https://user-images.githubusercontent.com/18453570/81143823-b8c12000-8fad-11ea-83a3-112844067438.png)

localhost:8761로 접속해보면, 로그인 창이 뜨는데 이때 Id와 Pw는 둘다 `admin`을 입력한다.

![image](https://user-images.githubusercontent.com/18453570/81143960-0b9ad780-8fae-11ea-9f6f-8d358c46547d.png)

위 이미지처럼 보인다면 Registry가 정상적으로 등록된 것이다.

Jhipster Registry가 무엇인지는 아래 링크를 통해 설명하였다.

- [Jhipster Registry 알아보기](/contents/jhipster_registry.md)

이제 gateway를 실행시켜보자.
build tool을 maven으로 선택하였기 때문에 gateway Directory 내에서 아래 command를 입력한다. (Registry가 실행중인 창은 닫지 않고, 새 창을 열어 입력한다.)

```
./mvnw
```
아래 이미지처럼 실행되었다면 정상이다. 
이미지에 보이는 주소로 접속해 gateway가 정상적으로 작동하는지 확인해본다.
![image](https://user-images.githubusercontent.com/18453570/81144345-cb882480-8fae-11ea-8417-967b21ece2f4.png)

localhost:8080으로 접속한다. 

![image](https://user-images.githubusercontent.com/18453570/81144436-02f6d100-8faf-11ea-8cbe-359bdbdd828a.png)

Registry와 마찬가지로 Id와 Pw 둘다 `admin`을 입력해 로그인이 가능하다.
admin은 관리자 권한을 가지고 있으며, admin으로 로그인 시 상단 메뉴 바가 아래 이미지처럼 변경된다.

![image](https://user-images.githubusercontent.com/18453570/81144563-4e10e400-8faf-11ea-81c2-e1e5d463ad7a.png)

메뉴를 탐색해보면, Entities에 아무것도 등록되지 않았을 것이다. 아직 마이크로서비스와 엔티티를 생성/등록하지 않았기 때문이다. 이제 마이크로서비스를 만들고 entity를 등록해보자.

-> [백엔드 서비스 개발](/contents/jhipster_guide2.md)


**끝!**



