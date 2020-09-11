# 사용자(User)서비스 구현
**코드는 아무 예고 없이 언제든 변경될 수 있으니, 실제 코드는 해당 서비스 애플리케이션의 Repository에서 확인하세요.**

Jhister Project는 Gateway내에 사용자 서비스를 제공한다.

기본적인 Spring Security부터 권한관리, 로그인, 회원가입 등의 기능을 제공하며 이메일 인증 또한 제공한다.
보안과 권한관리, 로그인, 회원가입 등 기본적으로 Jhipster가 제공하는 UserService를 되도록 그대로 사용하였다. 
하지만, 회원가입시 사용되는 이메일 인증의 경우 외부 이메일 서비스를 연결하지 않으면 사용할 수 없어 이메일 인증 메소드는 모두 주석처리하였다. 해당 부분은 아래와 같다.

## AccountResource.java
```java
@RestController
@RequestMapping("/api")
public class UserResource {
    ...(중략)...
    @PostMapping("/register")
    @ResponseStatus(HttpStatus.CREATED)
    public void registerAccount(@Valid @RequestBody ManagedUserVM managedUserVM) throws InterruptedException, ExecutionException, JsonProcessingException {
        if (!checkPasswordLength(managedUserVM.getPassword())) {
            throw new InvalidPasswordException();
        }
        User user = userService.registerUser(managedUserVM, managedUserVM.getPassword());
       // mailService.sendActivationEmail(user);
    }
}
``` 
위 코드의 맨 마지막줄인 `mailService.sendActivationEmail(user);` 를 주석처리한다.

## UserResource.java

```java
@RestController
@RequestMapping("/api")
public class UserResource {
    ...(중략)...
    @PostMapping("/users")
    @PreAuthorize("hasAuthority(\"" + AuthoritiesConstants.ADMIN + "\")")
    public ResponseEntity<User> createUser(@Valid @RequestBody UserDTO userDTO) throws URISyntaxException, InterruptedException, ExecutionException, JsonProcessingException {
        log.debug("REST request to save User : {}", userDTO);

        if (userDTO.getId() != null) {
            throw new BadRequestAlertException("A new user cannot already have an ID", "userManagement", "idexists");
            // Lowercase the user login before comparing with database
        } else if (userRepository.findOneByLogin(userDTO.getLogin().toLowerCase()).isPresent()) {
            throw new LoginAlreadyUsedException();
        } else if (userRepository.findOneByEmailIgnoreCase(userDTO.getEmail()).isPresent()) {
            throw new EmailAlreadyUsedException();
        } else {
            User newUser = userService.createUser(userDTO);
            //mailService.sendCreationEmail(newUser);
            return ResponseEntity.created(new URI("/api/users/" + newUser.getLogin()))
                .headers(HeaderUtil.createAlert(applicationName,  "userManagement.created", newUser.getLogin()))
                .body(newUser);
        }
    }
}
```
위 코드에서도 마찬가지로 `mailService.sendActivationEmail(user);` 를 주석처리한다.

위의 두 부분을 제외하고도 sendActivationEmail메소드를 직접 검사하여 모두 주석처리 하도록 한다. 

Sample에서 보여줄 기능은 아래와 같다.

## 구현기능
  - 사용자 관리
    1. 사용자 역할관리
  - 포인트 관리
    1. 포인트 부여
    2. 포인트 적립 및 결제

## API설계

|API명|사용자 정보수정|
|----|------|
|리소스URI|"/users"|
|Method|PUT|
|Request|UserDTO|
|Response| |

리소스로 예를 들면 /users를 PUT방식으로 호출하여 requestBody로 UserDTO를 보내 사용자 정보를 수정한다.
이때, 사용자 역할관리 또한 사용자 정보수정 방식으로 구현되었으며 ADMIN만 위 URI 접근이 허용된다.

|API명|포인트 결제|
|----|------|
|리소스URI|"/users/latefee"|
|Method|PUT|
|Request|LatefeeDTO|
|Response| |

/users/latefee를 PUT방식으로 호출하여 LatefeeDTO를 받아 사용자의 정보와 연체료 정보를 받아 연체료 결제를 수행한다.

## 도메인 모델링

사용자 서비스의 도메인 모델은 권한과 사용자로 구성된다. 권한과 사용자 모두 루트 엔티티이며 사용자와 권한은 다대다의 관계를 갖는다.
사용자의 권한은 'ROLE_ADMIN'인 관리자와 'ROLE_USER'인 일반 사용자로 구성되어있으며 사용자 생성 시 기본 권한인 'ROLE_USER'로 설정된다.
관리자 권한의 경우, 오직 관리자에 의해서만 부여받을 수 있다.

## 유스케이스 흐름

사용자 서비스의 경우, 기본적인 사용자 관리와 포인트 관리 기능을 포함하고 있는데 두 기능 모두 특별한 비즈니스 로직은 없다.
사용자 관리는 관리자만이 사용자 역할 관리를 수행할 수 있으며, 포인트 관리 기능은 Rental 서비스의 동기/비동기 호출에 의해 적립 또는 결제되기 때문이다.

