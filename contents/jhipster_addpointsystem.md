# Point System 소개

도서 대여 시스템은 도서를 대여할 때 포인트를 제공하며, 연체시 포인트 결제를 완료 해야만 연체 상태를 해지할 수 있다.

즉, 로직은 다음과 같다.

1. 도서 대여 -> kafka send 포인트 추가 -> kafka consume 포인트 추가 실행
2. 도서 연체해제 요청 -> feign 포인트 차감 요청 -> 포인트 차감완료 -> feign Rental 상태 업데이트

>이때, User 서비스를 처음엔 만들었으나 gateway에 User Service가 구현되어있는 관계로 기존의 User 서비스를 삭제했다.

## Point System 구현

gateway의 User Domain에 point를 추가한다.

User Domain 뿐만 아니라, User DTO, User ServiceImpl도 모두 수정한다.
