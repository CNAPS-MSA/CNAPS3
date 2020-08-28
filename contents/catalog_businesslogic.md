# 카탈로그(Catalog)서비스 구현
**코드는 아무 예고 없이 언제든 변경될 수 있으니, 실제 코드는 해당 서비스 애플리케이션의 Repository에서 확인하세요.**

Sample에서 보여줄 기능은 아래와 같다.

## 구현기능
  - 도서목록 및 검색기능
    1. 도서입고 시 Catalog생성
  - 인기도서목록 기능   
    1. 대출기록으로 인기도서 목록 생성 

## API설계
주요한 API 설계는 다음과 같다.

|API명|인기도서목록 조회|
|----|------|
|리소스URI|/book-catalogs/top-10|
|Method|GET|
|Request| |
|Response| |

대여 횟수가 많은 순으로 상위 10개의 도서목록을 조회한다.

## 도메인 모델링

BookCatalog 서비스는 도서 정보와 도서 목록 조회를 위한 서비스이다. Book 서비스에서 도서가 등록/수정/삭제 됨에 따라 BookCatalog 또한 등록/수정/삭제된다.
또한, BookCatalog는 조회와 검색을 위한 서비스인만큼 NoSQL인 MongoDB로 구현하였다.

## 유스케이스 흐름

## 내부영역 - 도메인 모델 개발

### BookCatalog.java

Catalog 엔티티이다. 조회/검색 용도이기 때문에 사용자가 도서 조회/검색시 필요한 속성들만을 가지고 있다.
따라서, 도서 제목, 설명, 작가, 출판사 등의 속성을 갖고 있다.
DB가 MongoDB이기 때문에 기존의 다른 서비스들과 다르게 엔티티에 `@Entity`가 아닌 `@Document`라는 어노테이션이 쓰인다. 

```java
@Document(collection = "book_catalog")
@Data
@ToString
public class BookCatalog implements Serializable {

    private static final long serialVersionUID = 1L;

    @Id
    private String id;

    @Field("title")
    private String title;

    @Field("description")
    private String description;

    @Field("author")
    private String author;

    @Field("publication_date")
    private LocalDate publicationDate;

    @Field("classification")
    private String classification;

    @Field("rented")
    private Boolean rented;

    @Field("rent_cnt")
    private Long rentCnt;

    @Field("book_id")
    private Long bookId;
  
  ...(중략)...

```

## 내부영역 - 서비스 개발

BookCatalogService에서는 도서 정보 생성/수정/삭제기능을 구현하였다.
도서 정보 생성/수정/삭제/상태변경은 외부서비스인 Book서비스의 비동기호출에 의해 이뤄지므로 인바운드 어댑터 개발에서 다루도록 한다.
다음은 도서 제목으로 도서를 검색하는 기능과 인기도서 목록을 불러오는 기능이다.

### BookCatalogService.java

```java
public interface BookCatalogService {
 ...(중략)...
 Page<BookCatalog> findBookByTitle(String title, Pageable pageable);

 List<BookCatalog> loadTop10();

}
```

### BookCatalogServiceImpl.java

```java

@Service
public class BookCatalogServiceImpl implements BookCatalogService {
  ...(중략)...
    @Override
    public Page<BookCatalog> findBookByTitle(String title, Pageable pageable)
    {
        return bookCatalogRepository.findByTitleContaining(title, pageable);
    }

    @Override
    public List<BookCatalog> loadTop10() {
        return bookCatalogRepository.findTop10ByOrderByRentCntDesc();
    }

}
```
도서 검색기능과 인기도서 검색 기능 모두 BookCatalogRepository의 JPA 메소드를 실행시켜 목록을 불러온다. 

## 내부영역 - 레파지토리 개발

BookCatalogService에서 쓰인 도서 검색 메소드와 인기도서 검색 메소드를 살펴보자.

### BookCatalogRepository.java

```java
@Repository
public interface BookCatalogRepository extends MongoRepository<BookCatalog, String> {
    
    Page<BookCatalog> findByTitleContaining(String title, Pageable pageable);

    List<BookCatalog> findTop10ByOrderByRentCntDesc();

    ..(중략)...
}
```

도서 제목으로 검색하는 메소드는 `findByTitleContaining`으로, 받아온 title을 포함하고 있는 BookCatalog 리스트를 반환한다.

인기도서를 검색하는 메소드는 `findByTitle10ByOrderByRentCntDesc()`로, 메소드를 천천히 해석해보면
- 상위10개의 BookCatalog검색
- RentCnt(대여횟수) 순으로 정렬
- 내림차순
즉, 대여횟수 순으로 내림차순 정렬하여 상위10개의 BookCatalog 리스트를 반환한다. 

