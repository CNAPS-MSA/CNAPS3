# 대여(Rental)서비스 구현
**코드는 아무 예고 없이 언제든 변경될 수 있으니, 실제 코드는 해당 서비스 애플리케이션의 Repository에서 확인하세요.**

## 도서 대여 기능 구현
Sample에서 보여줄 기능은 아래와 같다.

- 구현기능
   - 도서대여처리
      1. 대여 처리 비즈니스 로직 구현
      2. 대여 시, 도서 서비스 연계하여 상세 도서 정보(id, title) 조회 (feign 동기 호출)
      3. 대여 시, 도서 서비스 연계하여 재고 처리 (kafka 비동기 메시지 전송)
      4. 대여 시, 카탈로그 서비스에 대여 도서 집계 처리(kafka 비동기 메시지 전송) 
   - 반납처리
      1. 반납처리 비즈니스 로직 구현
      2. 반납 시, 도서 서비스 연계하여 재고 처리  (kafka 비동기 메시지 전송)
   - 연체 처리
      1. 연체 처리 비즈니스 로직 구현
      2. 연체 시 대여불가처리 
   - 연체된 도서 반납 처리

## API설계
|API명|도서대여|
|----|------|
|리소스URI|/rentals/{userid}/rentedItem/{books}|
|Method|POST|
|Request| |
|Response| |

리소스로 예를 들면 /rentals/scant/rentedItem/10001를 post방식으로 호출하므로 
scant라는 사용자의 대여카드에 10001의 일련번호 서적이 대여 된다는 의미이다.

|API명|도서반납|
|----|------|
|리소스URI|/rentals/{userid}/rentedItem/{books}|
|Method|DELETE|
|Request| |
|Response| |

마찬가지로 같은 리소스에 delete방식으로 호출하므로 대여가 취소되는 반납처리임을 알 수 있다.

|API명|도서연체처리|
|----|------|
|리소스URI|/rentals/{userid}/OverdueItem/{books}|
|Method|POST|
|Request| |
|Response| |

예를 들면 /rentals/scant/OverdueItem/10001를 post방식으로 호출하므로 
scant라는 사용자의 대여 카드에 10001의 일련번호 서적이 연체 등록된다는 의미이다.

|API명|도서연체처리|
|----|------|
|리소스URI|/rentals/{userid}/OverdueItem/{books}|
|Method|DELETE|
|Request| |
|Response| |

