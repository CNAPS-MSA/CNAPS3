# 타서비스 동기호출처리 : Feign Client 연결하기

Kafka는 비동기식 메세징 방식이며 Feign은 동기식 메세징 방식이다.

Feign Client를 직접 구현하기 전, Feign을 적용할 Application들의 `application.yml`을 아래와 같이 수정한다. (Rental, Book)

```yaml
feign:
  hystrix:
    enabled: true
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000

# See https://github.com/Netflix/Hystrix/wiki/Configuration
hystrix:
  command:
    default:
      execution:
        isolation:
          strategy: SEMAPHORE
          # See https://github.com/spring-cloud/spring-cloud-netflix/issues/1330
          thread:
            timeoutInMilliseconds: 10000
```

위 코드는 feign과 hystrix설정이다.
먼저, feign을 사용한다고 하더라도 feign의 설정보다는 hystrix의 설정이 우선된다.

Hystrix를 사용하는 경우 기본적으로 thread time out이 1초이기 때문에 기본 설정으로는 feign의 connection, read timeout이 1초 이상인 경우라도 1초 안에 응답이 오지 않으면 fallback이 실행된다.
현재 Sample에서는 fallback을 구현하지 않은 상태이기 때문에 Timeout시간을 10초로 길게 두었다. 

추후, circuit breaker를 적용하면서 fallback을 처리할 예정이다.

## Rental에서 Book을 호출 -> Book의 응답 받기

1. Rental의 adaptor 패키지에 BookClient생성

- BookClient.java

```java
package com.skcc.rental.adaptor;

import com.skcc.rental.config.FeignConfiguration;
import com.skcc.rental.web.rest.dto.BookInfo;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

import java.util.List;

@FeignClient(name= "book", configuration = {FeignConfiguration.class})
public interface BookClient {
    @GetMapping("/api/getBookInfo/{bookIds}")
    List<BookInfo> getBookInfo(@PathVariable("bookIds") List<Long> bookIds);
}
```

위와 같이, BookClient라는 interface를 작성한다. 위 코드는 bookId리스트를 보내면 BookInfo라는 obejct리스트를 리턴받는 메소드이다.

BookInfo는 아래와 같다.

2. BookInfo.java

BookInfo는 Book과 동기식 통신을 위한 객체클래스 이기 때문에 dto 패키지에 생성하였다.

```java
package com.skcc.rental.web.rest.dto;

import java.io.Serializable;

public class BookInfo implements Serializable {
    private Long id;

    private String title;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }
}
```

일반적인 DTO 형식과 동일하다. 단, 이 클래스에 constructor는 생성하지 않아야한다. 


3. RentalResource에서 BookClient 사용하기

- RentalResource.java

```java

private final BookClient bookClient;

public RentalResource(RentalService rentalService, RentalMapper rentalMapper, BookClient bookClient) {
        this.rentalService = rentalService;
        this.rentalMapper = rentalMapper;
        this.bookClient = bookClient;
}

...

List<BookInfo> bookInfoList = bookClient.getBookInfo(books);

...

```

위와 같이 BookClient를 상단에 선언하고 Constructor에 포함시킨다.

그 다음, 도서 정보를 받아와야하는 로직 부분에 bookClient에 선언한 메소드를 불러 통신하고 결과를 받아온다.


## Book에서 Rental의 호출 받기

응답하는 서비스에서의 구현은 간단하다.

1. BookResource.java
   
아래와 같이 BookResource에 Rental에서 생성한 BookClient에 선언했던 메소드와 동일하게 구현한다.

Book Id 리스트를 받고, 받은 Id리스트를 bookService에 BookInfo로 조합해주는 메소드를 부른 뒤, 결과를 리턴한다.

```java

    @GetMapping("/getBookInfo/{bookIds}")
    public List<BookInfo> getBookInfo(@PathVariable("bookIds") List<Long> bookIds){
        log.debug("Got feign request!!");
        List<BookInfo> bookInfoList= bookService.getBookInfo(bookIds);

        return bookInfoList;
    }
```

2. BookServiceImpl.java

```java
   @Override
    @Transactional(readOnly = true)
    public List<BookInfo> getBookInfo(List<Long> bookIds) {
        List<BookInfo> bookInfoList = bookIds.stream()
            .filter(b -> bookRepository.findById(b).get().getBookStatus().equals(BookStatus.AVAILABLE)) //불가능상태인 book에 대해 rental에 알람 또는 예외처리 필요
            .map(b -> new BookInfo(b, bookRepository.findById(b).get().getTitle()))
            .collect(Collectors.toList());
        return bookInfoList;
    }
```

BookInfo리스트로 만드는 방식은 java stream으로 구현하였다. 받은 BookId로 해당 도서를 찾고, 도서가 대여 가능 상태이면 BookInfo로 만들어 리스트로 변환시켰다.

3. BookInfo.java

```java  
package com.skcc.book.web.rest.dto;

import javax.validation.constraints.NotNull;
import java.io.Serializable;

public class BookInfo implements Serializable {
    private Long id;

    private String title;

    public BookInfo(Long id, String title) {
        this.id = id;
        this.title = title;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }
}
```

BookInfo 또한 Rental에서와 마찬가지로 dto 패키지 아래 생성하였다. Rental과의 차이점은 BookInfo의 Constructor가 선언되어있다는 것인데, 이는 BookService에서 BookId를 BookInfo로 재조합하는 과정에서 필요하기 때문에 선언하였다.

**Rental의 BookInfo에서는 Constructor를 절대 선언해서는 안된다.** 선언하는 경우 No such a constructor(?)과 같은 엄청난 오류 코드를 볼 수 있다...

## EnableFeignClients처리해주기

Rental과 Book 모두 Application에 EnableFeignClients라는 어노테이션 처리를 해주어야한다.
`@EnableFiegnClients`는 돌아다니면서 `@FeignClient`를 찾아 구현체를 만들어 준다. @FeignClient는 우리가 앞서 구현한 BookClient.java에서 처리해주었다.

- BookApp.java

```java

@SpringBootApplication
@EnableConfigurationProperties({LiquibaseProperties.class, ApplicationProperties.class})
@EnableDiscoveryClient
public class BookApp {

```

- RentalApp.java

```java

@SpringBootApplication
@EnableConfigurationProperties({LiquibaseProperties.class, ApplicationProperties.class})
@EnableDiscoveryClient
@EnableFeignClients
public class RentalApp {
```

이제 Feign Client 구현이 끝났다!

생각보다 간단한데, 사실 FallBack처리, Exception 처리 등을 추가하면 더 복잡해질 것이다.

우선 로직이 정상적으로 작동하는지 확인해보고 처리할 예정이다.

