# Jhipster를 활용한 쿠버네티스 배포

Jhipster 는 쿠버네티스 설정 파일을 자동 생성해주고 이를 기반으로 쉽게 배포가 가능하다.

##쿠버네티스 설정 파일 생성
###1. 프로젝트 root directory 로 이동한다.
```
cd ~/
```
 - 현재 총 3개의 Microservice 와 gateway 로 구성되어있다.
```
book
bookCatalog
gateway
rental
```

###2. 설정 파일용 폴더를 생성 후, 해당 폴더로 이동한다.
```
mkdir k8s && cd k8s
```

###3. 쿠버네티스 설정파일을 생성한다.

  - jhipster 명령어를 실행한다.
```
jhipster kubernetes
```
  - 아래 스크립트를 참고하여 옵션을 선택한다.
```
INFO! Using JHipster version installed globally
INFO! Executing jhipster:kubernetes
⎈ Welcome to the JHipster Kubernetes Generator ⎈
Files will be generated in folder: /Users/ejchoi/git/cnaps/k8s
✔ Docker is installed
WARNING! kubectl 1.2 or later is not installed on your computer.
Make sure you have Kubernetes installed. Read https://kubernetes.io/docs/setup/

? Which *type* of application would you like to deploy?
  Monolithic application
❯ Microservice application
? Enter the root directory where your gateway(s) and microservices are located (../)
4 applications found at /Users/ejchoi/git/cnaps/

? Which applications do you want to include in your configuration?
 ◉ book
 ◉ bookCatalog
 ◉ gateway
❯◉ rental
? Do you want to setup monitoring for your applications ? (Use arrow keys)
❯ No
  Yes, for logs and metrics with the JHipster Console (based on ELK and Zipkin)
  Yes, for metrics only with Prometheus
? Which applications do you want to use with clustered databases (only available with MongoDB and Couchbase)?
❯◉ bookCatalog
? Which applications do you want to use with clustered databases (only available with MongoDB and Couchbase)? bookCatalog
JHipster registry detected as the service discovery and configuration provider used by your apps
? Enter the admin password used to secure the JHipster Registry (admin)
? What should we use for the Kubernetes namespace? (default)
? What should we use for the base Docker repository name? gcr.io/cnaps-project
? What command should we use for push Docker image to repository? (docker push)
? Do you want to enable Istio? (Use arrow keys)
❯ No
  Yes
? Choose the Kubernetes service type for your edge services (Use arrow keys)
❯ LoadBalancer - Let a Kubernetes cloud provider automatically assign an IP
  NodePort - expose the services to a random port (30000 - 32767) on all cluster nodes
  Ingress - create ingresses for your services. Requires a running ingress controller
? Do you want to use dynamic storage provisioning for your stateful services? (Use arrow keys)
❯ No
  Yes
```
  - 아래의 파일들이 자동생성된다.
```
   create kubectl-apply.sh
   create book-k8s/book-deployment.yml
   create book-k8s/book-service.yml
   create book-k8s/book-mariadb.yml
   create bookcatalog-k8s/bookcatalog-deployment.yml
   create bookcatalog-k8s/bookcatalog-service.yml
   create bookcatalog-k8s/bookcatalog-mongodb.yml
   create gateway-k8s/gateway-deployment.yml
   create gateway-k8s/gateway-service.yml
   create gateway-k8s/gateway-mariadb.yml
   create rental-k8s/rental-deployment.yml
   create rental-k8s/rental-service.yml
   create rental-k8s/rental-mariadb.yml
   create K8S-README.md
   create messagebroker-k8s/kafka.yml
   create registry-k8s/jhipster-registry.yml
   create registry-k8s/application-configmap.yml
   create kustomization.yml
   create skaffold.yml
```
## 쿠버네티스 설정 파일 수정
자동 생성된 설정 파일에 일부 수정이 필요하다.
###1. Maria DB 설정 파일 수정
  - containers 내 args 항목을 추가한다. (해당 항목을 추가하지 않으면 한글 지원이 되지 않는다.)
    - book-k8s/book-mariadb.yml
    - gateway-k8s/gateway-mariadb.yml
    - rental-k8s/rental-mariadb.yml
```
<변경 전 - 예시 : book>
...
      containers:
        - name: mariadb
          image: mariadb:10.5.3
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: book-mariadb
                  key: mariadb-root-password
            - name: MYSQL_DATABASE
              value: book
...
<변경 후>
...
      containers:
        - name: mariadb
          image: mariadb:10.5.3
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: book-mariadb
                  key: mariadb-root-password
            - name: MYSQL_DATABASE
              value: book
          args:
            - --lower_case_table_names=1
            - --skip-ssl
            - --character_set_server=utf8mb4
            - --explicit_defaults_for_timestamp
...
```

###2. Mongo DB 설정 파일 수정
  - replicas 가 3으로 기본 설정되는데 이를 1로 수정한다.
    - bookcatalog-k8s/bookcatalog-mongodb.yml
```
<변경 전>
...
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: bookcatalog-mongodb
  namespace: default
spec:
  serviceName: bookcatalog-mongodb
  replicas: 3
  selector:
    matchLabels:
      app: bookcatalog-mongodb
...
<변경 후>
...
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: bookcatalog-mongodb
  namespace: default
spec:
  serviceName: bookcatalog-mongodb
  replicas: 1
  selector:
    matchLabels:
      app: bookcatalog-mongodb
...
``` 
###3. mongo db 접속 정보를 수정
  - bookcatalog-k8s/bookcatalog-deployment.yml
```
<변경 전>
...
  - name: SPRING_DATA_MONGODB_URI
    value: 'mongodb://bookcatalog-mongodb-0.bookcatalog-mongodb.default:27017,bookcatalog-mongodb-1.bookcatalog-mongodb.default:27017,bookcatalog-mongodb-2.bookcatalog-mongodb.default:27017'
...
<변경 후>
...
  - name: SPRING_DATA_MONGODB_URI
    value: 'mongodb://bookcatalog-mongodb-0.bookcatalog-mongodb.default:27017'
...
```
###4. 쿠베네티스 배포 실행 파일 수정
  - 실행 순서를 변경한다.
```
<변경 전>
...
default() {
    suffix=k8s
    kubectl apply -f registry-${suffix}/
    kubectl apply -f book-${suffix}/
    kubectl apply -f bookcatalog-${suffix}/
    kubectl apply -f gateway-${suffix}/
    kubectl apply -f rental-${suffix}/
    kubectl apply -f messagebroker-${suffix}/

}
...
<변경 후>
...
default() {
    suffix=k8s
    kubectl apply -f registry-${suffix}/
    kubectl apply -f messagebroker-${suffix}/
    kubectl apply -f gateway-${suffix}/
    kubectl apply -f book-${suffix}/
    kubectl apply -f bookcatalog-${suffix}/
    kubectl apply -f rental-${suffix}/

}
...
```

여기까지 완료되면 다음 단계 [GCP 에 배포하기](/contents/cd_gcp.md) 를 참고한다.