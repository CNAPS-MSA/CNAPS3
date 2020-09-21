# 백엔드 동작 확인하기

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
9. 연체도서를 반납한다.
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
Jhipster 레지스트리, Kafka, MongoDB 모두 Jhipster가 생성해준 Docker파일을 활용하여 설치/실행을 진행해보자.


1. Jhipster 레지스트리 실행시키기
   
   Jhipster 레지스트리는 Gateway에서만 실행시킬 수 있다. 따라서, Gateway 디렉토리로 이동하여 아래 커맨드를 입력해 실행시킨다.

   ```bash
   docker-compose -f src/main/docker/jhipster-registry.yml up
   ```
2. Kafka 실행시키기
   
   Kafka는 앞서 개발한 모든 서비스에서 실행가능하다. 하지만 모든 서비스에서 실행할 필요 없이 한가지 서비스에서만 Kafka를 실행시켜도 모든 서비스가 Kafka를 이용할 수 있다.
   따라서, 새로운 커맨드 창을 열어 Gateway 디렉토리로 이동해 아래 커맨드를 입력해 실행시킨다.

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

```bash
./mvnw
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





