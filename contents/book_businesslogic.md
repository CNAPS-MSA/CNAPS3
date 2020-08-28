# 도서(Book)서비스 구현

Sample에서 보여줄 기능은 아래와 같다.

## 구현기능
  - 입고도서등록, 도서등록
    1. 입고도서등록 : 기증 및 도서제공업체에 의해 도서정보가 등록된다.
    2. 대여도서등록 : 도서가 대여할수 있는 대여도서로 등록된다. 등록 시 Catagory서비스에서 검색되도록 정보전송
    3. 도서상태변경 : 대여서비스에서 도서대여시 도서상태가 반영됨 --> '대여가능/대여불가' 
    
## API설계
주요한 API설계는 다음과 같다.

|API명|도서정보조회|
|----|------|
|리소스URI|/books/bookInfo/{bookId}|
|Method|GET|
|Request| |
|Response| |

리소스로 예를 들면 /books/findBookInfo/10001를 GET방식으로 호출하므로 
10001의 일련번호 서적 정보를 조회한다. 

|API명|입고도서등록|
|----|------|
|리소스URI|/in-stock-books|
|Method|POST|
|Request|inStockBookDTO|
|Response| |

/in-stock-books에 requestBody형식으로 입고 도서 정보를 등록한다.

|API명|도서등록|
|----|------|
|리소스URI|/books/{inStockId}|
|Method|POST|
|Request|bookDTO|
|Response| |

예를 들면 /books/10001를 post방식으로 호출하므로 
신규도서정보를 requestBody형식으로 받아와 도서를 등록하고, 입고도서 일련번호 10001에 해당하는 도서는 입고도서에서 삭제된다.



## 도메인 모델링



모델은 도메인 엔티티인 대여도서와 입고도서로 구성된다.
도서공급자나 도서기증에 의해 도서가 입고되면 입고도서 객체가 생성되고, 이 입고된 도서는 유형에 따라 분류되고 도서관이 지정되어 대여 가능한 상품인 대여도서로 등록된다.
대여할수 있게 등록된 객체가 대여도서이다. 도서는 이용가능을 판단하는 도서상태를 가지고 있어 대여되면 대여불가 상태가되고 반납되면 대여가능상태가 된다.
입고도서는 루트엔티티이며 어그리게잇이고 대여도서도 역시 루트엔티티이며 어그리게잇이다. 

## 유스케이스 흐름

- 입고도서등록
- 대여도서등록처리
  
  입고도서등록과 대여도서등록처리는 프론트에서 받아온 데이터를 등록하는 절차로 특별한 비즈니스 로직은 없다.
  대여도서등록처리는 도서공급자나 도시기증자에 의해 입고된 도서들이 대여할 수 있는 정식도서로 등록되는 절차이다.
  언듯 입고도서객체와 연관관계가 있어 보이나 입고된 도서들을 살펴본 후 하나하나 분류하여 대여할 수 있는 프로세스이기 때문에 백엔드 데이터간의 직접적인 연관관계는 없다.
  프론트 엔드에서는 새로 입고된 도서 정보를 기반으로 도서분류등의 도서정보를 보완하여 새로운 대여도서로 등록하게 된다. 
  입고도서와 대여도서는 프론트를 매개로 관계를 가지므로 도메인 모델에서의 특별한 연관관계는 없다.
    
## 내부영역 - 도메인 모델 개발

### InstockBook.java

  입고도서 엔티티이다. id, 제목, 설명, 저자, 출판사, IBSN, 출간일 등의 속성등을 가지고 있다.
  또한 표준타입으로 입고 출처인 Source ENUM 클래스가 있다. 
  
  ```java
  @Entity
  @Table(name = "in_stock_book")
  @Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
  @Data
  @ToString
  public class InStockBook implements Serializable {

    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "title")
    private String title;

    @Column(name = "description")
    private String description;

    @Column(name = "author")
    private String author;

    @Column(name = "publisher")
    private String publisher;

    @Column(name = "isbn")
    private Long isbn;

    @Column(name = "publication_date")
    private LocalDate publicationDate;

    @Enumerated(EnumType.STRING)
    @Column(name = "source")
    private Source source;
    ...(중략)...
  ```

