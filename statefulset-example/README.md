### `mysql-deployment.yaml`
- PersistenceVolume을 `/var/lib/mysql/`에 마운트
- `MYSQL_ROOT_PASSWORD`: secret에서 가져와 데이터베이스 암호로 설정

### `wordpress-deployment.yaml`
- PersistenceVolume을 `/var/www/html`에 마운트
- `WORDPRESS_DB_HOST`: MySQL Service 이름이 설정
- `WORDPRESS_DB_PASSWORD`: kustomize가 생성한 데이터베이스 비밀번호가 설정

### `kubectl apply -k ./` 확인
```console
$ kubectl get secrets
NAME                    TYPE                                  DATA   AGE
mysql-pass-c57bb4t7mf   Opaque                                1      9s
```

```console
$ kubectl get pvc
NAME             STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
mysql-pv-claim   Bound     pvc-8cbd7b2e-4044-11e9-b2bb-42010a800002   20Gi       RWO            standard           77s
wp-pv-claim      Bound     pvc-8cd0df54-4044-11e9-b2bb-42010a800002   20Gi       RWO            standard           77s
```

```console
$ kubectl get pods
NAME                               READY     STATUS    RESTARTS   AGE
wordpress-mysql-1894417608-x5dzt   1/1       Running   0          40s
```

```console
$ kubectl get services wordpress
NAME        TYPE            CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
wordpress   LoadBalancer    10.0.0.89    <pending>     80:32406/TCP   4m
```


