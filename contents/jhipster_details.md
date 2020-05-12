# Jhipster Project Structure

## 1. Microservice Application Structure
   
   Jhipster Sample에서 Rental, book, user를 Microservice Application으로 등록하였다.
   3가지 Application은 프로젝트 생성 시 옵션을 동일하게 선택하였기 때문에, 동일한 project structure를 가지고 있다.
   그 중 Rental Service Application의 Structure를 예시로 살펴본다.
   >파일 수가 많아 directory level까지만 표시하였다.


```
.
└── src
    └── main
        ├── docker
        │   ├── central-server-config
        │   │   ├── docker-config
        │   │   └── localhost-config
        │   ├── grafana
        │   │   └── provisioning
        │   │       ├── dashboards
        │   │       └── datasources
        │   └── prometheus
        ├── java
        │   └── com
        │       └── skcc
        │           └── rental
        │               ├── aop
        │               │   └── logging
        │               ├── client
        │               ├── config
        │               │   └── audit
        │               ├── domain
        │               │   └── enumeration
        │               ├── repository
        │               ├── security
        │               │   └── jwt
        │               ├── service
        │               │   ├── dto
        │               │   ├── impl
        │               │   └── mapper
        │               └── web
        │                   └── rest
        │                       ├── errors
        │                       └── vm
        ├── jib
        └── resources
            ├── config
            │   ├── liquibase
            │   │   ├── changelog
            │   │   └── fake-data
            │   └── tls
            ├── i18n
            ├── static
            └── templates

```

### - **Sources : Build System & Configuration**
  - pom.xml : Maven 프로젝트이기 때문에 Build와 관련된 java version, dependencies에 관한 정보들이 포함되어있다.
  - package.json : Angular 또한 build file을 포함하고 있는데, 이와 관련된 dependencies는 package.json 파일에 포함되어있다.
  - YAML configuration files : src/main/resources/config/
    - Jhipster는 YAML 형식의 파일만 configuration 파일로 사용한다.
      - application.yml : common config
      - application-dev.yml : 개발 관련 config로 swagger, Spring Boot Developer Tools, H2 Setup info 등이 포함되어있다.
  - Server configuration files : src/main/docker/
    - src/main/docker/의 하위파일 : Jhipster-registry, kafka, mariaDB configuration 파일이 포함되어있다.
    - src/main/docker/central-server-config : Docker 환경과 Local 환경의 configuration 파일이 포함되어있다. 파일에는 공통 정보로 JWT와 Eureka 정보가 있다.
