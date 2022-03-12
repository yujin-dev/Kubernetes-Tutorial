## Tips
### Imperative Commands with Kubectl

`--dry-run` : `--dry-run=client`를 설정하면 해당 resource를 생성하지 않는 대신 필요한 조건을 추출한다.

`-o yaml`: resource definition을 yaml파일로 출력한다. 

#### Example

Create an NGINX Pod
```console
$ kubectl run nginx --image=nginx
```

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)
```console
$ kubectl run nginx --image=nginx --dry-run=client -o yaml
```

Create a deployment
```console
$ kubectl create deployment --image=nginx nginx
```
Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)
```console
$ kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```
Generate Deployment with 4 Replicas

```console
$ kubectl create deployment nginx --image=nginx --replicas=4
```
You can also scale a deployment using the kubectl scale command.
```console
$ kubectl scale deployment nginx --replicas=4
```
Another way to do this is to save the YAML definition to a file and modify
```console
$ kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```
Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
```console
$ kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
# This will automatically use the pod's labels as selectors
```

```console
$ kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 
# This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service
```

Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:
```console
$ kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
# This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod
```
Or
```console
$ kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
# This will not use the pods labels as selectors
```