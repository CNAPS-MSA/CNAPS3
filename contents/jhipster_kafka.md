# 타서비스 비동기호출처리 : Kafka를 통한 EDA구현 (Rental과 Book)

도서 대여와 반납을 진행한 후, 결과에 따라 해당 도서의 상태를 대여 불가능과 가능으로 변경해야한다.

도서 대여 완료시 -> 도서 상태 : 대여불가능
도서 반납 완료시 -> 도서 상태 : 대여 가능

![image](https://user-images.githubusercontent.com/18453570/83232000-5d0e3f00-a1c7-11ea-9ae8-0cff1cd756e6.png)

## Rental에 Kafka Producer 만들기

`com.skcc.rental`의 하위 패키지로 `adaptor`패키지를 생성한뒤, `adaptor`패키지 안에 `RentalKafkaProducer.java`를 생성한다.

adaptor 패키지는 외부 서비스와 통신할 때 필요한 것들을 구현하는 패키지이다.

1. RentalKafkaProducer.java

Kafka Producer의 코드는 아래와 같다.


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
public class RentalKafkaProducer {

    private final Logger log = LoggerFactory.getLogger(RentalKafkaProducer.class);

    private static final String TOPIC = "topic_kafka";

    private final KafkaProperties kafkaProperties;

    private final static Logger logger = LoggerFactory.getLogger(RentalKafkaProducer.class);
    private KafkaProducer<String, String> producer;
    private final ObjectMapper objectMapper = new ObjectMapper();


    public RentalKafkaProducer(KafkaProperties kafkaProperties) {
        this.kafkaProperties = kafkaProperties;
    }

    @PostConstruct
    public void initialize(){
        log.info("Kafka producer initializing...");
        this.producer = new KafkaProducer<>(kafkaProperties.getProducerProps());
        Runtime.getRuntime().addShutdownHook(new Thread(this::shutdown));
        log.info("Kafka producer initialized");
    }

    public void updateBookStatus(Long bookId, String bookStatus){
        try {
                UpdateBookEvent updateBookEvent = new UpdateBookEvent(bookId, bookStatus);
                String message = objectMapper.writeValueAsString(updateBookEvent);
                ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC, message);
                producer.send(record);

        } catch (JsonProcessingException e) {
            logger.error("Could not send book List",e);
            e.printStackTrace();
        }
    }


    @PreDestroy
    public void shutdown(){
        log.info("Shutdown Kafka producer");
        producer.close();
    }
}

```

위 코드에서 UpdateBookEvent라는 object를 생성하여 메세지를 보내는 것을 확인할 수 있다.
메세지를 보낼 때는 objectMapper로 UpdateBookEvent를 String message로 형태를 변경하고 Producer Recored라는 리스트에 담아 발송한다. 




2. UpdateBookEvent.java

UpdateBookEvent는 말 그대로, Book상태 변경을 위한 Event Object로, domain 패키지 아래 생성한다.

```java
package com.skcc.rental.domain;

public class UpdateBookEvent {

    private Long bookId;
    private String bookStatus;

    public UpdateBookEvent(Long bookId, String bookStatus){
        this.bookId = bookId;
        this.bookStatus = bookStatus;

    }

    public Long getBookId() {
        return bookId;
    }

    public void setBookId(Long bookId) {
        this.bookId = bookId;
    }

    public String getBookStatus() {
        return bookStatus;
    }

    public void setBookStatus(String bookStatus) {
        this.bookStatus = bookStatus;
    }
}

```


3. RentalResource.java

그리고 book에 메세지를 보낼 때는 아래와같이 보내면된다.

```java

    ...

    ...


    books.stream().forEach(b -> rentalService.updateBookStatus(b, "AVAILABLE"));

