# kubernetes object

## Pod

- Pod나 Deployment와 같은 오브젝트는 모두 `apiVersion`에 속해있다. 따라서 Pod를 배포할 땐 그에 적합한 API 버전을 명시해야 한다.
- `kind`는 해당 오브젝트의 종류다.
- `spec`이 배포할 컨테이너에 대한 주요 정보를 가지고 있다.

```shell
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
```

<br/>

## Deployment

- `spec.replicas`를 통해 리플리카셋을 변경을 할 수 있다.
- `spec.slector`: 템플릿을 선택하는 설정
- `spec.template`: 똑같은 형태로 찍어내는 것. 
  - `spec.selector.matchLabels.app`과 `spec.template.metadata.labels.app`이 동일한 것을 확인할 수 있다.
- Deployment의 `spec`에 속한 정보를 자세히 살펴보면, Pod의 `metadata`와 `spec`을 그대로 긁어온 것 같아 보인다.
- Deployment는 템플릿을 사용해 여러 개의 Pod를 생성하는 오브젝트다. (Deployment는 레플리카셋과 파드를 한 번에 관리하는 오브젝트다.)

```shell

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
```

<br/>

## ReplicaSet

- kind가 `ReplicaSet`으로 바뀐 것 빼고는 Deployment와 차이점이 거의 안 보인다.
- 하지만 큰 차이점이 있다.
  - 레플리카셋은 파드를 업데이트 하는 기능을 포함하지 않는다.
  - Deployment에서 롤링 업데이트를 실행을 시작하면 레플리카셋의 복제본을 하나 더 만들어 차례로 파드를 하나씩 띄운다. 파드가 다 뜨면 기존의 레플리카셋을 내린다.

```shell
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

<br/>

## Job

- 위의 다른 오브젝트와는 다르게 `apiVersion`에서 `batch/v1`을 사용하는 것을 확인할 수 있다.

```shell
apiVersion: batch/v1
kind: Job
metadata:
  name: job-curl-succ
spec:
  template:
    spec:
      containers:
      - name: net-tools
        image: sysnet4admin/net-tools
        command: ["curlchk",  "nginx"]
      restartPolicy: Never
```

- Job은 실행이 끝나면 `Completed` 상태가 되면서 파드가 내려간다.
  - Job은 한 번만 실행되어야 하기 때문에 `restartPolicy`를 `Never`로 둔다.
  - 만약 Job에 `restartPolicy`를 지정해주지 않으면 아래와 같은 오류가 발생하고 Job이 만들어지지 않는다.

```shell
The Job "이름" is invalid: spec.template.spec.restartPolicy: Required value: valid values: "OnFailure", "Never"
```

### Job 병렬 실행

- 아래 `spec.completions`: Job을 몇 번 시도할 지 설정한다. 순차적으로 실행한다. 
  - Job 하나가 끝난 게 확인되면 그 다음 Job을 시작한다.

```shell
apiVersion: batch/v1
kind: Job
metadata:
  name: job-completions 
spec:
  completions: 3
  template:
    spec:
      containers:
      - name: net-tools
        image: sysnet4admin/net-tool
        command: ["curlchk",  "nginx"]
      restartPolicy: Never
```

- Job을 병렬로 실행하려면 `spec.parallelism`을 사용하면 된다.

```shell
apiVersion: batch/v1
kind: Job
metadata:
  name: job-parallelism
spec:
  parallelism: 3 
  template:
    spec:
      containers:
      - name: net-tools
        image: sysnet4admin/net-tools
        command: ["curlchk",  "nginx"]
      restartPolicy: Never
```

### Job의 자동 종료

- `spec.activeDeadlineSeconds` 설정은 Job이 작업을 마치지 못해도 그 시간이 지나면 Job을 내려버린다.
- 이 방식은 Job이 실행되기 전에 종료 시켜버릴 수 있는 위험한 방법이기 때문에 `ttlSecondsAfterFinished` 설정을 사용하는 것이 좋다.

```shell
apiVersion: batch/v1
kind: Job
metadata:
  name: job-activedeadlineseconds
spec:
  backoffLimit: 3
  activeDeadlineSeconds: 30
  template:
    spec:
      containers:
      - name: net-tools
        image: sysnet4admin/net-tools
        command: ["/bin/sh", "-c"]
        args:
        - sleep 60;
          curlchk nginx;  
      restartPolicy: Never
```

- `spec.ttlSecondsAfterFinished`은 Job이 completed 상태가 된 후부터의 유예 시간을 주는 설정이다. 유예 시간이 끝나면 파드가 자동으로 종료된다.
- 여기서 ttl이란 time to leave를 뜻한다.

```shell
apiVersion: batch/v1
kind: Job
metadata:
  name: job-ttlsecondsafterfinished
spec:
  backoffLimit: 3
  ttlSecondsAfterFinished: 30
  template:
    spec:
      containers:
      - name: net-tools
        image: sysnet4admin/net-tools
        command: ["/bin/sh", "-c"]
        args:
        - sleep 60;
          curlchk nginx;  
      restartPolicy: Never
```

<br/>

## CronJob

<br/>

## DaemonSet

<br/>

## StatefulSet