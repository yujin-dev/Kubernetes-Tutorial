# Kubernetes 
[Udemy CKA 강의](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests) 를 듣고 정리한 공간. 

## Core Concept
![](./img/2021-11-20-11-11-44.png)

### ETCD
분산된 reliable key-value store이다.

```console
$
```
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
유일하게 ETCD와 직접 소통하여 사용자를 인증하고, 요청을 허가하여 데이터를 불러온다.

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
쿠버네티스에서 지속적으로 모니터링한다.
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
pod를 지정하기 위해 가장 적절한 node를 선정한다.
**[설치]**
```console
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kube-scheduler
```
**[kubeadm]** 에서 확인할 수 있다.


### Kubelet
worker node를 등록하거나 master Scheduler에 의해 POD를 생성하여 docker image를 빌드하거나 POD의 상태를 모니터링하여 Kube-API에 보고한다.

Kubeadm에 의해 설치되지 않기에 worker node에 직접 설치해야 한다.
**[설치]**
```console
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubelet
```

### Kube Proxy
IP table rules을 이용하여 POD 네트워크를 연결해 클러스터 내에서 서로 통신할 수 있도록 한다.
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
### PODs

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