```

위 코드는 책을 반납한 뒤, 책 상태를 변경해주기 위해 보내는 메세지이다.

book ID List를 받아 각각 bookId와`AVAILABLE`이란 상태 메세지를 담아 보내는 것이다. 
    
또, 바로 kafka producer에 접근하지 않고 service를 통해 메소드를 실행시켰다.

4. RentalServiceImpl.java

```java

    private final RentalKafkaProducer rentalKafkaProducer;

    public RentalServiceImpl(RentalRepository rentalRepository, RentedItemRepository rentedItemRepository, ReturnedItemRepository returnedItemRepository,
                             RentalKafkaProducer rentalKafkaProducer) {
        this.rentalRepository = rentalRepository;
        this.rentedItemRepository = rentedItemRepository;
        this.returnedItemRepository = returnedItemRepository;
        this.rentalKafkaProducer = rentalKafkaProducer;
    }
    
    ...

    @Override
    public void updateBookStatus(Long bookId, String bookStatus) {
        rentalKafkaProducer.updateBookStatus(bookId, bookStatus);
    }
```

Service에 updateBookStatus 메소드를 선언하고 ServiceImpl에서 구현한다. 
이때, Kafka Producer를 private final로 선언하고 constructor에 포함시키는 것을 잊지말자.


이제 Rental이 보내는 Kafka메세지를 구독할 Consumer를 만들어보자.

## Book에 consumer 구현하기


Rental에 구현한 것과 마찬가지로, `com.skcc.book`의 하위 패키지로 `adaptor`패키지를 생성한뒤, `adaptor`패키지 안에 `BookKafkaConsumer.java`를 생성한다.

adaptor 패키지는 외부 서비스와 통신할 때 필요한 것들을 구현하는 패키지이다.

1. BookKafkaConsumer.java


```java
package com.skcc.book.adaptor;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.skcc.book.config.KafkaProperties;
import com.skcc.book.domain.Book;
import com.skcc.book.domain.enumeration.BookStatus;
import com.skcc.book.repository.BookRepository;
import com.skcc.book.domain.UpdateBookEvent;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.errors.WakeupException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

import javax.annotation.PostConstruct;
import java.time.Duration;
import java.util.Collections;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicBoolean;

@Service
public class BookKafkaConsumer {

    private final Logger log = LoggerFactory.getLogger(BookKafkaConsumer.class);

    private final AtomicBoolean closed = new AtomicBoolean(false);

    public static final String TOPIC ="topic_kafka";

    private final KafkaProperties kafkaProperties;

    private KafkaConsumer<String, String> kafkaConsumer;

    private BookRepository bookRepository;

    private ExecutorService executorService = Executors.newCachedThreadPool();


    public BookKafkaConsumer(KafkaProperties kafkaProperties, BookRepository bookRepository) {
        this.kafkaProperties = kafkaProperties;
        this.bookRepository = bookRepository;
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
                        UpdateBookEvent updateBookEvent = objectMapper.readValue(record.value(), UpdateBookEvent.class);
                        Book book = bookRepository.findById(updateBookEvent.getBookId()).get();
                        book.setBookStatus(BookStatus.valueOf(updateBookEvent.getBookStatus()));
                        bookRepository.save(book);

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

2. UpdateBookEvent.java

```java
public class UpdateBookEvent {

    private Long bookId;
    private String bookStatus;

    public Long getBookId() {
        return bookId;
    }

    public void setBookId(Long bookId) {
        this.bookId = bookId;
    }

    public String getBookStatus() {
        return bookStatus;
    }

    public void setBookStatus(String bookStatus) {
        this.bookStatus = bookStatus;
    }
}
```
UpdateBookEvent는 Rental과 마찬가지로 domain 패키지 아래에 위치하게 하였다.
여기서 Rental의 UpdateBookEvent와 다른점은 Contructor가 없다는 점이다. 현재는 Constructor를 만들 필요가 없었기 때문에 만들지 않았다.

이제 다음은 도서 대여시 책의 ID로 책의 정보를 가져오는 Feign을 구현해보자.

- [Feign Client 구성하기](/contents/jhipster_feign.md)
