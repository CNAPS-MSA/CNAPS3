# 지속적배포
- 쿠버네티스 아키텍처
![8_gcp](https://user-images.githubusercontent.com/62231786/94516142-1a856d80-0260-11eb-8e82-53e9b8fb2654.png)

- 쿠버네티스 배포전략
  - [Jhipster 를 활용하여 쿠베네티스 배포 준비](/contents/jhipster_k8s.md) 
  - [GCP 에 배포하기](/contents/cd_gcp.md)

## 1.배포도구들 
- 젠킨스
- 앤시블
- AWS
- Azure

## 2.샘플
- Azure DevOps.yaml

## 3.교육및실습자료
- 쿠베네티스배포실습


$ helm install cd-jenkins -f jenkins/values.yaml stable/jenkins --version 1.2.2 --wait
Error: open jenkins/values.yaml: no such file or directory
sqsp27@cloudshell:~ (jenkins-project-286914)$ cd continuous-deployment-on-kubernetes/
sqsp27@cloudshell:~/continuous-deployment-on-kubernetes (jenkins-project-286914)$ helm install cd-jenkins -f jenkins/values.yaml stable/jenkins --version 1.2.2 --wait



NAME: cd-jenkins
LAST DEPLOYED: Wed Aug 19 14:22:22 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get your 'admin' user password by running:
  printf $(kubectl get secret --namespace default cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
2. Get the Jenkins URL to visit by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd-jenkins" -o jsonpath="{.items[0].metadata.name}")
  echo http://127.0.0.1:8080
  kubectl --namespace default port-forward $POD_NAME 8080:8080

3. Login with the password from step 1 and the username: admin


For more information on running Jenkins on Kubernetes, visit:
https://cloud.google.com/solutions/jenkins-on-container-engine


