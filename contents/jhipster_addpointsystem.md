# Point System 소개

도서 대여 시스템은 도서를 대여할 때 포인트를 제공하며, 연체시 포인트 결제를 완료 해야만 연체 상태를 해지할 수 있다.

즉, 로직은 다음과 같다.

1. 도서 대여 -> kafka send 포인트 추가 -> kafka consume 포인트 추가 실행
2. 도서 연체해제 요청 -> feign 포인트 결제 요청 -> 포인트 결제 실행 -> 완료시 : feign User 상태 업데이트

>이때, User 서비스를 처음엔 만들었으나 gateway에 User Service가 구현되어있는 관계로 기존의 User 서비스를 삭제했다.

## Point System 구현

Point System도 다른 서비스들과 마찬가지로 Jhipster를 활용하였다.

```
mkdir point
cd point
jhipster
```

옵션선택

![image](https://user-images.githubusercontent.com/18453570/83717959-fda0ab00-a66e-11ea-8104-0adcc5484941.png)