### Source.java

  ```java
  public enum Source {
    Donated, Purchased
  }
  ```

표준타입 Enum 클래스로 선언한 입고출처이다. 입고 출처는 기부 또는 구매로 구분된다. 

### Book.java
  대여도서엔티티이다. 도서엔티티는 실제로 대여가능한 도서이기 때문에 입고도서가 가진 속성에 추가하여 도서대여상태 및 도서분류, 보유도서관 등의 속성을 가지고 있다.
  또한 표준타입으로 대여가능,불가능의 도서상태를 보유한 BookStatus ENUM 클래스와 도서가 보관된 도서관을 의미하는 Location ENUM 클래스, 도서 분류를 위한 Classification ENUM 클래스가 있다.

  ```java
  @Entity
  @Table(name = "book")
  @Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
  @Data
  @ToString 
  public class Book implements Serializable {

    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "title")
    private String title;

    @Column(name = "description")
    private String description;

    @Column(name = "author")
    private String author;

    @Column(name = "publisher")
    private String publisher;

    @Column(name = "isbn")
    private Long isbn;

    @Column(name = "publication_date")
    private LocalDate publicationDate;

    @Enumerated(EnumType.STRING)
    @Column(name = "classification")
    private Classification classification;

    @Enumerated(EnumType.STRING)
    @Column(name = "book_status")
    private BookStatus bookStatus;

    @Enumerated(EnumType.STRING)
    @Column(name = "location")
    private Location location;
    ...(중략)...
  ```

### Classification.java

```java
public enum Classification {
    Arts, Photography, Biographies, BusinessMoney, Children, ComputerTechnology, History, Medical, Travel, Romance, Science, Math, SelfHelp
}
```
도서의 분류를 의미하는 Classification ENUM 클래스이다. 예술, 사진, 경제 등이 있다.

### BookStatus.java

```java
public enum BookStatus {
    AVAILABLE, UNAVAILABLE
}
```

이용가능,불가능을 의미하는 도서의 대여 가능 상태를 나타내는 BookStatus ENUM 클래스이다. 

### Location.java

```java
public enum Location {
    JEONGJA, PANGYO
}
```
도서가 보관된 도서관을 의미하는 Location Enum 클래스이다. 정자 또는 판교로 분류하였다. 


## 내부영역 - 서비스 개발

서비스는 어그리게잇 단위로 서비스을 생성하기 때문에  InStockBookService, BookService 인터페이스를 각각 생성한다. 

InStockBookService의 구현체인 InStockBookServiceImpl는 특별한 로직이 없고, 
BookService는 Rental Service에서 요청한 도서정보조회 기능, 도서 정보 업데이트 시 Catalog Service에 이벤트를 전송하는 기능을 담고있다. 

먼저 도서 정보조회 기능부터 살펴본다.

### BookService.java

```java
public interface BookService {
...(중략)...

BookInfoDTO findBookInfo(Long bookId);

}

```
도서정보조회시, 조회하고자 하는 책의 id를 받는다.

### BookServiceImpl.java

```java
@Service
@Transactional
public class BookServiceImpl implements BookService {

    ...(중략)...

    @Override
    @Transactional
    public Book findBookInfo(Long bookId) {
       return bookRepository.findById(bookId).get();

    }
}
```
도서정보조회 메소드이다.
- bookId로 도서를 조회해 반환한다.

도서정보조회 기능은 Rental 서비스의 도서 대여 기능과 관련된 것으로, Rental 서비스의 동기 호출 응답한다.
동기호출 매커니즘은 [Feign 동기 메세지 호출처리](/contents/jhipster_feign.md)에서 확인할 수 있다.

도서 정보업데이트 기능은 외부 서비스인 Rental서비스, BookCatalog 서비스와 이벤트를 수신/발신해야하기 때문에 외부 어댑터 개발에서 살펴보자. 


## 내부영역 - 레파지토리 개발

레파지토리는 엔티티당 1개식 만든다.

다음은 BookRepository 인터페이스이다. 

```java
@SuppressWarnings("unused")
@Repository
public interface BookRepository extends JpaRepository<Book, Long> {
}
```

