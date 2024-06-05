# 시작하기 전

- Virtual Box 설치
- Vagrant 설치
- PuTTY 설치
- SuperPuTTY 설치

# Vagrant 실행

- nodes 디렉토리의 있는 vagrant 파일을 실행합시다.
- 해당 디렉토리에 들어가 아래 명령어를 실행해줍니다.

```shell
vagrant up
```

- 위 명령어를 실행하면 자동으로 CentOs 컨테이너가 4대가 뜬다. (Virtual Box 프로그램에서 확인할 수 있다.)
- 처음에 컨테이너를 띄울 때만 해당 명령어를 실행하고, 그 다음부터는 Virtual Box UI에서 시작하기 버튼만 누르면 된다.

# SuperPuTTY 설정

- SuperPuTTY를 실행한다.
- 메뉴의 `file` => `import Sessions` => `from file` 을 클릭하자.
- 현재 디렉토리에 있는 `Sessions(k8s_learning).XML`을 선택하면 세션이 자동으로 등록된다!
- 이제 `m-k8s` 파일에 접속해보자. root의 비밀번호는 `vagrant`이다. (`Sessions(k8s_learning).XML` 파일에 설정되어 있음)
- `kubectl`, `kubeadm` 등의 명령어를 입력해 쿠버네티스 환경이 잘 설치됐는지 확인해보자.

### kube가 실행되고 있는지 확인하려면?

- 아래 명령어를 실행했을 때, Active가 activating 상태라면 정상적으로 동작하고 있는 것이다.

```shell
> systemctl status kubelet
```

<br/>

# kubeadm을 사용해 클러스터 구축하기

- 먼저 vagrant up을 통해 컨테이너가 실행되었다는 것을 가정하고 실습을 진행한다.
- 각 컨테이너에서 아래에서 설명하는 명령어를 순서대로 실행해주면 된다. (쉘 스크립트 실행)
- 마스터 노드를 먼저 띄우고 그 다음에 워커 노드를 마스터 노드에 조인시킨다.

## 명령어

### 마스터 노드 설정

- SuperPuTTY를 사용해 m-k8s 노드에 접속해 아래 명령어를 실행하자. (root에 설정에 필요한 파일이 저장되어 있을 것이다.)

```shell
_Lecture_k8s_learning.kit/ch1/1.6/WO_master_node.sh
```

- 아래 명령어를 실행하면 `kubeadm config images pull'` 명령어를 통해 kubeadm을 설정하기 위한 이미지를 다운로드 받기 시작한다.
- 그리고 내려받은 이미지를 통해 아래와 같은 구성요소를 설치한다.

```shell
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
```

- 설치가 완료되었다면 아래 명령어를 사용해 설치가 잘 됐는지 확인해보자.

```shell
> kubectl get po -A 
```

- 아래와 같이 kube 시스템 구성요소를 확인할 수 있다.

```shell
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-744cfdf676-9hgfj   1/1     Running   0          3m12s
kube-system   calico-node-lxbwt                          1/1     Running   0          3m13s
kube-system   coredns-74ff55c5b-2jbtr                    1/1     Running   0          3m12s
kube-system   coredns-74ff55c5b-cfwhg                    1/1     Running   0          3m12s
kube-system   etcd-m-k8s                                 1/1     Running   0          3m24s
kube-system   kube-apiserver-m-k8s                       1/1     Running   0          3m24s
kube-system   kube-controller-manager-m-k8s              1/1     Running   0          3m24s
kube-system   kube-proxy-t27fd                           1/1     Running   0          3m13s
kube-system   kube-scheduler-m-k8s                       1/1     Running   0          3m24s
```

### 워커 노드 설정

- 나머지 워커 노드에 접속해 아래 명령어를 실행하자.

```shell
kubeadm join --token 123456.1234567890123456 \
             --discovery-token-unsafe-skip-ca-verification 192.168.1.10:6443
```

## Master Node

- kubeadm init
  - `--token`: 토큰을 임의의 숫자 값으로 지정
  - `token-ttl 0`: 토큰 수명을 무한대로
  - `--pod-netword-cidr`: pod가 assign 받을 수 있는 네트워크 지정
  - `--apiserver-advertise-address`: k8s api-server의 광고 주소를 master node의 ip 주소로 fix

```shell
#!/usr/bin/env bash

# init kubernetes 
kubeadm init --token 123456.1234567890123456 --token-ttl 0 \
--pod-network-cidr=172.16.0.0/16 --apiserver-advertise-address=192.168.1.10

# config for master node only 
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# raw_address for gitcontent
raw_git="raw.githubusercontent.com/sysnet4admin/IaC/master/manifests" 

# config for kubernetes's network 
kubectl apply -f https://$raw_git/172.16_net_calico_v1.yaml
```

## Worker Node

- 마스터 노드는 이미 구성이 완료된 상태에서 kubeadm으로 조인하는 명령어만 있으면 된다.
- 토큰 값을 master node와 동일하게 가져가야 한다.
- 원래는 인증 과정을 거쳐야 하지만 실습의 편의성을 위해 skip!
- 그리고 api server의 광고 주소로 사용하는 마스터 노드의 ip를 조인하는 값으로 사용한다.
- API 서버의 포트번호는 `6443`으로 고정이다.
- 이제 워커 노드가 마스터 노드에 조인을 완료하면 쿠버네티스 클러스터 구성이 완료된다.

```shell
#!/usr/bin/env bash

# config for work_nodes only 
kubeadm join --token 123456.1234567890123456 \
             --discovery-token-unsafe-skip-ca-verification 192.168.1.10:6443

```