# Rental과 Book, Kafka를 통한 비동기식 연결

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

    public void updateBookStatus(List<Long> bookIds, String bookStatus){
        try {
            for(Long bookId : bookIds) {
                UpdateBookEvent updateBookEvent = new UpdateBookEvent(bookId, bookStatus);
                String message = objectMapper.writeValueAsString(updateBookEvent);
                ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC, message);
                producer.send(record);
            }
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

UpdateBookEvent는 말 그대로, Book상태 변경을 위한 Event Object로, domain 패키지 아래 생성한다.

2. UpdateBookEvent.java

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


이제 Rental이 보내는 Kafka메세지를 구독할 Consumer를 만들어보자.

## Book