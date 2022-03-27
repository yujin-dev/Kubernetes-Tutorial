# Kubernetes Overview

*reference*
- https://nirsa.tistory.com
- https://deveric.tistory.com/116
- [Udemy CKA 강의](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests)

## Overview
Kubernetes는 Container+Orchestration의 집합이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3a4c81c9-5069-4de6-93a5-b5538bcca642/Untitled.png)

일단 **Node**( 과거에는 Minion이라고 함 )가 있다. 물리적 또는 가성 머신이다. 클러스터는 Node 여러 개로 구성된 집합이다. 

Node에는 **Master** 와 **Worker**가 있다. Master에는 **kube-api server**가 있어 Worker의 **kubelet**과 통신하며 관리할 수 있다. **ETCD**라는 key-value 저장소는 Node에 대한 정보를 저장하고 관리하는데 사용한다. **Scheduler**를 통해 Node에 워크와 컨테이너를 분산시킨다. 

Worker Node에는 컨테이너를 실행시킬 수 있는 도커와 같은 **런타임** 환경이 있다. 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ef5d6f31-43b7-44e0-9ad3-ca7bef6cb2c2/Untitled.png)

**Minikube**는 위의 모든 것을 통합하여 로컬 시스템에서 실행시킬 수 있다. 대신 minikube는 단일 Worker Node 쿠버네티스 클러스터만 세팅하는 경우에 이용한다. kubernetes CLI인 kubectl을 함께 설치하여 쿠버네티스를 활용한다. 여러 node cluster를 구성하는 경우 **kubeadm**을 사용해야 한다. 

**Pod**는 쿠버네티스 내에서 가장 작은 단위로 실행시켜야 할 컨테이너들을 하나의 application으로 묶은 인스턴스이다. 보통 하나의 docker container를 하나의 Pod에서 실행시킨다. 사용자가 늘어남에 따라 더 많은 트래픽을 감당하기 위해서는 Node내에서 Pod를 늘리거나 Node 자체를 늘려 서비스를 확장할 수 있다.

주로 Pod는 YAML 파일로 작성하여 생성한다.

## Core Concept
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbnNh5w%2FbtqB2wa5BW4%2FKnbI4zn8QsIK4jZ8rbyEF0%2Fimg.png)

- Master : Node를 제어하고 전체 클러스터를 관리해주는 controller이며 관리 서버이다.
- Node : 컨테이너가 배포될 물리 서버 또는 가상 머신( Worker Node )
- Pod : 단일 Node에 배포된 하나 이상의 컨테이너 그룹으로 여러 개의 컨테이너를 묶어서 Pod단위로 관리할 수 있다. 

### ETCD
- 클러스터의 모든 데이터를 저장하는 데이터베이스 역할로 분산형 key-value store이다.
- 모든 데이터를 저장하는 용도로 데이터 백업이나 클러스터링 후 여러 master server에 분산 실행하여 안정성을 보장한다.

**[설치 및 실행]**
```console
$ curl -L https://github.com/etcd-io/etcd/releases/download/v3.3.11/etcdv3.3.11-linux-amd64.tar.gz -o etcd-v3.3.11-linux-amd64.tar.gz
$ tar xzvf etcd-v3.3.11-linux-amd64.tar.gz
$ ./etcd
```

#### key - value를 저장
```console
$ ./etcd set key_1 value_1
```
#### key값 가져오기
```console
$ ./etcd get key_1
```
**[kubeadm]** 에서 확인할 수 있다.
```console
$ kubectl get pods -n kube-system
---------------------------------
NAMESPACE NAME READY STATUS RESTARTS AGE
...
kube-system etcd-master 1/1 Running 0 1h
```
etcd master POD의 key들을 확인할 수 있다. registry안에 pods, replicasets, roles 등에 정보가 있다.
```console
$ kubectl exec etcd-master –n kube-system etcdctl get / --prefix –keys-only
---------------------------------
/registry/apiregistration.k8s.io/apiservices/v1.
/registry/apiregistration.k8s.io/apiservices/v1.apps
/registry/apiregistration.k8s.io/apiservices/v1.authentication.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.authorization.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.autoscaling
/registry/apiregistration.k8s.io/apiservices/v1.batch
/registry/apiregistration.k8s.io/apiservices/v1.networking.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.rbac.authorization.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.storage.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1beta1.admissionregistration.k8s.io
```

### Kube-API Server
- Pod, Service, Replication controller와 같은 API에 대한 검증과 구성을 수행하는 REST API
- 유일하게 ETCD와 직접 소통하여 사용자를 인증하고, 요청을 허가하여 데이터를 불러온다.

```console
$ kubectl get nodes
---------------------------------
NAME STATUS ROLES AGE VERSION
master Ready master 20m v1.11.3
node01 Ready <none> 20m v1.11.3
```
**[설치]**
```console
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-apiserver
```
**[kubeadm]** 에서 확인할 수 있다.

```console
$ kubectl get pods -n kube-system
---------------------------------
NAMESPACE NAME READY STATUS RESTARTS AGE
...
kube-system kube-apiserver-master 1/1 Running 0 15m
```

### Kube Controller Manager
- 쿠버네티스에 탑재된 핵심 제어 루프를 포함하는 데몬
- 쿠버네티스에서 지속적으로 모니터링한다.
**[설치]**
```console
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-controller-manager
```
**[kubeadm]** 에서 확인할 수 있다.

