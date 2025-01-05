
```
k get nodes -o wide
```

### how to json path in kubectl

1. identify the kubectl command
2. familiarize with json output
   `k get pods -o json
   ![[Pasted image 20240721220130.png|444]]
3. form the json path query
   `.items[0].spec.containers[0].image`
4. use the json path query with kubectl command
   `k get po -o=jsonpath='{.items[0].spec.containers[0].image}'`

![[Pasted image 20240721220720.png|500]]

![[Pasted image 20240721220859.png|500]]

아래의 방식을 더 추천한다.
![[Pasted image 20240721221019.png|500]]


![[Pasted image 20240721221114.png|400]]


## Lightning lab-1

1. Upgrade the current version of kubernetes from `1.29.0` to `1.30.0` exactly using the `kubeadm` utility. Make sure that the upgrade is carried out one node at a time starting with the controlplane node. To minimize downtime, the deployment `gold-nginx` should be rescheduled on an alternate node before upgrading each node.

  

Upgrade `controlplane` node first and drain node `node01` before upgrading it. Pods for `gold-nginx` should run on the `controlplane` node subsequently.

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#upgrading-control-plane-nodes

![[Pasted image 20240721225522.png|400]]
![[Pasted image 20240721224151.png|400]]
![[Pasted image 20240721224317.png|400]]

![[Pasted image 20240721224217.png]]


**Upgrade `controlplane`**

1. Update package & Check madison 
```shell
apt update
apt-cache madison kubeadm
```

```shell
vi /etc/apt/sources.list.d/kubernetes.list
```

	1.29 ->1.30

Drain node
```
kubectl drain controlplane --ignore-daemonsets
```
    
Upgrade kubeadm
```shell
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.0-1.1' && \
sudo apt-mark hold kubeadm
```

    
```
kubeadm upgrade plan
kubeadm upgrade apply v1.30.0    
```

 Upgrade the kubelet
```shell
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.0-1.1' kubectl='1.30.0-1.1' && \
sudo apt-mark hold kubelet kubectl

  ```
    
restart kubelet
```shell
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```
    
Reinstate controlplane node
    
```
kubectl uncordon controlplane
```


**Upgrade `node01`**

Go to worker node
    
```
ssh node01
```

  Upgrade kubeadm
    
```shell
vi /etc/apt/sources.list.d/kubernetes.list
# 1.29 -> 1.30

sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.0-1.1' && \
sudo apt-mark hold kubeadm	

```
    
Upgrade node
   ![[Pasted image 20240721230855.png|300]]
   
```
sudo kubeadm upgrade node

exit
```

Drain the worker node
    
```
kubectl drain node01 --ignore-daemonsets --force
```
    

Upgrade the kubelet
    
```
ssh node01

sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.30.0-1.1' kubectl='1.30.0-1.1' && \
sudo apt-mark hold kubelet kubectl
```

    estart kubelet
```
systemctl daemon-reload
systemctl restart kubelet    
```

Rehold packages
    
```
apt-mark hold kubeadm kubelet
    
exit
```

Reinstate worker node
    
```
kubectl uncordon node01
```
    
Verify `gold-nginx` 
    
```
kubectl get pods -o wide | grep gold-nginx
```

![[Pasted image 20240722001337.png]]




2. Print the names of all deployments in the `admin2406` namespace in the following format:
```
k -n admin2406 get deploy -o custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[0].image,READY_REPLICAS:.spec.replicas,NAMESPACE:.metadata.namespace --sort-by=.metadata.name > /opt/admin2406_data
```


3. A kubeconfig file called `admin.kubeconfig` has been created in `/root/CKA`. There is something wrong with the configuration. Troubleshoot and fix it.
   ![[Pasted image 20240721234102.png]]
	![[Pasted image 20240721234215.png]]
server port 6443으로 fix

![[Pasted image 20240721234319.png]]


4.Create a new deployment called `nginx-deploy`, with image `nginx:1.16` and `1` replica. Next upgrade the deployment to version `1.17` using `rolling update`.

```
k create deploy nginx-deploy --image=nginx:1.16 --replicas=1 --dry-run=client -o yaml > nginx.yaml

k apply -f nginx.yaml 

k set image deploy nginx-deploy nginx=nginx:1.17
```


5. A new deployment called `alpha-mysql` has been deployed in the `alpha` namespace. However, the pods are not running. Troubleshoot and fix the issue. The deployment should make use of the persistent volume `alpha-pv` to be mounted at `/var/lib/mysql` and should use the environment variable `MYSQL_ALLOW_EMPTY_PASSWORD=1` to make use of an empty root password.

Important: Do not alter the persistent volume.

![[Pasted image 20240721234852.png]]

![[Pasted image 20240721234926.png]]

![[Pasted image 20240721235335.png|400]]


6. Take the backup of ETCD at the location `/opt/etcd-backup.db` on the `controlplane` node.

etcd 정보조회

`k describe pod/etcd-controlplane -n kube-system | grep '\-\-'`

![[Pasted image 20240721235532.png]]

```
ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key snapshot save /opt/etcd-backup.db
```


7. Create a pod called `secret-1401` in the `admin1401` namespace using the `busybox` image. The container within the pod should be called `secret-admin` and should sleep for `4800` seconds.  

The container should mount a `read-only` secret volume called `secret-volume` at the path `/etc/secret-volume`. The secret being mounted has already been created for you and is called `dotfile-secret`.


```
k run secret-1401 -n admin1401 --image=busybox --dry-run=client -o yaml --command -- sleep 4800 > admin.yaml
```
