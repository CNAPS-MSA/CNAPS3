# JHipster 를 이용한 애플리케이션 배포
앞서 말했듯이 쿠버네티스 환경에 배포하기 위해서는 몇 가지 설정 파일이 필요하다. JHipster 는 쿠버네티스 이런 설정 파일을 자동 생성해주고 이를 기반으로 쉽게 배포가 가능하다. 가장 먼저 GCP 환경에서 애플리케이션을 도커 이미지로 빌드한 후에 컨테이너 레지스트리에 등록할 것이다. 그리고 나서 JHipster 이용하여 쿠버네티스 환경을 위한 배포 구성 파일을 생성하고, 이 구성 파일과 등록된 컨테이너 이미지로 애플리케이션을 배포하고 확인해보자.

## 지속적 통합
### 도커 이미지 빌드 및 컨테이너 레지스트리 등록
1. GCP 콘솔 활면에서 상단의 터미널 아이콘 (그림 10.8 빨간 박스 영역) 을 선택하면 하단에 터미널 창이 열린다. 이제부터 터미널에서 작업을 진행하면 된다.
<img width="1570" alt="10_6" src="https://user-images.githubusercontent.com/62231786/98325802-02f58d80-2033-11eb-9f18-8e399836dcd7.png">
2. 소스를 다운받을 폴더를 생성하고 해당 폴더로 이동한다.

```
$ mkdir cnaps && cd cnaps
```

3.	깃헙에서 소스를 다운로드 받는다.
- k8s, gateway, book, bookCatalog, rental

```
$ git clone https://github.com/CNAPS-MSA/k8s.git
$ git clone https://github.com/CNAPS-MSA/gateway.git
$ git clone https://github.com/CNAPS-MSA/book.git
$ git clone https://github.com/CNAPS-MSA/bookCatalog.git
$ git clone https://github.com/CNAPS-MSA/rental.git
```

4.	도커 이미지를 빌드 후, 컨테이너 레지스트리에 푸시한다.
- jib 라이브러리를 사용하면 별도의 도커 설치없이 도커 이미지를 빌드하고 컨테이너 레지스트리에 푸시까지 한 번에 수행할 수 있다.

```
$ cd gateway
$ ./mvnw package -Pprod -DskipTests jib:build -Dimage=gcr.io/$PROJECT_ID/gateway:latest
$ cd book
$ ./mvnw package -Pprod -DskipTests jib:build -Dimage=gcr.io/$PROJECT_ID/book:latest
$ cd bookCatalog
$ ./mvnw package -Pprod -DskipTests jib:build -Dimage=gcr.io/$PROJECT_ID/bookcatalog:latest
$ cd rental
$ ./mvnw package -Pprod -DskipTests jib:build -Dimage=gcr.io/$PROJECT_ID/rental:latest
```

- 콘솔 화면에서 Container Registry 의 이미지 메뉴로 이동하면 아래와 같이 방금 빌드한 도커 이미지가 등록된 것을 확인할 수 있다.
<img width="1570" alt="10_9" src="https://user-images.githubusercontent.com/62231786/98325811-04bf5100-2033-11eb-91f2-fab906c2c772.png">

## 지속적 배포
### 애플리케이션 배포
1.	쿠버네티스 설정 파일 폴더로 이동한다.

```
$ ~/cnaps/k8s/
```

2.	쿠버네티스 설정 파일 수정
- 쿠버네티스  디플로이먼트 리소스 생성 시, 컨테이너 레지스트리에 등록한 이미지를 내려받을 수 있도록 아래와 같이 수정한다.
- 아래 4개 파일 모두 프로젝트 ID 부분을 수정해준다.
    - gateway-k8s/gateway-deployment.yml
    - book-k8s/book-deployment.yml
    - bookcatalog-k8s/bookcatalog-deployment.yml
    - rental-k8s/rental-deployment.yml
    
```
<변경 전>
...
        containers:
          - name: gateway-app
            image: gcr.io/cnaps-project/gateway
...
<변경 후>
...
        containers:
          - name: gateway-app
            image: gcr.io/cnaps-project-286804/gateway
...
```

3.	이제 쿠버네티스 리소스를 생성하는 쉘 스크립트를 실행한다. 해당 스크립트를 실행하면 각 서비스 설정 파일을 기반으로 리소스가 생성된다.

