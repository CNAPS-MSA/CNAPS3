# 지속적통합
- 저장소 Push
- build수행 
 - 저장소 Pull 
 - 컴파일, 테스트 , 바이너리 생성 
    메이븐, 그레들 
 - 컨테이너 구성 
    Dockerfile
 - 레지스터리 등록 
    도커레지스터리, 각 벤더의 레지스터리 서비스
- 배포 수행
 - 레지스터리 Pull
 - 배포 구성 파일 수행 
    
 
 
-----------------------------------------------



- 지속적통합이란? 

- 파이프라인
  - build
    - 컨테이너 base Image 구성
    
    빌드는 다양한 jar.war등 다양한 artifact를 생성할 수 있으나 Cloud Infra를 컨테이너 플랫폼으로 선정한 경우, 컨테이너 이미지로 구성해야 한다.
    따라서 빌드 후 Docker Container Image를 생성한다.
    
    - push Registery 
    
    생성된 이미지를 배포시 사용하기 위해 Image Registery에 push 한다

## 1.지속적통합도구들
- 메이븐 & 그래들
- 젠킨스
- 도커
- AWS도구
- Azure

## 2.샘플
- DockerFile



