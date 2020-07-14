# 타서비스 동기호출처리 : Feign Client 연결하기

Feign은 REST 기반 동기 서비스 호출을 추상화한 Spring Cloud Netflix라이브러리이다. 
Feign을 사용하면 웹 서비스 클라이언트를 보다 쉽게 작성할 수 있다. 인터페이스를 통해 클라이언트를 작성해주는데 Spring이 런타임에 구현체를 제공해준다.
Feign Client를 직접 구현하기 전, Feign을 적용할 Application들의 application.yml을 아래와 같이 수정한다. (Rental, Book)
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


## Rental서비스에서 Book서비스 호출 -> Book의 응답 받기

다음은 대여 서비스의 adaptor패키지를 생성하고  BookClient인터페이스 객체를 생성한다.

### BookClient.java
```java
package com.skcc.rental.adaptor;

import com.skcc.rental.config.FeignConfiguration;
import com.skcc.rental.web.rest.dto.BookInfoDTO;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

import java.util.List;

@FeignClient(name= "book", configuration = {FeignConfiguration.class})
public interface BookClient {
    @GetMapping("/api/BookInfo/{bookIds}/{userid}")
    ResponseEntity<List<BookInfoDTO>> getBookInfo(@PathVariable("bookIds") List<Long> bookIds, @PathVariable("userid")Long userid);
}
```

위 코드는 bookId리스트를 보내면 Book서비스의 REST API를 호출하여 BookInfo라는 obejct리스트를 리턴받는다.  BookInfo는 아래와 같다.
BookInfo는 Book과 동기식 통신을 위한 DTO객체이기 때문에 web.rest 패키지내의dto 패키지에 생성한다.

### BookInfoDTO.java
```java
package com.skcc.rental.web.rest.dto;

import lombok.Getter;
import lombok.Setter;

import java.io.Serializable;

@Getter
@Setter
public class BookInfoDTO implements Serializable {
    private Long id;
    private String title;
}
```
일반적인 DTO 형식과 동일하다. 단, 이 클래스에 constructor는 생성하지 않아야한다. 
RentalResource에서 사용은 다음과 같다.

### RentalResource.java

```java
public class RentalResource {
...
private final BookClient bookClient;
…
public RentalResource(RentalService rentalService, RentalMapper rentalMapper, BookClient bookClient, UserClient userClient) {
    this.rentalService = rentalService;
    this.rentalMapper = rentalMapper;
    this.bookClient = bookClient;
    this.userClient = userClient;
}
…

public ResponseEntity rentBooks(…)
…
ResponseEntity<List<BookInfoDTO>> bookInfoResult = bookClient.getBookInfo(books, userid); //feign - 책 정보 가져오기
...
```

위와 같이 BookClient를 상단에 선언하고 생성자에 포함시킨다.
그 다음, 도서 정보를 받아와야하는 로직 부분에 bookClient에 선언한 메소드를 불러 통신하고 결과를 받아온다.
응답하는 도서 서비스에서의 구현은 간단하다.
아래와 같이 BookResource에 Rental에서 생성한 BookClient에 선언했던 메소드와 동일하게 구현한다.
Book Id 리스트를 받고, 받은 Id리스트를 bookService에 BookInfo로 조합해주는 메소드를 부른 뒤, 결과를 리턴한다.

### BookResource.java
```java
    @GetMapping("/BookInfo/{bookIds}")
    public List<BookInfo> getBookInfo(@PathVariable("bookIds") List<Long> bookIds){
        log.debug("Got feign request!!");
        List<BookInfo> bookInfoList= bookService.getBookInfo(bookIds);

        return bookInfoList;
    }
```

### BookServiceImpl.java
```java
@Override
@Transactional(readOnly = true)
public List<BookInfo> getBookInfo(List<Long> bookIds) {
    List<BookInfo> bookInfoList = bookIds.stream()
        .filter(b -> bookRepository.findById(b).get().getBookStatus().equals(BookStatus.AVAILABLE))
        .map(b -> new BookInfo(b, bookRepository.findById(b).get().getTitle()))
        .collect(Collectors.toList());
    return bookInfoList;
}
```
BookInfo리스트로 만드는 방식은 java stream으로 구현하였다. 받은 BookId로 해당 도서를 찾고, 도서가 대여 가능 상태이면 BookInfo로 만들어 리스트로 변환시켰다.
도서서비스에도 BookInfoDTO가 존재해야 한다.

### BookInfoDTO.java

```java  
package com.skcc.book.web.rest.dto;

import javax.validation.constraints.NotNull;
import java.io.Serializable;

@Getter
@Setter
@AllArgsConstructor
public class BookInfoDTO implements Serializable {
    private Long id;
    private String title;
}
```
BookInfo 또한 Rental에서와 마찬가지로 dto 패키지 아래 생성하였다. Rental과의 차이점은 BookInfo의 Constructor가 선언되어있다는 것인데, 이는 BookService에서 BookId를 BookInfo로 재조합하는 과정에서 필요하기 때문에 선언하였다.

## EnableFeignClients처리해주기

마지막으로Feign Client를 사용하기 위해서는 Feign Client를 사용하는 대여서비스의 Application에 EnableFeignClients라는 어노테이션 처리를 해주어야한다. 
@EnableFiegnClient를 설정하면 @FeignClient라고 설정되어 있는 모든 인터페이스를 찾아 구현체를 만들어 준다. @FeignClient는 우리가 앞서 구현한 BookClient.java에서 처리해주었다.

### RentalApp.java
```java
@SpringBootApplication
@EnableConfigurationProperties({LiquibaseProperties.class, ApplicationProperties.class})
@EnableDiscoveryClient
@EnableFeignClients
public class RentalApp {
```
