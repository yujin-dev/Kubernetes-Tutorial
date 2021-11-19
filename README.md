# Kubernetes 
[Udemy CKA 강의](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests) 를 듣고 정리한 공간. 

## Core Concept
### ETCD

### Kube-API Server
유일하게 ETCD와 직접 소통한다.

### Kube Controller Manager
쿠버네티스에서 지속적으로 모니터링한다.

### Kube Scheduler
pod를 지정하기 위해 가장 적절한 node를 선정한다.

### Kubelet
worker node를 등록하거나 master Scheduler에 의해 POD를 생성하여 docker image를 빌드하거나 POD의 상태를 모니터링하여 Kube-API에 보고한다.

Kubeadm에 의해 설치되지 않기에 worker node에 직접 설치해야 한다.

### Kube Proxy
IP table rules을 이용하여 POD 네트워크를 연결해 클러스터 내에서 서로 통신할 수 있도록 한다.