```console
$ kubectl get pods -n kube-system
---------------------------------
NAMESPACE NAME READY STATUS RESTARTS AGE
...
kube-system kube-controller-manager-master 1/1 Running 0 15m
```

### Kube Scheduler
- 쿠버네티스에서 가용성, 성능 및 용량을 관리하는 스케줄러
- pod를 지정하기 위해 가장 적절한 node를 선정한다.
**[설치]**
```console
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-scheduler
```
**[kubeadm]** 에서 확인할 수 있다.


### Kubelet
- 각 노드에서 구동되는 주요 에이전트로 kubelet은 PodSpecs 집합을 가지며 컨테이너가 구동되고 있는지, 정상 작동하는지 보장한다.
- worker node를 등록하거나
- master Scheduler에 따라 POD를 생성하여 docker image를 빌드하거나 POD의 상태를 모니터링하여 Kube-API에 보고한다.

Kubeadm에 의해 설치되지 않기에 worker node에 직접 설치해야 한다.
**[설치]**
```console
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubelet
```

### Kube Proxy
- TCP/UDP 스트림 포워딩이나 백엔드 집합에 걸쳐 라운드 로빈 TCP/UDP 포워딩을 할 수 있다.
- IP table rules을 이용하여 POD 네트워크를 연결해 클러스터 내에서 서로 통신할 수 있도록 한다.
**[설치]**
```console
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-proxy
```
**[kubeadm]** 에서 확인할 수 있다.

```console
$ kubectl get pods -n kube-system
---------------------------------
NAMESPACE NAME READY STATUS RESTARTS AGE
...
kube-system kube-proxy-lzt6f 1/1 Running 0 16m
kube-system kube-proxy-zm5qd 1/1 Running 0 16m
```
### POD

#### nginx 이미지를 pod를 생성
```console
$ kubectl run nginx --image nginx
```

#### 현재 실행중인 Pod 파악
```console
$ kubectl get nodes
```


#### YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx 
  # 여러 컨테이너를 입력할 수 있다.
  # - name: busybox
  #   image: busybox
```

### ReplicaSets
Replicaset은 실행되는 pod 갯수에 대한 가용성을 보장하고 지정한 pod 갯수만큼 항상 실행될 수 있도록 관리한다.
 
#### Replication Controller
- 항상 정해진 수의 replicat가 실행 중임을 보장한다.
- replication controller는 정해진 pod를 유지하거나 로드 밸런싱을 위해 scale up햔다.
```yaml
apiVersion: v1
kind: ReplicationContoller
metadata:
  name: myapp-rc
  labels:
    app: myapp
    tier: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        tier: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx 
  replicas: 3
```
#### ReplicaSets
yaml 파일에서 replication controller와 차이점은 selector가 포함된다.
Replicaset은 직접적으로 Pod와 연결되어 있지 않기에 관리할 Pod를 찾기 위해 selector를 사용한다.

```yaml
apiVersion: v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
...
  replicas: 3
  selector: # 어떤 레이블의 pod를 선택하여 관리할지에 대한 설정
    matchLabels:
      type: front-end
```

**[ Commands ]**
- replicaset을 생성, 삭제 및 업데이트
  ```console
  $ kubectl create -f replicaset-definition.yml
  $ kubectl get replicaset # 1개의 replicaset가 생성되었고, 4개의 Pod가 필요함
  ------------------------
  NAME              DESIRED   CURRENT   READY   AGE
  new-replica-set   4         4         0       9s

  $ kubectl delete replicaset myapp-replicaset
  $ kubectl replace -f replicaset-definition.yml
  ```
- replicaset 정보 확인
  ```
  controlplane ~ ➜  kubectl explain replicaset
  KIND:     ReplicaSet
  VERSION:  apps/v1

  DESCRIPTION:
      ReplicaSet ensures that a specified number of pod replicas are running at
      any given time.

  FIELDS:
    apiVersion   <string>
      APIVersion defines the versioned schema of this representation of an
      object. Servers should convert recognized schemas to the latest internal
      value, and may reject unrecognized values. More info:
      https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources
  ```
- 파드 갯수를 실시간으로 조정
  ```console
  $ kubectl scale replicaset test-replicaset --replicas=5
  ```

### Deployment
Replicaset의 상위 개념으로 볼 수 있다. Replicaset을 생성하는 deployment를 정의하거나 배포 작업을 롤링 업데이트(인스턴스를 하나씩 돌면서 업그레이드) 하는 기능이 있다.


```yaml
apiVersion: v1 -> apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 2 
  selector: # 어떤 레이블의 pod를 선택하여 관리할지에 대한 설정
    matchLabels:
      tier: frontend
  template: # 어떤 파드를 실행할지에 대한 정보
    metadata:
      labels:
        tier: nginx -> frontend
    spec: # 컨데이터에 대한 설정
      containers:
      - name: nginx
        image: nginx
```
#### Deployment Update
```console
$ kubectl set image deployment/{deployment name} {container name}={image}:{version}
```

### Namespace
```console
$ kubectl get pods --nampspace=dev
```

```yaml
apiVersion: v1 -> apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
  namespace=dev
```

