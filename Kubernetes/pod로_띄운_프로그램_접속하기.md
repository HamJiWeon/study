### 1. Pod의 내부로 들어가서 접근하기
```bash
kubectl exec -it nginx-pod -- bash
root@nginx-pod:/# 
```

```bash
root@nginx-pod:/# curl localhost:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
이렇게 bash로 들어와서 localhost:80를 접속해보면 응답이 잘 날라오는 것을 볼 수 있다.  
쿠버네티스에서는 이 pod 내부의 네트워크를 container와 같이 공유해서 사용하기 때문에 이렇게 정상적으로 응답이 날라온 것이다.

### 2. Pod의 내부 네트워크를 외부에서도 접속할 수 있도록 포트 포워딩을 활용하기
```bash
kubectl port-forward pod/nginx-pod 80:80
Unable to listen on port 80: Listeners failed to create with the following errors: [unable to create listener: Error listen tcp4 127.0.0.1:80: bind: permission denied unable to create listener: Error listen tcp6 [::1]:80: bind: permission denied]
error: unable to listen on any of the requested ports: [{80 80}]
```
여기서 80:80으로 포트포워딩을 했을 때 권한이 필요하다고 에러를 띄운다. 그래서 sudo를 붙여서 사용하면
```bash
sudo kubectl port-forward pod/nginx-pod 80:80
Password:
Forwarding from 127.0.0.1:80 -> 80
Forwarding from [::1]:80 -> 80
```

<img width="530" height="260" alt="image" src="https://github.com/user-attachments/assets/6664292e-4f1a-4876-a0cc-69d51d005ce8" />

접속이 잘 되는 것을 볼 수 있다.
