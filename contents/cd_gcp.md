# GCP (Google Cloud Platform) 배포 환경 구성
GCP (Google Cloud Platform) 는 구글에서 서비스하고 있는 퍼블릭 클라우드 컴퓨팅 서비스이다. 신규 가입 시, 300 달러 상당의 무료 사용이 가능하여 본 실습을 진행하는데 용이하다. 회원가입 이후 콘솔에 접속하면 아래와 같다. 이제부터 애플리케이션 배포를 위해 GCP (Google Cloud Platform) 기본 환경 설정 및 쿠버네티스 생성을 해보도록 하겠다.

<img width="1570" alt="10_4" src="https://user-images.githubusercontent.com/62231786/98325797-012bca00-2033-11eb-9752-b38e24f41315.png">

## GCP 환경설정
1.	API 및 서비스 > 라이브러리 메뉴에서 'Google Container Registry API' 서비스를 검색하고 ‘사용’ 버튼을 선택한다.
<img width="945" alt="10_5" src="https://user-images.githubusercontent.com/62231786/98325800-025cf700-2033-11eb-87c1-3b7d3050c93f.png">
2.	상단의 터미널 아이콘 (빨간 박스 영역) 을 선택하면 하단에 터미널 창이 열린다. 이제부터 터미널에서 작업을 진행하면 된다.
<img width="1570" alt="10_6" src="https://user-images.githubusercontent.com/62231786/98325802-02f58d80-2033-11eb-9f18-8e399836dcd7.png">
3. project, zone 을 설정한다.

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

4. project 아이디를 환경 변수 등록한다.

```
$ export PROJECT_ID=$(gcloud config get-value core/project)
$ echo $PROJECT_ID
cnaps-project-286804
```

## GKE (Google Kubernetes Engine/구글 쿠버네티스 엔진) 생성
1.	아래 명령어로 쿠버네티스 클러스터를 생성한다.
- 약 5분 정도 소요된다.

```
$ gcloud container clusters create cnaps-cluster \
        --zone asia-northeast3-a  --machine-type n1-standard-2 --num-nodes 3 \
        --enable-autoscaling --min-nodes 1 --max-nodes 5
```

- 생성이 완료되면 콘솔의 Kubernetes Engine > 클러스터 에서 아래와 같이 확인할 수 있다. 이 화면에서도 위 옵션을 참고하여 클러스터 생성 가능하다.
<img width="1571" alt="10_7" src="https://user-images.githubusercontent.com/62231786/98325808-0426ba80-2033-11eb-9fa8-c1f70e959fec.png">
2.	생성한 클러스터를 인증한다.

```
$ gcloud container clusters get-credentials cnaps-cluster
Fetching cluster endpoint and auth data.
kubeconfig entry generated for cnaps-cluster.
```
 
3. cluster 목록 및 서비스 상태 확인
  - 아래와 같이 조회되는지 확인한다.
  
```
$ gcloud container clusters list
  NAME           LOCATION           MASTER_VERSION  MASTER_IP     MACHINE_TYPE   NODE_VERSION   NUM_NODES  STATUS
  cnaps-cluster  asia-northeast3-a  1.15.12-gke.2   34.64.164.10  n1-standard-2  1.15.12-gke.2  3          RUNNING
$ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.0.0.1     <none>        443/TCP   14m
```

이제 GCP (Google Colud Platfrom) 설정이 끝났다. 다음으로 JHipster 를 사용하여 애플리케이션을 빌드, 배포해보자.
- [jhispter를 이용한 애플리케이션 배포](/contents/jhipster_k8s.md) 
