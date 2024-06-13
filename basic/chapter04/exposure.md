# 애플리케이션 노출

## 간단한 방법

### Port-Forward

- 포트포워딩은 확인 용도로 사용하기 편하지만, 영속적으로 사용하기는 어렵다.
- 포트 포워딩할 파드인 아래 내용을 deploy한다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fwd-chk-hn
spec:
  containers:
  - name: chk-hn 
    image: sysnet4admin/chk-hn
```

- 아래와 같이 명령어를 치면 포트 포워딩이 진행된다.

```shell
$ kubectl port-forward fwd-chk-hn 80:80
Forwarding from 127.0.0.1:80 -> 80
Handling connection for 80
```

- 아래와 같이 curl 명령어로 접근해보자.

```shell
$ curl 127.0.0.1
fwd-chk-hn
```

- 아래와 같이 명령어를 작성하면 모든 주소에 대해서 접근이 허용된다.
- 
```shell
$ kubectl port-forward --address 0.0.0.0 fwd-chk-hn 80:80
```

### HostPost

- 어떤 노드에 디플로이됐는지 파악해야 하고 그 노드에 hostPort(아래의 경우 8080)로 접근해야 한다.
- 현실적으로 사용하기 어려운 방식이다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hp-chk-hn 
spec:
  containers:
  - name: chk-hn
    image: sysnet4admin/chk-hn
    ports:
    - containerPort: 80
      hostPort: 8080
```

### HostNetwork

- `spec.hostNetwork`에 `true`를 주면 된다.
- 사용자는 어떤 노드에 디플로이 됐는지 알아야 하고 그 노드의 80으로 접속해야 한다.
- 현실적으로 사용하기 어렵다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hnet-chk-hn
spec:
  hostNetwork: true
  containers:
    - name: chk-hn
      image: sysnet4admin/chk-hn 
```

<br/>

## 노드포트 NodePort

- `spec.type`을 NodePort로 설정해주면 된다.
- 포트에 대한 설명 (노드포트 <-> 서비스 <-> 파드)
  - `spec.ports.targetPort`: 파드와 서비스가 연결되는 포트번호이다.
  - `spec.ports.port`: 해당 포트 번호로 노드 포트와 서비스가 연결된다.
  - `spec.ports.nodePort`: 실제로 클러스터 외부에 노출되는 포트이다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: np-nginx 
spec:
  selector:
    app: deploy-nginx  
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30000 #option 
  type: NodePort
```

<br/>

## 로드밸런서 LoadBalancer

- selector를 deployment에 있는 pod와 일치시켜줘야 한다.
- 포트에 대한 설명 (서비스 <-> 파드)
  - `spec.ports.port`: 외부에서 서비스에 접근할 때 사용하는 포트
  - `spec.ports.targetPort`: 파드와 서비스가 연결되는 포트번호

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lb-nginx 
spec:
  selector:
    app: deploy-nginx  
  ports:
    - name: http
      port: 80
      targetPort: 80 
  type: LoadBalancer
```

<br/>

## 외부이름 ExternalName

- 도메인 이름을 위한 서비스 객체다.
- `spec.type`을 ExternalName으로 설정한다.
- `spec.externalName`은 외부 도메인 주소를 의미한다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ex-url-1 
  namespace: default
spec:
  type: ExternalName
  externalName: sysnet4admin.github.io
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ex-url-2 
  namespace: default
spec:
  type: ExternalName
  externalName: k8s-edu.github.io
```

- 클러스터 내부에서 외부이름이 잘 동작하는지를 확인하기 위해 net-tools 파드 하나를 띄워보자.

```shell
$ kubectl run net --image=sysnet4admin/net-tools-ifn
```

- 이제 net-tools에 접속해서(쿠버네티스 클러스터 안에서) nslookup 명령어를 실행해보자.

