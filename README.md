[1. docker](#docker)  
[1.1. 설치](#docker-설치) 

[2. minikube](#minikube)  
[2.1 minikube 설치](#minikube-설치)  
[2.2. minikube 실행](#minikube-실행)  
[2.3. service, pods, deployment 확인](#service,-pods,-deployment-확인)  
[2.4. gcr을 이용한 service 실행](#gcr을-이용한-service-실행)  

[3. github package repository](#github-package-repository)  
[3.1. docker file 생성](#docker-file-생성)  
[3.2. maven install](#Maven-install)  
[3.3. docker image 생성](#docker-image-생성)  
[3.4. docker container 실행](#docker-container-실행)  
[3.5. github access token 발행](#github-access-token-발행)  
[3.6. docker tag](#docker-tag)  
[3.7. docker push](#docker-push)  
[3.8. change new version](#change-new-version)  

[4. skaffold](#skaffold)  
[4.1. skaffold 설치](#skaffold-설치)  
[4.2. skaffold yml 생성](#skaffold-yml-생성)  
[4.3. skaffold 실행](#skaffold-실행)  
[4.4. resource 변경 확인](#resource-변경-확인)  

[5. discovery client](#discovery-client)  
[5.1. discovery client dependency](#discovery-client-dependency)  
[5.2. discovery client annotation](#discovery-client-annotation)  
[5.3. users-service 호출](#users-service-호출)  
[5.4. discovery server](#discovery-server)  
[5.4.1. discovery server skaffold run](#discovery-server-skaffold-run)  
[5.4.2. discovery clinet skaffold run](#discovery-client-skaffold-run)

[Z. Appendex](#Appendex)

 # docker
 ## docker 설치
 https://www.docker.com/products/docker-desktop
  

# minikube
## minikube 설치
https://kubernetes.io/ko/docs/tasks/tools/install-minikube/

## minikube 실행
PowerShell (관리자 모드) 실행
```
minikube strart -driver=docker
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
  
  # skaffold
  ## skaffold-설치 (windows)
  ```
  choco install skaffold
  ```
  ## skaffold yml 생성
  
  skaffold.yaml
  ```
apiVersion: skaffold/v1
kind: Config
build:
  artifacts:
    - image: config-example
      docker:
        dockerfile: Dockerfile
  local: {}

deploy:
  kubectl:
    manifests:
      - config.yaml
      - deploy.yaml

  ```
  
  deploy.yaml
  ```
  kind: Service
apiVersion: v1
metadata:
  name: config-example
spec:
  selector:
    app: config-example
  ports:
    - protocol: TCP
      port: 8080
      nodePort: 30083
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: config-example
spec:
  selector:
    matchLabels:
      app: config-example
  replicas: 1
  template:
    metadata:
      labels:
        app: config-example
    spec:
      containers:
        - name: config-example
          image: config-example
          imagePullPolicy: Never
          ports:
            - containerPort: 8080
  ```
  
  config.yaml
  ```
  apiVersion: v1
kind: ConfigMap
metadata:
  name: config-example
  namespace: default
data:
  application.yml: |-
    welcome:
      message: Hello Spring Cloud from application.yml
    message: Hi from application.yml
  ```
  
  application.yml
  ```
  logging.level.org.springframework.cloud.kubernetes: DEBUG
spring.application.name: config-example

management:
  endpoints:
    restart:
      enabled: true
    web:
      exposure:
        include: info,refresh,keepalive,health
  ```
  
  bootstrap.yml
  ```
  spring:
  cloud:
    kubernetes:
      config:
        enabled: true
        name: config-example
        namepsace: default
        sources:
          - namespace: default
            name: config-example
      reload:
        enabled: true
        mode: event
        strategy: refresh
  ```
  
  ## skaffold 실행
  ```
  skaffold dev
  ```
    
  ## resource 변경 확인
  local 소스 변경 후 cmd 확인 
  -> image 재 생성 및 빌드 후, 실행됨
  
  # discovery client
  
  ## discovery client dependency
  spring-cloud-kubernetes-config-example/pom.xml 
  ```
		<dependency>
   <groupId>org.springframework.cloud</groupId>
	 		<artifactId>spring-cloud-starter-kubernetes</artifactId>
		 	<version>1.1.0.RELEASE</version>
		</dependency>
  ```
  
  ## discovery client annotation
  SpringCloudKubernetesConfigExampleApplication.java
  ```
  @EnableDiscoveryClient <----- add
  @EnableScheduling
  @SpringBootApplication
  public class SpringCloudKubernetesConfigExampleApplication {
  ...
  
  
  	@Bean
  	public RestTemplate restTemplate() {
		    return new RestTemplate();
	  }
  ```
  
  ## users-service 호출
  SchedulerComponent.java
  ```
  public void schedule() {
     ...
  
     String url = "http://users-service:8080/users";
     ResponseEntity<String> responseEntity = restTemplate.getForEntity(url, String.class);
     System.out.println("Calling via Discovery Client... " + responseEntity.getBody());
  }
  ```
  
  ## discovery server
  ```
  git clone https://github.com/atropos0116/spring-cloud-kubernetes-server-example.git
  ```
  
  ### discovery server skaffold run
  ```
  .../spring-cloud-kubernetes-server-example>skaffold run
  ```
  
  ### discovery client skaffold dev
  ```
  .../spring-cloud-kubernetes-config-example>skaffold dev
  ```

  # Appendex
  ```
  Can't read configMap with name: [config-example] in namespace:[default]. Ignoring... 
  ```
  
  Solution :
      https://devopscube.com/kubernetes-api-access-service-account/

 
