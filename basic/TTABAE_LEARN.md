> 따배쿠 - 쿠버네티스 시리즈

# 쿠버네티스 아키텍쳐

1. 이미지를 docker hub에 저장한다.
2. kubectl를 통해 명령어를 master( control plane )에 전송한다.
3. master에는 REST API server가 있어 kubectl 명령어가 REST를 호출한다. 
4. scheduler가 worker node의 상태를 확인하여 노드의 kubelet에 전송한다. 
5. kubelet을 통해 docker image를 받아 컨테이너로 실행하는데 pod를 생성하게 된다.

## master( control-plane ) component

- ETCD : key-value database로, worker node에 대한 상태나 도커 컨테이너 등의 상태에 대한 정보가 있다. kubelet 내의 component cAdvisor를 통해 수집된 데이터가 저장된다.
- kube-apiserver : kubectl 명령어가 API에 요청되어 실행된다. 모든 것을 요청하고 제어하는 부분이다.
- kube-schduler : API를 통해 ETCD에서 노드 정보를 가져와 적절한 노드를 선택하도록 응답한다. API를 통해 worker node에 통신하여 kubelet에 요청하고 docker를 통해 이미지를 받아 실행한다.
- kube-controller-manager : 요구하는만큼의 컨테이너가 작동하는지 모니터링하여 API에 필요한 만큼의 갯수가 실행 중이도록 보장한다.


## worker node component
- kubelet
- kube-proxy : 네트워크 담당
- container runtime : docker

