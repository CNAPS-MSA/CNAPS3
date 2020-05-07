# Jhipster를 활용한 Microservice application 개발

## Jhipster란 무엇인가?

![image](https://user-images.githubusercontent.com/18453570/81132833-b26d7c80-8f8a-11ea-8ce5-95b841fe6fae.png)

**Jhipster**란, modern 웹 애플리케이션과 마이크로서비스 아키텍처를 빠르게 적용, 개발, 배포할 수 있도록 도와주는 오픈소스 개발 플랫폼이다.
지원해주는 영역은 아래와 같다.

- Front-end 영역 지원 : Angular, React, Vue.js
- Back-end 영역 지원 : Spring Boot (Java & Kotlin 포함), Micronaut, Quarkus, Node.js, and .NET.
- Deployment 영역 지원 : Docker and Kubernetes for AWS, Azure, Cloud Foundry, Google Cloud Platform, Heroku, and OpenShift.

## Jhipster의 Goal

Jhipster 사용의 목적은 완전하고 현대적인 웹 애플리케이션과 마이크로서비스 아키텍처의 생성이며 다음과 같은 항목들을 통합하는 것이 목표이다.

- 광범위한 테스트를 커버할 수 있는 우수한 성능으 강력한 서버 스택
- 세련되고 현대적인 모바일 친화적 UI를 위한 Angular, React, Vue + Bootstrap를 갖춘 CSS
- Webpack 및 Maven 또는 Gradle을 사용하여 애플리케이션을 빌드하는 강력한 워크 플로우
- 클라우드에 빠르게 배포할 수있는 코드 기반 인프라
  
실제로 Jhipster는 설치가 간편하고, Directory 생성 후 몇가지 옵션을 선택하면 바로 실행가능한 웹 애플리케이션으로 만들어진다. 모놀리틱, 마이크로서비스, 게이트웨이를 선택할 수 있고 그 외에 Swagger나 Docker, kafka, JPA 등 현대적인 애플리케이션 개발에 필요한 환경설정과 라이브러리를 자동으로 설치해준다. 또한 기본적인 인증처리, Rest API를 이용한 통신 내용이 포함되어있다.

## Jhipster 프로젝트의 Microservice 구성요소

- JHipster 레지스트리 : MSA의 필수요소, 다른 모든 구성요소를 서로 연결하고 서로 통신할 수 있게 함

- Microservice : 백엔드 코드가 들어있고, 실행 후 도메인에 대한 API를 노출. 여러 마이크로서비스로 구성 될 수 있으며 몇 개의 엔티티와 비즈니스 규칙이 포함된다.

- Gateway : 모든 프론트 엔드 코드를 가지고 있으며 전체 마이크로 서비스에서 생성한 API 를 사용한다.

- Back-End : src/ main / java 폴더에 존재한다.

- Front-End : src / main /webapp 폴더에 존재하고, Angular JS 모듈의 대부분을 포함한다.

## Jhipster Architecture

![image](https://user-images.githubusercontent.com/18453570/81143306-806d1200-8fac-11ea-9c7f-3506f51fccd8.png)


# Jhipster 환경구축 (Mac & Windows 동일)

1. Java 11 설치 ->  [AdoptOpenJDK builds](https://adoptopenjdk.net/)
2. Node.js 설치 -> **반드시 LTS 64-bit version 설치** [Node.js website](https://nodejs.org/en/) 
3. JHipster 설치 -> 
```
npm install -g generator-jhipster
```

외에 Git, Docker은 애플리케이션 실행 및 사용환경에 따라 필요한 경우에 설치한다.

- git : https://git-scm.com/
- Docker : https://www.docker.com/products/docker-desktop

## MacOS

Mac을 사용하는 경우 brew를 사용하여 설치할 수 있다. 

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

따라서, Sample 프로젝트에서는 book, user, rental 3개의 Microservie를 생성하고 gateway에 등록해 Simple CRUD게시판까지 연결해보자.

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


## Service 만들기

먼저, 도서대여시스템의 core service 인 book, user, rental 서비스부터 생성한다.

이때 service를 만드는 방식은 동일하나, port와 package명을 다르게 해야한다.

- book :
  - port: 8081
  - package : com.skcc.book
- user :
  - port : 8082
  - package : com.skcc.user
- rental :
  - port : 8083
  - package : com.skcc.rental

### core service 1 : book

1. book 폴더 생성
2. book 폴더를 jhipster 프로젝트로 변경

![image](https://user-images.githubusercontent.com/18453570/81146423-38052280-8fb3-11ea-8397-bd615fa6f08b.png)

```
mkdir book
cd book
jhipster
```
3. 옵션 선택

**port설정과 package 설정을 잊지말자**

![image](https://user-images.githubusercontent.com/18453570/81146788-ec9f4400-8fb3-11ea-90ee-5e3f5a4d860a.png)

-------------------------옵션선택 설명--------------------

### core service 2 : user

core service 1의 book과 같은 방식으로 user service를 생성한다.

**port설정과 package 설정을 잊지말자**

```
mkdir user
cd user
jhipster
```

- 옵션 선택

![image](https://user-images.githubusercontent.com/18453570/81147097-8c5cd200-8fb4-11ea-98b4-88dfae9d1059.png)


### core service 3 : rental

core service 1의 book과 같은 방식으로 rental service를 생성한다.

**port설정과 package 설정을 잊지말자**

```
mkdir rental
cd rental
jhipster
```

![image](https://user-images.githubusercontent.com/18453570/81147568-861b2580-8fb5-11ea-8f0c-545d23d60041.png)

## service 실행시키기

이제 생성했던 서비스들을 실행시켜 registry를 확인해 정상적으로 동작하는지 확인해본다.

아래의 이미지처럼 각 서비스의 Directory에 들어가 `./mvnw`를 입력해 서비스를 실행시킨다.

![image](https://user-images.githubusercontent.com/18453570/81147751-f2962480-8fb5-11ea-9283-0aa58fbd15c6.png)

먼저, 각 서비스의 Directory에 아래와 같은 이미지가 뜨면 정상적으로 실행된 것이다.

1. book

![image](https://user-images.githubusercontent.com/18453570/81148014-69cbb880-8fb6-11ea-93c2-16c1aaa9aa49.png)

2. user

![image](https://user-images.githubusercontent.com/18453570/81148038-7b14c500-8fb6-11ea-8f54-dc98ec8851a5.png)

3. rental

![image](https://user-images.githubusercontent.com/18453570/81148066-88ca4a80-8fb6-11ea-9cb4-797e8061408a.png)

그 다음, localhost:8761에 접속해 Registry를 확인해보면 gateway, book, user, rental 서비스가 등록되어있는 것을 볼 수 있다.

![image](https://user-images.githubusercontent.com/18453570/81148406-2faee680-8fb7-11ea-92dc-22cfcaeee6ae.png)

## 생성한 service에 Entity를 추가하기

생성한 service에 Entity를 추가하는 방법은 크게 2가지로 나눌 수 있다.

- [Jhipster Online](https://start.jhipster.tech/#/) 을 통해 Entity 및 Relationship을 설정할 수 있다.
- Terminal에 command를 입력해 Entity 및 Relationship을 설정할 수 있다. 


1. book service에 book entity 생성

book Directory에 들어가 아래 command를 입력하여 book Entity를 생성한다.
생성 시, author, title, description으로 책 저자, 책 이름, 책 설명을 변수로 선언하고 이는 반드시 입력되어야하는 값이기 때문에 required로 하였다.

```
cd book
jhipster entity book
```

그 다음 entity에 추가할 변수와 그 변수의 자료형, 옵션, 다른 entity와의 관계를 설정할 수 있다. 

![image](https://user-images.githubusercontent.com/18453570/81156459-ecf20c00-8fc0-11ea-92ad-6e33ce9753ad.png)

entity생성과 변수 설정 완료 후, 추가적인 옵션을 선택할 수 있는데 그 선택은 아래와 같이 한다. 

![image](https://user-images.githubusercontent.com/18453570/81157507-0182d400-8fc2-11ea-94c6-a707e2595ebd.png)

마지막으로 master.xml을 overwrite할 것이냐는 옵션이 나오는데, 이때 y를 선택해야한다.

![image](https://user-images.githubusercontent.com/18453570/81157614-1ceddf00-8fc2-11ea-8a41-34def05dcb46.png)

2. user service에 user entity 생성

book service와 마찬가지로 진행한다.
Entity 생성 시, 변수로는 name과 email을 선언했다.

```
cd user
jhipster entity user
```

3. rental service에 rental entity와 rentalItem entity생성

이번에는 terminal이 아닌 Jhipster Online에서 entity와 relationship을 설정해보았다.

- rental Entity 생성 시, 변수로 userId, rentalCnt,  rentalStatus를 선언하였다. rentalStatus는 rental의 상태를 나타내는 값으로 enum으로 처리했다.
- rentalItem Entity 생성 시, 변수로 bookId와 rentalItemStatus를 선언하였다. rentalItemStuts는 현재 대출 중인 책의 상태를 나타내는 값으로 enum 처리하였다.

이때, rental 과 rentalItem은 oneToMany관계인데, 이를 rentalItem과 rental 관계로 바꿔 ManyToOne으로 설정하였다.


Jhipster Online에 접속해 `Design Entities`을 클릭-> `Create a new JDL model`버튼을 클릭하여 JDL studio에 접속해 아래와 같이 코드를 입력한다.

```
entity Rental{
	id Long,
	userId Long,
    rentalItemCnt Integer ,
    rentalStatus RentalStatus
}

entity RentalItem {
	id Long,
    bookId Long,
    rentalItemStatus RentalItemStatus
}


enum RentalStatus {
    OK, RENTALED, OVERDUE
}

enum RentalItemStatus{
	RENTALED, OVERDUE
}

// defining ManyToOne relationships


relationship ManyToOne {
	RentalItem{rental} to Rental
}

// Set pagination options
paginate * with pagination 

// Use Data Transfert Objects (DTO)
dto * with mapstruct

// Set service options to all except few
service all with serviceImpl 

```

코드를 저장한 후, 해당 파일을 내려받아 rental Directory의 최상위로 옮긴다.

그 다음 terminal에서 아래 command를 입력해 import 한다.

```
jhipster import-jdl ./my-jdl-file.jdl --force
```

## 추가한 entity를 gateway에 등록시키기

마지막으로 지금까지 추가한 entity들을 gateway에 등록시켜야한다.
book, user, rental, rentalItem을 차례로 등록시킨다.


```
cd gateway
jhipster entity book
```
위 command입력을 통해 book entity를 gateway에 추가시킬 수 있다.

입력 후, 몇가지 옵션을 선택하게 되는데 아래 이미지와 같이 선택한다.

이때 Entity의 path는 각 microservice가 존재하는 path로 수정해야한다. 

![image](https://user-images.githubusercontent.com/18453570/81278941-60634e80-9091-11ea-8e61-ac87302aa29a.png)

이후, overwrite하겠냐는 질문에는 `a`를 입력해야한다.


# 재실행 후, 테스트

이제 entity 생성까지 모두 마쳤으니, 다시 모든 application을 재실행시켜 제대로 동작하는지 확인한다.

먼저 gateway 폴더에서 아래와 같은 command를 입력한다.

```
docker-compose -f src/main/docker/jhipster-registry.yml up
```

새로운 terminal 창에서 다시 gateway 폴더에 들어가 아래 command를 입력한다.

```
./mvnw
```

book, user, rental 서비스 또한 실행시켜준다.

```
cd book
./mvnw

cd user
./mvnw

cd rental
./mvnw
```

이제 모두 잘 동작하는지 확인해본다.

1. Jhipster Registry 확인

- localhost:8761에 접속하여 아래 이미지처럼 뜨는지 확인

![image](https://user-images.githubusercontent.com/18453570/81279412-12027f80-9092-11ea-95f5-cfe93233ab99.png)

2. Gateway 확인

- localhost:8080에 접속하여 `admin`으로 접속하고, 아래 이미지처럼 뜨는지 확인

![image](https://user-images.githubusercontent.com/18453570/81279558-44ac7800-9092-11ea-9bcc-8a3d574f9b86.png)
![image](https://user-images.githubusercontent.com/18453570/81279591-4d9d4980-9092-11ea-9eed-8a2b0d79fb2c.png)
![image](https://user-images.githubusercontent.com/18453570/81279612-555cee00-9092-11ea-857a-9498192eeb25.png)
![image](https://user-images.githubusercontent.com/18453570/81279633-5beb6580-9092-11ea-8b51-dd705a32d9bc.png)
![image](https://user-images.githubusercontent.com/18453570/81279659-6443a080-9092-11ea-8636-62698889edd5.png)

위 이미지처럼 Jhipster가 Fake DB를 repository에 저장하여 화면을 보여준다. 물론, FakeDB는 추후 삭제할 수 있다.
또, 마지막 rental Items 화면의 경우, 현재 서비스와 서비스끼리의 business logic을 추가하지 않았기 때문에 Rental부분에 빈 영역으로 나와도 정상이다.

**끝!**

----------------------추가해야될 내용-------------------

- Jhipster Registry에서 확인할 수 있는 내용 & 왜 사용해야하는지 
- Jhipster 프로젝트 생성 후 내부 구조 및 클래스, config 내용 설명, 정제해야될 파일 확인
- JDL 문법 정리
- Jhipster 프로젝트 내에 Business Logic 처리 방법 


