# Jhipster Registry 소개

Jhipster Registry는 Jhipster 개발자들이 제공하는 runtime application이다. Jhipster Registry도 Jhipster처럼 오픈소스이며, 현재 소스는 github을 통해 사용가능하다. -> [jhipster registry source](https://github.com/jhipster/jhipster-registry)

Jhipster는 프로젝트에 사용되는 모든 애플리케이션을 등록하고 애플리케이션의 configuration을 가져온다. 또한, 애플리케이션의 상태를 모니터링 할 수 있는 dashboard도 포함되어있다.

또한, Jhipster Registry는 코드를 수정할 필요없이, registry가 실행 중인 상태에서 gateway를 먼저 동작시키고 service application들을 띄우면 Jhipster Registry가 해당 service들을 **자동으로** 등록한다. 

## Jhipster Registry 사용 목적

Jhipster Registry는 크게 3가지 목적으로 사용된다.

1. Eureka Server
2. Spring Cloud Config
3. Monitoring Dashboard

### Jhipster Registry's Eureka Server

Jhipster Registry는 **Netflix Eureka Server**로 구성되어있고, 모든 애플리케이션의 routing, load balancing, scalability를 포함한 서비스 검색 기능을 제공한다.

MSA에선 필수로 사용되는 것으로, gateway가 어떤 Microservice와 instance가 실행가능한지 알게 해준다.
또한, Monoliths를 포함한 모든 애플리케이션의 Hazelcast 분산 캐시를 자동으로 확장해주는 기능도 포함한다.
> [Hazelcast cache 공식문서](https://www.jhipster.tech/using-cache/)

![image](https://user-images.githubusercontent.com/18453570/81362681-5f2b3380-911c-11ea-8fe4-5a1178cdb7e3.png)

Jhipster Registry를 실행한 후, localhost:8761로 접속하면 현재 동작 중인 Microservice의 상태를 확인할 수 있다. 

### Jhipster Registry's Spring cloud config

Jhipster Registry에서 제공하는 Spring cloud config는 Spring에서 제공하는 Spring cloud config와 동일하다. (Spring cloud config에 관한 자세한 설명은 생략)

Jhipster는 프로젝트 생성 시, configuration에 필요한 application-*.yml file들을 모두 생성해준다.
생성되는 yml파일은 아래와 같다.

- application.yml
- application-dev.yml
- application-prod.yml
- application-tls.yml
- bootstrap.yml
- bootstrap-prod.yml

Jhipster Registry와 Spring cloud config는, application이 실행되었을 때 Jhipster Registry에 연결하여 application의 configuration을 가져오는 방식으로 동작한다. 

### Monitoring Dashboard

Jhipster Registry는 모든 타입의 application에 관리자용 Dashboards를 제공한다. Application이 Eureka server에 등록되면 바로 dashboard에서 확인이 가능하다.

이 dashboard에는 애플리케이션의 중요한 정보들을 포함하고 있는데, 이 정보들에 접근하기 위해 Jhipster Registry는 JWT 토큰을 사용한다. JWT 토큰은 애플리케이션 생성 시, 옵션에서 선택할 수 있다. 따라서, Jhipster Registry를 사용하기 위해선 JWT토큰 옵션 사용에 동의해야만 한다.


1. The metrics dashboard

![image](https://www.jhipster.tech/images/jhipster-registry-metrics.png)

The metrics dashboard에서 제공하는 세부정보는 아래와 같다.

- JVM
- HTTP Requests
- cache usage
- database connection pool

2. The health dashboard

![image](https://www.jhipster.tech/images/jhipster-registry-health.png)

The health dashboard는 Spring Boot Actuator의 health 엔드 포인트를 사용하여 애플리케이션의 다양한 부분에 대한 health 정보를 제공한다. Spring Boot Actuator는 많은 상태 점검을 기본적으로 제공하며 애플리케이션 별 상태 점검을 추가 할 수 있다.

3. The configuration dashboard

![image](https://www.jhipster.tech/images/jhipster-registry-configuration.png)

The configuration dashboard는 Spring Boot Actuator의 configuration 엔드 포인트를 사용하여 현재 애플리케이션의 Spring configuration에 대한 전체정보를 제공한다.

4. The logs dashboard

![image](https://www.jhipster.tech/images/jhipster-registry-logs.png)

The logs dashboard를 사용하면 런타임 시 실행 중인 application의 Logback configuration을 관리 할 수 있다. 버튼을 클릭하여 Java 패키지의 로그 레벨을 변경할 수 있고, 개발 및 프로덕션 상태에서 유용하게 활용할 수 있는 기능이다.

>참조: [Jhipster Registry 공식문서](https://www.jhipster.tech/jhipster-registry/)