### - **Sources : java**
  - Database
    - src/main/resources/config/liquibase/master.xml
        <img width="802" alt="image" src="https://user-images.githubusercontent.com/18453570/81539051-9a429680-93aa-11ea-9d26-a8139c682beb.png">
      - master.xml은 Jhipster Database 구조의 메인 config file 이다.
        - Red box : Database system에 따라 다른 타입과 functions의 정의를 담고 있다.
        - Green box : Table의 정의를 포함하고 있는데, Jhipster 테이블(000000~으로 시작)과 JDL 파일을 통해 생성된 entity 테이블(20200507~~~로 시작) 정보 모두 포함한다.
          ![image](https://user-images.githubusercontent.com/18453570/81540518-bba48200-93ac-11ea-8494-b87a5d67d11b.png)
          위 이미지는 Rental Entity 생성과 함께 만들어진 Table의 정보이다. primary key, parameter type, constraints 정보가 포함되어있는 것을 확인할 수 있다.
        - Blue box : index와 foreign key들의 정의가 포함된 파일이다.
          ![image](https://user-images.githubusercontent.com/18453570/81539844-c0b50180-93ab-11ea-8a2a-cc33c10a82c2.png)
          위 이미지는 Rental과 RentalItem간 관계를 추가했을 때 생성된 정보이다. Entity간 Relationship을 추가했을 때 생성되는 index와 정보를 포함하고 있음을 확인할 수 있다.
      - fake-data : Application을 최초 실행시켰을 때, DB를 추가하지 않았음에도 dummy DB가 들어가 있는 것을 확인했을 것이다. 이 dummy DB는 src/main/resourecs/config/liquibase/fake-data/에서 확인할 수 있다.
  - Entity : /src/main/java/com/skcc/rental/domain에서 확인할 수 있다. Enum은 domain 폴더 하위에 enumeration 폴더에 생성된다. 
  - Repository : /src/main/java/com/skcc/rental/repository에서 확인할 수 있다. JPA repository로 생성되는데, 기본 CRUD 외의 database queries는 없는 상태로 생성된다.
  - DTO : /src/main/java/com/skcc/rental/service/dto/에서 확인할 수 있다. 
    - Mapstruct : Mapstruct는 JPA entities와 DTOs간의 번역하는 라이브러리 역할을 한다. Mapstruct의 경우 EntityNameMapper의 형식으로 생성되며, /src/main/java/com/skcc/rental/service/mapper/에서 확인할 수 있다.
  - Service interface : /src/main/java/com/skcc/rental/service/에서 확인할 수 있다. Entity 생성시 선택한 옵션에 따라 Entity별로 Service interface가 생성된다. 하지만 Aggregate 단위로 Service interface를 구성해야하기 때문에 추후 Aggregate을 선정하여 해당 Aggregate 단위로 service interface를 재구성해야한다.
  - Service implementation : /src/main/java/com/skcc/rental/service/impl/ 에서 확인할 수 있다. Service interface와 마찬가지로 Entity 별로 생성되며, 기본적으로 Logging과 mapping 처리, 기본 CQRS가 구현되어있다. 추후 Aggregate 단위로 재구성한 Service interface 에 따라 implementation을 수정해야한다. 
  - Rest Controller : /src/main/java/com/skcc/web/rest/에서 확인할 수 있다. 각 Entity 별 controller와 kafka용 controller가 생성된다. 기본적으로 logging과 기본 CQRS가 구현되어있다.

## 2. Microservice Gateway Structure

Jhipster Sample은 Gateway에서 Application의 모든 Front-end를 구성한다. 따라서, Gateway의 structure를 확인해보면 Microservice application의 structure와 조금 다른 부분이 있음을 확인할 수 있다.

>파일 수가 많아 directory level까지만 표시하였다.

```
.
├── node
├── src
│   └── main
│       ├── docker
│       │   ├── central-server-config
│       │   │   ├── docker-config
│       │   │   └── localhost-config
│       │   ├── grafana
│       │   │   └── provisioning
│       │   │       ├── dashboards
│       │   │       └── datasources
│       │   └── prometheus
│       ├── java
│       │   └── com
│       │       └── skcc
│       │           └── gateway
│       │               ├── aop
│       │               │   └── logging
│       │               ├── client
│       │               ├── config
│       │               │   ├── apidoc
│       │               │   └── audit
│       │               ├── domain
│       │               ├── gateway
│       │               │   ├── accesscontrol
│       │               │   ├── ratelimiting
│       │               │   └── responserewriting
│       │               ├── repository
│       │               ├── security
│       │               │   └── jwt
│       │               ├── service
│       │               │   ├── dto
│       │               │   └── mapper
│       │               └── web
│       │                   └── rest
│       │                       ├── errors
│       │                       └── vm
│       ├── jib
│       ├── resources
│       │   ├── config
│       │   │   ├── liquibase
│       │   │   │   ├── changelog
│       │   │   │   └── data
│       │   │   └── tls
│       │   ├── i18n
│       │   └── templates
│       │       └── mail
│       └── webapp
│           ├── WEB-INF
│           ├── app
│           │   ├── config
│           │   ├── entities
│           │   │   ├── book
│           │   │   │   └── book
│           │   │   ├── rental
│           │   │   │   ├── rental
│           │   │   │   └── rental-item
│           │   │   └── user
│           │   │       └── user
│           │   ├── modules
│           │   │   ├── account
│           │   │   │   ├── activate
│           │   │   │   ├── password
│           │   │   │   ├── password-reset
│           │   │   │   │   ├── finish
│           │   │   │   │   └── init
│           │   │   │   ├── register
│           │   │   │   └── settings
│           │   │   ├── administration
│           │   │   │   ├── audits
│           │   │   │   ├── configuration
│           │   │   │   ├── docs
│           │   │   │   ├── gateway
│           │   │   │   ├── health
│           │   │   │   ├── logs
│           │   │   │   ├── metrics
│           │   │   │   └── user-management
│           │   │   ├── home
│           │   │   └── login
│           │   └── shared
│           │       ├── auth
│           │       ├── error
│           │       ├── layout
│           │       │   ├── footer
│           │       │   ├── header
│           │       │   ├── menus
│           │       │   └── password
│           │       ├── model
│           │       │   ├── book
│           │       │   ├── enumerations
│           │       │   ├── rental
│           │       │   └── user
│           │       ├── reducers
│           │       └── util
│           ├── content
│           │   ├── css
│           │   └── images
│           ├── i18n
│           │   ├── en
│           │   └── ko
│           └── swagger-ui
│               └── dist
│                   └── images
└── webpack
```

Gateway의 structure를 살펴보았을 때,  Microservice Application의 structure와의 가장 큰 차이점은 /src/main/아래 `webapp`이라는 폴더가 추가된 점이다. `webapp` 엔 Jhipster Client 코드가 포함되어있다. Jhipster Sample 프로젝트 구성요소 선택에서 React를 선택하였기 때문에 React style의 structure 및 file system을 따르고 있다.

Gateway의 Server contribution과 domain/repository/service 구성방식은 동일하며, domain/repository/service 내용을 살펴보면 `localhost:8080`으로 접속하였을 때를 위한 로그인/인증 방식과 user정보가 포함되어있다. 구현방식은 Microservice application의 구현방식과 동일하므로 설명하지 않는다.

### - Sources : React

`webapp` Directory 내용을 중점으로 다룰 것이기 때문에 webapp Directory 위주로 살펴보겠다. 아래는 Jhipster 공식 문서에서 설명하는 `React+Redux style`의 structure와 그 세부 내용 설명이다.
실제로 우리가 개발한 gateway를 살펴보면 이와 동일한 구조이다.

> [Jhipster React 관련 공식 문서](https://www.jhipster.tech/using-react/)

```
webapp
├── app                             - Your application
│   ├── config                      - General configuration (redux store, middleware, etc.)
│   ├── entities                    - Generated entities
│   ├── modules                     - Main components directory
│   │   ├── account                 - Account related components
│   │   ├── administration          - Administration related components
│   │   ├── home                    - Application homepage
│   │   └── login                   - Login related components
│   ├── shared                      - Shared elements such as your header, footer, reducers, models and util classes
│   ├── app.scss                    - Your global application stylesheet if you choose the Sass option
│   ├── app.css                     - Your global application stylesheet
│   ├── app.tsx                     - The application main class
│   ├── index.tsx                   - Index script
│   ├── routes.tsx                  - Application main routes
│   └── typings.d.ts                -
├── i18n                            - Translation files
├── static                          - Contains your static files such as images and fonts
├── swagger-ui                      - Swagger UI front-end
├── 404.html                        - 404 page
├── favicon.ico                     - Fav icon
├── index.html                      - Index page
├── manifest.webapp                 - Application manifest
└── robots.txt                      - Configuration for bots and Web crawlers

```

아래는 gateway와 service의 Entities를 연결 시 생성되는 내용이다. 

```
webapp
├── app                                        
│   └── entities
│       ├── foo                           - CRUD front-end for the Foo entity
│       │   ├── foo-delete-dialog.tsx     - Delete dialog component
│       │   ├── foo-detail.tsx            - Detail page component
│       │   ├── foo-dialog.tsx            - Creation dialog component
│       │   ├── foo.reducer.ts            - Foo entity reducer
│       │   ├── foo.tsx                   - Entity main component
│       │   └── index.tsx                 - Entity main routes
│       └── index.tsx                     - Entities routes    
└── i18n                                  - Translation files
     ├── en                               - English translations
     │   ├── foo.json                     - English translation of Foo name, fields, ...
     └── fr                               - French translations
         └── foo.json                     - French translation of Foo name, fields, ...
```

실제 우리가 개발한 sample gateway는 위와 동일한 구성임을 아래 이미지를 통해 확인할 수 있다.
i18n은 다국어를 위한 것인데, 우리는 Korean과 English를 선택하였기 때문에 ko과 en임을 확인할 수 있다.

<img width="442" alt="image" src="https://user-images.githubusercontent.com/18453570/81649956-118c2f00-946c-11ea-9b8d-a879b7da7896.png">


> React관련 설명 참고자료
> 1. https://medium.com/@ca3rot/아마-이게-제일-이해하기-쉬울걸요-react-redux-플로우의-이해-1585e911a0a6
> 2. https://github.com/piotrwitek/react-redux-typescript-guide/blob/master/README.md


----------------------추가해야될 내용-------------------
- JDL 문법 정리
- Jhipster 프로젝트 내에 Business Logic 처리 방법 
- 기본 제공 페이지 외 추가 페이지 만들어보기

