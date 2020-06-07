# Jhipster를 활용한 Microservice application 개발

## Service 만들기

먼저, 도서대여시스템의 core service 인 book, user, rental 서비스부터 생성한다.

이때 service를 만드는 방식은 동일하나, port와 package명을 다르게 해야한다.

- book :
  - port: 8081
  - package : com.skcc.book
- user :
  - port : 8082
  - package : com.skcc.user
- rental :
  - port : 8083
  - package : com.skcc.rental

### core service 1 : book

1. book 폴더 생성
2. book 폴더를 jhipster 프로젝트로 변경

![image](https://user-images.githubusercontent.com/18453570/81146423-38052280-8fb3-11ea-8397-bd615fa6f08b.png)

```
mkdir book
cd book
jhipster
```
3. 옵션 선택

**port설정과 package 설정을 잊지말자**

![image](https://user-images.githubusercontent.com/18453570/81146788-ec9f4400-8fb3-11ea-90ee-5e3f5a4d860a.png)

-------------------------옵션선택 설명--------------------

### core service 2 : user

core service 1의 book과 같은 방식으로 user service를 생성한다.

**port설정과 package 설정을 잊지말자**

```
mkdir user
cd user
jhipster
```

- 옵션 선택

![image](https://user-images.githubusercontent.com/18453570/81147097-8c5cd200-8fb4-11ea-98b4-88dfae9d1059.png)


### core service 3 : rental

core service 1의 book과 같은 방식으로 rental service를 생성한다.

**port설정과 package 설정을 잊지말자**

```
mkdir rental
cd rental
jhipster
```

![image](https://user-images.githubusercontent.com/18453570/81147568-861b2580-8fb5-11ea-8f0c-545d23d60041.png)

## service 실행시키기

이제 생성했던 서비스들을 실행시켜 registry를 확인해 정상적으로 동작하는지 확인해본다.

아래의 이미지처럼 각 서비스의 Directory에 들어가 `./mvnw`를 입력해 서비스를 실행시킨다.

![image](https://user-images.githubusercontent.com/18453570/81147751-f2962480-8fb5-11ea-9283-0aa58fbd15c6.png)

먼저, 각 서비스의 Directory에 아래와 같은 이미지가 뜨면 정상적으로 실행된 것이다.

1. book

![image](https://user-images.githubusercontent.com/18453570/81148014-69cbb880-8fb6-11ea-93c2-16c1aaa9aa49.png)

2. user

![image](https://user-images.githubusercontent.com/18453570/81148038-7b14c500-8fb6-11ea-8f54-dc98ec8851a5.png)

3. rental

![image](https://user-images.githubusercontent.com/18453570/81148066-88ca4a80-8fb6-11ea-9cb4-797e8061408a.png)

그 다음, localhost:8761에 접속해 Registry를 확인해보면 gateway, book, user, rental 서비스가 등록되어있는 것을 볼 수 있다.

![image](https://user-images.githubusercontent.com/18453570/81148406-2faee680-8fb7-11ea-92dc-22cfcaeee6ae.png)

## 생성한 service에 Entity를 추가하기

생성한 service에 Entity를 추가하는 방법은 크게 2가지로 나눌 수 있다.

- [Jhipster Online](https://start.jhipster.tech/#/) 을 통해 Entity 및 Relationship을 설정할 수 있다.
- Terminal에 command를 입력해 Entity 및 Relationship을 설정할 수 있다. 


1. book service에 book entity 생성

book Directory에 들어가 아래 command를 입력하여 book Entity를 생성한다.
생성 시, author, title, description으로 책 저자, 책 이름, 책 설명을 변수로 선언하고 이는 반드시 입력되어야하는 값이기 때문에 required로 하였다.

```
cd book
jhipster entity book
```

그 다음 entity에 추가할 변수와 그 변수의 자료형, 옵션, 다른 entity와의 관계를 설정할 수 있다. 

![image](https://user-images.githubusercontent.com/18453570/81156459-ecf20c00-8fc0-11ea-92ad-6e33ce9753ad.png)

entity생성과 변수 설정 완료 후, 추가적인 옵션을 선택할 수 있는데 그 선택은 아래와 같이 한다. 

![image](https://user-images.githubusercontent.com/18453570/81157507-0182d400-8fc2-11ea-94c6-a707e2595ebd.png)

마지막으로 master.xml을 overwrite할 것이냐는 옵션이 나오는데, 이때 y를 선택해야한다.

![image](https://user-images.githubusercontent.com/18453570/81157614-1ceddf00-8fc2-11ea-8a41-34def05dcb46.png)

2. user service에 user entity 생성

book service와 마찬가지로 진행한다.
Entity 생성 시, 변수로는 name과 email을 선언했다.

```
cd user
jhipster entity user
```

3. rental service에 rental entity와 rentalItem entity생성

이번에는 terminal이 아닌 Jhipster Online에서 entity와 relationship을 설정해보았다.

- rental Entity 생성 시, 변수로 userId, rentalCnt,  rentalStatus를 선언하였다. rentalStatus는 rental의 상태를 나타내는 값으로 enum으로 처리했다.
- rentalItem Entity 생성 시, 변수로 bookId와 rentalItemStatus를 선언하였다. rentalItemStuts는 현재 대출 중인 책의 상태를 나타내는 값으로 enum 처리하였다.

이때, rental 과 rentalItem은 oneToMany관계인데, 이를 rentalItem과 rental 관계로 바꿔 ManyToOne으로 설정하였다.


Jhipster Online에 접속해 `Design Entities`을 클릭-> `Create a new JDL model`버튼을 클릭하여 JDL studio에 접속해 아래와 같이 코드를 입력한다.

```
entity Rental{
	id Long,
	userId Long,
    rentalItemCnt Integer ,
    rentalStatus RentalStatus
}

entity RentalItem {
	id Long,
    bookId Long,
    rentalItemStatus RentalItemStatus
}


enum RentalStatus {
    OK, RENTALED, OVERDUE
}

enum RentalItemStatus{
	RENTALED, OVERDUE
}

// defining ManyToOne relationships


relationship ManyToOne {
	RentalItem{rental} to Rental
}

// Set pagination options
paginate * with pagination 

// Use Data Transfert Objects (DTO)
dto * with mapstruct

// Set service options to all except few
service all with serviceImpl 

```

코드를 저장한 후, 해당 파일을 내려받아 rental Directory의 최상위로 옮긴다.

그 다음 terminal에서 아래 command를 입력해 import 한다.

```
jhipster import-jdl ./my-jdl-file.jdl --force
```

## 추가한 entity를 gateway에 등록시키기

마지막으로 지금까지 추가한 entity들을 gateway에 등록시켜야한다.
book, user, rental, rentalItem을 차례로 등록시킨다.


```
cd gateway
jhipster entity book
```
위 command입력을 통해 book entity를 gateway에 추가시킬 수 있다.

입력 후, 몇가지 옵션을 선택하게 되는데 아래 이미지와 같이 선택한다.

이때 Entity의 path는 각 microservice가 존재하는 path로 수정해야한다. 

![image](https://user-images.githubusercontent.com/18453570/81278941-60634e80-9091-11ea-8e61-ac87302aa29a.png)

이후, overwrite하겠냐는 질문에는 `a`를 입력해야한다.


# 재실행 후, 테스트

이제 entity 생성까지 모두 마쳤으니, 다시 모든 application을 재실행시켜 제대로 동작하는지 확인한다.

먼저 gateway 폴더에서 아래와 같은 command를 입력한다.

```
docker-compose -f src/main/docker/jhipster-registry.yml up
```

새로운 terminal 창에서 다시 gateway 폴더에 들어가 아래 command를 입력한다.

```
./mvnw
```

book, user, rental 서비스 또한 실행시켜준다.

```
cd book
./mvnw

cd user
./mvnw

cd rental
./mvnw
```

이제 모두 잘 동작하는지 확인해본다.

1. Jhipster Registry 확인

- localhost:8761에 접속하여 아래 이미지처럼 뜨는지 확인

![image](https://user-images.githubusercontent.com/18453570/81279412-12027f80-9092-11ea-95f5-cfe93233ab99.png)

2. Gateway 확인

- localhost:8080에 접속하여 `admin`으로 접속하고, 아래 이미지처럼 뜨는지 확인

![image](https://user-images.githubusercontent.com/18453570/81279558-44ac7800-9092-11ea-9bcc-8a3d574f9b86.png)
![image](https://user-images.githubusercontent.com/18453570/81279591-4d9d4980-9092-11ea-9eed-8a2b0d79fb2c.png)
![image](https://user-images.githubusercontent.com/18453570/81279612-555cee00-9092-11ea-857a-9498192eeb25.png)
![image](https://user-images.githubusercontent.com/18453570/81279633-5beb6580-9092-11ea-8b51-dd705a32d9bc.png)
![image](https://user-images.githubusercontent.com/18453570/81279659-6443a080-9092-11ea-8636-62698889edd5.png)

위 이미지처럼 Jhipster가 Fake DB를 repository에 저장하여 화면을 보여준다. 물론, FakeDB는 추후 삭제할 수 있다.
또, 마지막 rental Items 화면의 경우, 현재 서비스와 서비스끼리의 business logic을 추가하지 않았기 때문에 Rental부분에 빈 영역으로 나와도 정상이다.

# Jhipster Project Package Refactoring

Jhipster에서 기본으로 생성해주는 Directory 구조도 훌륭하지만, CNAPS 3.0에서 추구하는 방향성과는 다른 부분들이 있다.
따라서 해당 부분들을 수정해주었다. 세부사항은 아래 링크에서 확인해보자.

>[Jhipster Project Package Refactoring](/contents/jhipster_package_ref.md)

# Jhipster Project Business Logic

Jhipster를 활용해 기본적인 CQRS와 화면구성, package 구조 또한 변경을 완료하였다. 이제 각 service와 service간의 Business Logic을 추가해보자.

-> [Jhipster Business Logic추가](/contents/jhipster_businesslogic.md)


**끝!**



