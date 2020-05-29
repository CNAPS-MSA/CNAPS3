# Jhipster Business Logic 

먼저, business logic을 추가하기 전 각 service를 gateway의 end-point에 등록시켜주어야한다.
등록하지 않을 경우, Access policy Filter에 request가 걸려져, 추가한 로직이 실행되지않을 수 있기 때문이다.

gateway directory -> resources -> config -> **application-dev.yml 과 application-prod.yml 모두 수정**

<img width="738" alt="image" src="https://user-images.githubusercontent.com/18453570/82303226-a2778300-99f5-11ea-972d-3c122d1ae752.png">

위 이미지처럼 빨간 박스 부분을 수정해주면 된다.

위 이미지엔 Rental만 추가 되었는데, Book 서비스, Delivery, Payment등 새로운 서비스를 추가할 때마다 Rental을 추가한 형식으로 추가해야한다.

```yaml
jhipster:
  gateway:
    rate-limiting:
      enabled: false
      limit: 100000
      duration-in-seconds: 3600
    authorized-microservices-endpoints: # Access Control Policy, if left empty for a route, all endpoints will be accessible
      rental: /api,/v2/api-docs # recommended dev configuration
      book: /api,/v2/api-docs 
````

## Rent book 서비스 구현하기

다시 Rental Directory로 이동한다.

1. RentalService.java

2. RentalServiceImpl.java

3. Rental.java

4. RentalItem.java

5. RentalResource.java