## 내부영역 - 도메인 모델 개발

사용자는 login(로그인에 필요한 Id를 의미한다), password(비밀번호), firstName(이름), lastName(성), email(이메일 주소), authorities(권한) 등을 기본적인 속성으로 갖는다. 

권한 엔티티는 name(권한 이름)만을 기본 속성으로 갖는다.

## 내부영역 - 서비스 개발

위에서 언급했다시피 사용자의 권한관리는 사용자 정보 수정에 의해 이뤄진다. 그렇다면 사용자 정보 수정이 어떻게 이루어지는 지 살펴보자.

### UserService.java

```java
@Service
@Transactional
public class UserService {
    ...(중략)...

      public Optional<UserDTO> updateUser(UserDTO userDTO) {
        return Optional.of(userRepository
              .findById(userDTO.getId()))
              .filter(Optional::isPresent)
              .map(Optional::get)
              .map(user -> {
                this.clearUserCaches(user);
                user.setLogin(userDTO.getLogin().toLowerCase());
                user.setFirstName(userDTO.getFirstName());
                user.setLastName(userDTO.getLastName());
                if (userDTO.getEmail() != null) {
                    user.setEmail(userDTO.getEmail().toLowerCase());
                }
                user.setImageUrl(userDTO.getImageUrl());
                user.setActivated(userDTO.isActivated());
                user.setLangKey(userDTO.getLangKey());
                Set<Authority> managedAuthorities = user.getAuthorities();
                managedAuthorities.clear();
                userDTO.getAuthorities().stream()
                    .map(authorityRepository::findById)
                    .filter(Optional::isPresent)
                    .map(Optional::get)
                    .forEach(managedAuthorities::add);
                this.clearUserCaches(user);
                log.debug("Changed Information for User: {}", user);
                return user;
            })
            .map(UserDTO::new);
    }

}
```

사용자 정보수정 메소드를 살펴보면 기존의 CRUD처럼 레파지토리에 단순 저장하는 방식이 아닌, Java Stream을 활용하여 구현하였음을 알 수 있다. 
사용자 정보 수정 메소드는 권한 수정 뿐만 아니라 모든 정보를 userDTO에 담긴 정보로 수정한다.

권한 수정 부분을 살펴보면, 기존 사용자의 권한을 가져온뒤 이를 clear()를 통해 삭제한다.
그리고 userDTO에 담긴 권한을 add하여 권한을 수정한다.

## 내부영역 - 레파지토리 개발

UserRepository 또한 Jhipster에서 구현한 코드를 그대로 사용하였다. 

### UserRepository.java

```java
package com.skcc.gateway.repository;

import com.skcc.gateway.domain.User;

import org.springframework.cache.annotation.Cacheable;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.EntityGraph;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;
import java.time.Instant;

/**
 * Spring Data JPA repository for the {@link User} entity.
 */
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    String USERS_BY_LOGIN_CACHE = "usersByLogin";

    String USERS_BY_EMAIL_CACHE = "usersByEmail";

    Optional<User> findOneByActivationKey(String activationKey);

    List<User> findAllByActivatedIsFalseAndActivationKeyIsNotNullAndCreatedDateBefore(Instant dateTime);

    Optional<User> findOneByResetKey(String resetKey);

    Optional<User> findOneByEmailIgnoreCase(String email);

    Optional<User> findOneByLogin(String login);

    @EntityGraph(attributePaths = "authorities")
    @Cacheable(cacheNames = USERS_BY_LOGIN_CACHE)
    Optional<User> findOneWithAuthoritiesByLogin(String login);

    @EntityGraph(attributePaths = "authorities")
    @Cacheable(cacheNames = USERS_BY_EMAIL_CACHE)
    Optional<User> findOneWithAuthoritiesByEmailIgnoreCase(String email);

    Page<User> findAllByLoginNot(Pageable pageable, String login);
}
```
UserRepository는 위과 같이 대부분 사용자관리와 관련된 메소드로 이루어져있다.


## 외부영역 - REST 컨트롤러 개발

사용자 서비스에서 사용자 관리와 관련된 REST 컨트롤러는 두가지로 AccountResource.java와 UserResource.java로 이루어져있다.
AccountResource.javad와 UserResource.java의 용도는 비슷하면서도 다른데, AccountResource.java의 경우 회원가입, 사용자 본인의 정보수정, 사용자 본인의 비밀번호 찾기 등의 요청을 받는 컨트롤러이다.
UserResource.java는 대부분 관리자에 의한 `사용자 관리` 요청을 중점적으로 받는 컨트롤러로 관리자에 의한 사용자 생성, 정보수정, 사용자 삭제 등이 해당된다. 따라서, UserResource.java에서는 요청을 받을 때에 요청을 보낸 사용자가 ADMIN인지 판단하여 요청을 받아들인다.

그 중 권한관리와 관련된 정보 수정 API를 살펴보자.

### UserResource.java

