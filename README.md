[1. minikube](#minikube)  
[1.1 minikube 설치](#minikube-설치)  
[1.2. minikube 실행](#minikube-실행)  
[1.3. service, pods, deployment 확인](#service,-pods,-deployment-확인)  
[1.4. gcr을 이용한 service 실행](#gcr을-이용한-service-실행)  

[2. docker](#docker)  
[2.1. 설치](#docker-설치)  

[3. github package repository](#github-package-repository)  
[3.1. docker file 생성](#docker-file-생성)
[3.2. maven install](#Maven-install)
[3.3. docker image 생성](#docker-image-생성)
[3.4. docker container 실)(#docker-container-실행)
[3.5. github access token 발행](#github-access-token-발행)
[3.6. docker tag](#docker-tag)
[3.7. docker push](#docker-push)
[3.8. change new version](#change-new-version)





# minikube
## minikube 설치
https://kubernetes.io/ko/docs/tasks/tools/install-minikube/

## minikube 실행
PowerShell (관리자 모드) 실행
```
minikube strart
```

## service, pods, deployment-확인
```
kubectl get all
```

## gcr을 이용한 service 실행
https://github.com/atropos0116/spring-boot-kubernetes-example-doc

kube.yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-boot-example
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spring-boot-example
  template:
    metadata:
      labels:
        app: spring-boot-example
    spec:
      containers:
        - name: spring-boot-example
          image: 'gcr.io/nifty-inn-274523/spring-boot-example:v1'
          ports:
            - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: spring-boot-example
  labels:
    name: spring-boot-example
spec:
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: spring-boot-example
  type: LoadBalancer
  ```
  
  # docker
  ## docker 설치
  https://www.docker.com/products/docker-desktop
  
  
  # github package repository
  sample source : https://github.com/atropos0116/container-demo
   
  # docker file 생성
  Dockerfile
  ```
  FROM openjdk:8-jdk-alpine
  COPY target/container-demo.jar container-demo.jar
  EXPOSE 8080
  ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "container-demo.jar"]
  ```
  
  # maven install
  ```
  ./mvnw clean install
  ```
  
  # docker image 생성
  ```
  cd {path}/container-demo
  docker build . -t coatiner-demo:v1
  ```
  # docker container 실행
  ```
  docker run container-demo:v1 -p 8080:8080
  ```
  
  # github access token 발행
  1. github 로그인
  2. user status > Settings
  3. Developer settings > Personal access token
  4. generate new token
  5. 권한 체크
     - repo 
     - write:packages
     - read:packages
  6. Generate Token
  7. Token 복사
  
  # access token 을 통한 인증 로그인
    1. GH_TOKEN.txt 생성
    2. 복사한 access token 붙여넣기
    3. access token을 통한 로그인
   ```
   type ~/GH_TOKEN.txt | docker login docker.pkg.github.com -u {user} --password-stdin
   ```

  # docker tag 
  ```
  docker tag container-demo:v1 docker.pkg.github.com/atropos0116/container-demo/container-demo:v1
  ```
  
  # docker push
  ```
  docker push container-demo:v1 docker.pkg.github.com/atropos0116/container-demo/container-demo:v1
  ```
  
  # change new version 
  ```
  cd {path}/container-demo
  docker build . -t coatiner-demo:v2
  docker tag container-demo:v2 docker.pkg.github.com/atropos0116/container-demo/container-demo:v2
  docker push container-demo:v2 docker.pkg.github.com/atropos0116/container-demo/container-demo:v2
  ```
  
  
  
  
  



 
