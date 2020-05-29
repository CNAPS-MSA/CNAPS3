# Jhipster Business Logic 

## Business Logic 추가 전, Configuration 설정

1. 각 service를 gateway의 end-point에 등록시켜주어야한다.
   
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


    Rental 서비스부터 구현할 예정이기 때문에 우선 rental만 등록해두었다.

1. Security 설정 변경
   
    기존의 Gateway, Rental, Book의 `SecurityConfiguration.java`를 살펴보면 아래와 같이 작성되어있다.

    ```java
    @Override
        public void configure(HttpSecurity http) throws Exception {
            // @formatter:off
            http
                .csrf()
                .disable()
                .addFilterBefore(corsFilter, UsernamePasswordAuthenticationFilter.class)
                .exceptionHandling()
                    .authenticationEntryPoint(problemSupport)
                    .accessDeniedHandler(problemSupport)
            .and()
                .headers()
                .contentSecurityPolicy("default-src 'self'; frame-src 'self' data:; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://storage.googleapis.com; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self' data:")
            .and()
                .referrerPolicy(ReferrerPolicyHeaderWriter.ReferrerPolicy.STRICT_ORIGIN_WHEN_CROSS_ORIGIN)
            .and()
                .featurePolicy("geolocation 'none'; midi 'none'; sync-xhr 'none'; microphone 'none'; camera 'none'; magnetometer 'none'; gyroscope 'none'; speaker 'none'; fullscreen 'self'; payment 'none'")
            .and()
                .frameOptions()
                .deny()
            .and()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
                .authorizeRequests()
                .antMatchers("/api/authenticate").permitAll()
                .antMatchers("/api/register").permitAll()
                .antMatchers("/api/activate").permitAll()
                .antMatchers("/api/account/reset-password/init").permitAll()
                .antMatchers("/api/account/reset-password/finish").permitAll()
                .antMatchers("/api/**").authenticated()
                .antMatchers("/management/health").permitAll()
                .antMatchers("/management/info").permitAll()
                .antMatchers("/management/prometheus").permitAll()
                .antMatchers("/management/**").hasAuthority(AuthoritiesConstants.ADMIN)
            .and()
                .apply(securityConfigurerAdapter());
            // @formatter:on
        }
    ```

    위의 코드에서 `.antMatchers("/api/**").authenticated()`부분을 `.antMatchers("/api/**").permitAll()`로 수정해준다.
    `.antMatchers("/api/**").authenticated()`는 해당 경로로 들어오는 요청이 인증된 요청인 경우에만 허용한다는 의미인데, Swagger로 테스트 하던 중 SecurityFilter에 오류가 생기는 현상이 있어, 로직 개발 중엔 `permitAll()`로 수정하여 개발 및 테스트를 진행하였다. 
    >Gateway, Rental, Book의 각 SecurityConfiguration.java을 모두 위와 같이 수정하였다.


# Business Logic 구현

Sample에서 보여줄 Logic은 아래와 같다.

1. 도서 대여하기
2. 도서 반납 후, 도서 상태 변경하기 -> kafka 사용하기
3. 도서 연체 시 포인트 결제 연결하기 -> FeignClient 사용하기

**코드는 아무 예고 없이 언제든 변경될 수 있으니, 실제 코드는 해당 서비스 애플리케이션의 Repository에서 확인하세요.**

## 도서 대여 서비스 구현하기

아래 코드를 살펴보면, rentBooks()를 RentalServiceImpl에서 개발하지 않고, **Rental Entity 내부**에 진행한 것을 확인할 수 있다. 그 이유는, Entity 생명주기 내에서 처리할 수 있는 로직을 ServiceImple에서 구현하는 경우 ServiceImple의 크기가 비대해지는데, 이 현상을 **Fat Service**라 한다. *(맞나..?)* 이를 방지하기 위해 생명주기가 같은 Entity들끼리의 로직, Repository접근 없이 가능한 로직들은 Entity내에 개발하였다.


Rental Directory로 이동한다.

1. RentalService.java
    
    아래 코드를 RentalService.java에 추가한다.

    ```java

    /****
        *
        * Business Logic
        *
        * 책 대여하기
        *
        * ****/
        RentalDTO rentBooks(Long userId, List<Long> bookIds);

    ```

    -> 책 대여 시, 대여하는 사용자의 Id와 대여하고자 하는 책들의 Id를 받아 진행한다. 


