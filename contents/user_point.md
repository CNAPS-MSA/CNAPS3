# 포인트 관리기능 구현

- [사용자(User) 서비스 구현](/contents/user_businesslogic.md) 

사용자 서비스 구현에서 소개한 두가지의 기능 중 앞서 사용자 역할관리 기능을 소개하였다.
이번 페이지에서는 사용자 서비스 구현에서 소개하였던 포인트 관리 기능에 대해 살펴보자.

- 포인트 관리
    1. 포인트 부여
    2. 포인트 적립 및 결제
   
## 내부영역 - 도메인 모델 개발

포인트 관리 기능을 위해 새로운 도메인 모델을 생성하지 않았으며, 기존의 Jhipster에서 생성한 User에 Integer형식의 `Point`라는 새로운 속성을 생성하였다.
Jhipster에서 생성해주는 도메인 모델에 새로운 속성을 부여할 때에는 여러가지 수정해야하는 부분이 많은데, 특히 Gateway같은 경우 외부 서비스들이 모두 접근하기 때문에 수정에 의해 오류가 생길 경우 클라이언트가 각 서비스에 접근할 수 없다. 따라서, Jhipster의 User 서비스 수정 가이드 내용을 충분히 숙지한 뒤 수정해야한다.

포인트 속성을 추가할 때에도 Jhipster의 가이드에 따라 추가하였으며 아래 링크를 통해 해당 상세 내용을 확인할 수 있다.

