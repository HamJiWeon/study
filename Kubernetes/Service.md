## Service란?
외부로부터 들어오는 트래픽을 받아, Pod에 균등하게 분배해주는 로드밸런서 역할을 하는 기능이다.
> 실제 서비스에서 Pod에 요청을 보낼 때, 포트 포워딩이나 Pod 내로 직접 접근해서 요청을 보내지 않는다. 서비스를 통해 요청을 보내는게 일반적이다.

### Service 구조
```
┌─────────────────────────────────────────┐  
|  Computer                               |  
|  ┌───────────────────────────────────┐  |  
|  |  Kubernetes                       |  |  
|  |          ┌────────────┐           |  |  
|  |          | Deployment |           |  |  
|  |          └──────|─────┘           |  |  
|  |  ┌──────────────↓──────────────┐  |  |  
|  |  |  ReplicaSet                 |  |  |  
|  |  |  ┌─────┐  ┌─────┐  ┌─────┐  |  |  |  
|  |  |  | Pod |  | Pod |  | Pod |  |  |  |  
|  |  |  └──↑──┘  └──↑──┘  └──↑──┘  |  |  |  
|  |  └─────|────────|────────|─────┘  |  |  
|  |        └────────|────────┘        |  |  
|  |            ┌────|────┐            |  |  
|  |            | Service |            |  |  
|  |            └────↑────┘            |  |  
|  └─────────────────|─────────────────┘  |  
|                    |                    |  
└────────────────────|────────────────────┘  
                 ┌───|────┐  
                 | Client |  
                 └────────┘  
```
### Service 코드
```yaml
apiVersion: v1
kind: Service

metadata:
  name: spring-service

spec:
  type: NodePort
  selector:
    app: backend-app
  ports:
    - protocol: TCP      # Service에 접속하기 위한 프로토콜
      port: 8080         # Kubernetes 내부에서 Service에 접속하기 위한 포트 번호
      targetPort: 8080   # 매핑하기 위한 Pod의 포트 번호
      nodePort: 30000    # 외부에서 사용자들이 접급하게 될 포트 번호
```
### Service 타입
|타입|설명|
|---|---|
|`NodePort`|Kubernetes 내부에서 해당 서비스에 접속하기 위한 포트를 열고 외부에서 접속 가능하도록 한다.|
|`ClusterIP`|Kubernetes 내부에서만 통신할 수 있는 IP 주소를 부여. 외부에서는 요청할 수 없다.|
|`LoadBalancer`|외부의 로드밸런서를 활용해 외부에서 접속할 수 있도록 연결한다.|

### Service 파일 실행하기
```bash
kubectl apply -f spring-service.yaml
service/spring-service created
```

### Service 확인하기
```bash
kubectl get service
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes       ClusterIP   10.96.0.1      <none>        443/TCP          4h22m
spring-service   NodePort    10.103.8.145   <none>        8080:30000/TCP   8s
```
