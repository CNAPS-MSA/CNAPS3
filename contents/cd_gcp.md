# GCP 에 배포하기

## GCP 환경설정
### 1. project 생성
  - Home > Dashboard > CREATE PROJECT 를 선택하고 'CNAPS Project' 프로젝트를 생성한다.
  ![cd_gcp_01](https://user-images.githubusercontent.com/62231786/90473310-8d79cf80-e15d-11ea-8ed2-025cba3165da.png)

### 2. API 설정
  - APIs & Services > Library 에서 'Google Container Registry API' 서비를 활성화한다.
  ![cd_gcp_02](https://user-images.githubusercontent.com/62231786/90473185-4095f900-e15d-11ea-873e-6358e73fd4e2.png)
  
### 3. Cloud Shell 실행
  - 상단의 콘솔 아이콘을 선택한다.
  ![cd_gcp_03](https://user-images.githubusercontent.com/62231786/90473507-0bd67180-e15e-11ea-81ee-e71ffc154ea7.png)

### 4. Cloud 환경 설정
  - project, zone 을 설정한다.
```
$ gcloud config set project [PROJECT_ID]    // 1. 에서 생성한 프로젝트 아이디로 설정
$ gcloud config set compute/zone asia-northeast3-a
$ gcloud config list    // 설정 확인
[compute]
gce_metadata_read_timeout_sec = 5
zone = asia-northeast3-a
[core]
project = cnaps-project-286804
```
  - project 아이디를 환경 변수 등록한다.
```
$ export PROJECT_ID=$(gcloud config get-value core/project)
$ echo $PROJECT_ID
cnaps-project-286804
```

## GKE 설정
### 1. Kubernetes Cluster 생성
  - 약 5분 정도 소요된다.
  - 콘솔에서 Google Kubernetes Engine > Clusters 에서도 생성할 수 있다.
```
$ gcloud container clusters create cnaps-cluster \
        --zone asia-northeast3-a  --machine-type n1-standard-2 --num-nodes 3 \
        --enable-autoscaling --min-nodes 1 --max-nodes 5
```
### 2. kubeconfig 생성
  - kubectl 명령어 실행을 하기 위해서 해당 작업이 필요하다.
 ```
$ gcloud container clusters get-credentials cnaps-cluster --zone asia-northeast3-a
Fetching cluster endpoint and auth data.
kubeconfig entry generated for cnaps-cluster.
 ```
### 3. cluster 목록 및 서비스 상태 확인
  - 아래와 같이 조회되는지 확인한다.
 ```
$ gcloud container clusters list
  NAME           LOCATION           MASTER_VERSION  MASTER_IP     MACHINE_TYPE   NODE_VERSION   NUM_NODES  STATUS
  cnaps-cluster  asia-northeast3-a  1.15.12-gke.2   34.64.164.10  n1-standard-2  1.15.12-gke.2  3          RUNNING
$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.0.0.1     <none>        443/TCP   14m
 ```

## 컨테이너라이징
### 1. git clone
  - git 에서 작업할 소스를 내려받는다.
```
$ mkdir cnaps && cd cnaps
$ git clone https://github.com/CNAPS-MSA/k8s.git
$ git clone https://github.com/CNAPS-MSA/gateway.git
$ git clone https://github.com/CNAPS-MSA/book.git
$ git clone https://github.com/CNAPS-MSA/bookCatalog.git
$ git clone https://github.com/CNAPS-MSA/rental.git
```
### 2. docker image 생성 및 Registry 등록
  - gateway 외 book, bookCatalog, rental 도 동일하게 아래 작업을 수행한다.
```
$ cd gateway
$ ./mvnw package -Pprod -DskipTests jib:dockerBuild -Dimage=gcr.io/$PROJECT_ID/gateway:latest
$ cd book
$ ./mvnw package -Pprod -DskipTests jib:dockerBuild -Dimage=gcr.io/$PROJECT_ID/book:latest
$ cd bookCatalog
$ ./mvnw package -Pprod -DskipTests jib:dockerBuild -Dimage=gcr.io/$PROJECT_ID/bookcatalog:latest
$ cd rental
$ ./mvnw package -Pprod -DskipTests jib:dockerBuild -Dimage=gcr.io/$PROJECT_ID/rental:latest
```
 - 생성된 이미지를 확인한다.
```
$ docker images
REPOSITORY                                TAG                 IMAGE ID            CREATED             SIZE
gcr.io/cnaps-project-286804/book          latest              45c7d41a2017        3 minutes ago       331MB
gcr.io/cnaps-project-286804/bookcatalog   latest              052ddd2f9852        4 minutes ago       323MB
gcr.io/cnaps-project-286804/rental        latest              c9cac9c1040c        6 minutes ago       332MB
gcr.io/cnaps-project-286804/gateway       latest              2a7988ddf51a        28 minutes ago      339MB
```
 - GCP Container Registry 에 등록한다.
```
$ docker push gcr.io/cnaps-project-286804/gateway:latest
$ docker push gcr.io/cnaps-project-286804/book
$ docker push gcr.io/cnaps-project-286804/bookcatalog
$ docker push gcr.io/cnaps-project-286804/rental
```
  - 콘솔에서 Container Registry > images 에서 등록된 이미지들을 확인할 수 있다.
![cd_gcp_04](https://user-images.githubusercontent.com/62231786/90486926-ad68bd80-e174-11ea-9333-b42c471db183.png)

## 배포하기
### 1. deployment.yaml 파일 수정
  - 이미지명 내 프로젝트 아이디 부분을 생성한 프로젝트로 수정한다.
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
###2. 배포
  - 아래의 명령어를 수행한다.
```
$ ./kubectl-apply.sh -f
```
###3. 배포 서비스 확인
  - 모든 서비스가 정상적으로 기동되면 아래와 같다.
  - service/gateway 서비스에 접속하여 확인할 수 있다.
    - http://{EXTERNAL-IP}:8080
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
###4. 배포 서비스 삭제
  - 모든 서비스를 중단한다.
```
$ kubectl delete all --all
```