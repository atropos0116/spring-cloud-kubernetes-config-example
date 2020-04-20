[1. Minikube](#Minikube)
[1.1 Minikube 설치](#Minikube-설치)
[1.2. Minikube 실행](#Minikube-실행)
[1.3. Service, Pods, Deployment 확인](#Service,-Pods,-Deployment-확인)
[1.4. Running Container in Google Cloud Platform](#Running-Container-in-Google-Cloud-Platform)


# Minikube
## Minikube 설치
https://kubernetes.io/ko/docs/tasks/tools/install-minikube/

## Minikube 실행
PowerShell (관리자 모드) 실행
```
minikube strart
```

## Service, Pods, Deployment 확인
```
kubectl get all
```

## Running Container in Google Cloud Platform
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
  
  



 