다음은 InStockBookRepository 인터페이스인데 제목으로 도서찾는 기능을 추가했다. 

```java
@SuppressWarnings("unused")
@Repository
public interface InStockBookRepository extends JpaRepository<InStockBook, Long> {
    Page<InStockBook> findByTitleContaining(String title, Pageable pageable);
}
```
사용자가 도서의 제목을 모두 입력하지 않아도 조회할 수 있도록 `findByContaining`로 조회하도록 하였다. 


## 외부영역 - REST 컨트롤러 개발

프론트에 제공하는 REST API는 다음 기능을 제공해야 한다.  

- 도서정보조회
- 입고도서등록
- 도서등록
- 도서수정
- 도서삭제

입고도서 등록은 단순히 InstockBook Entity 생성/저장이기 때문에 생략하였다.
도서 정보조회 API는 GET방식으로 ("/books/bookInfo/{bookId}")으로 선언하였다. 도서 조회 비즈니스로직 처리는 bookService.findBookInfo로 위임하였다.
도서 등록/수정/삭제는 클라이언트 요청을 받은 후 도서 서비스를 호출하여 위임하였으며, bookCatalog로 이벤트를 전송하기 때문에 아웃바운드 어댑터 개발에서 살펴보도록한다.

### BookResource.java

```java
@RestController
@RequestMapping("/api")
public class BookResource {

    ...(중략)...
    
    //도서정보조회
    @GetMapping("/books/bookInfo/{bookId}")
    public ResponseEntity<BookInfoDTO> findBookInfo(@PathVariable("bookId") Long bookId){
        Book book = bookService.findBookInfo(bookId);
        BookInfoDTO bookInfoDTO = new BookInfoDTO(bookId, book.getTitle());
        log.debug(bookInfoDTO.toString());
        return ResponseEntity.ok().body(bookInfoDTO);
    }   
}

```

도서 정보 조회 API 처리 흐름을 살펴보면
- Http Get 방식으로 조회할 도서의 id를 받는다. 
- 도서 서비스를 호출하여 조회할 도서를 반환 받는다.
- Rental서비스로 반환할 BookInfoDTO를 생성한다.
- 도서정보조회 API는 Rental 서비스에서 호출하였기 때문에 Rental 서비스로 요청결과를 반환한다. 


## 외부영역 - 아웃바운드 어댑터 개발
 
도서서비스는 도서가 등록/수정/삭제 되었을 때, 사용자가 업데이트된 도서 정보를 조회할 수 있도록 Catalog서비스에 도서 정보를 전송해야 한다.
따라서, 도서 등록/수정/삭제 시 비동기 메시지가 카프카로 전송되게 한다.

도서등록/수정/삭제 기능이 존재하는 BookServiceImpl 서비스에서 도서 등록/수정/삭제시 비동기 호출로 이벤트를 처리하도록 구현해 보자.

우선 도메인이벤트를 만들자.

### bookChanged.java

```java
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
public class BookChanged {

    private String title;

    private String description;

    private String author;

    private String publicationDate;

    private String classification;

    private Boolean rented;

    private String eventType;

    private Long rentCnt;

    private Long bookId;


}
```

도서 등록/수정/삭제 시 위의 도메인 이벤트를 생성하게 된다. BookChanged는 도서 정보와 어떤 이벤트인지 구분하기 위한 이벤트 타입을 담고 있다. 

다음은 이 도메인 이벤트를 생성해서 아웃바운드 어댑터를 호출하는 로직이다. 

먼저 클라이언트의 요청이 들어오고 도서 서비스를 호출하는 REST 컨트롤러부터 살펴보자.

### BookResource.java