## 도메인 모델 
![image](https://user-images.githubusercontent.com/15258916/87246499-d072c400-c488-11ea-9df3-193f5d6b4763.png)

- 도메인 모델에서는 비지니스 개념을 표현한다. 비지니스 개념은 객체로 표현되고 도메인 주도 설계의 전술적 설계 기법인 어그리게잇, 엔티티, VO, 표준타입 패턴을 적용한다.
- 위 그림은 그렇게 정의된 대여 서비스의 도메인 모델이다. 대여와 반납의 책임을 가지고 있는 어그리게잇이며 루트 엔티티인 대여카드(Rental) , Rental과 일대다 관계인 엔티티 유형의 대여도서(RentedItem), 엔티티 연체도서(OverdueItem), 엔티티 반납도서(RetrurnItem)로 구성된다. 대여도서(OverdueItem) 와 반납도서(RetrurnItem)은 대여도서(RentedItem)과 마찬가지로 Rental과 일대다관계이다.
- Rental의 개념은 대여카드이다. 모든 사용자는 대여를 위한 대여카드를 하나씩 보유한다. 대여카드는 대여, 반납, 연체 처리, 연체도서반납,연체해제처리의 책임을 가진다.
- 대여 시 빌린도서만큼 대여도서(RentedItem)가 생성되고, 연체되면 연체도서(OverdueItem) 로 이동하고, 반납 시 반납도서(RetrurnItem)로 최종 이동된다. 
- 개인 5권의 대여 한도가 체크되고 1권의 도서라도 연체 되면 대여할 수 없다. 이런 대여가능여부는 표준 타입인 대여가능여부(RentalStatus)에 의해 규정된다.

## 유스케이스 흐름
![image](https://user-images.githubusercontent.com/15258916/87246908-5728a080-c48b-11ea-90c3-86a10a3c27c1.png)


위의 그림은 도서대여와 도서반납의 유스케이스 흐름을 시퀀스 다이어그램 으로 작성한 것이다.
도서대여 및 도서반납의 비지니스 로직은 도메인 모델에 응집되어 있는 것을 확인할 수 있다. 서비스는 그외 흐름제어 및 저장처리, 이벤트 메시지 처리를 담당한다. 그럼 각 영역별로 상세히 살펴보자.

## 내부영역 - 도메인 모델 개발하기 
### Rental.java

대여카드(Rental)은 사용자아이디(userid), 대여가능상태(RentalStatus), 연체료(lateFee), 대여도서(RentedItem), 연체도서(OverdueItem), 반납도서(ReturnedItem)의 속성을 가지고 있다.

#### CASCADE 설정

Rental.java에는 3개의 OneToMany관계가 선언되어있다.
대여 중인 도서 리스트/ 연체 도서 리스트/ 반납된 도서 리스트이다.
3가지 리스트 모두 Rental과 생명 주기가 같기 때문에 `CascadeType.ALL`로 설정하였다.

```java
    /**
    * 대여카드 어그리게잇(루트 엔티티) 클래스
    */
      @Entity
      @Table(name = "rental")
      @Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
      public class Rental implements Serializable {
         @Id
         @GeneratedValue(strategy = GenerationType.IDENTITY)
         private Long id;
         @Column(name = "user_id")
         private Long userId;

         @Enumerated(EnumType.STRING)
         @Column(name = "rental_status")
         private RentalStatus rentalStatus;

         @Column(name = "late_fee")
         private Long lateFee;

         @OneToMany(mappedBy = "rental", cascade = CascadeType.ALL)
         @Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
         private Set<RentedItem> rentedItems = new HashSet<>();

         @OneToMany(mappedBy = "rental", cascade = CascadeType.ALL)
         @Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
         private Set<OverdueItem> overdueItems = new HashSet<>();

         @OneToMany(mappedBy = "rental", cascade = CascadeType.ALL)
         @Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
         private Set<ReturnedItem> returnedItems = new HashSet<>();
      …(중략)…
```
 
### Rental

```java
/**
  * 대여카드 생성
  * @param userId
  * @return
  */
public static Rental createRental(Long userId) {
    Rental rental = new Rental();
    rental.setUserId(userId);
    //대여 가능하게 상태 변경
    rental.setRentalStatus(RentalStatus.RENT_AVAILABLE);
    rental.setLateFee(0);
    return rental;
}
```
**createRental(대여카드 생성 메소드)**

대여카드 생성메소드는 Renta내부에서 사용자id만 받아 생성될 수 있도록 캡슐화한다. 
이때 대여카드에 사용자id을 부여하고 RentalStatus는 RENT_AVAILABLE로 설정하며, LateFee는 0으로 설정한다.

```java
//대여 가능 여부 체크 //
public boolean checkRentalAvailable(Integer newBookListCnt) throws Exception{
    if(this.rentalStatus.equals(RentalStatus.RENT_UNAVAILABLE )) throw new Exception("연체 상태입니다.");
    if(this.getLateFee()!=0) throw new Exception("연체료를 정산 후, 도서를 대여하실 수 있습니다.");
    if(newBookListCnt+this.getRentedItems().size()>5) throw new Exception("대출 가능한 도서의 수는 "+( 5- this.getRentedItems().size())+"권 입니다.");

    return true;
}

/**
 * 대여하기
 *
 * @param bookid
 * @param title
 * @return
 */
public Rental rentBook(Long bookid, String title) {
    this.addRentedItem(RentedItem.createRentedItem(bookid, title, LocalDate.now()));
    return this;
}

/**
 * 반납하기
 *
 * @param bookId
 * @return
 */
public Rental returnbook(Long bookId) {
    RentedItem rentedItem = this.rentedItems
.stream().filter(item -> item.getBookId().equals(bookId)).findFirst().get();
    this.addReturnedItem(ReturnedItem.createReturnedItem(rentedItem.getBookId(), 
rentedItem.getBookTitle(), LocalDate.now()));
    this.removeRentedItem(rentedItem);
    return this;
}
```
**checkRentalAvailable(대여가능여부체크)**
   - 대여가능여부(RentalStatus)가 RENT_UNAVAILABLE 이거나 Latefee가 0이 아니면 연체 상태로, 대여가 불가능하며 Exception을 던진다. 
   - 대여 중인 책과 대여하고자 하는 책 개수의 합이 5권이 넘는 경우 대여가 불가능하다. 이때, 대여가 가능한 책 권 수를 알려주고 Exception을 던진다.

**rentBooks 메소드 (대여하기)** 
  - 도서id와 이름으로 대여 도서 객체(RentedItem)를 생성한 후에 대여 카드(rental)에 추가한다. 

**returnbook(반납하기)**
  - 도서id로 대여 카드에 존재했던 대여도서 객체(rentedItem)를 찾아 삭제하고, 그 정보로 반납도서 객체(returnedItem)를 만든 후 대여카드(rental)에 추가한다.

### RentedItem.java

```java
/**
 * RentedItem 엔티티 클래스
 */
@Entity
@Table(name = "rented_item")
@Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
public class RentedItem implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "book_id")
    private Long bookId;

    @Column(name = "rented_date")
    private LocalDate rentedDate;

    @Column(name = "due_date")
    private LocalDate dueDate;

    @Column(name = "book_title")
    private String bookTitle;

    @ManyToOne
    @JsonIgnoreProperties("rentedItems")
    private Rental rental;

    public static RentedItem createRentedItem(Long bookId, String bookTitle, 
                                                            LocalDate rentedDate) 
{
    RentedItem rentedItem = new RentedItem();
    rentedItem.setBookId(bookId);
    rentedItem.setBookTitle(bookTitle);
    rentedItem.setRentedDate(rentedDate);
    rentedItem.setDueDate(rentedDate.plusWeeks(2));
    return rentedItem;
}

…(중략)…
```
**createRentedItem(대여도서 생성 메소드)**
   - 대여도서(RentedItem) 생성 시, 연결되어있는 rental의 정보, 대여 도서 정보, 대여 시작 날짜, 반납 날짜가 지정된다. 대여 기간은 총 2주로 설정한다

### ReturnedItem.java
```java
public static ReturnedItem createReturnedItem(Long bookId, String bookTitle, LocalDate now) {
    ReturnedItem returnedItem = new ReturnedItem();
    returnedItem.setBookId(bookId);
    returnedItem.setBookTitle(bookTitle);
    returnedItem.setReturnedDate(now);
    return returnedItem;
}
```
반납도서(ReturnedItem) 생성 메소드이다

### RentalStatus.java
```java
public enum RentalStatus {
    RENT_AVAILABLE(0,"대여가능","대여가능상태"),
    RENT_UNAVAILABLE(1,"대여불가","대여불가능상태");
}
```
표준타입인 enum class로 정의된 대여가능여부이다

## 내부영역 - 서비스 구현
서비스는 비지니스 흐름을 처리한다.
위에 구현한 도메인 모델을 기반으로 도서 대여의  비지니스 처리 흐름부터 살펴보자. 

### RentalService.java
```java
public interface RentalService {
…(중략)…
/****
 *
 * Business Logic
 *
 * 책 대여하기
 *
 * ****/
Rental rentBooks(Long userId, List<BookInfo> books);
```
대여 서비스 인터페이스이다. 책 대여 시, 대여 카드를 찾기 위해 대여하는 사용자의 Id와 대여하고자 하는 책들의 Id를 받는다.

### RentalServiceImpl.java
```java
@Service
@Transactional
public class RentalServiceImpl implements RentalService {
…(중략)…
/**
 * 여러권 대여하기
 *
 * @param userId
 * @param books
 * @return
 */
@Transactional
public Rental rentBooks(Long userId, List<BookInfoDTO> books) {
    log.debug("Rent Books by : ", userId, " Book List : ", books);
    Rental rental = rentalRepository.findByUserId(userId).get();
    try {
      Boolean checkRentalStatus = rental.checkRentalAvailable(books.size());
       if (checkRentalStatus) {

        books.forEach(bookInfo -> rental.rentBook(bookInfo.getId(), bookInfo.getTitle()));
        rentalRepository.save(rental);

        books.forEach(b -> {
          try {
                updateBookStatus(b.getId(), "UNAVAILABLE");
                updateBookCatalog(b.getId(), "RENT_BOOK");
               } catch (ExecutionException | InterruptedException 
| JsonProcessingException e) {
                    e.printStackTrace();
                }
            });
            savePoints(userId, books.size());
        }
    } catch (Exception e) {
        String errorMessage = e.getMessage();
        System.out.println(errorMessage);
        return null;
    }
    return rental;
}
```
도서대여 메소드이다. 
   - 사용자id에 해당하는 대여카드 (Rental)를 찾는다.
   - 먼저 대출할 도서 갯수를 넣어 해당 대여카드가 도서대여 가능 상태인지 확인한다.
   - 도서카드(Renta)에 빌리려는 도서정보를 넣어 도서 대여 처리를 위임한다.
   - 대여 처리가 된 대여카드(Rental)는 RentalRepository에 save 한다.
   - 도서 상태변경 이벤트 처리를 전송 한다.
   - 도서 카탈로그 변경 이벤트 처리를 전송한다.

비동기 이벤트 처리를 위해 아웃바운드 어댑타를 호출하는 부분은 서비스에 아래와 같이 연계된다. 메시지를 카프카에 직접 던지는 구현부분은 아웃 바운드 어답터 영역에서 살펴보겠다. 

### RentalServiceImpl.java
```java
@Override
public void updateBookStatus(Long bookId, String bookStatus) 
throws ExecutionException, InterruptedException, JsonProcessingException {
    rentalProducer.updateBookStatus(bookId, bookStatus);
}

@Override
public void savePoints(Long userId, int bookCnt) throws ExecutionException, InterruptedException, JsonProcessingException {
    rentalProducer.savePoints(userId, bookCnt * pointPerBooks);
}

@Override
public void updateBookCatalog(Long bookId, String eventType) throws InterruptedException, ExecutionException, JsonProcessingException {
    rentalProducer.updateBookCatalogStatus(bookId, eventType);
}
```
다음은 반납처리의 서비스 흐름을 살펴보자.

### RentalService.java
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
### RentalServiceImpl.java
```java
/**
 * 여러 권 반납하기
 *
 * @param userId
 * @param bookIds
 * @return
 */
@Transactional
public Rental returnBooks(Long userId, List<Long> bookIds) {
    log.debug("Return books by ", userId, " Return Book List : ", bookIds);
    Rental rental = rentalRepository.findByUserId(userId).get();

    Rental finalRental = rental;
    bookIds.forEach(bookid -> finalRental.returnbook(bookid));
    rental = rentalRepository.save(finalRental);

    bookIds.forEach(b -> {
        try {
            updateBookStatus(b, "AVAILABLE");
            updateBookCatalog(b, "RETURN_BOOK");
        } catch (ExecutionException | InterruptedException | JsonProcessingException e) {
            e.printStackTrace();
        }
    });
    return rental;
}
```
도서대여처리와 비슷하다.
   - 해당 Userid의  대여카드(Rental)을 찾는다.
   - for loop를 통해 대여카드에게 위임하여 도서반납처리를 차례로 진행한다.
   - 반납 처리가 된 대여카드(Rental)는 RentalRepository로 저장(save) 한다.
   - 도서 상태변경 이벤트 처리를 전송 한다.
   - 도서 카탈로그 변경 이벤트 처리를 전송한다.

## 내부영역 - 레파지토리 개발
위의 서비스 처리에서 저장처리를 하는 레파지토리가 등장했는데 구현은 아래와 같다.
### RentalRepository.java
```java
/**
 * Rented엔티티의 레파지토리
 */
@Repository
public interface RentalRepository extends JpaRepository<Rental, Long> {

    Optional<Rental> findByUserId(Long userId);
}
```
도메인 주도 설계의 레파지토리 패턴을 영향 받은 Spring Data JPA의 Repository인터페이스이다. 
레파지토리 패턴은 도메인 영역과 인프라스트럭처 계층을 분리하여 계층간의 결합도를 낮추기 위한 패턴으로 도메인 객체(어그리게잇)의 생명주기 즉 영속성을 관리한다. 
Spring Data JPA의 Repository인터페이스를 활용하면 Sql문 작성으로 지루하게 반복되는 CRUD문제를 쉽게 해결할 수 있다. CRUD 데이터 처리에 대한 공통 인터페이스를 제공하고, 인터페이스만 작성하면 런타임시 동적으로 구현체를 주입해 준다. 

기본적인 CRUD 메소드를 제공해 주는데 다음과 같다.
- findAll() : 전체 목록을 조회
- findOne(ID) :  id로 단건 조회
- save() : 저장 (Insert,update)
- count() : 갯수조회 
- delete() : 삭제

추가적인 메서드가 필요하면 메서드이름으로 쿼리를 생성할 수도 있고, 직접 JPQL를 사용할 수도 있고 쿼리문을 객체로 표현할 수 있는QueryDsl를 활용할 수도 있다. 
위의 findByUserId메서드는 기본 메서드 외에 사용자id로 검색하기 위해 메서드이름으로 쿼리를 생성해주는 메소드를 선언한 것이다. 런타임 시 적절한 JPQL 쿼리가 자동으로 생성되어 실행될 것이다.  

## 외부영역 – REST 컨트롤러 개발 
도메인과 서비스를 통해 대략적인 비지니스 개념을 정의하고 비지니스 흐름을 처리하는 방식을 살펴보았다. 이렇게 완성된 비지니스로직은 프론트엔드와 약속된 API로 외부로 공개되어 프론트 엔드에 의해 활용 되야 한다. REST컨트롤러는 구현된 서비스의 REST API를 발행한다. 

컨트롤러는 프로트엔드에 제공할 API를 내부영역의 도메인 기능을 활용하여 적절히 제공해야 한다. 또한 API 변환 외의 비지니스 로직 처리는 내부영역의 서비스에 위임해야 한다. 
RentalResource에서 대여API를 쉽게 인지할 수 있는 적절한 리소스명("/rentals/{userid}/RentedItem/{books}")으로 http 표준 메소드 POST방식으로 선언하여 제공하고 있다. 주요 비지니스로직처리는  rentalService. rentBooks를 호출하여 위임한다. 
### RentalResource.java

```java
@RestController
@RequestMapping("/api")
public class RentalResource {
…중략…
/**
 * 도서 대여 하기
 * @param userid
 * @param books
 * @return
 * @throws InterruptedException
 * @throws ExecutionException
 * @throws JsonProcessingException
 */
@PostMapping("/rentals/{userid}/RentedItem/{books}")
public ResponseEntity rentBooks(@PathVariable("userid") Long userid, 
@PathVariable("books") List<Long> books)
 throws InterruptedException, ExecutionException,
 JsonProcessingException {
    log.debug("rent book request");

//feign - 책 정보 가져오기
    ResponseEntity<List<BookInfoDTO>> bookInfoResult = bookClient.getBookInfo(books, 
userid); 
    List<BookInfoDTO> bookInfoDTOList = bookInfoResult.getBody();
    log.debug("book info list", bookInfoDTOList.toString());

    Rental rental = rentalService.rentBooks(userid, bookInfoDTOList);

    if (rental != null) {
        RentalDTO result = rentalMapper.toDto(rental);
        return ResponseEntity.ok().body(result);
    } else {
        log.debug("대여 할 수 없는 상태입니다.");
        return ResponseEntity.badRequest().build();
    }
}
```
대여처리 API 구현을 위한 컨트롤러 처리 흐름을 살펴보면
   - Http Post방식으로 사용자id , 대여할 도서 정보 목록을 받는다.
   - 도서목록을 동기 호출로 도서 서비스를 호출하여 검증하고 상세도서정보를 가져온다.  
   - 서비스를 호출하여 도서대여처리를 수행한다.
   - 도서대여처리한 대여 카드를 DTO로 변경하여 클라이언트에 반환한다.

도서 서비스를 호출하여 동기 호출한 상세 부분은 다음의 외부 영역 아웃바운드 처리에서 살펴보기로 한다.

## 아웃바운드 어댑터 처리 
이전의 서비스 흐름처리와 API 컨트롤러에서 아웃바운드 어댑터를 이용해서 데이터를 가져오고 보내는 흐름이 있다는 것을 확인했다. 
즉 도서 대여 API에서 도서정보를 검증하고 상세정보를 요청하기 위해  도서서비스를 향한 아웃바운드 동기호출 아답터를 이용한다. 또한 대여 완료 시 도서서비스에 재고처리를 해야 하고, 카탈로그 서비스에  해당 도서가 대여 중이라는 처리를 해야 한다. 이 두개의 처리는 아웃바운드 아댑터를 통해 비동기 메시지 이벤트를 전송한다.  아래 그림은 그러한 과정을 보여준다. 
![image](https://user-images.githubusercontent.com/15258916/87247736-8db4ea00-c490-11ea-8748-fa1b5602fffa.png)

지금부터는 이러한 아웃바운드 어댑터 처리의 구현에 대해 알아보자. 다음 순서로 살펴보겠다. 
- [Feign 동기 메시지 호출처리](/contents/jhipster_feign.md)
- [Kafka 비동기 메시지 전송처리](/contents/jhipster_kafka.md)

