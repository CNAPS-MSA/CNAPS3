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
    ```


2. Security 설정 변경
   
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
    >>오직 개발 또는 테스트 단계일 때만 `permitAll()`로 설정해야하며, prod단계에선 `authenticated()`를 적용해야한다.


# Business Logic 구현

Sample에서 보여줄 Logic은 아래와 같다.

1. 도서 대여하기
   1. 대여시, book 정보(id, title)를 feign을 통해 가져오기
   2. 대여 완료된 book 상태 kafka를 통해 업데이트하기 
2. 도서 반납 후, 도서 상태 변경하기 -> kafka 사용하기

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
    Rental rentBooks(Long userId, List<BookInfo> books);


    ```

    -> 책 대여 시, 대여하는 사용자의 Id와 대여하고자 하는 책들의 Id를 받아 진행한다. 


2. RentalServiceImpl.java

    ```java
       
    @Transactional
    public Rental rentBooks(Long userId, List<BookInfo> books) {
        log.debug("Rent Books by : ", userId, " Book List : ", books);
        Rental rental = new Rental();
        if(rentalRepository.findByUserId(userId).isPresent()){
            rental = rentalRepository.findByUserId(userId).get();
        }else{
            //도서카드 생성 -> rental과 user 연결 후 삭제해야함
            log.debug("첫 도서 대여 입니다.");
            rental = Rental.createRental(userId);
        }

        try{
            Boolean checkRentalStatus = rental.checkRentalAvailable(books.size());
            if(checkRentalStatus){
            List<RentedItem> rentedItems = books.stream()
                .map(bookInfo -> RentedItem.createRentedItem(bookInfo.getId(), bookInfo.getTitle(), LocalDate.now()))
                .collect(Collectors.toList());

            for (RentedItem rentedItem : rentedItems) {
                rental = rental.rentBook(rentedItem);


            }
            rentalRepository.save(rental);


            }

        }catch (Exception e){
            String errorMessage = e.getMessage();
            System.out.println(errorMessage);
            return null;
        }
        return rental;

    }

    ```

    - 기존 내역이 있는 경우 -> 해당 User의 Rental을 찾아 rentBooks를 진행한다. : 이 부분은 User Service구현 완료 시 User 생성완료 후 Rental을 생성하는 방식으로 변경할 예정임. 따라서, 추후 이부분은 삭제될예정임.
    - Controller에서 넘겨받은 BookInfo를 가지고 rentedItem을 생성하여 Rental Entity내의 rentBook을 진행한다.
        >이부분의 경우, RentedItem List를 생성시에는 Java Stream API를, rentBook 메소드를 반복 실행할 때에는 for loop를 사용한 것을 확인할 수 있다.
        >왜냐하면, Java Stream은 List Object내 Item의 map과 collect 작업에서 성능이 유효하다. 간단한 for loop의 경우 stream이 아닌 일반적인 방식의 for loop가 성능이 3배정도 빠르기 때문이다. 따라서, 메소드 반복 실행의 경우엔 일반적인 for loop로 구현하였다. 
      - 가장 먼저, 현재 Rent가 가능한 상태인지 확인한다.
        >상태 체크 메소드는 Rental.java에 구현한다.
        -> RentalStatus가 OVERDUE이거나 Latefee가 0이 아니면 연체 상태로, 대여가 불가능하며 Exception을 던진다.
        -> 대여 중인 책과 대여하고자 하는 책 개수의 합이 5권이 넘는 경우 대여가 불가능하다. 이때, 대여가 가능한 책 권 수를 알려주고 Exception을 던진다.
        -> 대여 가능 상태인 경우 rentBook을 진행한다.
    - 작업한 Rental은 RentalRepository에 save 한다.
    - 예외 상황에 대해 모두 Exception처리를 했다. 구체적인 Exception 구현은 추후 개발할 예정이다.

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
    
    1. Rental 생성 메소드
   
    첫 대여인 경우 Rental을 생성한다. 이때 RentalStatus는 RENT_AVAILABLE로 설정하며, LateFee는 0으로 설정한다.
    
    ```java

            //생성메소드//
        /**
        *
        * @param userId
        * @return
        */
        public static Rental createRental(Long userId){
            Rental rental = new Rental();
            rental.setUserId(userId);
            //대여 가능하게 상태 변경
            rental.setRentalStatus(RentalStatus.RENT_AVAILABLE);
            rental.setLateFee((long)0);
            return rental;
        }

    ```
    1. rentBooks 메소드 (책 대여하기)
    
    
    ```java
    //대여하기 메소드//
    public Rental rentBook(RentedItem rentedItem){
        //현재 대여목록 갯수와 대여할 도서 갯수 파악

        this.addRentedItem(rentedItem);

        return this;
    }

    
    //대여 가능 여부 체크 //
    public boolean checkRentalAvailable(Integer newBookListCnt) throws Exception{
        if(this.rentalStatus.equals(RentalStatus.RENT_UNAVAILABLE )) throw new Exception("연체 상태입니다.");
        if(this.getLateFee()!=0) throw new Exception("연체료를 정산 후, 도서를 대여하실 수 있습니다.");
        if(newBookListCnt+this.getRentedItems().size()>5) throw new Exception("대출 가능한 도서의 수는 "+( 5- this.getRentedItems().size())+"권 입니다.");

        return true;
    }
    

    ```

    - ServiceImpl에서 생성한 RentedItem을 받아, rental의 rentedItems에 add 한다.
    - 대여 가능 여부 체크에서는 rentalStatus, LateFee, 현재 대여한 책의 개수를 기준으로 가능여부를 체크한다.

