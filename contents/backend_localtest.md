# 시연 : 단위테스트 수행

앞서 개발한 대여서비스, 도서서비스, 사용자서비스(게이트웨이), 도서 카탈로그 서비스가 테스트 시나리오대로 잘 동작하는지 확인해보자.

먼저, 테스트 시나리오는 다음과 같다.

1. 사용자 2명 등록한다. USER1,USER2
2. USER2에게 운영자 권한을 준다.
3. 운영자가 3권의 도서정보를 등록한뒤 입고처리한다.(2권은 대출가능,1권의 대여중)
4. USER1으로 로그인한다.
5. USER1이 도서정보를 검색한다.
6. 대출가능한 도서를 2권 대출한다.
    1. 대여중인 도서는 대출할 수 없다.
7. 대출한 도서중 1권을 반납한다.
8. 2주가 지난도서는 연체된다.(USER2로 로그인하여 1권을 연체처리한다.)
9. USER1으로 다시 로그인하여 연체도서를 반납한다.
10. USER1이 다시 도서를 대출하려고 하나 시스템은 대출불가 메시지를 보낸다.
11. USER1은 연체일자를 확인하고 포인트로 가감하여 연체를 해제한다.
12. USER1은 다시 대출가능상태가 되고 대출을 수행한다.

서비스들을 동작시키기 전에 서비스 동작에 필요한 Jhipster 레지스트리, Kafka, MongoDB 등을 먼저 실행시켜보자.

## 서비스 동작 환경 실행시키기

먼저, 샘플에서 서비스 동작을 위한 환경을 구성하기 위해 Docker를 활용하였다. Kafka와 MongoDB를 로컬데스크탑에 구축해도 되지만, 로컬환경에 따라 구축방법과 명령어가 상이할 수 있기 때문에 간편하게 구축이 가능한 Docker설치를 선택하였다.
환경을 구축하기 전에 Docker가 로컬환경에 미리 설치되어있어야한다.

-> Docker Desktop 다운로드 링크 : https://www.docker.com/products/docker-desktop

설치가 완료되었으면 Docker Desktop을 실행시킨 뒤, Docker 회원가입과 로그인을 진행한다.

Docker를 실행하고 로그인을 완료하면 Docker의 상태가 아래와 같이 변경된다.

