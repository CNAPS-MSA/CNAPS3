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


The JHipster Registry is a Spring Config Server: when applications are launched they will first connect to the JHipster Registry to get their configuration. This is true for both gateways and microservices.

This configuration is a Spring Boot configuration, like the one found in the JHipster application-*.yml files, but it is stored in a central server, so it is easier to manage.

On startup, your gateways and microservices app will query the Registry’s config server and overwrite their local properties with the ones defined there.

Two kinds of configurations sources are available (defined by the spring.cloud.config.server.composite property):

A native configuration, which is used by default in development (using the JHipster dev profile), and which uses the local filesystem.
A Git configuration, which is used by default in production (using the JHipster prod profile), and which stores the configuration in a Git server. This allows to tag, branch or rollback configurations using the usual Git tools, which are very powerful in this use-case.
To manage your centralized configuration you need to add appname-profile.yml files in your configuration source where appname and profile correspond to the application’s name and current profile of the service that you want to configure. For example, adding properties in a gateway-prod.yml file will set those properties only for the application named gateway started with a prod profile. Moreover, properties defined in application[-dev|prod].yml will be set for all your applications.

As the Gateway routes are configured using Spring Boot, they can also be managed using the Spring Config Server, for example you could map application app1-v1 to the /app1 URL in your v1 branch, and map application app1-v2 to the /app1 URL in your v2 branch. This is a good way of upgrading microservices without any downtime for end-users.


>참조: [Jhipster Registry 공식문서](https://www.jhipster.tech/jhipster-registry/)
