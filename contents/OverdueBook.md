# 도서 연체 및 연체된 도서반납하기 기능 구현

초반에 1.1.1 구현기능에서 언급한 기능 중 연체 처리와 연체된 도서 반납 처리가 남았다.
도서대여 및 반납하기라는 중요한 흐름을 따라가 봤기 때문에 도서 연체 및 연체된 도서반납처리는 어렵지 않을 것이다. 위에 시퀀스 다이어그림처럼 도서대여 흐름과  거의 비슷하니 비지니스 로직 처리 흐름상 다른 부분만 살펴보자. 

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
    RentedItem rentedItem = this.rentedItems
.stream()
.filter(item -> item.getBookId().equals(bookId)).findFirst().get();
    this.addOverdueItem(OverdueItem.createOverdueItem(rentedItem.getBookId(), 
rentedItem.getBookTitle(), rentedItem.getDueDate()));
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
