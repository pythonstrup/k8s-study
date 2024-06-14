# Volume

## EmptyDir

- EmptyDir은 보통 컨테이너가 공유해야할 공간이나 내용이 있으면 사용한다. 성능을 위해 공유하는 공간을 메모리로도 설정할 수 있다.
- `spec.containers.volumeMounts`를 통해 마운트 되는 볼륨의 이름과 마운트 경로를 정하는 설정이다.
- `spec.volumes`는 볼륨을 직접적으로 설정하는 부분이다. 이 볼륨이 2개의 컨테이너를 연결해준다고 보면 된다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-emptydir 
  labels:
    app: nginx 
spec:
  containers:
  - name: web-page
    image: nginx 
    volumeMounts:
    - mountPath: /usr/share/nginx/html 
      name: empty-directory 

  - name: html-builder 
    image: alpine 
    volumeMounts:
    - mountPath: /html-dir 
      name: empty-directory 
    command: ["/bin/sh", "-c"]
    args: 
      - echo "This page created on $(date +%Y-%m-%d)" > /html-dir/index.html;
        sleep infinity;

  volumes:
  - name: empty-directory 
    emptyDir: {}
```

- yaml 파일을 적용해보고 확인해보자.

```shell
$ kubectl get pod -o wide
NAME           READY   STATUS    RESTARTS      AGE     IP                    NODE     NOMINATED NODE   READINESS GATES
pod-emptydir   2/2     Running   0             3m15s   172.16.103.160       w2-k8s   <none>           <none>
$ curl 172.16.103.160
This page created on 2024-06-14
```

<br/>

## HostPath

> HostPath 볼륨에는 많은 보안 위험이 있으며 가능하면 HostPath를 사용하지 않는 것이 좋다.

- hostPath의 경우 워커 노드의 로컬 파일시스템에 있는 파일이나 디렉토리를 pod에 마운트한다.
- `spec.template.spec.containers.volumeMounts.mountPath`가 `/host-log`로 잡혀 있는 것을 확인할 수 있다.
  - `spec.template.spec.containers.volumes.hostPath`는 `/var/log`로 설정되어 있다.
  - `spec.template.spec.containers.volumes.name`이 `spec.template.spec.containers.volumeMounts.name`을 통해 컨테이너의 경로와 연결된다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-hostpath
  labels:
    app: deploy-hostpath
spec:
  replicas: 3
  selector:
    matchLabels:
      app: deploy-hostpath 
  template:
    metadata:
      labels:
        app: deploy-hostpath 
    spec:
      containers:
      - name: host-mon
        image: sysnet4admin/sleepy
        volumeMounts:
        - mountPath: /host-log  
          name: hostpath-directory 
      volumes:
      - name: hostpath-directory 
        hostPath:
          path: /var/log
```

- `Deployment`의 경우 **노드마다 1개씩** 호스트 패스를 이용해 로그를 확인하기는 어렵다. (어떤 노드에 어떻게 배포될 지 모르기 때문이다.)
- 이럴 때 사용하는 k8s 오브젝트가 `DaemonSet`이다. (데몬셋은 한 노드에 하나의 파드가 올라간다는 특징이 있다.)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-hostpath
  labels:
    app: ds-hostpath
spec:
  selector:
    matchLabels:
      app: ds-hostpath 
  template:
    metadata:
      labels:
        app: ds-hostpath 
    spec:
      containers:
      - name: host-mon
        image: sysnet4admin/sleepy
        volumeMounts:
        - mountPath: /host-log  
          name: hostpath-directory 
      volumes:
      - name: hostpath-directory 
        hostPath:
          path: /var/log
```

- 데몬셋을 배포하고 마운트가 잘 되어 있는지 확인해보자.

```shell
$ kubectl get pod
NAME                READY   STATUS    RESTARTS      AGE
ds-hostpath-c525q   1/1     Running   0             3m27s
ds-hostpath-fxfdn   1/1     Running   0             3m27s
ds-hostpath-zknmd   1/1     Running   0             3m27s
$ kubectl get pod -o wide
NAME                READY   STATUS    RESTARTS      AGE     IP               NODE     NOMINATED NODE   READINESS GATES
ds-hostpath-c525q   1/1     Running   0             5m42s   172.16.132.32    w3-k8s   <none>           <none>
ds-hostpath-fxfdn   1/1     Running   0             5m42s   172.16.103.161   w2-k8s   <none>           <none>
ds-hostpath-zknmd   1/1     Running   0             5m42s   172.16.221.154   w1-k8s   <none>           <none>
```

- 워커 노드 3번에 있는 `ds-hostpath-c525q`에 접근해볼 것이다. 그리고 워커 노드 3번의 로그를 검색해보자.

```shell
kubectl exec ds-hostpath-c525q -it -- /bin/bash
[root@ds-hostpath-c525q /]# cat host-log/messages | grep w1-k8s
~~~ 
엄청나게 많은 로그
~~~
```

- 워커 노드 1번의 내용으로 메시지를 찾으면 아무런 내용도 찾을 수 없다.

```shell
[root@ds-hostpath-c525q /]# cat host-log/messages | grep w1-k8s
cat: can't open 'host-log/message': No such file or directory
```

<br/>

## NFS Volume

<br/>

## Persistence Volume & Persistence Volume Claim

<br/>

## Storage Class

<br/>

## VolumeClaimTemplate

<br/>

# 참고자료

- [그림으로 배우는 쿠버네티스](https://www.inflearn.com/course/%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EB%B0%B0%EC%9A%B0%EB%8A%94-%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4)
