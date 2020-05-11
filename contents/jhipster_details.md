# Jhipster Project Structure

1. Microservice Application Structure
   
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

- **Sources : Build System & Configuration**
  - pom.xml : Maven 프로젝트이기 때문에 Build와 관련된 java version, dependencies에 관한 정보들이 포함되어있다.
  - package.json : Angular 또한 build file을 포함하고 있는데, 이와 관련된 dependencies는 package.json 파일에 포함되어있다.
  - YAML configuration files : src/main/resources/config/
    - Jhipster는 YAML 형식의 파일만 configuration 파일로 사용한다.
      - application.yml : common config
      - application-dev.yml : 개발 관련 config로 swagger, Spring Boot Developer Tools, H2 Setup info 등이 포함되어있다.
  - Server configuration files : src/main/docker/
    - src/main/docker/의 하위파일 : Jhipster-registry, kafka, mariaDB configuration 파일이 포함되어있다.
    - src/main/docker/central-server-config : Docker 환경과 Local 환경의 configuration 파일이 포함되어있다. 파일에는 공통 정보로 JWT와 Eureka 정보가 있다.
- **Sources : java**
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
  - DTO : /src/main/java/com/skcc/rental/service/dto에서 확인할 수 있다. 
    - Mapstruct : Mapstruct는 JPA entities와 DTOs간의 번역하는 라이브러리 역할을 한다. Mapstruct의 경우 EntityNameMapper의 형식으로 생성되며, /src/main/java/com/skcc/rental/service/mapper에서 확인할 수 있다.
  - Service interface
  - Service implementation
  - Rest Controller


     






----------------------추가해야될 내용-------------------

- Jhipster 프로젝트 생성 후 내부 구조 및 클래스, config 내용 설명, 정제해야될 파일 확인


- JDL 문법 정리
- Jhipster 프로젝트 내에 Business Logic 처리 방법 