[Jhipster의 User Service 수정 가이드](https://www.jhipster.tech/tips/022_tip_registering_user_with_additional_information.html)

포인트 관리의 두번째 기능인 포인트 적립 및 결제는 User 도메인 내부에서 처리하도록 구현하였다.

### User.java

**포인트 적립메소드**

```java

    public User savePoints(int points){
        this.point+=points;
        return this;
    }
}
```
포인트 적립 메소드는 적립 포인트를 받아 보유하고 있는 포인트와 합산하고 해당 사용자정보를 반환한다.

**포인트 결제 메소드**

```java
 public User usePoints(int points) throws UsePointsUnavailableException{
        if(this.point>=points) {
            this.point -=points;
            return this;
        }else{
            throw new UsePointsUnavailableException("잔여 포인트가 모자라 결제가 불가능 합니다.");
        }
    }
```
포인트 결제 메소드는 보유하고 있는 포인트가 결제요청된 포인트보다 같거나 큰 경우 결제된다. 결제요청된 포인트보다 적을 경우 `UsePointsUnavailableException` 예외를 던져 예외처리되도록 하였다.

## 내부영역 - 서비스 개발

사용자 생성 시의 포인트 부여부터 살펴보면, 사용자가 생성될 때 기본 포인트가 1000점 부여되도록 하였다. 사용자 생성 시에 호출되는 UserService.registerUser와 UserService.createUser에서 `setPoint(1000);`으로 포인트를 부여한다.

### UserService.java

```java
@Service
@Transactional
public class UserService {
    ...(중략)...
    public User registerUser(UserDTO userDTO, String password) throws InterruptedException, ExecutionException, JsonProcessingException {
        ..생략..
        User newUser = new User();
        ..생략..
        newUser.setPoint(1000);
        userRepository.save(newUser);
        createRental(newUser.getId());
        this.clearUserCaches(newUser);
        log.debug("Created Information for User: {}", newUser);
        return newUser;
    }

     public User createUser(UserDTO userDTO) throws InterruptedException, ExecutionException, JsonProcessingException {
        User user = new User();
        ..생략..
        user.setPoint(1000);
        userRepository.save(user);
        createRental(user.getId());
        this.clearUserCaches(user);
        log.debug("Created Information for User: {}", user);
        return user;
     }
}
```

포인트 적립과 결제 기능은 외부 서비스의 비동기/동기 호출에 의해 이루어지므로 인바운드 어댑터 개발에서 살펴보자.

## 외부영역 - 인바운드 어댑터 개발

사용자 서비스는 비동기 호출에 의해 포인트 적립이 이뤄지고, 동기 호출에 의해 포인트 결제가 이뤄진다.

먼저, 포인트 적립 기능 구현부터 살펴보자. 

포인트 적립기능은 사용자가 도서를 대여한 뒤, 사용자 서비스를 비동기 호출하여 포인트를 적립하도록 한다. 따라서, GatewayKafkaConsumer인 인바운드 어댑터를 통해 포인트 적립 이벤트를 받는다.

### GatewayKafkaConsumer.java

```java
@Service
public class GatewayKafkaConsumer {
    ...(중략)...
    while (!closed.get()){
        ConsumerRecords<String, String> records = kafkaConsumer.poll(Duration.ofSeconds(3));
            for(ConsumerRecord<String, String> record: records){
                log.info("Consumed message in {} : {}", TOPIC, record.value());
                ObjectMapper objectMapper = new ObjectMapper();
                SavePointsEvent savePointsEvent= objectMapper.readValue(record.value(), SavePointsEvent.class);
                userService.savePoint(savePointsEvent.getUserId(), savePointsEvent.getPoints());


            }

    }
}
```
인바운드 어댑터는 Rental서비스가 전송한 SavePointEvent라는 포인트 적립 이벤트를 받아, userService.savePoint를 호출하여 해당 사용자 정보와 적립할 포인트 정보를 넘겨 책임을 위임한다.

아래는 인바운드 어댑터에서 호출되는 userService.savePoint 메소드이다. 

### UserService.java

```java
@Service
@Transactional
public class UserService {
    ...(중략)...
    public void savePoint(Long userId, int points) {
        User user = userRepository.findById(userId).get();
        user=user.savePoints(points);
        userRepository.save(user);
    }
```
userService.savePoint는 레파지토리에서 해당 사용자를 검색한 뒤, 해당 사용자 도메인 내부의 savePoints메소드를 호출하여 포인트를 적립하도록한다.

다음은 포인트 결제 기능 구현을 살펴보자.

포인트 결제는 포인트 적립과는 다르게 동기 호출에 의해 이뤄진다. 포인트 결제의 결과에 따라 포인트 결제를 요청한 외부 서비스에서 실행할 로직이 달라지기 때문이다.
따라서, 포인트 결제는 Feign Client를 사용하였고 REST 컨트롤러에서 결제 요청을 받는다.

### UserResource.java

```java
@RestController
@RequestMapping("/api")
public class UserResource {

    ...(중략)...

    @PutMapping("/users/latefee")
    public ResponseEntity usePoint(@RequestBody LatefeeDTO latefeeDTO) throws UsePointsUnavailableException {
        userService.usepoints(latefeeDTO.getUserId(), latefeeDTO.getLatefee());
        return new ResponseEntity<>(HttpStatus.OK);
    }

}
```
포인트 결제 API는 PUT 방식에 "/users/latefee"로 명명하였고 LatefeeDTO를 전달받아, userService.usepoints를 호출하여 LatefeeDTO에 담긴 포인트를 결제할 사용자 정보와 연체료 결제를 전달한다.

### UserService.java

```java
@Service
@Transactional
public class UserService {
    ...(중략)...
    public User usepoints(Long userId, int latefee) throws UsePointsUnavailableException {
        User user = userRepository.findById(userId).get();
        user= user.usePoints(latefee);
        return userRepository.save(user);
    }
}
```
호출된 userService.usepoints는 전달받은 사용자 정보를 통해 해당 사용자를 검색하고 해당 사용자 도메인 내부의 usePoints를 호출하여 포인트를 결제한다.
이때, 포인트 결제 요청이 실패하는 경우 UsePointsUnavailableException을 던져 예외처리 되도록 구현하였다.

기본 예외처리와 Feign Client 관련 예외처리는 아래 페이지를 참고하도록 한다.

[Jhipster와 Spring 기본 예외처리](https://engineering-skcc.github.io/msa/jhipster-exception/)
[Feign Client의 예외처리](https://engineering-skcc.github.io/msa/jhipster-feign/)