또한, 위에서 살펴볼 수 있듯이 NoSQL인 MongoDB를 사용한다하더라도 Spring Data JPA 쿼리메소드를 동일하게 사용할 수 있다.
단, NoSQL은 Document 형식이며 ID의 속성이 String인 점을 감안하여 약간의 차이가 있을 수 있으므로 Spring Data MongoDB의 공식 문서를 살펴보는 것을 권장한다.

## 외부영역 - REST 컨트롤러 개발

BookCatalog는 요청하는 도서 리스트를 반환해주는 REST API로만 구성되어있다.
또한, REST 컨트롤러는 기본적으로 Entity가 아닌 DTO로 클라이언트와 데이터를 주고 받기 때문에, REST 컨트롤러에서 bookMapper를 실행하여 bookCatalog 서비스에서 반환한 entity를 DTO로 변환시킨다. 
도서 리스트 반환 API 메소드를 살펴보자.

### BookCatalogResource.java

```java
@RestController
@RequestMapping("/api")
public class BookCatalogResource {

    ...(중략)...

    @GetMapping("/book-catalogs/top-10")
    public ResponseEntity<List<BookCatalog>> loadTop10Books(){
        List<BookCatalog> bookCatalogs = bookCatalogService.loadTop10();
        return ResponseEntity.ok().body(bookCatalogs);
    }

}
```

- 인기도서 목록 조회 API : GET방식이며 ("/book-catalogs/top-10")으로 선언, book서비스를 호출하여 대여횟수를 기준으로 상위 10개의 도서 리스트를 반환한다.
  
## 인바운드 어댑터 개발

Book서비스에서 도서를 생성/수정/삭제하면 BookCatalog서비스로 이벤트를 발송한다. 또한 Rental서비스에서 도서를 대여/반납하면 BookCatalog서비스로 도서 상태 변경 이벤트를 발송한다.
BookCatalog의 인바운드 어댑터에서 받는 BookChanged이벤트 내의 이벤트 종류는 총 4가지(생성/수정/삭제/상태변경)로, 메세지를 수신한 뒤 bookCatalog서비스를 호출하여 해당 이벤트 종류에 따라 각각 다른 메소드를 실행시켜 처리한다.  

메세지 수신부터 BookCatalog 서비스를 호출하여 생성/수정/삭제/상태변경하는 프로세스까지 살펴보자.

### BookCatalogConsumer.java

```java
@Service
public class BookCatalogConsumer {
  
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
                            BookChanged bookChanged = objectMapper.readValue(record.value(), BookChanged.class);
                            bookCatalogService.processCatalogChanged(bookChanged);
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

Book서비스에서 카프카 메세지를 보낼때 명명하였던 Topic과 동일한 Topic을 수신한다. 따라서 "topic_catalog"라는 토픽을 수신하여 Object Mapper를 통해 수신한 카프카메세지를 BookChanged로 변환하여 메세지를 읽는다.
bookCatalogService.processCatalogChanged를 호출하여 변환된 BookChanged를 보내 bookCatalog 생성/수정/삭제 이벤트를 수행한다.

### BookCatalogService.java

```java
public interface BookCatalogService {
    ...(중략)...

    //kafka 이벤트 종류별 카테고라이징 처리
    void processCatalogChanged(BookChanged bookChanged);
    //신규 도서 등록 
    BookCatalog registerNewBook(BookChanged bookChanged);
    //도서 삭제
    void deleteBook(BookChanged bookChanged);
    //도서 상태 수정
    BookCatalog updateBookStatus(BookChanged bookChanged);
    //도서 정보 수정
    BookCatalog updateBookInfo(BookChanged bookChanged);

}
```

bookCatalogService에 BookChanged를 받아 이벤트 종류별로 메소드를 호출하는 processCatalogChanged와 도서 등록/삭제/수정을 위한 메소드를 선언하였다.

### BookCatalogServiceImpl.java

```java
@Service
public class BookCatalogServiceImpl implements BookCatalogService {
  
  ...(중략)...

    //이벤트 카테고라이징
    @Override
    public void processCatalogChanged(CatalogChanged catalogChanged) {
        String eventType  = catalogChanged.getEventType();
        switch (eventType) {
            case "NEW_BOOK":
               registerNewBook(catalogChanged);
                break;
            case "DELETE_BOOK":
                deleteBook(catalogChanged);
                break;
            case "RENT_BOOK":
            case "RETURN_BOOK":
                updateBookStatus(catalogChanged);
                break;
            case "UPDATE_BOOK":
                updateBookInfo(catalogChanged);
                break;
        }
    }
}

