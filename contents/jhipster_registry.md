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



>참조: [Jhipster Registry 공식문서](https://www.jhipster.tech/jhipster-registry/)