4. RentedItem.java

    ServiceImpl에서 RentedItem 생성시에도 RentedItem Entity를 호출하여 생성한다. 

    RentedItem 생성 시, 연결되어있는 rental의 정보, 대여 도서 정보, 대여시작 날짜, 반납 날짜가 포함되어야한다. 대여기간은 총 2주로 설정하였다.

    ```java
      public static RentedItem createRentedItem(Long bookId, String bookTitle, LocalDate rentedDate) {
        RentedItem rentedItem = new RentedItem();
        rentedItem.setBookId(bookId);
        rentedItem.setBookTitle(bookTitle);
        rentedItem.setRentedDate(rentedDate);
        rentedItem.setDueDate(rentedDate.plusWeeks(2));
        return rentedItem;

    }
    ```

5. RentalResource.java

    이제 RentalResource에서 mapping을 선언하여 RestAPI로 구현해보자.
    Input으로는 `userId`와 `bookIdList`를 받는다. 각각 userid와 books로 매핑하였다.
    >RestController에서는 되도록 DTO를 사용하였다. 

    - rentBooks 요청이 실행되면, rentalService의 rentBooks메소드를 실행시켜 rental을 반환받는다.
      - 과정 중 Exception이 발생하면, Exception을 던진다. -> 우선은 null check를 통해 체크하였다. 추후 Exception 처리를 추가할 예정이다.
    - 정상적으로 진행된 경우, kafka를 통해 rentalService에 bookStatus를 업데이트 시키는 메소드를 실행시킨다. 

    ```java
       
    @PostMapping("/rentbooks/{userid}/{books}")
    public ResponseEntity rentBooks(@PathVariable("userid")Long userid, @PathVariable("books") List<Long> books) {
        log.debug("rent book request");
        List<BookInfo> bookInfoList = bookClient.getBookInfo(books);
        log.debug("book info list",bookInfoList.toString());

        Rental rental = rentalService.rentBooks(userid, bookInfoList);

        //추후 Exception처리//
        if(rental!=null) {
            //kafka - 책 상태 업데이트
            bookInfoList.stream().forEach(b -> rentalService.updateBookStatus(b.getId(), "UNAVAILABLE"));

            RentalDTO result = rentalMapper.toDto(rental);
            return ResponseEntity.ok().body(result);
        }else {

            log.debug("대여 불가:");

            return ResponseEntity.badRequest().build();

        }

    }
    ```