```java
@RestController
@RequestMapping("/api")
public class UserResource {

      ...(중략)...

    /**
     * {@code PUT /users} : Updates an existing User.
     *
     * @param userDTO the user to update.
     * @return the {@link ResponseEntity} with status {@code 200 (OK)} and with body the updated user.
     * @throws EmailAlreadyUsedException {@code 400 (Bad Request)} if the email is already in use.
     * @throws LoginAlreadyUsedException {@code 400 (Bad Request)} if the login is already in use.
     */
    @PutMapping("/users")
    @PreAuthorize("hasAuthority(\"" + AuthoritiesConstants.ADMIN + "\")")
    public ResponseEntity<UserDTO> updateUser(@Valid @RequestBody UserDTO userDTO) {
        log.debug("REST request to update User : {}", userDTO);
        Optional<User> existingUser = userRepository.findOneByEmailIgnoreCase(userDTO.getEmail());
        if (existingUser.isPresent() && (!existingUser.get().getId().equals(userDTO.getId()))) {
            throw new EmailAlreadyUsedException();
        }
        existingUser = userRepository.findOneByLogin(userDTO.getLogin().toLowerCase());
        if (existingUser.isPresent() && (!existingUser.get().getId().equals(userDTO.getId()))) {
            throw new LoginAlreadyUsedException();
        }
        Optional<UserDTO> updatedUser = userService.updateUser(userDTO);

        return ResponseUtil.wrapOrNotFound(updatedUser,
            HeaderUtil.createAlert(applicationName, "userManagement.updated", userDTO.getLogin()));
    }
}
```
정보 수정 API는 PUT 방식으로 ("/users")로 선언되어 UserDTO를 클라이언트로부터 받아 정보 수정을 처리한다. 
이때, @PreAuthorize라는 어노테이션을 사용하여 요청을 보낸 사용자의 권한이 ADMIN인 경우에만 요청을 허용한다.

정보수정로직을 살펴보면,
해당 유저가 존재하는지 확인하고, 존재하는 경우 userRepository에 접근하여 해당 유저의 정보를 가져온다.
그 다음 userService.updateUser를 호출하여 사용자 정보를 수정한다.

## 외부영역 - 아웃바운드 어댑터 개발

회원가입을 하여 사용자가 신규 등록되거나 관리자가 사용자를 등록하여 새로운 사용자가 생성되면, Rental서비스로 비동기 호출하여 Rental(도서 카드)를 생성하도록 요청한다.
이때, 사용자가 직접 회원가입하는 경우 userService.registerUser가 호출되고 관리자가 사용자를 생성하는 경우 userService.createUser가 호출된다.

따라서, userService.registerUser와 userService.createUser 두 개의 메소드에서 kafka를 통해 비동기 호출하여 도서카드를 생성하도록 하였다.


### UserService.java
```java
@Service
@Transactional
public class UserService {
    ...(중략)...
    public User registerUser(UserDTO userDTO, String password) throws InterruptedException, ExecutionException, JsonProcessingException {
        ...(중략)...
        User newUser = new User();
        String encryptedPassword = passwordEncoder.encode(password);
        newUser.setLogin(userDTO.getLogin().toLowerCase());
        // new user gets initially a generated password
        newUser.setPassword(encryptedPassword);
        ...(중략)...
        userRepository.save(newUser);
        createRental(newUser.getId());
        this.clearUserCaches(newUser);
        log.debug("Created Information for User: {}", newUser);
        return newUser;
    }

    public User createUser(UserDTO userDTO) throws InterruptedException, ExecutionException, JsonProcessingException {
        User user = new User();
        user.setLogin(userDTO.getLogin().toLowerCase());
        ...(중략)...
        user.setPoint(1000);
        userRepository.save(user);
        createRental(user.getId());
        this.clearUserCaches(user);
        log.debug("Created Information for User: {}", user);
        return user;
    }

    public void createRental(Long id) throws InterruptedException, ExecutionException, JsonProcessingException {
        gatewayProducer.createRental(id);
    }
}
```

사용자를 신규 등록한 뒤, createRental 메소드를 호출, gatewayProducer.createRental을 호출하여 비동기 호출로 Rental 서비스로 도서 카드를 생성하도록 한다.

### GatewayProducer.java

```java
@Service
public class GatewayProducer {
  
    public PublishResult createRental(Long userId) throws ExecutionException, InterruptedException, JsonProcessingException{

        UserIdCreated userIdCreated = new UserIdCreated(userId);
        String message = objectMapper.writeValueAsString(userIdCreated);
        RecordMetadata metadata = producer.send(new ProducerRecord<>(TOPIC_RENTAL, message)).get();
        return new PublishResult(metadata.topic(), metadata.partition(), metadata.offset(), Instant.ofEpochMilli(metadata.timestamp()));


    }

}
```

아웃바운드 어댑터인 GatewayProducer.createRental은 사용자의 Id정보를 받아 UserIdCreated라는 도메인 이벤트를 생성한 뒤 Rental서비스가 구독 중인 Topic과 함께 kafka 메세지를 전송한다.
전송된 메세지는 Rental 서비스의 인바운드 어댑터를 통해 소비되어 Rental(도서 카드)를 생성한다.
