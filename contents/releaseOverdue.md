# 대여불가 해제처리 기능 구현


- [도서 연체/연체된 도서 반납 기능 구현](/contents/OverdueBook.md) 

사용자는 도서가 연체되면 대여불가 처리가 되며 도서카드에 연체비 30포인트가 부여된다.
부여된 연체비를 모두 결제해야만 대여불가 처리가 해제된다.
아래의 시퀀스 다이어그램은 대여불가 해제처리의 흐름이다.

```
대여불가 해제처리 시퀀스 다이어그램 이미지 삽입
```

## 도메인 모델 객체
### Rental.java
```java
/
    /**
     * 대여불가 해제
     *
     * @param lateFee
     * @return
     *
     * */
    public Rental releaseOverdue(int lateFee) {
        this.setLateFee(this.lateFee - lateFee);
        this.setRentalStatus(RentalStatus.RENT_AVAILABLE);
        return this;
    }


```
대여카드(Rental)객체는 연체 상태 해제의 책임을 가지고 있다. 주요한 비지니스로직처리 메소드는 다음과 같다.
- **대여불가 해제()**: 포인트 결제가 완료되면 연체상태가 해제되어야한다. 결제한 연체료를 현재 연체료에서 차감하고, 대여가능여부를 가능으로 변경한다. 

## 서비스 흐름 처리 
서비스에서 이렇게 도메인 모델에서 정의된 로직들을 사용하여 비지니스 로직 구현을 완료한다.

### RentalServiceImpl
```java

    /**
     * @param userId
     * 대여불가 해제하기
     */
    @Override
    public Rental releaseOverdue(Long userId) {
        Rental rental = rentalRepository.findByUserId(userId).get();
        rental=rental.releaseOverdue(rental.getLateFee());
        return rentalRepository.save(rental);
    }

}
```
연체해제 처리에서는
- 해당 사용자Id에 맞는 도서 카드를 검색 한 후에
- 해당 도서 카드 대여 가능 처리 후 저장한다.


## 외부영역 – REST 컨트롤러 개발

대여불가 해제 처리는 RentalResource에서 http 표준 메소드 방식으로 제공된다.
대여불가 해제 처리 API("/rentals/release-overdue/user/{userId}")으로 PUT 방식으로 선언하였다.
주요 비지니스로직처리는 동기 이벤트인 userClinet를 호출하여 포인트결제를 처리하고, 결과에 따라 rentalService.releseOverdue를 호출하여 위임한다.


### RentalResource.java

```java
@RestController
@RequestMapping("/api")
public class RentalResource {
…중략…
    /**
     * 대여불가 해제하기
     *
     * @param userId
     * @return
     */
    @PutMapping("/rentals/release-overdue/user/{userId}")
    public ResponseEntity releaseOverdue(@PathVariable("userId") Long userId)  {
        LatefeeDTO latefeeDTO = new LatefeeDTO();
        latefeeDTO.setUserId(userId);
        latefeeDTO.setLatefee(rentalService.findLatefee(userId));
        try{
            userClient.usePoint(latefeeDTO);

        }catch (FeignClientException e){
            if (!Integer.valueOf(HttpStatus.NOT_FOUND.value()).equals(e.getStatus())) {
                throw e;
            }
        }
        RentalDTO rentalDTO = rentalMapper.toDto(rentalService.releaseOverdue(userId));
        return ResponseEntity.ok().body(rentalDTO);


}

```

대여불가 해제 처리 API 구현을 위한 컨트롤러 처리 흐름을 살펴보면 
   - 외부 서비스인 User 서비스와의 통신을 위한 유저 정보와 연체료 정보를 담을 객체인 LatefeeDTO를 생성한다.
   - 결제 처리는 동기 호출로 User 서비스를 호출하여 연체료를 결제하고 결제 결과를 가져온다.
   - 결제 처리가 완료된 경우 서비스를 호출하여 대여불가 해제처리를 수행한다.
   - 결제 처리가 실패한 경우 예외처리한다.

결체 실패 처리 구현은 [외부 서비스 예외처리 구현](/contents/feign_exception.md)에서 상세하게 살펴보기로 한다.
