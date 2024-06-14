# Intro

## Pod, Container, Application

### Pod

- 파드는 볼륨을 가지고 있고, 다수의 컨테이너가 존재한다.
- 한 개의 컨테이너를 가진 파드 예시: Nginx
- 다수의 컨테이너를 가진 파드 예시: Spring Boot Server, MySQL 등

### Container

- 파드에 포함.

### Application

- 기능이 동작하는 단위를 애플리케이션이라고 표현한다.
- 애플리케이션은 단일 컨테이너일 수도 있고 복수의 컨테이너일 수도 있다.
- 파드와 1대 1로 매핑될 수도 있고, 파드가 여러 개 모여 동작하는 기능일 수도 있다.

<br/>

## 자주 사용하는 kubectl 명령어

- `get`: 오브젝트를 조회하는 명령어
- `run, create, apply`: 오브젝트 생성
- `delete`: 오브젝트 삭제
- `exec`: 파드 내부에 컨테이너로 접속
- `scale`: 파드 갯수를 늘리거나 줄임
- `edit`: 배포된 오브젝트를 수정

<br/>

## 자주 사용하는 kubectl 옵션 명령어

### `-o yaml`

- yaml 파일로 추출
- 아래 명령어를 사용하면 pod가 yaml 파일로 추출될 것이다.

```shell
$ kubectl get pod nginx -o yaml
```

### `--dry-run=cliet`

- 실제로 클러스터에 적용하진 않지만, 실험적으로 실행만 해보는 기능이다.
- 아래 명령어를 실행하면 dry run도 실행되면서 yaml 파일도 추출해준다. 
  - `-o yaml` 옵션은 해당 컨테이너가 실행 중일 때만 yaml 파일을 추출할 수 있다.
  - 그런 dry run을 하면 실제로 실행되진 않지만 실행되는 것처럼 동작하기 때문에 `-o yaml`을 사용할 수 있게 된다. 
  - 그래서 둘을 자주 혼용해 사용한다.

```shell
$ kubectl run nginx --image=nginx --dry-run=client -o yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

- 아래와 같이 리눅스 명령어를 활용하면 파일로 바로 저장할 수 있다.

```shell

$ kubectl run nginx --image=nginx --dry-run=client -o yaml > pod-nginx.yaml
$ ls
... pod-nginx.yaml
```

- deployment 오브젝트에도 적용 가능하다.

```shell
$ kubectl create deploy nginx --image=nginx -o yaml --dry-run=client > deploy-nginx.yaml
$ kubectl apply -f deploy-nginx.yaml
$ kubectl get pod
NAME                     READY   STATUS              RESTARTS   AGE
nginx-6799fc88d8-wmphf   0/1     ContainerCreating   0          2s

$ kubectl delete deploy nginx
```

### 상태를 알 수 있는 명령어 01: Events

- namespace 단위로 이벤트를 확인할 때 사용한다.

```shell
$ kubectl get events
LAST SEEN   TYPE      REASON                    OBJECT                            MESSAGE
17m         Normal    Scheduled                 pod/apy-nginx-558dd8d455-k9cnx    Successfully assigned default/apy-nginx-558dd8d455-k9cnx to w3-k8s
17m         Normal    Pulling                   pod/apy-nginx-558dd8d455-k9cnx    Pulling image "nginx"
...
```

```shell
$ kubectl get events -n kube-system
LAST SEEN   TYPE      REASON              OBJECT                                          MESSAGE
30h         Warning   FailedScheduling    pod/calico-kube-controllers-57c5b6487c-hnm2g    0/1 nodes are available: 1 node(s) had taint {node.kubernetes.io/not-ready: }, that the pod didn't tolerate.
30h         Warning   FailedScheduling    pod/calico-kube-controllers-57c5b6487c-hnm2g    0/1 nodes are available: 1 node(s) had taint {node.kubernetes.io/not-ready: }, that the pod didn't tolerate.
...
```

### 상태를 알 수 있는 명령어 02: Describe

- 오브젝트의 상태를 보기 위해 사용하는 명령어다.
- events 명령어보다는 describe 명령어를 더 자주 사용한다.

```shell
$ kubectl describe pod nginx
Name:         nginx
Namespace:    default
Priority:     0
Node:         w2-k8s/192.168.1.102
Start Time:   Fri, 07 Jun 2024 22:51:54 +0900
Labels:       run=nginx
Annotations:  cni.projectcalico.org/podIP: 172.16.103.136/32
              cni.projectcalico.org/podIPs: 172.16.103.136/32
Status:       Running
IP:           172.16.103.136
IPs:
  IP:  172.16.103.136
Containers:
  nginx:
    Container ID:   docker://8d276f14b432415b20aaa9f27581164238c4c7e3915d0fbb60618398388c9e35
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:0f04e4f646a3f14bf31d8bc8d885b6c951fdcf42589d06845f64d18aec6a3c4d
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 07 Jun 2024 22:51:58 +0900
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2kxl7 (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  kube-api-access-2kxl7:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  12s   default-scheduler  Successfully assigned default/nginx to w2-k8s
  Normal  Pulling    11s   kubelet            Pulling image "nginx"
  Normal  Pulled     8s    kubelet            Successfully pulled image "nginx" in 2.778277444s
  Normal  Created    8s    kubelet            Created container nginx
  Normal  Started    8s    kubelet            Started container nginx
```

### 상태를 알 수 있는 명령어 03: Logs

- 컨테이너의 안의 로그를 확인할 수 있는 명령어다.

```shell
$ kubectl logs nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2024/06/07 13:51:58 [notice] 1#1: using the "epoll" event method
2024/06/07 13:51:58 [notice] 1#1: nginx/1.27.0
2024/06/07 13:51:58 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
2024/06/07 13:51:58 [notice] 1#1: OS: Linux 3.10.0-1160.90.1.el7.x86_64
2024/06/07 13:51:58 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2024/06/07 13:51:58 [notice] 1#1: start worker processes
2024/06/07 13:51:58 [notice] 1#1: start worker process 29
```

<br/>

# 참고자료

- [그림으로 배우는 쿠버네티스](https://www.inflearn.com/course/%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EB%B0%B0%EC%9A%B0%EB%8A%94-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4)
