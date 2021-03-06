

# 지속적 통합과 지속적 배포 (CI/CD)

지속적 통합 (CI) 이란 애플리케이션 소스 변경 사항들이 지속적으로 병합, 빌드, 테스트되는 것을 의미한다. 그리고 지속적 배포 (CD) 란 지속적 통합을 통해 준비된 애플리케이션이 테스트 및 프로덕션 환경에 자동으로 배포되는 것을 의미한다.<br>
애플리케이션 개발을 한다고 생각해보자. 개발자들은 작업한 소스 코드를 형상 관리에 커밋, 푸시하게 된다. 그 후에 소스 변경 사항들은 빌드, 테스트 과정을 거쳐 테스트 또는 프로덕션 환경으로 배포되게 된다. 이런 일련의 작업들을 자동화한 것이 지속적 통합과 지속적 배포 (CI/CD) 라고 할 수 있다.<br>

<img src="https://user-images.githubusercontent.com/62231786/98325793-fec97000-2032-11eb-9401-4d10cbd5a2db.png" /> <br><br>
위 그림은 샘플 프로젝트의 애플리케이션 빌드/배포 프로세스를 도식화한 것이다. 이는 일반적인 클라우드 환경에서 의 빌드/배포 프로세스이다. 각 단계별 상세 태스크를 정리하면 다음과 같다.<br>

|프랙티스|주요 활동|태스크|작업 도구|
|------|------|---|---|
|지속적통합|소스 코드 확보 및 통합|- 소스코드 커밋<br>- 소스코드 풀<br>|형상관리<br>예) 깃허브|
| |빌드, 컨테이너|- 소스코드 빌드<br>- 도커 이미지 생성<br>|빌드 도구<br>예) 도커, 코드빌드|
| |컨테이너 레지스트리 등록|- 레지스트리 푸쉬|컨테이너 레지스트리 서비스<br>예)도커허브, ECR, ACR, GCR|
|지속적배포|컨테이너 이미지 확보|- 컨테이너 이미지 풀|컨테이너 레지스트리 서비스<br>예)도커허브, ECR, ACR, GCR|
| |배포|- 배포 수행|배포 도구<br>예) 젠킨스, AWS CodePipeline, Azure DevOps Pipeline, GCP Code Build|

가장 먼저, 형상관리에 통합된 애플리케이션 소스코드를 빌드하게 된다. 여기서 컨테이너 기술을 적용하지 않는다면 빌드 단계에서 생성된 war, jar 등 의 결과물을 바로 배포하면 된다. 하지만 컨테이너 기술을 적용한다면 도커 이미지를 생성하고 이를 관리하기 위해 컨테이너 레지스트리에 등록한다. 다음으로 컨테이너 레지스트리에 등록한 도커 이미지를 사용하여 쿠버네티스 환경에 애플리케이션을 배포한다.<br>
이런 모든 과정 또는 일부과정을 빌드/배포 도구를 사용하여 파이프라인을 구성하여 자동화할 수 도 있다. 대표적인 도구로 젠킨스가 있으며 각 클라우드 벤더마다 빌드/배포를 지원하는 도구들이 있는데 대표적으로 AWS CodePipeline, Azure DevOps Pipeline, GCP Cloud Build 가 있다. 

# jhispter를 이용한 애플리케이션 배포
  - [GCP (Google Cloud Platform) 배포 환경 구성](/contents/cd_gcp.md)
  - [jhispter를 이용한 애플리케이션 배포](/contents/jhipster_k8s.md) 