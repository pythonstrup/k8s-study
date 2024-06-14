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
