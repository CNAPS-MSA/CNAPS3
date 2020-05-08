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

위 이미지처럼 현재 동작 중인 서비스를 확인할 수 있다. 

### Jhipster Registry's Spring cloud config



It is a an Eureka server, that serves as a discovery server for applications. This is how JHipster handles routing, load balancing and scalability for all applications.
It is a Spring Cloud Config server, that provide runtime configuration to all applications.
It is an administration server, with dashboards to monitor and manage applications.
All those features are packaged into one convenient application with a modern Angular-based user interface



>참조: [Jhipster Registry 공식문서](https://www.jhipster.tech/jhipster-registry/)