```
$ ./kubectl-apply.sh -f
configmap/application-config created
secret/registry-secret created
service/jhipster-registry created
statefulset.apps/jhipster-registry created
deployment.apps/jhipster-kafka created
service/jhipster-kafka created
deployment.apps/jhipster-zookeeper created
service/jhipster-zookeeper created
deployment.apps/gateway created
secret/gateway-mariadb created
deployment.apps/gateway-mariadb created
service/gateway-mariadb created
service/gateway created
deployment.apps/book created
secret/book-mariadb created
deployment.apps/book-mariadb created
service/book-mariadb created
service/book created
deployment.apps/bookcatalog created
configmap/bookcatalog-mongodb-config created
configmap/bookcatalog-mongodb-init created
statefulset.apps/bookcatalog-mongodb created
service/bookcatalog-mongodb created
service/bookcatalog created
deployment.apps/rental created
secret/rental-mariadb created
deployment.apps/rental-mariadb created
service/rental-mariadb created
service/rental created
```

4.	쿠버네티스 리소스 상태 확인
- 쿠버네티스 리소스 전체 조회를 한다. 해당 명령어로 서비스가 정상적으로 기동이 되었는지 확인할 수 있다. 모든 서비스가 아래와 같이 STATUS 항목이 모둔 Running 이면 정상적으로 기동된 것이다.

```
$ kubectl get all
NAME                                      READY   STATUS    RESTARTS   AGE
pod/book-7d67596bb9-b7cbn                 1/1     Running   0          33m
pod/book-mariadb-86c84b4dc4-dcv7g         1/1     Running   0          33m
pod/bookcatalog-7f64d657d4-xk9rt          1/1     Running   0          33m
pod/bookcatalog-mongodb-0                 1/1     Running   0          33m
pod/gateway-5f8bb9f8f5-zq98x              1/1     Running   0          33m
pod/gateway-mariadb-7958dbcffc-6xbb8      1/1     Running   0          33m
pod/jhipster-kafka-85f64cc674-s89x8       1/1     Running   0          33m
pod/jhipster-registry-0                   1/1     Running   0          33m
pod/jhipster-registry-1                   1/1     Running   0          33m
pod/jhipster-zookeeper-55755c6f65-5fxnj   1/1     Running   0          33m
pod/rental-7c868d57f4-pvg28               1/1     Running   0          33m
pod/rental-mariadb-759fcf5f9c-zdr4k       1/1     Running   0          33m

NAME                          TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)          AGE
service/book                  ClusterIP      10.0.2.41     <none>          8081/TCP         33m
service/book-mariadb          ClusterIP      10.0.2.90     <none>          3306/TCP         33m
service/bookcatalog           ClusterIP      10.0.2.132    <none>          8082/TCP         33m
service/bookcatalog-mongodb   ClusterIP      None          <none>          27017/TCP        33m
service/gateway               LoadBalancer   10.0.13.1     34.64.121.108   8080:32740/TCP   33m
service/gateway-mariadb       ClusterIP      10.0.2.157    <none>          3306/TCP         33m
service/jhipster-kafka        ClusterIP      10.0.15.99    <none>          9092/TCP         33m
service/jhipster-registry     ClusterIP      None          <none>          8761/TCP         33m
service/jhipster-zookeeper    ClusterIP      10.0.6.45     <none>          2181/TCP         33m
service/kubernetes            ClusterIP      10.0.0.1      <none>          443/TCP          46m
service/rental                ClusterIP      10.0.11.254   <none>          8083/TCP         33m
service/rental-mariadb        ClusterIP      10.0.4.5      <none>          3306/TCP         33m

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/book                 1/1     1            1           33m
deployment.apps/book-mariadb         1/1     1            1           33m
deployment.apps/bookcatalog          1/1     1            1           33m
deployment.apps/gateway              1/1     1            1           33m
deployment.apps/gateway-mariadb      1/1     1            1           33m
deployment.apps/jhipster-kafka       1/1     1            1           33m
deployment.apps/jhipster-zookeeper   1/1     1            1           33m
deployment.apps/rental               1/1     1            1           33m
deployment.apps/rental-mariadb       1/1     1            1           33m

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/book-7d67596bb9                 1         1         1       33m
replicaset.apps/book-mariadb-86c84b4dc4         1         1         1       33m
replicaset.apps/bookcatalog-7f64d657d4          1         1         1       33m
replicaset.apps/gateway-5f8bb9f8f5              1         1         1       33m
replicaset.apps/gateway-mariadb-7958dbcffc      1         1         1       33m
replicaset.apps/jhipster-kafka-85f64cc674       1         1         1       33m
replicaset.apps/jhipster-zookeeper-55755c6f65   1         1         1       33m
replicaset.apps/rental-7c868d57f4               1         1         1       33m
replicaset.apps/rental-mariadb-759fcf5f9c       1         1         1       33m

NAME                                   READY   AGE
statefulset.apps/bookcatalog-mongodb   1/1     33m
statefulset.apps/jhipster-registry     2/2     34m
```