```java
@RestController
@RequestMapping("/api")
public class BookResource {
  ...(중략)...

  //도서 등록
    @PostMapping("/books/{inStockId}")
    public ResponseEntity<BookDTO> registerBook(@RequestBody BookDTO bookDTO, @PathVariable Long inStockId) throws  URISyntaxException, InterruptedException, ExecutionException, JsonProcessingException {
        if (bookDTO.getId() != null) {
            throw new BadRequestAlertException("A new book cannot already have an ID", ENTITY_NAME, "idexists");
        }
        Book newBook = bookService.registerNewBook(bookMapper.toEntity(bookDTO), inStockId);
        BookDTO result = bookMapper.toDto(newBook);
        return ResponseEntity.created(new URI("/api/books/" + result.getId()))
            .headers(HeaderUtil.createEntityCreationAlert(applicationName, true, ENTITY_NAME, result.getId().toString()))
            .body(result);
    }

  //도서 정보 수정
    @PutMapping("/books")
    public ResponseEntity<BookDTO> updateBook(@RequestBody BookDTO bookDTO) throws URISyntaxException, InterruptedException, ExecutionException, JsonProcessingException {
        log.debug("REST request to update Book : {}", bookDTO);
        if (bookDTO.getId() == null) {
            throw new BadRequestAlertException("Invalid id", ENTITY_NAME, "idnull");
        }
        Book book = bookService.updateBook(bookMapper.toEntity(bookDTO));
        BookDTO result = bookMapper.toDto(book);
        return ResponseEntity.ok()
            .headers(HeaderUtil.createEntityUpdateAlert(applicationName, true, ENTITY_NAME, bookDTO.getId().toString()))
            .body(result);
    }

  //도서 삭제
   @DeleteMapping("/books/{id}")
    public ResponseEntity<Void> deleteBook(@PathVariable Long id) throws InterruptedException, ExecutionException, JsonProcessingException {
        log.debug("REST request to delete Book : {}", id);
        bookService.delete(id);
        return ResponseEntity.noContent().headers(HeaderUtil.createEntityDeletionAlert(applicationName, true, ENTITY_NAME, id.toString())).build();
    }


}
```

REST 컨트롤러에서 도서 등록/수정/삭제에 대한 클라이언트의 요청이 들어오면 해당 도서를 등록/수정/삭제하는 로직을 수행한다.

도서 등록의 로직은 다음과 같다.
- 입고도서 id와 등록할 도서 정보를 담고있는 bookDTO를 받는다. 
- 도서 서비스를 호출하여 위임한 후 결과를 클라이언트에 반환한다.

도서 수정의 로직은 다음과 같다.
- 수정할 도서 정보를 담고있는 bookDTO를 받는다.
- 도서 서비스를 호출하여 위임한 후 결과를 클라이언트에 반환한다.

도서 삭제의 로직은 다음과 같다.
- 삭제할 도서의 id를 받는다.
- 도서 서비스를 호출하여 위임한 후 결과를 클라이언트에 반환한다.

### BookService.java

```java
public interface BookService {

...(중략)...
Book createBook(Book book) throws InterruptedException, ExecutionException, JsonProcessingException;

Book updateBook(Book book) throws InterruptedException, ExecutionException, JsonProcessingException;

void delete(Long id) throws InterruptedException, ExecutionException, JsonProcessingException;

}
```
컨트롤러에서 호출한 도서 생성/저장/삭제 메소드이다.

### BookServiceImple.java

```java
@Service
@Transactional
public class BookServiceImpl implements BookService {

...(중략)...
    
    @Override
    public Book createBook(Book book) throws InterruptedException, ExecutionException, JsonProcessingException {
        Book createdBook = bookRepository.save(book);
        sendBookCatalogEvent("NEW_BOOK",createdBook.getId());
        return createdBook;
    }

    @Override
    public Book updateBook(Book book) throws InterruptedException, ExecutionException, JsonProcessingException {
        Book updatedBook = bookRepository.save(book);
        sendBookCatalogEvent("UPDATE_BOOK",book.getId());
        return updatedBook;
    }

    @Override
    public void delete(Long id) throws InterruptedException, ExecutionException, JsonProcessingException {
        log.debug("Request to delete Book : {}", id);
        sendBookCatalogEvent("DELETE_BOOK", id);
        bookRepository.deleteById(id);
    }

    
}
```