2. RentalServiceImpl.java

    ```java
        @Override
    public RentalDTO rentBooks(Long userId, List<Long> bookIds) {
        log.debug("Rent Books by : ", userId, " Book List : ", bookIds);

        if(rentalRepository.findByUserId(userId).isPresent()){ //기존에 대여 내역이 있는 경우
            Rental rental = rentalRepository.findByUserId(userId).get();
            rental = rental.rentBooks(bookIds);
            if(rental!=null)
            {
                log.debug(" 대여 완료 되었습니다.", rental);
                return rentalMapper.toDto(rental);
            }else{
                log.debug("대여 불가능 상태입니다.");
                return null;
            }
        }else{ // 첫 대여인 경우
            log.debug("첫 도서 대여입니다.");
            Rental rental = Rental.createRental(userId);
            rentalRepository.save(rental);
            rental=rental.rentBooks(bookIds);

            log.debug(" 대여 완료 되었습니다.", rental);
            return rentalMapper.toDto(rental);
        }
    }
    ```

    - 기존 내역이 있는 경우 -> 해당 User의 Rental을 찾아 rentBooks를 진행한다. 
      - 해당 User가 대여 가능 상태인지 체크하고, 불가능인 경우 null을 return, 가능한 경우 rent를 진행한다.
    - 첫 대여인 경우 -> 해당 User의 Rent를 생성하여 저장한다. 이후, rentBooks를 진행한다.

3. Rental.java

    1. CASCADE 설정

    Rental.java에는 3개의 OneToMany관계가 선언되어있다.
    대여 중인 도서 리스트/ 연체 도서 리스트/ 반납된 도서 리스트이다.
    3가지 리스트 모두 Rental과 생명 주기가 같기 때문에 `CascadeType.ALL`로 설정하였다.

    ```java
    @OneToMany(mappedBy = "rental", cascade = CascadeType.ALL)
    @Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
    private Set<RentedItem> rentedItems = new HashSet<>();

    @OneToMany(mappedBy = "rental", cascade = CascadeType.ALL)
    @Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
    private Set<OverdueItem> overdueItems = new HashSet<>();

    @OneToMany(mappedBy = "rental", cascade = CascadeType.ALL)
    @Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
    private Set<ReturnedItem> returnedItems = new HashSet<>();
    ```
    
    2. Rental 생성 메소드
   
    첫 대여인 경우 Rental을 생성한다. 이때 RentalStatus는 OK로 설정하며, LateFee는 0으로 설정한다.
    
    ```java
        //생성메소드//
    public static Rental createRental(Long userId){
        Rental rental = new Rental();
        rental.setUserId(userId);
        rental.setRentalStatus(RentalStatus.OK);
        rental.setRentedItems(new HashSet<>());
        rental.setOverdueItems(new HashSet<>());
        rental.setReturnedItems(new HashSet<>());
        rental.setLateFee((long)0);
        return rental;
    }
    ```
    3. rentBooks 메소드 (책 대여하기)
    
    가장 먼저, 현재 Rent가 가능한 상태인지 확인한다.
    -> RentalStatus가 OVERDUE이거나 Latefee가 0이 아니면 연체 상태로, 대여가 불가능하다. 
    -> 대여 중인 책과 대여하고자 하는 책 개수의 합이 5권이 넘는 경우 대여가 불가능하다. 이때, 대여가 가능한 책 권 수를 알려주고 return 한다.

    Book Id 리스트를 받아 각 book마다 rentedItem을 생성하고 Rental의 rentedItems에 add한다.
    Rental의 상태는 RENTED로 변경하고, Latefee를 0으로 설정한다.
    
    ```java

    //대여 가능 여부 체크 //
    public boolean checkRentalAvailable(Integer newBookCnt){
        if(this.rentalStatus!=RentalStatus.OVERDUE){

            if(this.rentedItems.size()+newBookCnt >5){
                System.out.println("대출 가능한 도서의 수는 "+( 5- this.getRentedItems().size())+"권 입니다.");
                return false;
            }else{
                return true;
            }
        }else{
            System.out.println("연체 상태입니다.");
            return false;
        }
    }


    //대여하기 메소드//
    public Rental rentBooks(List<Long> bookIds){
        if(checkRentalAvailable(bookIds.size())){
            for(Long bookId : bookIds){
                RentedItem rentedItem = RentedItem.createRentedItem(this, bookId, LocalDate.now());
                this.addRentedItem(rentedItem);
            }
            this.setRentalStatus(RentalStatus.RENTED);
            this.setLateFee((long)0);
            return this;

        }else{
            return null;
        }
    }


    ```