## 도서 반납 서비스 구현하기

1. RentalResource.java

    도서 반납 요청 또한 대여와 마찬가지로 userId와 book Id List를 Input으로 받도록하였다.
    이때 대여 기록이 없는 책을 반납시도하는 경우 null을 던지게 했다. 이 부분은 추후 Exception으로 처리할 예정이다.
    ```java
    
     @PutMapping("/returnbooks/{userid}/{books}")
    public ResponseEntity returnBooks(@PathVariable("userid")Long userid, @PathVariable("books") List<Long> books){


            Rental rental=rentalService.returnBooks(userid,books);
            log.debug("returned books");
            log.debug("SEND BOOKIDS for Book: {}", books);

            //추후 Exception처리//
            if(rental!=null) {
                books.stream().forEach(b -> rentalService.updateBookStatus(b, "AVAILABLE"));
                RentalDTO result = rentalMapper.toDto(rental);
                return ResponseEntity.ok().body(result);
            }else {
                log.debug("대여기록에 없는 도서입니다.");
                return ResponseEntity.badRequest().build();
            }

    }
    ```
    
    returnBook을 마친 후에, book의 상태를 변경한다.

2. RentalService.java

    ```java

    /****
     *
     * Business Logic
     *
     * 책 반납하기
     *
     * ****/

    Rental returnBooks(Long userId, List<Long> bookIds);
    ```

3. RentalServiceImpl.java

    ```java

   
    @Transactional
    public Rental returnBooks(Long userId, List<Long> bookIds) {
        log.debug("Return books by ", userId, " Return Book List : ", bookIds);
        Rental rental = rentalRepository.findByUserId(userId).get();

        List<RentedItem> rentedItems = rental.getRentedItems().stream()
            .filter(rentedItem -> bookIds.contains(rentedItem.getBookId()))
            .collect(Collectors.toList());
        log.debug("bookIds contain :" , rentedItems.size());

        if(rentedItems.size()>0) {
            for (RentedItem rentedItem : rentedItems) {
                rental.returnbook(rentedItem);
            }

            rental = rentalRepository.save(rental);

            return rental;
        }else{

            return null;
        }

    }

    ```
    - 해당 User의 Rental을 찾는다.
    - Controller에서 넘겨받은 BookId와 rental의 rentedItems 중 일치하는 book을 필터링한다. 일치하는 book만 모아 새로운 rentedItemList를 만든다.
    - for loop에서 Rental의 returnBook을 차례로 진행한다.
    - 진행 후, rental을 저장한다.

4. Rental.java

    ```java
     //반납 하기//
    public Rental returnbook(RentedItem rentedItem) {

        this.removeRentedItem(rentedItem);
        this.addReturnedItem(ReturnedItem.createReturnedItem(rentedItem.getBookId(), rentedItem.getBookTitle(), LocalDate.now()));
        return this;

    }
    ```

    받은 rentedItem을 기존의 rentedItemList에서 제거하고, returnedItem을 생성하여 returnedItemList에 추가한다.

5. ReturnedItem.java

    ```java
        public static ReturnedItem createReturnedItem(Long bookId, String bookTitle, LocalDate now) {
        ReturnedItem returnedItem = new ReturnedItem();
        returnedItem.setBookId(bookId);
        returnedItem.setBookTitle(bookTitle);
        returnedItem.setReturnedDate(now);
        return returnedItem;
    }
    ```
    ReturnedItem 생성 메소드이다. 

### kafka와 Feign

Rent와 Return에서 Book 상태를 업데이트하고, 해당 Book의 정보 또한 가져와야했다.
Book상태의 경우 Kafka를 통해, Logic이 완료되면 업데이트하였고, Book정보는 Logic 시작 전 Feign을 통해 가져왔다. 
Book Service와의 Kafka, Feign Client연결은 아래 링크 페이지로 이동하여 확인하자.

- [Kafka 구성하기](/contents/jhipster_kafka.md)
- [Feign Client 구성하기](/contents/jhipster_feign.md)