- 도서 생성 메소드는 createBook으로 컨드롤러에서 전달받은 Book을 저장한 후, sendBookCatalogEvent를 호출하여 비동기 이벤트를 전송한다. 이때, 이벤트 타입은 `NEW_BOOK`이다.
- 도서 수정 메소드는 updateBook으로 컨드롤러에서 전달받은 Book을 저장한 후, sendBookCatalogEvent를 호출하여 비동기 이벤트를 전송한다. 이때, 이벤트 타입은 `UPDATE_BOOK`이다.
- 도서 삭제 메소드는 deleteBook으로 sendBookCatalogEvent를 호출하여 비동기 이벤트를 전송한다. 이때, 이벤트 타입은 `DELETE_BOOK`이다. 이후, 컨드롤러에서 전달받은 bookId로 도서를 삭제한다. 

```java
public interface BookService {

...(중략)...

void sendBookCatalogEvent(String eventType, Long bookId) throws InterruptedException, ExecutionException, JsonProcessingException;

}
```
 도서 Catalog 서비스에 발송할 이벤트를 생성하는 메소드이다. 이벤트 타입과 도서id를 받는다.


### BookServiceImpl.java

```java
@Service
@Transactional
public class BookServiceImpl implements BookService {

...(중략)...
   
    @Override
    public void sendBookCatalogEvent(String eventType,Long bookId) throws InterruptedException, ExecutionException, JsonProcessingException {
        Book book = bookRepository.findById(bookId).get();
        BookChanged bookChanged = new BookChanged();
        if(eventType.equals("NEW_BOOK") || eventType.equals("UPDATE_BOOK")) {
            bookChanged.setBookId(book.getId());
            bookChanged.setAuthor(book.getAuthor());
            bookChanged.setClassification(book.getClassification().toString());
            bookChanged.setDescription(book.getDescription());
            bookChanged.setPublicationDate(book.getPublicationDate().format(fmt));
            bookChanged.setTitle(book.getTitle());
            bookChanged.setEventType(eventType);
            bookChanged.setRented(!book.getBookStatus().equals(BookStatus.AVAILABLE));
            bookChanged.setRentCnt((long) 0);
            bookProducer.sendBookCreateEvent(bookChanged);
        }else if(eventType.equals("DELETE_BOOK")){
            bookChanged.setEventType(eventType);
            bookChanged.setBookId(book.getId());
            bookProducer.sendBookDeleteEvent(bookChanged);
        }
    }
}
```

Catalog에 전송할 비동기 이벤트를 생성하는 메소드이다.
- 이벤트 타입과 도서 id를 받는다.
- 받은 도서 id로 해당 도서를 찾는다.
- CatalogChanged라는 이벤트를 생성한다.
- 이벤트 타입에 따라 도서 정보를 CatalogChanged에 담는다.
  - 도서의 생성과 수정의 경우 도서의 모든 정보를 담는다.
  - 도서의 삭제의 경우 해당 도서의 id만 알아도 삭제할 수 있기 때문에 id만 담는다. 
- 아웃바운드 어댑터인 BookProducer를 호출하여 카프카로 이벤트를 보낸다.
  - 도서의 생성/수정의 경우 bookProducer.sendBookCreateEvent를 호출하고
  - 도서의 삭제의 경우 bookProducer.sendBookDeleteEvent를 호출한다.

다음은 아웃바운드 어댑터이다.

### BookProducer.java

```java

@Service
public class BookProducer {
    ...(중략)...
    public PublishResult sendBookCreateEvent(BookChanged bookChanged)throws ExecutionException, InterruptedException, JsonProcessingException{

        String message = objectMapper.writeValueAsString(bookChanged);
        RecordMetadata metadata = producer.send(new ProducerRecord<>(TOPIC_CATALOG, message)).get();
        return new PublishResult(metadata.topic(), metadata.partition(), metadata.offset(), Instant.ofEpochMilli(metadata.timestamp()));
    }

    public PublishResult sendBookDeleteEvent(BookChanged bookDeleteEvent)throws ExecutionException, InterruptedException, JsonProcessingException{

        String message = objectMapper.writeValueAsString(bookDeleteEvent);
        RecordMetadata metadata = producer.send(new ProducerRecord<>(TOPIC_CATALOG, message)).get();
        return new PublishResult(metadata.topic(), metadata.partition(), metadata.offset(), Instant.ofEpochMilli(metadata.timestamp()));
    }
}
```

