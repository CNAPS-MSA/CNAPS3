# 사용자(User)서비스 구현
**코드는 아무 예고 없이 언제든 변경될 수 있으니, 실제 코드는 해당 서비스 애플리케이션의 Repository에서 확인하세요.**


Sample에서 보여줄 기능은 아래와 같다.

## 구현기능
  - 사용자 관리
    1. 사용자 역할관리
  - 포인트 관리
    1. 포인트 부여,감소
  - 대출금지 해제처리
    1. 연체일만큼 포인트로 감소
    2. Rental서비스 '대출불가처리' 해제

## API설계

|API명|사용자 생성|
|----|------|
|리소스URI|"/users"|
|Method|POST|
|Request| |
|Response| |

리소스로 예를 들면 /users를 post방식으로 호출하여 requestBody로 UserDTO를 보내 신규 사용자를 등록한다.

|API명|사용자 정보수정|
|----|------|
|리소스URI|"/users"|
|Method|PUT|
|Request| |
|Response| |

리소스로 예를 들면 /users를 PUT방식으로 호출하여 requestBody로 UserDTO를 보내 사용자 정보를 수정한다.

|API명|사용자 조회|
|----|------|
|리소스URI|"/users"|
|Method|GET|
|Request| |
|Response| |

리소스로 예를 들면 /users를 GET방식으로 호출하여 

## 도메인 모델링

## 유스케이스 흐름

## 내부영역 - 도메인 모델 개발

## 내부영역 - 서비스 개발

## 내부영역 - 레파지토리 개발

## 외부영역 - REST 컨트롤러 개발

## 외부영역 - 아웃바운드 어댑터 개발

## 단위테스트 수행