```

kafka 이벤트를 카테고라이징 하는 메소드이다. BookChanged의 이벤트 타입에 따라 생성/삭제/수정 메소드를 실행시킨다.

### BookCatalogServiceImpl.java

```java
@Service
public class BookCatalogServiceImpl implements BookCatalogService {
   
   ...(중략)...

    //신규도서 등록
    @Override
    public BookCatalog registerNewBook(CatalogChanged catalogChanged) {
        System.out.println("register new book");
        BookCatalog bookCatalog = new BookCatalog();
        bookCatalog.setBookId(catalogChanged.getBookId());
        bookCatalog.setAuthor(catalogChanged.getAuthor());
        bookCatalog.setClassification(catalogChanged.getClassification());
        bookCatalog.setDescription(catalogChanged.getDescription());
        bookCatalog.setPublicationDate(LocalDate.parse(catalogChanged.getPublicationDate(), fmt));
        bookCatalog.setRented(catalogChanged.getRented());
        bookCatalog.setTitle(catalogChanged.getTitle());
        bookCatalog.setRentCnt(catalogChanged.getRentCnt());
        bookCatalog= bookCatalogRepository.save(bookCatalog);
        return bookCatalog;
    }
}
```

BookChanged의 eventType이 "NEW_BOOK"일때 실행시키는 메소드로, BookCatalog를 생성한다.
이때, 전달받은 BookChanged의 정보에 따라 BookCatalog를 생성 후, 저장한다.

### BookCatalogServiceImpl.java

```java
@Service
public class BookCatalogServiceImpl implements BookCatalogService {
   
   ...(중략)...

    //도서 삭제
    @Override
    public void deleteBook(CatalogChanged catalogChanged) {
        bookCatalogRepository.deleteByBookId(catalogChanged.getBookId());
    }
}

```

BookChanged의 eventType이 "DELETE_BOOK"일때 실행시키는 메소드로, 전달받은 BookChanged의 도서 id로 BookCatalog를 찾아 삭제한다.

### BookCatalogServiceImpl.java

```java
@Service
public class BookCatalogServiceImpl implements BookCatalogService {
   
   ...(중략)...

    //도서 상태 수정
    @Override
    public BookCatalog updateBookStatus(BookChanged bookChanged) {
        BookCatalog bookCatalog = bookCatalogRepository.findByBookId(bookChanged.getBookId());
        if(catalogChanged.getEventType().equals("RENT_BOOK")) {
            Long newCnt = bookCatalog.getRentCnt() + (long) 1;
            bookCatalog.setRentCnt(newCnt);
            bookCatalog.setRented(true);
            bookCatalog= bookCatalogRepository.save(bookCatalog);
        }else if(catalogChanged.getEventType().equals("RETURN_BOOK")){
            bookCatalog.setRented(false);
            bookCatalog= bookCatalogRepository.save(bookCatalog);

        }
        return bookCatalog;

    }
}
```

BookChanged의 eventType이 "RENT_BOOK" 또는 "RETURN_BOOK" 일때 실행시키는 메소드이다.
처리 흐름은 다음과 같다.
- 전달받은 BookChanged의 해당 도서 id로 해당 BookCatalog를 찾는다.
- 이벤트 타입이 "RENT_BOOK"인 경우
  - 대여 횟수를 +1 하고, 대여 중으로 상태를 수정한다.
- 이벤트 타입이 "RETURN_BOOK"인 경우
  - 대여 상태를 대여 가능으로 수정한다.
- 수정한 bookCatalog를 저장한다.

### BookCatalogServiceImpl.java

```java
@Service
public class BookCatalogServiceImpl implements BookCatalogService {
   
   ...(중략)...

    //도서 정보 수정
    @Override
    public BookCatalog updateBookInfo(BookChanged bookChanged) {
        BookCatalog bookCatalog = bookCatalogRepository.findByBookId(bookChanged.getBookId());
        bookCatalog.setAuthor(bookChanged.getAuthor());
        bookCatalog.setClassification(bookChanged.getClassification());
        bookCatalog.setDescription(bookChanged.getDescription());
        bookCatalog.setPublicationDate(LocalDate.parse(bookChanged.getPublicationDate(), fmt));
        bookCatalog.setRented(bookChanged.getRented());
        bookCatalog.setTitle(bookChanged.getTitle());
        bookCatalog.setRentCnt(bookChanged.getRentCnt());
        bookCatalogRepository.save(bookCatalog);
        return bookCatalog;
    }

}
```

BookChanged의 eventType이 "UPDATE_BOOK"일때 실행시키는 메소드로, 전달받은 BookChanged의 도서 id로 BookCatalog를 찾아 도서정보를 수정한다.

## 단위테스트 수행