![image](https://user-images.githubusercontent.com/18453570/93732288-9ef64180-fc0b-11ea-9f73-54d83ea02275.png)



설치 및 구동이 완료된 Docker 컨테이너에 Jhipster 레지스트리, Kafka, MongoDB를 설치하고 실행시켜보자. 이때 MariaDB설치를 제외한 이유는 대여, 도서 서비스에 사용된 MariaDB는 in-memory로 설정하였기 때문에 서비스를 구동시키는 것 만으로도 데이터베이스를 사용할 수 있기 떄문이다.
Jhipster 프로젝트는 기본적으로 Microservice Application이나 Microservice Gateway를 생성하면 선택한 개발환경에 맞게 Docker 파일을 생성해준다.
생성이 완료된 Docker 파일을 살펴보면 port 설정, Docker 이미지 정보 등이 모두 yaml 형식으로 정의되어있다.

Jhipster에서 생성한 Docker 파일을 활용해 컨테이너를 생성 및 실행시킬때 모두 공통적으로 `docker-compose`명령어를 입력한다. docker-compose는 컨테이너를 생성 및 실행하는 명령어입니다. docker-compose는 yaml 형식의 파일을 통해 컨테이너를 생성하고 실행하게 되는데, yaml형식의 파일에는 개발환경 구성와 컨테이너 실행에 필요한 옵션, 의존성 등의 정보가 담긴다.
즉, docker-compose 명령어와 함께 쓰이는 yaml 파일은 복잡한 도커 실행옵션들을 미리 적어둔 문서라고 볼 수 있다.
이러한 yaml파일을 docker-compose 명령어로 실행시킴으로써 기존의 사용자가 일일히 도커 실행옵션을 입력할 필요없이 한번에 개발환경 세팅과 컨테이너 생성, 실행을 할 수 있다.

그럼 이제, Jhipster 레지스트리, Kafka, MongoDB 모두 Jhipster가 생성해준 Docker파일을 활용하여 설치/실행을 진행해보자.



1. Jhipster 레지스트리 실행시키기
   
   Jhipster 레지스트리는 Gateway에서만 실행시킬 수 있다. 따라서, Gateway 디렉토리로 이동하여 아래 커맨드를 입력해 실행시킨다.

   ```bash
   docker-compose -f src/main/docker/jhipster-registry.yml up
   ```
2. Kafka 실행시키기
   
   Kafka는 앞서 개발한 모든 서비스에서 실행가능하다. 하지만 모든 서비스에서 실행할 필요 없이 한가지 서비스에서만 Kafka를 실행시켜도 모든 서비스가 Kafka를 이용할 수 있다.
   따라서, 새로운 커맨드 창을 열어 Gateway 디렉토리로 이동해 아래 커맨드를 입력해 실행시킨다. 이때 `-d` 옵션은 백그라운드에서 실행하는 옵션으로 kafka 실행과 DB 실행시 주로 사용된다.

    ```bash
    docker-compose -f src/main/docker/kafka.yml up -d
    ```
    
    단, Kafka를 실행시킬때에는 한 가지 주의해야할 점이 있다. 위 커맨드를 입력하면 Zookeepr, Kafka 두개의 컨테이너가 생성되는데 Zookeeper 실행이 완료되지 않은 상태에서 Kafka가 실행되는 경우 서비스에서 Kafka 메세지를 사용할 수 없다.
    Zookeeper만 실행되고 Kafka는 실행되지 않은 경우 Docker 대쉬보드로 이동하여 직접 실행시킬 수 있다. 아래 이미지를 참고하여 Kafka를 개별 실행시키고 Zookeeper와 Kafka 모두 정상적으로 실행 중인지 반드시 확인하도록 한다.

    - Docker 대쉬보드를 클릭한다.
    ![image](https://user-images.githubusercontent.com/18453570/93734003-8b020e00-fc12-11ea-8fc5-91970b02a8ca.png)

    - Kafka 컨테이너 옆 Start 버튼을 클릭하여 Kafka를 실행한다.
    ![image](https://user-images.githubusercontent.com/18453570/93734068-c3a1e780-fc12-11ea-8b02-e1ebb15ebec6.png)

    - Kafka가 정상적으로 실행된 경우 아래 이미지와 같다.
    ![image](https://user-images.githubusercontent.com/18453570/93734320-c3561c00-fc13-11ea-92ac-23522c973f1b.png)


3. MongoDB 실행시키기
   
   MongoDB는 도서 카탈로그 서비스에서만 실행가능하다. 따라서, 도서 카탈로그 디렉토리로 이동하여 아래 커맨드를 입력해 실행시킨다.

   ```bash
   docker-compose -f src/main/docker/mongodb.yml up -d
   ```

4. 결과 확인하기
   
   Jhipter 레지스트리, Kafka, MongoDB 모두 정상적으로 실행되었는지 확인해보자.
   정상적으로 실행 중인 경우 아래 이미지와 같이 컨테이너 이미지가 모두 초록색으로 변경되어있을 것이다.

    ![image](https://user-images.githubusercontent.com/18453570/93734320-c3561c00-fc13-11ea-92ac-23522c973f1b.png)



## 게이트웨이와 마이크로서비스 동작시키기

이제, 게이트웨이와 각각의 마이크로서비스를 동작시켜보자. 

각 서비스가 게이트웨이에 등록할 수 있도록 게이트웨이를 먼저 실행한 뒤 나머지 서비스들을 실행시킨다. 

실행시킬 때에는 각 서비스별로 Cmd 창 또는 터미널 새창을 열어 실행시키도록 하며, 서비스 테스트를 종료하기 전까진 터미널 창을 닫지 않도록 한다.

먼저 각 서비스 디렉토리의 루트폴더에서 아래 커맨드를 입력해 실행시킨다.

- Linux 또는 MacOs인경우
```bash
./mvnw
```

- Windows인 경우
```bash
mvnw
```

게이트웨이 및 서비스들을 모두 정상적으로 실행시키면 아래 이미지와 같은 화면을 확인할 수 있다.

```
2020-09-21 14:30:49.083  INFO 99272 --- [  restartedMain] com.skcc.gateway.GatewayApp              : Started GatewayApp in 22.021 seconds (JVM running for 22.692)
2020-09-21 14:30:49.089  INFO 99272 --- [  restartedMain] com.skcc.gateway.GatewayApp              :
----------------------------------------------------------
	Application 'gateway' is running! Access URLs:
	Local: 		http://localhost:8080/
	External: 	http://10.250.66.104:8080/
	Profile(s): 	[dev, swagger]
----------------------------------------------------------
2020-09-21 14:30:49.089  INFO 99272 --- [  restartedMain] com.skcc.gateway.GatewayApp              :
----------------------------------------------------------
	Config Server: 	Connected to the JHipster Registry running in Docker
----------------------------------------------------------
```

```
2020-09-21 14:30:31.040  INFO 98974 --- [  restartedMain] com.skcc.rental.RentalApp                : Started RentalApp in 25.556 seconds (JVM running for 26.667)
2020-09-21 14:30:31.045  INFO 98974 --- [  restartedMain] com.skcc.rental.RentalApp                :
----------------------------------------------------------
	Application 'rental' is running! Access URLs:
	Local: 		http://localhost:8083/
	External: 	http://10.250.66.104:8083/
	Profile(s): 	[dev, swagger]
----------------------------------------------------------
2020-09-21 14:30:31.045  INFO 98974 --- [  restartedMain] com.skcc.rental.RentalApp                :
----------------------------------------------------------
	Config Server: 	Connected to the JHipster Registry running in Docker
----------------------------------------------------------
```

```
2020-09-21 14:30:38.011  INFO 99140 --- [  restartedMain] com.skcc.book.BookApp                    : Started BookApp in 22.316 seconds (JVM running for 23.017)
2020-09-21 14:30:38.015  INFO 99140 --- [  restartedMain] com.skcc.book.BookApp                    :
----------------------------------------------------------
	Application 'book' is running! Access URLs:
	Local: 		http://localhost:8081/
	External: 	http://10.250.66.104:8081/
	Profile(s): 	[dev, swagger]
----------------------------------------------------------
2020-09-21 14:30:38.015  INFO 99140 --- [  restartedMain] com.skcc.book.BookApp                    :
----------------------------------------------------------
	Config Server: 	Connected to the JHipster Registry running in Docker
----------------------------------------------------------
```

```
2020-09-21 14:30:41.886  INFO 99216 --- [  restartedMain] com.skcc.bookcatalog.BookCatalogApp      :
----------------------------------------------------------
	Application 'bookCatalog' is running! Access URLs:
	Local: 		http://localhost:8082/
	External: 	http://10.250.66.104:8082/
	Profile(s): 	[dev, swagger]
----------------------------------------------------------
2020-09-21 14:30:41.887  INFO 99216 --- [  restartedMain] com.skcc.bookcatalog.BookCatalogApp      :
----------------------------------------------------------
	Config Server: 	Connected to the JHipster Registry running in Docker
----------------------------------------------------------
```
## 시나리오 테스트하기

게이트웨이와 서비스들이 정상적으로 실행되었으면, [http://localhost:8080]으로 접속하여 시나리오대로 테스트를 진행한다.

1. http://localhost:8080으로 접속한다.
   정상적으로 접속된 경우 아래 이미지와 같은 화면을 확인 할 수 있다.
   ![image](https://user-images.githubusercontent.com/18453570/94217423-f1da3c80-ff1c-11ea-9bda-1ba950463b99.png)
2. 사용자 2명을 등록한다. (USER1,USER2 으로 회원가입)
   아래 화면은 등록 예시이다. USER1의 아이디는 user1으로, USER2의 아이디는 user2로 등록하였다. 회원가입시 아이디와 이메일은 다른 유저와 중복되지 않아야함을 주의하자.
    ![image](https://user-images.githubusercontent.com/18453570/94217461-01f21c00-ff1d-11ea-9be7-8efbd62850b5.png)
3. USER2에게 운영자 권한을 준다.
   USER2에게 운영자 권한을 주기 위해선 운영자 권한을 가진 계정으로 로그인해야한다. 권한에 따라 접근할 수 있는 메뉴가 다르기 때문이다. 권한 관리는 사용자 관리 메뉴에서 수행할 수 있는데, 사용자 관리 메뉴는 운영자 권한을 가진 사용자만이 접근할 수 있다.
   기본 생성된 운영자의 아이디와 패스워드는 정보는 아래와 같다.
   - Id : admin
   - Pw : admin
   기본 운영자로 로그인 완료 후 관리자 탭으로 이동하여 사옹자 관리 메뉴로 이동한다. 사용자 관리 메뉴화면은 아래와 같다.
   ![image](https://user-images.githubusercontent.com/18453570/94218882-6fec1280-ff20-11ea-90f3-50039be25e72.png)
   2번 시나리오에서 생성한 user1과 user2 정보를 볼 수 있다. 사용자가 회원가입을 하는 경우 `ROLE_USER` 권한만을 가지고 있음을 확인할 수 있다.
   USER2에게 운영자 권한을 줘야하므로 user2의 우측 수정 버튼을 눌러 아래와 같이 `ROLE_USER`와 `ROLE_ADMIN` 권한 두개를 부여한다.(Shift를 누른 상태로 클릭한 뒤 저장을 클릭한다.)
   결과는 아래 이미지와 같이 user2 권한이 변경되어있음을 알 수 있다.
   ![image](https://user-images.githubusercontent.com/18453570/94226925-ce6ebc00-ff33-11ea-9eac-9e72fb423685.png)
4. 운영자가 3권의 도서정보를 등록한뒤 입고처리한다.(2권은 대출가능,1권의 대여중)
   본래 입고처리 프로세스는 출판사나 도서 구매 업체가 시스템에 입고도서를 등록하면 도서시스템 관리자 도서를 등록하는 프로세스를 갖고 있다. 하지만 샘플에서는 운영자가 출판사나 도서 구매 업체 역할을 대신 한다고 가정하고 진행하도록한다.
   따라서, 운영자가 입고도서를 등록하고 도서 등록 메뉴를 통해 도서로 등록하도록 한다.
   먼저, user2로 로그인한다.
   ![image](https://user-images.githubusercontent.com/18453570/94227028-21e10a00-ff34-11ea-86ea-76061b7e0385.png)

   로그인 완료 후, 아래 이미지와 같이 상단의 Entities 메뉴에서 InStockBook을 클릭하여 입력 폼을 채운 뒤 저장하여 입고도서를 등록한다.
   
   ![image](https://user-images.githubusercontent.com/18453570/94242163-96c23d00-ff50-11ea-81df-f62283987ce2.png)
   ![image](https://user-images.githubusercontent.com/18453570/94242301-cb35f900-ff50-11ea-82fa-ca3bc02f216b.png)
   
   그 결과, 아래 이미지와 같은 화면을 확인할 수 있다.
   
   ![image](https://user-images.githubusercontent.com/18453570/94242372-e6a10400-ff50-11ea-961d-a3381eff8c08.png)
   
   이제 입고처리를 진행해보자.
   상단의 도서관리 탭에서 도서 등록 메뉴로 이동한다.
   
   <img width="897" alt="image" src="https://user-images.githubusercontent.com/18453570/94407734-c8384400-01ae-11eb-836a-2b3c749a384b.png">
   
   각 도서마다 등록메뉴를 눌러 입력 폼을 채운 뒤 저장하여 도서를 등록한다. 이때 3권의 입고도서 중 한권은 대여중 처리를 한다. 결과는 아래 이미지와 같다.
   
   <img width="960" alt="image" src="https://user-images.githubusercontent.com/18453570/94407876-05043b00-01af-11eb-9e86-2f7231a7c21d.png">

5. 로그아웃한 뒤, USER1으로 로그인하여 도서 대여 탭으로 이동하여 도서 정보를 검색한다.
   검색어를 `JPA`로 입력하면 결과는 아래 이미지와 같다.

   <img width="960" alt="image" src="https://user-images.githubusercontent.com/18453570/94408043-3bda5100-01af-11eb-94fe-6cd86833bd2a.png">

6. 대출가능한 도서를 2권 대출한다.
   이제 도서 대출을 테스트해보자. 도서대여메뉴에는 등록된 도서 목록이 검색된다. 이때 대여 중인 도서는 대여버튼이 비활성화되어 대여할 수 없다.
   따라서, 대여가 가능한 나머지 도서 2권을 대여한다.
   
   <img width="960" alt="image" src="https://user-images.githubusercontent.com/18453570/94408326-9a073400-01af-11eb-893a-eaca579b665c.png">
   
   대여가 완료되면 아래 이미지와 같이 대여완료 알람이 뜨고 대여버튼이 비활성화된다.
   
   <img width="960" alt="image" src="https://user-images.githubusercontent.com/18453570/94408425-bd31e380-01af-11eb-8129-13ece00bf592.png">
   
7. 대출한 도서중 1권을 반납한다.
   이제 반납기능을 테스트해보자.
   대여 도서, 반납 도서, 연체 도서 등 사용자의 기록은 상단 메뉴의 Mypage를 통해 확인할 수 있다. 아래 이미지와 같이 Mypage로 이동하면 6번에서 테스트한 두권의 도서가 대여 도서목록에 표시됨을 확인할 수 있다.
   또한 도서를 대여하면 도서 당 30포인트를 적립할 수 있는데, 잔여 포인트가 1060인 것을 확인할 수 있다.
   
   <img width="948" alt="image" src="https://user-images.githubusercontent.com/18453570/94408639-071ac980-01b0-11eb-8c86-7be85c620232.png">
   
   위 이미지와 같은 페이지를 확인하였으면 나의 대여 도서 목록에 있는 도서 중 한 권을 반납하기 버튼을 눌러 반납한다.
8. 2주가 지난도서는 연체된다.(USER2로 로그인하여 1권을 연체처리한다.)
   이제 연체 기능을 테스트해본다. User1을 로그아웃한 뒤, 관리자 계정인 User2로 로그인하여 User1이 대여한 도서 중 한권을 연체처리해보자.
   User2로 로그인한 뒤, 도서관리 탭의 대여도서관리 메뉴로 이동한다.
   
   <img width="960" alt="image" src="https://user-images.githubusercontent.com/18453570/94408954-77c1e600-01b0-11eb-81a0-aa67dd31df15.png">
   
   대여도서관리 메뉴로 이동하면 User1이 대여 중인 도서 한권의 목록이 보인다. 연체처리 버튼을 눌러 연체처리한다.
   
   <img width="960" alt="image" src="https://user-images.githubusercontent.com/18453570/94409041-9627e180-01b0-11eb-9a00-85ac3ca10e9f.png">
   
9.  USER1으로 다시 로그인하여 연체도서를 반납한다.
    User2를 로그아웃 후 다시 User1으로 로그인하여 연체도서를 반납한다.
    Mypage메뉴로 이동하면 아래 이미지와 같이 User2가 연체처리한 도서가 나의 연체 도서 목록에 추가되어있을 것이다.
    
    <img width="940" alt="image" src="https://user-images.githubusercontent.com/18453570/94409241-de470400-01b0-11eb-885b-4cc6b72012d7.png">
    
    반납하기 버튼을 눌러 아래 이미지와 같이 반납 도서에 두권의 도서 목록이 있는 것을 확인한다.
    
    <img width="960" alt="image" src="https://user-images.githubusercontent.com/18453570/94409398-13535680-01b1-11eb-932e-064089ff98b4.png">
    
10. USER1이 다시 도서를 대출하려고 하나 시스템은 대출불가 메시지를 보낸다.
    User1은 대여했던 도서를 모두 반납하였지만, 한 권의 책이 연체처리되었기 때문에 현재 연체상태로 도서를 대여할 수 없다. 연체상태일때 도서를 대여할 수 없는지 테스트해보자.
    도서 대여 메뉴로 이동하여 대여 가능한 도서를 대여신청하면 아래 이미지와 같이 연체상태이기 때문에 대여를 할 수 없다는 메세지와 함께 대여가 완료되지 않는다.
    
    <img width="960" alt="image" src="https://user-images.githubusercontent.com/18453570/94409686-7c3ace80-01b1-11eb-981b-02e4fd44d05f.png">
    
11. USER1은 연체일자를 확인하고 포인트로 가감하여 연체를 해제한다.
    연체 상태를 해제하기 위해 다시 Mypage로 이동한다. 그 다음 연체 해제버튼을 클릭하여 연체료를 결제한다.
    연체료 결제가 완료되면 아래 이미지와 같이 대여 가능 상태가 대여 가능으로 변경되며, 연체료 30포인트를 결제하였기 때문에 잔여 Point는 1030으로, 연체료는 0으로 변경된다.
    
    <img width="960" alt="image" src="https://user-images.githubusercontent.com/18453570/94409907-bc01b600-01b1-11eb-8bd2-f71171c22586.png">
    
12. USER1은 다시 대출가능상태가 되고 대출을 수행한다.
    이제 다시 도서 대여를 시도해보자.
    도서 대여 메뉴로 이동하여 도서 대여버튼을 클릭해 도서 대여를 하면 아래 이미지와 같이 도서가 대여됨을 확인할 수 있다.
    
    <img width="960" alt="image" src="https://user-images.githubusercontent.com/18453570/94410023-dd62a200-01b1-11eb-92b0-98e5c08b87f0.png">
    
    또한 대여 완료 후, Mypage를 확인해보면 도서 한권을 대여하였기 때문에 30포인트가 적립되어 잔여 Point가 1060이 되었고 나의 대여도서 목록에 해당도서가 추가되어있다.
    
    <img width="958" alt="image" src="https://user-images.githubusercontent.com/18453570/94410085-ea7f9100-01b1-11eb-98b1-6ba6dc29421d.png">


