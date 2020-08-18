# 도서 연체 및 연체된 도서반납하기 기능 구현


- [대여(Rental)서비스 구현](/contents/jhipster_businesslogic.md) 

초반에 구현기능에서 언급한 기능 중 연체 처리와 연체된 도서 반납 처리를 구현해 보자.
대여서비스구현에서 이미 중요한 흐름을 따라가 봤기 때문에 도서 연체 및 연체된 도서반납처리는 어렵지 않을 것이다. 
아래의 시퀀스 다이어그램처럼 도서대여 흐름과  거의 비슷하니 비지니스 로직 처리 흐름상 다른 부분만 살펴보자. 
![image](https://user-images.githubusercontent.com/15258916/87248578-1fbef180-c495-11ea-90c2-cf3b6b399a38.png)

## 도메인 모델 객체
### Rental.java
```java
/**
 * 연체처리
 *
 * @param bookId
 * @return
 */
public Rental overdueBook(Long bookId) {
    RentedItem rentedItem = this.rentedItems.stream().filter(item -> item.getBookId().equals(bookId)).findFirst().get();
    this.addOverdueItem(OverdueItem.createOverdueItem(rentedItem.getBookId(), rentedItem.getBookTitle(), rentedItem.getDueDate()));
    this.removeRentedItem(rentedItem);
    return this;
}

/**
 * 연체된 책 반납
 *
 * @param bookId
 * @return
 */
public Rental returnOverdueBook(Long bookId) {
    OverdueItem overdueItem = this.overdueItems
        .stream()
.filter(item -> item.getBookId().equals(bookId)).findFirst().get();
    this.addReturnedItem(ReturnedItem.createReturnedItem(overdueItem.getBookId(),
 overdueItem.getBookTitle(), LocalDate.now()));
    this.removeOverdueItem(overdueItem);
    return this;
}

/**
 * 대여 불가 처리
 * @return
 */
public void makeRentUnable() {
    this.setRentalStatus(RentalStatus.RENT_UNAVAILABLE);
    this.setLateFee(this.getLateFee() + 30); //연체시 연체비 30포인트 누적
}

```
대여,반납처리와 마찬가지로 대여카드(Rental)객체가 연체처리와 연체된 도서의 반납처리의 책임을 가지고 있다. 주요한 비지니스로직처리 메소드는 다음과 같다.
- **연체처리()**: 도서Id로 도서카드에서 해당 대여 도서(RentedItem)를 찾은 뒤 연체도서객체(OverdueItem)를 만들고 도서카드에 추가한 뒤 기존 대여 도서는 삭제한다.
- **연체된도서반납처리()**: 도서Id로 도서카드에서 연체도서(OverdueItem)를 찾은 뒤 반납도서 객체(ReturnedItem)를 만들어 도서카드에 추가한 뒤 기존 연체 도서는 삭제한다. 
- **대여불가처리()**: 연체 시 대여불가처리가 되어야 한다. 대여가능여부를 불가로 변경하고 도서카드에 연체비 30포인트를 부여한다.

## 서비스 흐름 처리 
서비스에서 이렇게 도메인 모델에서 정의된 로직들을 사용하여 비지니스 로직 구현을 완료한다.

### RentalServiceImpl
```java
/**
 * 연체처리
 *
 * @param userId
 * @param book
 * @return
 */
@Override
public Long beOverdueBooks(Long rentalId, Long bookId) {
    Rental rental = rentalRepository.findById(rentalId).get();
    rental= rental.overdueBook(bookId);
    rental = rental.makeRentUnable();
    rentalRepository.save(rental);
    return bookId;
}


/**
 * 연체된 책 반납하기
 *
 * @param userid
 * @param book
 * @return
 */
@Override
public Rental returnOverdueBooks(Long userid, Long book) throws ExecutionException , InterruptedException , JsonProcessingException{
    Rental rental = rentalRepository.findByUserId(userid).get();

    rental = rental.returnOverdueBook(book);

    updateBookStatus(book, "AVAILABLE");
    updateBookCatalog(book, "RETURN_BOOK");

    return rentalRepository.save(rental);
}
```
연체처리에서는
- 해당 사용자Id에 맞는 도서 카드를 검색 한 후에
- 해당 도서들을 도서카드 객체에 위임하여 연체처리하고
- 해당 도서 카드 대여 불가 처리 후 저장한다.

연체된 서적 반납처리에서는 
- 해당 사용자Id에 맞는 도서 카드를 검색 한 후에
- 해당 도서들을 도서 카드 객체에 위임하여 반납처리한 뒤
- 카프카를 이용한 비동기 메시지 통신으로 도서 서비스 해당 도서 재고 상태와 카탈로그 서비스의 해당 도서 상태를 대여가능으로 변경한다.
- 해당 도서 카드 대여를 저장한다.


## 외부영역 – REST 컨트롤러 개발

연체도서 반납과 도서 연체 처리 또한 대여처리와 마찬가지로  RentalResource에서 http 표준 메소드 방식으로 제공된다.
연체도서 반납 API("/rentals/{userid}/OverdueItem/{book}")으로 DELETE 방식으로 선언하였고, 연체 처리 API ("/rentals/{rentalId}/OverdueItem/{bookId}")으로 POST 방식으로 선언하였다. 
주요 비지니스로직처리는 rentalService.returnOverdueBook과 rentalService.beOverdueBook을 호출하여 위임한다. 


### RentalResource.java

```java
@RestController
@RequestMapping("/api")
public class RentalResource {
…중략…

/**
 * 연체도서 반납하기
 *
 * @param userid
 * @param book
 * @return
 */
@DeleteMapping("/rentals/{userid}/OverdueItem/{book}")
public ResponseEntity returnOverdueBook(@PathVariable("userid") Long userid, @PathVariable("book") Long book) throws InterruptedException, ExecutionException, JsonProcessingException {
    Rental rental = rentalService.returnOverdueBook(userid, book);
    RentalDTO result = rentalMapper.toDto(rental);
    return ResponseEntity.ok().body(result);
}

/**
 * 도서 연체처리 하기
 *
 * @param rentalId
 * @param bookId
 *
 */

@PostMapping("/rentals/{rentalId}/OverdueItem/{bookId}")
public ResponseEntity<Void> BeOverdue(@PathVariable("rentalId")Long rentalId, @PathVariable("bookId")Long bookId){
    rentalService.beOverdueBook(rentalId, bookId);
    return ResponseEntity.ok().build();
}
```

연체도서 반납처리 API 구현을 위한 컨트롤러 처리 흐름을 살펴보면 
   - 서비스를 호출하여 연체도서 반납처리를 수행한다.
   - 연체도서 반납처리한 대여 카드를 DTO로 변경하여 클라이언트에 반환한다.

연체처리 API 구현을 위한 컨트롤러 처리 흐름을 살펴보면 
   - 서비스를 호출하여 연체처리를 수행한다.
   - 처리 완료 응답을 클라이언트에 반환한다.
