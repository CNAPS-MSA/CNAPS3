# Jhipster Business Logic 

먼저, business logic을 추가하기 전 각 service를 gateway의 end-point에 등록시켜주어야한다.
등록하지 않을 경우, Access policy Filter에 request가 걸려져, 추가한 로직이 실행되지않을 수 있기 때문이다.

gateway directory -> resources -> config -> **application-dev.yml 과 application-prod.yml 모두 수정**

<img width="738" alt="image" src="https://user-images.githubusercontent.com/18453570/82303226-a2778300-99f5-11ea-972d-3c122d1ae752.png">

위 이미지처럼 빨간 박스 부분을 수정해주면 된다.

Rental 서비스부터 구현할 예정이기 때문에 우선 rental만 등록해두었다.

## Rent book 서비스 구현하기

다시 Rental Directory로 이동한다.

1. RentalService.java

2. RentalServiceImpl.java

3. Rental.java

4. RentalItem.java

5. RentalResource.java
