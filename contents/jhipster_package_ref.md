# Jhipster Project Package Refactoring

Jhipster에서 기본으로 생성해주는 Directory 구조도 훌륭하지만, CNAPS 3.0에서 추구하는 방향성과는 다른 부분들이 있다.
해당 부분은 아래와 같다.

1. DTO, Mapper, Kafka, feign의 위치

DTO, Mapper, Kafka, fiegn은 외부 다른 서비스와의 통신에서 사용된다.
예를 들어, Rental에서 Book의 정보를 가져온다거나 Rental의 비즈니스 로직흐름 상 Book의 상태를 업데이트해주는 상황에 쓰인다.
Jhipster는 DTO와 Mapper는 Service 패키지에, Kafka는 Web.rest에 구성해두었다. (Feign은 config외엔 없다.)
하지만 외부 서비스와 통신하는 데 쓰이는 경우 client와 연결되는 곳인 controller에 필요한 DTO나 Mapper는 web.rest에, 이때 쓰이는 도구인 Kafka나 feign은 adaptor라는 패키지를 따로 만들어 관리하는 것이 바람직하다.

따라서 패키지의 구조 아래 이미지와 같이 수정하였다.

<img width="178" alt="image" src="https://user-images.githubusercontent.com/18453570/92840345-ef0b2200-f41b-11ea-9759-ab376f2422f0.png">

2. DTO, Mapper를 사용하는 방식

Jhipster가 DTO와 Mapper를 Service 패키지에 둔 이유는 `Service`에서 DTO와 Mapper를 사용하기 때문이다. 즉, Service가 Return하는 모든 Object가 DTO인 것이다. 하지만 CNAPS 3.0에서는 이 방식을 지양한다. 따라서, Service는 Entity형태로 Return하게 하였고 Controller에서 내부 서비스에서 받아온 Entity를 DTO로 변환시키도록하였다.

변경 후, 예시 코드는 아래와 같다. Full code의 경우 해당 서비스의 repository를 참고하자.

<RentalResource.java> - Controller

```java
    /**
     * {@code POST  /rentals} : Create a new rental.
     *
     * @param rentalDTO the rentalDTO to create.
     * @return the {@link ResponseEntity} with status {@code 201 (Created)} and with body the new rentalDTO, or with status {@code 400 (Bad Request)} if the rental has already an ID.
     * @throws URISyntaxException if the Location URI syntax is incorrect.
     */
    @PostMapping("/rentals")
    public ResponseEntity<RentalDTO> createRental(@RequestBody RentalDTO rentalDTO) throws URISyntaxException {
        log.debug("REST request to save Rental : {}", rentalDTO);
        if (rentalDTO.getId() != null) {
            throw new BadRequestAlertException("A new rental cannot already have an ID", ENTITY_NAME, "idexists");
        }
        RentalDTO result = rentalMapper.toDto(rentalService.save(rentalMapper.toEntity(rentalDTO)));
        return ResponseEntity.created(new URI("/api/rentals/" + result.getId()))
            .headers(HeaderUtil.createEntityCreationAlert(applicationName, true, ENTITY_NAME, result.getId().toString()))
            .body(result);
    }
```

<RentalService.java> -Service

```java
   /**
     * Save a rental.
     *
     * @param rentalDTO the entity to save.
     * @return the persisted entity.
     */
    Rental save(Rental rentalDTO);
```

<RentalServiceImpl.java> -Service Implementation

```java

    /**
     * Save a rental.
     *
     * @param rental
     * @return the persisted entity.
     */
    @Override
    public Rental save(Rental rental) {
        log.debug("Request to save Rental : {}", rental);
        return rentalRepository.save(rental);
    }
  ```