도서 서비스에서 호출한 아웃바운드 어댑터, BookProducer이다.
sendBookCreateEvent와 sendBookDeleteEvent 메소드는 도서 서비스로부터 전달받은 CatalogChanged를 카프카 메세지로 변환하여 이벤트를 전송한다. 이때 Topic은 "topic_catalog"로 명명하였다.


## 인바운드 어댑터 개발

도서가 대여/반납되면 Rental 서비스에서 도서가 대여/반납되었다는 비동기 메세지를 발송해 도서 대여 가능상태를 업데이트 시켜야한다. 
따라서, Book 서비스는 도서가 대여/반납되었을 때 발송된 메세지를 수신하여 도서 대여 가능 상태를 업데이트한다.

발송된 비동기 메세지를 수신하는 것은 인바운드 어댑터로 구현한다.

다음은 인바운드 어댑터이다. 

```java
@Service
public class BookConsumer {

...(중략)...

@PostConstruct
    public void start(){
        log.info("Kafka consumer starting ...");
        this.kafkaConsumer = new KafkaConsumer<>(kafkaProperties.getConsumerProps());
        Runtime.getRuntime().addShutdownHook(new Thread(this::shutdown));
        kafkaConsumer.subscribe(Collections.singleton(TOPIC));
        log.info("Kafka consumer started");

        executorService.execute(()-> {
            try {
                while (!closed.get()){
                    ConsumerRecords<String, String> records = kafkaConsumer.poll(Duration.ofSeconds(3));
                    for(ConsumerRecord<String, String> record: records){
                        log.info("Consumed message in {} : {}", TOPIC, record.value());
                       ObjectMapper objectMapper = new ObjectMapper();
                        StockChanged stockChanged = objectMapper.readValue(record.value(), StockChanged.class);
                        bookService.processChangeBookState(stockChanged.getBookId(), stockChanged.getBookStatus());
                    }
                }
                kafkaConsumer.commitSync();
            }catch (WakeupException e){
                if(!closed.get()){
                    throw e;
                }
            }catch (Exception e){
                log.error(e.getMessage(), e);
            }finally {
                log.info("kafka consumer close");
                kafkaConsumer.close();
            }
            }
        );
    }

}
```

인바운드 어댑터는 Rental 서비스가 발송한 이벤트의 토픽을 구독하며 메세지를 polling한다. 
인바운드 어댑터를 살펴보면
- 토픽을 구독하며 메세지를 polling 한다.
- 메세지가 수신되면 해당 메세지를 ObjectMapper를 통해 StockChanged라는 도메인 객체로 변환한다.
- BookService.processChangedBookState를 호출하여 StockChanged를 전달한다. 

### BookService.java

```java
public interface BookService {

...(중략)...
void processChangeBookState(Long bookId, String bookStatus);

}
```

BookConsumer에서 호출한 processChangedBookState 메소드이다. 

### BookServiceImpl.java

```java

@Service
@Transactional
public class BookServiceImpl implements BookService {
...(중략)...
    @Override
    public void processChangeBookState(Long bookId, String bookStatus) {
        Book book = bookRepository.findById(bookId).get();
        book.setBookStatus(BookStatus.valueOf(bookStatus));
        bookRepository.save(book);
    }
}

```
도서상태를 수정하는 메소드는 전달받은 도서Id로 해당 도서를 조회한 뒤, 도서 상태를 수정 및 저장한다. 

## 단위테스트 수행

마이크로서비스가 분리되어 있기 때문에 동기 및 비동기 테스트는 여러 서비스를 모두 기동하고 수행할 수 밖에 없다. 따라서 이런 테스트는 통합테스트를 통해 수행한다.

서비스를 수행하는 단위테스트는 레이어의 여러 지점에서 수행할 수 있는데 현 시점에서는 서비스가 가진 로직구현을 점검하기 위한 지점인 서비스 인터페이스 앞에서 수행하자. 

다음과 같이 서비스를 테스트하는 Junit 테스트케이스를 생성한다.

여러개의 상태와 예약을 보유한 도서객체 샘플을 생성한다. 
도서대여가능여부확인 및 도서예약처리 기능을 호출하여 의도된 결과가 반환되는 지 체크한다.





