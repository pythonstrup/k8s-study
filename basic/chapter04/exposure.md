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

<br/>

## 클러스터주소 ClusterIP / 헤드리스 Headless

<br/>

## 엔드포인트 Endpoints

<br/>

## 인그레스 Ingress

<br/>