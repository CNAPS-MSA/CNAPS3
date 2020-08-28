# 타서비스 동기호출처리 : Feign Client 연결하기

Feign은 REST 기반 동기 서비스 호출을 추상화한 Spring Cloud Netflix라이브러리이다. 
Feign을 사용하면 웹 서비스 클라이언트를 보다 쉽게 작성할 수 있다. 인터페이스를 통해 클라이언트를 작성해주는데 Spring이 런타임에 구현체를 제공해준다.
Feign Client를 직접 구현하기 전, Feign을 적용할 Application들의 application.yml을 아래와 같이 수정한다. (Rental, Book)
```yaml
feign:
  hystrix:
    enabled: false
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
현재 Sample에서는 fallback을 구현하지 않은 상태이기 때문에 Timeout시간을 10초로 길게 두었고, circuit breaker 패턴을 적용하지 않았기 때문에 feign의 hystrix옵션을 false로 설정하였다. 


## Rental서비스에서 Book서비스 호출 -> Book의 응답 받기

다음은 대여 서비스의 adaptor패키지를 생성하고 BookClient인터페이스 객체를 생성한다.

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
    @GetMapping("/api/books/bookInfo/{bookId}")
    ResponseEntity<BookInfoDTO> findBookInfo(@PathVariable("bookId") Long bookId);
}
```

위 코드는 bookId를 보내면 Book서비스의 REST API를 호출하여 BookInfo라는 obejct를 리턴받는다.  BookInfo는 아래와 같다.
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
ResponseEntity<BookInfoDTO> bookInfoResult = bookClient.findBookInfo(bookId); //feign - 책 정보 가져오기
...
```

위와 같이 BookClient를 상단에 선언하고 생성자에 포함시킨다.
그 다음, 도서 정보를 받아와야하는 로직 부분에 bookClient에 선언한 메소드를 불러 통신하고 결과를 받아온다.
응답하는 도서 서비스에서의 구현은 간단하다.
아래와 같이 BookResource에 Rental에서 생성한 BookClient에 선언했던 메소드와 동일하게 구현한다.
BookId를 받고, bookService에 도서 정보를 가져오는 메소드 호출과 함께 bookId를 넘겨 BookInfoDTO로 결과를 리턴 받는다.

### BookResource.java
```java
    @GetMapping("/books/bookInfo/{bookId}")
    public ResponseEntity<BookInfoDTO> findBookInfo(@PathVariable("bookId") Long bookId){
        BookInfoDTO bookInfoDTO = bookService.findBookInfo(bookId);
        log.debug(bookInfoDTO.toString());
        return ResponseEntity.ok().body(bookInfoDTO);
    }
```

### BookServiceImpl.java
```java
@Override
@Transactional
public BookInfoDTO findBookInfo(Long bookId) {
    BookInfoDTO bookInfoDTO = new BookInfoDTO();
    Book book = bookRepository.findById(bookId).get();
    bookInfoDTO.setId(book.getId());
    bookInfoDTO.setTitle(bookRepository.findById(book.getId()).get().getTitle());
    return bookInfoDTO;
}
```
받은 BookId로 해당 도서를 찾고, BookInfo로 만들어 리스트로 변환시켰다.
도서서비스에도 BookInfoDTO가 존재해야 한다.

### BookInfoDTO.java

```java  
ppackage com.skcc.book.web.rest.dto;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import java.io.Serializable;
@Getter
@Setter
@NoArgsConstructor
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

## Feign Client 예외처리하기

Feign Client는 외부 서비스를 동기호출 할 떄에 사용된다. 하지만 동기호출의 결과가 항상 성공하진 않기 때문에 결과에 따라 예외처리를 해야한다.
특히, Feign은 예외 발생 시 결과를 받는 서비스에선 무조건 `500 Internal Server Error`로 처리되기 때문에 예외처리를 하지 않으면 동기 호출했던 서비스에서 어떠한 에러가 발생했는지 확인할 수가 없다.
따라서, Sample 에서는 외부 서비스에서 어떤 에러가 발생했는지 확인하고 이를 Client로 보내도록 예외처리를 구현하였다.
구체적인 구현 내용은 외부 서비스 예외처리 구현을 통해 살펴보자.

- [서비스 외부 에러 관리하기 - Feign Exception](/contents/feign_exception.md)