- 쿠버네티스 클러스터에 배포가 완료되면 아래 그림과 같은 모습이다. 애플리케이션은 게이트웨이 서비스를 통해 외부 네트워크와 통신하게 된다.
<img width="1569" alt="10_11" src="https://user-images.githubusercontent.com/62231786/98325814-05f07e00-2033-11eb-854e-c814f26d1838.png">

### 오토스케일링
위 그림을 보면 레지스트리를 제외한 나머지 서비스들은 모두 1개의 파드만 생성되어있다. 만약 도서 카탈로그의 서비스의 사용량이 증가하여 인스턴스를 늘리고 싶을 때는 어떻게 해야할까? 쿠버네티스에서는 간단하게 명령어를 통해 오토스케일링을 할 수 있다.<br>

1.	오토스케일링 명령어를 수행한다.

```
$ kubectl scale deployment bookcatalog --replicas=3
```

2.	오토스케일링 결과를 확인한다.
- 파드가 총 3개로 생성된 것을 확인할 수 있다.

```
$ kubectl get pod | grep bookcatalog
bookcatalog-675954cd6b-bzpvx   1/1     Running           0          4s
bookcatalog-675954cd6b-kz8c6   1/1     Running           1          2m48s
bookcatalog-675954cd6b-vn7hk   1/1     Running           0          5s
bookcatalog-mongodb-0          1/1     Running           0          3m49s

```

### 서비스 확인
1.	게이트웨이 서비스 리소스를 조회한다.

```
$ kubectl get service/gateway
NAME      TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)          AGE
gateway   LoadBalancer   10.3.241.6   34.xx.xx.xxx   8080:31348/TCP   10m
```

2.	웹 브라우저에서 1번에서 확인한 EXTERNAL-IP 에 접속한다. 포트까지 포함하여 http://EXTERNAL-IP:8080 로 접속하면 된다. 아래와 같이 게이트웨이 화면이 뜨면 애플리케이션이 정상적으로 배포된 것이다.
<img width="1569" alt="10_12" src="https://user-images.githubusercontent.com/62231786/98325815-05f07e00-2033-11eb-9318-e9c58d057d7a.png">

### 서비스 삭제
1.	쿠버네티스 클러스터에 생성한 리소스를 모두 삭제한다. 해당 리소스를 삭제하지 않고 두면 과금일 발생할 수 있어 실습 후에는 반드시 삭제한다.

```
$ kubectl delete all --all
pod "book-7f896fc975-cbw2t" deleted
pod "book-mariadb-86c84b4dc4-jzddb" deleted
pod "bookcatalog-675954cd6b-vhkjd" deleted
pod "bookcatalog-mongodb-0" deleted
pod "gateway-7d9c88c94f-l2bq7" deleted
pod "gateway-mariadb-7958dbcffc-q4pwf" deleted
pod "jhipster-kafka-85f64cc674-lg7pt" deleted
pod "jhipster-registry-0" deleted
pod "jhipster-registry-1" deleted
pod "jhipster-zookeeper-55755c6f65-8mzz9" deleted
pod "rental-774f45d9fb-zpm8m" deleted
pod "rental-mariadb-759fcf5f9c-n9k6l" deleted
service "book" deleted
service "book-mariadb" deleted
service "bookcatalog" deleted
service "bookcatalog-mongodb" deleted
service "gateway" deleted
service "gateway-mariadb" deleted
service "jhipster-kafka" deleted
service "jhipster-registry" deleted
service "jhipster-zookeeper" deleted
service "kubernetes" deleted
service "rental" deleted
service "rental-mariadb" deleted
deployment.apps "book" deleted
deployment.apps "book-mariadb" deleted
deployment.apps "bookcatalog" deleted
deployment.apps "gateway" deleted
deployment.apps "gateway-mariadb" deleted
deployment.apps "jhipster-kafka" deleted
deployment.apps "jhipster-zookeeper" deleted
deployment.apps "rental" deleted
deployment.apps "rental-mariadb" deleted
statefulset.apps "bookcatalog-mongodb" deleted
statefulset.apps "jhipster-registry" deleted
```

2.	쿠버네티스 클러스터도 삭제한다.

```
$ gcloud container clusters delete cnaps-cluster
```
