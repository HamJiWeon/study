### 서버 3개 띄어보기
서버를 세개를 띄어보려면
```yaml
apiVersion: v1
kind: Pod

metadata:
  name: spring-pod-1

spec:
  containers:
    - name: spring-container
      image: spring-server
      ports:
        - containerPort: 8080
      imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Pod

metadata:
  name: spring-pod-2

spec:
  containers:
    - name: spring-container
      image: spring-server
      ports:
        - containerPort: 8080
      imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Pod

metadata:
  name: spring-pod-3

spec:
  containers:
    - name: spring-container
      image: spring-server
      ports:
        - containerPort: 8080
      imagePullPolicy: IfNotPresent
```

이렇게 수동으로 띄울 수 있다. 근데 만약 100개를 띄어야 한다면 너무 노가다 작업이 이어진다. 그래서 **Deployment**를 활용한다.

**Deployment란?**  
Pod를 묶음으로 쉽게 관리할 수 있는 기능을 말한다.

**Deployment의 장점**
- pod의 수를 지정하는대로 여러 개의 pod를 쉽게 생성할 수 있다.
  - ex. pod를 100개 생성하라고 시키면 deployment가 알아서 pod를 100개 생성해준다.
- pod가 비정상적으로 종료된 경우, 알아서 새로 pod를 생성해 pod의 수를 유지한다.
- 동일한 구성의 여러 pod를 일괄적으로 일시 중지, 삭제, 업데이트 하기가 쉽다.
  - ex. deployment를 활용하면 '100개의 pod로 띄어져있는 결제 서버'를 한 번에 일시 중지/삭제/업데이트 하는 것이 쉽다.
 
**Deployment의 구조**
> **Computer** > **Kubernetes** > **Deployment** → **ReplicaSet** → **Pods**
- Deployment가 ReplicaSet을 관리하고, ReplicaSet이 Pods를 관리하는 구조이다.

### Deployment 적용해보기
```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: spring-deployment

# Deployment 세부 정보
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend-app

  # 배포할 Pod 정의
  template:
    metadata:
      labels:
        app: backend-app
    spec:
      containers:
        - name: spring-container
          image: spring-server
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
```

### 실행하기
```bash
kubectl apply -f spring-deployment.yaml 
deployment.apps/spring-deployment created
```

### Pods replicas의 개수대로 잘 생성됐는지 확인
```bash
kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
spring-deployment-7b5d4d9965-8d2fn   1/1     Running   0          9s
spring-deployment-7b5d4d9965-f8tqn   1/1     Running   0          9s
spring-deployment-7b5d4d9965-jxz7h   1/1     Running   0          9s
```

### Deployment 확인하기
```bash
kubectl get deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
spring-deployment   3/3     3            3           17s
```

### ReplicaSet 확인하기
```bash
kubectl get replicaset
NAME                           DESIRED   CURRENT   READY   AGE
spring-deployment-7b5d4d9965   3         3         3       3m8s
```

백엔드 서버 3개를 각각의 pod에 띄었다. 실제 요청을 보낼 때는 각 서버에 균등하게 트래픽이 분배되어야 한다. 그런데 사용자보고 여러 백엔드 서버에 알아서 균등하게 요청을 하라고 시킬 수도는 없다.
따라서 pod 앞단에 알아서 여러 pod에 균등하게 요청을 해줄 무언가가 필요하다. Kubernetes에서는 Service가 여러 pod에 균등하게 요청을 분배해주는 역할을 한다.