```shell
$ kubectl exec net -it -- /bin/bash
[root@net /]# nslookup ex-url-1
Server:         10.96.0.10
Address:        10.96.0.10#53

ex-url-1.default.svc.cluster.local      canonical name = sysnet4admin.github.io.
Name:   sysnet4admin.github.io
Address: 185.199.108.153
Name:   sysnet4admin.github.io
Address: 185.199.109.153
Name:   sysnet4admin.github.io
Address: 185.199.110.153
Name:   sysnet4admin.github.io
Address: 185.199.111.153
Name:   sysnet4admin.github.io
Address: 2606:50c0:8003::153
Name:   sysnet4admin.github.io
Address: 2606:50c0:8002::153
Name:   sysnet4admin.github.io
Address: 2606:50c0:8001::153
Name:   sysnet4admin.github.io
Address: 2606:50c0:8000::153

[root@net /]# nslookup ex-url-2
Server:         10.96.0.10
Address:        10.96.0.10#53

ex-url-2.default.svc.cluster.local      canonical name = k8s-edu.github.io.
Name:   k8s-edu.github.io
Address: 185.199.110.153
Name:   k8s-edu.github.io
Address: 185.199.109.153
Name:   k8s-edu.github.io
Address: 185.199.108.153
Name:   k8s-edu.github.io
Address: 185.199.111.153
Name:   k8s-edu.github.io
Address: 2606:50c0:8001::153
Name:   k8s-edu.github.io
Address: 2606:50c0:8002::153
Name:   k8s-edu.github.io
Address: 2606:50c0:8003::153
Name:   k8s-edu.github.io
Address: 2606:50c0:8000::153
```

<br/>

## 클러스터주소 ClusterIP / 헤드리스 Headless

### clusterIP

- 클러스터 IP는 Pod와 Pod의 연결을 위한 내부용 IP이다. (클러스터 내부의 목적을 위해 사용된다.) 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cl-nginx 
spec:
  selector:
    app: deploy-nginx  
  ports:
    - name: http
      port: 80
      targetPort: 80
  type: ClusterIP
```

- 해당 서비스를 띄운 뒤 조회해보면 아래와 같다.

```shell
$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
cl-nginx     ClusterIP   10.102.51.251   <none>        80/TCP    14s
```

### headless

- 헤드리스는 `spec.clusterIP`에 `None`이라고 선언하면 headless 타입이 된다.
- 클러스터 IP와 동일하게 클러스터 내부에서 사용되지만 IP가 없는 상태다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hdl-nginx 
spec:
  selector:
    app: deploy-nginx  
  ports:
    - name: http
      port: 80
      targetPort: 80
  clusterIP: None
```

- 해당 서비스를 조회해보자. 실제로 IP가 없는 것을 확인할 수 있다.

```shell
$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
hdl-nginx    ClusterIP   None            <none>        80/TCP    10s
```

- Deployment로 pod를 띄우면 해시값이 뒤에 붙기 때문에 파드 이름이 매번 바뀌어 호스트 이름을 알 수가 없다.
- StatefulSet과 Headless를 결합해 사용하면 어떨까?
  - StatefulSet은 항상 같은 호스트 네임을 가지고 잇기 때문에 내부에서 도메인 이름으로 호출할 수 있는 구조다.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sts-chk-hn
spec:
  replicas: 3
  serviceName: sts-svc-domain #statefulset need it
  selector:
    matchLabels:
      app: sts
  template:
    metadata:
      labels:
        app: sts
    spec:
      containers:
      - name: chk-hn
        image: sysnet4admin/chk-hn
---
apiVersion: v1
kind: Service
metadata:
  name: sts-svc-domain
spec:
  selector:
    app: sts
  ports:
    - port: 80
  clusterIP: None
```

<br/>

## 엔드포인트 Endpoints

### 로드밸런서 엔드포인트

- 디플로이먼트와 로드밸런서를 띄우면 엔드포인트가 만들어진다.
- 아래와 같은 경우 Pod가 3개 띄워지는데, 그러면 엔드포인트도 3개가 만들어진다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-chk-ip
  labels:
    app: deploy-chk-ip
spec:
  replicas: 3
  selector:
    matchLabels:
      app: deploy-chk-ip
  template:
    metadata:
      labels:
        app: deploy-chk-ip
    spec:
      containers:
        - name: chk-ip
          image: sysnet4admin/chk-ip
---
apiVersion: v1
kind: Service
metadata:
  name: lb-chk-ip
spec:
  selector:
    app: deploy-chk-ip
  ports:
    - name: http
      port: 80
      targetPort: 80
  type: LoadBalancer
```

### 서비스 엔드포인트

- 먼저 서비스를 선언한다. (타입 선언이 되지 않으면 기본적으로 클러스터 IP로 생성된다.)
- 그리고 `kind`를 `Endpoints`로 한다.
  - ip(`192.168.1.11`)의 경우 로드밸런서로 미리 만들어둔 ip이다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-data
spec:
  ports:
    - name: http
      port: 80
      targetPort: 80
---
apiVersion: v1
kind: Endpoints
metadata:
  name: external-data
subsets:
  - addresses:
      - ip: 192.168.1.11
    ports:
      - name: http
        port: 80
```

<br/>

## 인그레스 Ingress

<br/>