4. RentedItem.java

    Rent에서 RentedItem 생성시에도 RentedItem Entity를 호출하여 생성한다. 

    RentedItem 생성 시, 연결되어있는 rental의 정보, 대여 도서 정보, 대여시작 날짜, 반납 날짜가 포함되어야한다. 대여기간은 총 2주로 설정하였다.

    ```java
     public static RentedItem createRentedItem(Rental rental, Long bookId, LocalDate rentedDate) {
        RentedItem rentedItem = new RentedItem();
        rentedItem.setBookId(bookId);
        rentedItem.setRental(rental);
        rentedItem.setRentedDate(rentedDate);
        rentedItem.setDueDate(rentedDate.plusWeeks(2)); //총 대여기간 2주 설정
        return rentedItem;
    }
    ```

5. RentalResource.java

    이제 RentalResource에서 mapping을 선언하여 RestAPI로 구현해보자.
    Input으로는 `userId`와 `bookIdList`를 받는다. 각각 userid와 books로 매핑하였다.
    >RestController에서는 되도록 DTO를 사용하였다. 

    rentBooks 요청이 실행되면, rentalService의 rentBooks메소드를 실행시켜, rentalDTO를 반환받았다.
    rentalDTO가 null인 경우, 에러 메세지를 출력한다. 
    정상적으로 진행된 경우 rentalService의 save메소드를 통해 repository에 저장하였다.

    ```java
        @PostMapping("/rentbooks/by/{userid}/books/{books}")
    public ResponseEntity<RentalDTO> rentBooks(@PathVariable("userid")Long userid, @PathVariable("books") List<Long> books){
        log.debug("rent book request");
        RentalDTO rentalDTO = rentalService.rentBooks(userid,books);

        if(rentalDTO==null){
            throw new BadRequestAlertException("Invalid ", ENTITY_NAME, "null");
        }
        RentalDTO result = rentalService.save(rentalDTO);
        log.debug("SEND BOOKIDS for Book: {}", books);

        return ResponseEntity.ok()
            .headers(HeaderUtil.createEntityUpdateAlert(applicationName, true, ENTITY_NAME, rentalDTO.getId().toString()))
            .body(result);
    }
    ```

## 도서 반납 서비스 구현하기

1. RentalResource.java

    도서 반납 요청 또한 대여와 마찬가지로 userId와 book Id List를 Input으로 받도록하였다.
   
   ```java

    @PutMapping("/returnbooks/by/{userid}/books/{books}")
    public ResponseEntity returnBooks(@PathVariable("userid")Long userid, @PathVariable("books") List<Long> books){
        rentalService.returnBooks(userid,books);
        log.debug("returned books");
        log.debug("SEND BOOKIDS for Book: {}", books);

        return ResponseEntity.ok().build();
    }
    ```

2. RentalService.java

    ```java

    /****
     *
     * Business Logic
     *
     * 책 반납하기
     *
     * ****/

    void returnBooks(Long userId, List<Long> bookIds);
    ```

3. RentalServiceImpl.java

    ```java


    @Override
    public void returnBooks(Long userId, List<Long> bookIds) {
        log.debug("Return books by ", userId, " Return Book List : ", bookIds);

        if(rentalRepository.findByUserId(userId).isPresent()){
            Rental rental = rentalRepository.findByUserId(userId).get();
            for(Long bookId: bookIds){
                RentedItem rentedItem = rentedItemRepository.findByBookId(bookId).get();
                rental.getRentedItems().remove(rentedItem);
                rentedItemRepository.delete(rentedItem);
                ReturnedItem returnedItem = ReturnedItem.createReturnedItem(rental, bookId , LocalDate.now());
                rental.addReturnedItem(returnedItem);
                returnedItemRepository.save(returnedItem);

            }

            if(rental.getRentedItems().size()==0 && rental.getRentalStatus()!= RentalStatus.OVERDUE){
                rental.setRentalStatus(RentalStatus.OK);
            }

            rentalRepository.save(rental);

            return ;

        }else{
            log.debug("대여 이력이 없습니다.");
            return ;
        }

    }

    ```

### kafka로 도서 대여/반납 시, Book상태 변경하기

