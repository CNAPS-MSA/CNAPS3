# 타서비스 비동기호출처리 : Kafka를 통한 EDA구현 (Rental과 Book)

다음은  Kafka를 활용한 비동기 메시지 전송 처리의 구현이다. 

![image](https://user-images.githubusercontent.com/18453570/83232000-5d0e3f00-a1c7-11ea-9ae8-0cff1cd756e6.png)

위 그림과 같이 도서 대여와 반납을 진행한 후, 결과에 따라 해당 도서의 상태를 대여 불가능과 가능으로 변경해야한다.
- 도서 대여 완료시 -> 도서 상태 : 대여불가능
- 도서 반납 완료시 -> 도서 상태 : 대여 가능
도메인 이벤트를 비동기 통신으로 전송하는 방식으로 구현하기로 결정했고, 비동기 메시지 매커니즘의 신뢰성을 보장하기 위한 메시지 큐가 필요하다. 우린 메시지 큐로 Apache Kafka를 선택했다. 
우선 개발환경에kaffa를 설치하고 설정해 보자.

- 카프카 설치과정

## Rental에 Kafka Producer 만들기

rental의 하위 패키지로 adaptor패키지를 생성한뒤, adaptor패키지 안에 RentalProducer.java를 생성한다. 
adaptor 패키지는 외부 영역의 아웃바운드 아답터를 구현하는 패키지이다. Producer의 코드는 아래와 같다.

### RentalProducer.java

RentalProducer의 코드는 아래와 같다.

```java
package com.skcc.rental.adaptor;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.skcc.rental.config.KafkaProperties;
import com.skcc.rental.domain.UpdateBookEvent;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import java.util.List;

@Service
public class RentalProducer {

    private final Logger log = LoggerFactory.getLogger(RentalProducer.class);

    private static final String TOPIC_BOOK = "topic_book";
    private static final String TOPIC_CATALOG = "topic_catalog";
    private static final String TOPIC_POINT = "topic_point";

    private final KafkaProperties kafkaProperties;
    private final static Logger logger = LoggerFactory.getLogger(RentalProducer.class);
    private KafkaProducer<String, String> producer;
    private final ObjectMapper objectMapper = new ObjectMapper();
    public RentalProducer(KafkaProperties kafkaProperties) {
        this.kafkaProperties = kafkaProperties;
    }

    @PostConstruct
    public void initialize(){
        log.info("Kafka producer initializing...");
        this.producer = new KafkaProducer<>(kafkaProperties.getProducerProps());
        Runtime.getRuntime().addShutdownHook(new Thread(this::shutdown));
        log.info("Kafka producer initialized");
    }

    //책 상태 업데이트
    public PublishResult updateBookStatus(Long bookId, String bookStatus) 
throws ExecutionException, InterruptedException, JsonProcessingException {

        StockChanged stockChanged = new StockChanged(bookId, bookStatus);
        String message = objectMapper.writeValueAsString(stockChanged);
        RecordMetadata metadata = producer.send
(new ProducerRecord<>(TOPIC_BOOK, message)).get();
        return new PublishResult(metadata.topic(), metadata.partition(), 
metadata.offset(), Instant.ofEpochMilli(metadata.timestamp()));

    }

    // 권당 포인트 적립
    public PublishResult savePoints(Long userId, int points) throws ExecutionException, InterruptedException, JsonProcessingException {
        PointChanged pointChanged = new PointChanged(userId, points);
        String message = objectMapper.writeValueAsString(pointChanged);
        RecordMetadata metadata = producer.send(new ProducerRecord<>(TOPIC_POINT, message)).get();
        return new PublishResult(metadata.topic(), metadata.partition(), metadata.offset(), Instant.ofEpochMilli(metadata.timestamp()));
    }

    //대여, 반납  시 book catalog의 책 상태 업데이트
    public PublishResult updateBookCatalogStatus(Long bookId, String eventType)
throws ExecutionException, InterruptedException,JsonProcessingException {
        BookCatalogChanged bookCatalogChanged = new BookCatalogChanged();
        bookCatalogChanged.setBookId(bookId);
        bookCatalogChanged.setEventType(eventType);
        String message = objectMapper.writeValueAsString(bookCatalogChanged);
        RecordMetadata metadata = producer.send(new ProducerRecord<>(TOPIC_CATALOG, message)).get();
        return new PublishResult(metadata.topic(), metadata.partition(), metadata.offset(), Instant.ofEpochMilli(metadata.timestamp()));
    }

    @PreDestroy
    public void shutdown(){
        log.info("Shutdown Kafka producer");
        producer.close();
    }
…

```

위 코드에서 StockChanged라는 도메인 이벤트를 생성하여 메세지를 보내는 것을 확인할 수 있다. 
메세지를 보낼 때는 objectMapper로 StockChanged를 String message로 형태를 변경하고 Producer Recored라는 리스트에 담아 발송한다.

### StockChanged.java
StockChanged는 말 그대로, 도서재고 상태 변경을 위한 도메인 이벤트 객체로, domain 패키지 아래 event 패키지에 생성한다.

```java
package com.skcc.rental.domain.event;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
@AllArgsConstructor
public class StockChanged {

    private Long bookId;
    private String bookStatus;

}

```
그리고 위에서도 한번 살펴보았지만 메세지를 보낼 때는 아래와 같이 보내면된다.

### RentalServiceImpl.java
```java
@Service
@Transactional
public class RentalServiceImpl implements RentalService {
…
private final RentalProducer rentalProducer;
…
public Rental returnBooks(Long userId, List<Long> bookIds) {
...
    updateBookStatus(b, "AVAILABLE");
…

@Override
public void updateBookStatus(Long bookId, String bookStatus) throws ExecutionException, InterruptedException, JsonProcessingException {
    rentalProducer.updateBookStatus(bookId, bookStatus);
}

```
위 코드는 책을 반납한 뒤, 책 상태를 변경해주기 위해 보내는 메세지이다.
RentalProducer 를 private final로 선언하고 constructor에 포함시킨 후에 반납하기(returnBooks)메소드에서 book ID List를 받아 각각 bookId와AVAILABLE이란 상태 메세지를 담아 보내는 것이다.
이제 대여서비스가 보내는 Kafka메세지를 구독할 도서서비스의Consumer를 만들어보자.

## 도서서비스에 consumer 구현하기
도서서비스에 구현한 것과 마찬가지로, 도서서비스의 하위 패키지로 adaptor패키지를 생성한뒤, adaptor패키지 안에 BookConsumer.java를 생성한다.

### BookConsumer.java
```java
package com.skcc.book.adaptor;

import …

@Service
public class BookConsumer {

    private final Logger log = LoggerFactory.getLogger(BookConsumer.class);
    private final AtomicBoolean closed = new AtomicBoolean(false);
    public static final String TOPIC ="topic_book";
    private final KafkaProperties kafkaProperties;
    private KafkaConsumer<String, String> kafkaConsumer;
    private BookService bookService;
    private ExecutorService executorService = Executors.newCachedThreadPool();

    public BookConsumer(KafkaProperties kafkaProperties, BookService bookService) {
        this.kafkaProperties = kafkaProperties;
        this.bookService = bookService;
    }

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
                        StockChanged stockChanged = 
objectMapper.readValue(record.value(), StockChanged.class);
                        Book book = bookService.findOne(stockChanged.getBookId()).get();
                        book.setBookStatus(BookStatus.valueOf
(stockChanged.getBookStatus()));
                        bookService.save(book);
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

    public KafkaConsumer<String, String> getKafkaConsumer() {
        return kafkaConsumer;
    }
    public void shutdown() {
        log.info("Shutdown Kafka consumer");
        closed.set(true);
        kafkaConsumer.wakeup();
    }
}
```
위의 코드를 보면 카프카에서 읽은 메시지를 대여서비스가 보낸 StockChaanged 도메인 이벤트로 변환하고 이 도메인 이벤트 정보를 가지고 bookService를 호출하여 도서의 재고 상태를 업데이트 하는 것을 볼 수 있다. 다음은 도서서비스에 존재하는 StockChaanged도메인 이벤트 클래스이다. 대여서비스의 클래스와 동일하다.

### StockChanged.java
```java
package com.skcc.book.domain;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class StockChanged {

    private Long bookId;
    private String bookStatus;
}
```
StockChaanged는 Rental과 마찬가지로 domain 패키지 아래 Event 패키지에 위치한다. 

다음은 도서 대여시 책의 ID로 책의 정보를 가져오는 Feign을 구현해보자.

- [Feign Client 구성하기](/contents/jhipster_feign.md)
