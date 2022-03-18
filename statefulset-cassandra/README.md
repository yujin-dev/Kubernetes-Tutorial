### 추가 설정
Minikube는 2048MB 메모리에 CPU 2개가 기본 설정

```console
$ minikube start --memory 5120 --cpus=4
```

### `kubectl apply -f https://k8s.io/examples/application/cassandra/cassandra-service.yaml` 확인

```console
$ kubectl get svc cassandra 
NAME        TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
cassandra   ClusterIP   None         <none>        9042/TCP   45s
```
### `kubectl apply -f https://k8s.io/examples/application/cassandra/cassandra-statefulset.yaml` 확인
```console
$ kubectl get statefulset cassandra
NAME        DESIRED   CURRENT   AGE
cassandra   3         0         13s
```

```console
$ kubectl get pods -l="app=cassandra"
(시간이 소요)
NAME          READY     STATUS    RESTARTS   AGE
cassandra-0   1/1       Running   0          10m
cassandra-1   1/1       Running   0          9m
cassandra-2   1/1       Running   0          8m
```

`cassandra-0` Pod내부의 링의 상태를 보여주는 `nodetool`

```console
$ kubectl exec -it cassandra-0 -- nodetool status
Datacenter: DC1-K8Demo
======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.5  83.57 KiB  32           74.0%             e2dd09e6-d9d3-477e-96c5-45094c08db0f  Rack1-K8Demo
UN  172.17.0.4  101.04 KiB  32           58.8%             f89d6835-3a42-4419-92b3-0e62cae1479c  Rack1-K8Demo
UN  172.17.0.6  84.74 KiB  32           67.1%             a6a1e8c2-3dc5-4417-b1a0-26507af2aaad  Rack1-K8Demo
```




