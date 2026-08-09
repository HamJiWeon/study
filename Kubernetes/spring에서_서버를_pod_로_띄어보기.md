우선 연습이기 때문에 간단한 get으로 controller를 만든다.
```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AppController {

    @GetMapping("/")
    public String home() {
        return "Hello, World";
    }
}
```

그 다음엔 이미지를 만들기 위해 Dockerfile을 작성한다.
```Dockerfile
FROM eclipse-temurin:17-jdk AS build

COPY build/libs/*SNAPSHOT.jar app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

그리고나서 `./gradlew clean build`를 써서 프로젝트의 기존 빌드 결과물을 깨끗이 지운 후, 소스코드를 새로 컴파일하고 테스트하여 실행 파일로 묶어준다.  

```bash
spring-server:latest
```
그러면 `BUILD SUCCESSFUL`이 되는데, `docker build -t spring-server .`로 이미지를 빌드하면 위와 같이 리스트에 잘 뜬 것을 볼 수 있다.

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: spring-pod

spec:
  containers:
    - name: spring-container
      image: spring-server
      ports:
        - containerPort: 8080
```
그 다음 manifest 파일을 작성하고, `kubectl apply -f spring-pod.yaml`로 pod를 띄어준다.

```bash
kubectl get pods
NAME         READY   STATUS         RESTARTS   AGE
spring-pod   0/1     ErrImagePull   0          12s

kubectl get pods
NAME         READY   STATUS             RESTARTS   AGE
spring-pod   0/1     ImagePullBackOff   0          26s
```

pod가 잘 생성이 되는데 pods를 띄어보면 `STATUS`가 `ErrImagePull`이라고 뜨는 것을 볼 수 있다. 그래서 한 번 더 `get`을 해보면 `STATUS`가 `ImagePullBackOff`로 바뀐 것을 볼 수 있다.  

우선 이게 뭔지 알기 전에 **Image Pull Policy** 를 알아야 한다.

### Image Pull Policy란?
이미지 풀 정책(Image Pull Policy)이란 쿠버네티스가 yaml파일을 읽어들여 pod를 생성할 때, 이미지를 어떻게 pull을 받아올 것인지에 대한 정책을 의미한다.

|정책|설명|
|---|---|
|`Always`|로컬에서 이미지를 가져오지 않고, 무조건 레지스트리(= Dockerhub, ECR과 같은 원격 이미지 저장소)에서 가져온다.|
|`IfNotPresent`|로컬에서 이미지를 먼저 가져온다. 만약 로컬에 이미지가 없는 경우에만 레지스트리에서 가져온다.|
|`Never`|로컬에서만 이미지를 가져온다.|

그래서 기존 manifest 파일을 다시 살펴보면 Image Pull Policy를 따로 설정하지 않아 이미지 태그가 `latest`여서 Default 값으로 `Always`로 설정된 것이다.
> 이미지의 태그가 `latest`가 아닌 경우에는 `IfNotPresent`로 설정된다.

### 수정하기
1. `yaml`에 `imagePullPolicy` 추가하기
```yaml
apiVersion: v1
kind: Pod

metadata:
  name: spring-pod

spec:
  containers:
    - name: spring-container
      image: spring-server
      ports:
        - containerPort: 8080
      imagePullPolicy: ifNotPresent
```

2. 기존 pod 삭제하기
```bash
kubectl delete pod spring-pod
```

3. 다시 pod 생성하기
```bash
kubectl apply -f spring-pod.yaml 
```

### 결과
```bash
NAME         READY   STATUS    RESTARTS   AGE
spring-pod   1/1     Running   0          7s
```
1. pod 내부 접속
```bash
kubectl exec -it spring-pod -- bash
root@spring-pod:/# curl localhost:8080
Hello, World
```
2. 포트포워딩 접속
```bash
kubectl port-forward pod/spring-pod 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080
```

잘 생성된 것을 볼 수 있다.
