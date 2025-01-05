
빠르게 pod 생성
```
k run (pod name) --image=(imagename) -n=(namespace)

k run redis --image=redis -n=finance
```

어느 한 pod가 어느 ns에 있는지 알고싶을때
```
k get po --all-namespace 
k get po -A

```

What DNS name should the Blue application use to access the database 'db-service' in its own namespace -marketing. you can try it in the web application UI. Use port 6379
```
terminal ui 오른쪽 위에 ui 접근할수 있는 버튼이 있음 - 여기서 테스트 가능 

k get po -n=marketing
redis-db, blue라는 pod가 있음을 확인 할 수 있다 

k get svc -n=marketing
blue-service, db-service 란 서비스가 있는 걸 확인 할 수 있다. 
```

*같은 ns상에 있다면 just 그 서비스 이름인 db-service라고 dns name을 부를 수 있다.*

같은 ns가 아닌 dev라는 다른 ns에 있는 'db-service'에 access 하기 위해서는 어떤 dns name을 써야할까?
```
db-servce.dev.svc.cluster.local
```


### Certification Tip: Imperative Commands
```
--dry-run=client
```
If you simply want to test your command, use it
This will not create the resource. Instead, tell you whether the resource can be created and if your command is right.
```
-o yaml
```
This will output the resource definition in YAML format on the screen.

#### create pod
```
kubectl run nginx --image=nginx
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```
k run (podname)이지
k run po (podname) 아님

#### create deployment
```
kubectl create deployment --image=nginx nginx
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
kubectl create deployment nginx --image=nginx --replicas=4

#scale
kubectl scale deployment nginx --replicas=4
```

### Service
**Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379**

`kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml`

(This will automatically use the pod's labels as selectors)

Or

`kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml` (This will not use the pods' labels as selectors; instead it will assume selectors as **app=redis.** [You cannot pass in selectors as an option.](https://github.com/kubernetes/kubernetes/issues/46191) So it does not work well if your pod has a different label set. So generate the file and modify the selectors before creating the service)
  
**Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:**

`kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml`

(This will automatically use the pod's labels as selectors, [but you cannot specify the node port](https://github.com/kubernetes/kubernetes/issues/25478). You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`

(This will not use the pods' labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the **`kubectl expose`** command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.
첫번째꺼는 nodeport의 포트번호를 지정할 수 없지만 label selector을 자동으로 해준다. 
2번은 label selector을 자동으로 해주지 않지만 nodeport의 포트번호를 자동으로 해준다. 

### pod 생성시 clusterip 함께생성

create a pod called httpd using the image httpd:alpine in the default ns. next create a service of type clusterip by the same name (httpd). the target port for the service should be 80.
```
난 
k run httpd --image=httpd:alpine 으로 pod 생성후 
k expose pod httpd --port=80 --name=httpd 으로 만들었는데 이를 한번에 해줄 수 있다. 

k run httpd --image=http:alpine --port=80 --expose=true
k run --help 
해서 보면 알수 있다. 
```