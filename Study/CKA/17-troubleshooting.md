### Application Failure

먼저 앱과 데이터베이스로 구성되어있는 구조에서 트러블슈팅을 한다고 가정해보자.
프론트엔드부터 체크해본다.
![[Pasted image 20240714193124.png|500]]
- curl을 사용하여 해당 웹서버 ip와 포트에 접근할 수 있는지 체크
- describe service - selector에 pod가 잘지정되어있는지, endpoints 등 확인.
- pod definitions 의 pod name과 동일한지 체크

![[Pasted image 20240714193132.png|500]]
- pod status 확인
- log 명령어로 로그 확인 가능
	- -f: --follow - 로그의 실시간 스트리밍
	- --previous: 이전 로그 확인


#### test


**Troubleshooting Test 1:** A simple 2 tier application is deployed in the `alpha` namespace. It must display a green web page on success. Click on the `App` tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

![[Pasted image 20240714212836.png|400]]

```
kubectl config set-context --help
kubectl config set-context --current --namespace=alpha
```
context를 추가하여 alpha namespace 별도 입력하지 않아도 되도록 설정 가능


![[Pasted image 20240714212830.png]]
```
k get deploy
k describe deploy webapp-mysql

k get svc
```
dbhost 명이 잘못 지정되어있었음.
```
k edit svc mysql
k delete svc mysql
k apply -f /tmp/kubectl-edit-2243266576.yaml
```


**Troubleshooting Test 2:** The same 2 tier application is deployed in the `beta` namespace. It must display a green web page on success. Click on the `App` tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

![[Pasted image 20240714214314.png]]
mysql 접근 불가상태
![[Pasted image 20240714214348.png|400]]
target port 3306이 아닌 8080으로 잘못 지정됨.


**Troubleshooting Test 3:** The same 2 tier application is deployed in the `gamma` namespace. It must display a green web page on success. Click on the `App` tab at the top of your terminal to view your application. It is currently failed or unresponsive. Troubleshoot and fix the issue.

1,2번 문제와 동일하게 mysql 접근 불가상태

![[Pasted image 20240714214820.png]]
pod의 로그를 보면 running 중이긴 한것을 알 수 있다. 
pod 문제가 아닌 연결부분의 문제인가


![[Pasted image 20240714215030.png|400]]
![[Pasted image 20240714215042.png|400]]

서로 연결되어야 할 pod와 svc의 selector label이 다름을 볼 수 있었다.


**Troubleshooting Test 4:** The same 2 tier application is deployed in the `delta` namespace. It must display a green web page on success. Click on the `App` tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.
![[Pasted image 20240714215347.png|400]]

![[Pasted image 20240714215308.png]]
Access denied!

![[Pasted image 20240714215509.png]]
db_user가 root가 아닌 sql-user로 되어있었음.

```
k edit deploy webapp-mysql 
```


**Troubleshooting Test 5:** The same 2 tier application is deployed in the `epsilon` namespace. It must display a green web page on success. Click on the `App` tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

4번과 동일한 문제로 수정 후 
![[Pasted image 20240714215819.png]]
root user도 access denied!

DB 자체 pod를 살펴보니
k describe po mysql 
![[Pasted image 20240714215906.png|400]]
password가 다름.

```
k edit po mysql 
k replace --force -f /tmp/kubectl-edit-3648967467.yaml
```


**Troubleshooting Test 6:** The same 2 tier application is deployed in the `zeta` namespace. It must display a green web page on success. Click on the `App` tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

-> 502 bad gateway 발생

![[Pasted image 20240714220318.png]]
앞단의 web-service nodeport 잘못지정.

![[Pasted image 20240714220347.png]]
또 유저 다름

```
k edit deploy webapp-mysql
```

그 다음 앞과 동일하게 db password 다름으로 
mysql pod 변경하여 해결


### Control Plane Failure

check node status
- `k get node`
- `k get pod

check controlplane pods
- `k get pods -n kube-system`

check controlplane services
- `service kube-apiserver status`
-  `service kube-controller-manager status`
-  `service kube-scheduler status`
-   `service kubelet status`
-   `service kube-proxy status`

check service logs
- `k logs kube-apiserver-master -n kube-system`
- `sudo journalctl -u kube-apiserver`

#### test

1. The cluster is broken. We tried deploying an application but it's not working. Troubleshoot and fix the issue.


![[Pasted image 20240714221544.png|400]]
```
k describe deploy app
```
![[Pasted image 20240714221649.png|500]]
ScalingReplicaSet

rs describe
![[Pasted image 20240714221721.png]]

pod describe
![[Pasted image 20240714221750.png|500]]
![[Pasted image 20240714221856.png|500]]
node 에 none으로 된 것을 볼 수 있다. 파드가 노드에 할당되지 않은 것을 의심

![[Pasted image 20240714221407.png]]
kube-scheduler describe
![[Pasted image 20240714222025.png]]

![[Pasted image 20240714222105.png|500]]
발견

![[Pasted image 20240714222202.png]]


2. Scale the deployment `app` to 2 pods.
![[Pasted image 20240714222643.png|500]]


3. Even though the deployment was scaled to 2, the number of `PODs` does not seem to increase. Investigate and fix the issue.

deploy가 2개 replica로 설정했지만 1개만 running임을 볼 수 있었다.
deploy 살펴본 결과,
![[Pasted image 20240714223110.png]]
![[Pasted image 20240714222909.png]]
kube-controller-manager가 crashloopbackoff 상태임을 알 수 있었다.

![[Pasted image 20240714223219.png]]
![[Pasted image 20240714223325.png]]XXXX 오타 발견


4. Something is wrong with scaling again. We just tried scaling the deployment to 3 replicas. But it's not happening.

![[Pasted image 20240714223812.png]]
3번과 동일하게 로그확인

/etc/kubernetes/pki/ca.crt: no such file or directory

```
cat /etc/kubernetes/manifests/kube-controller-manager.yaml 
```
![[Pasted image 20240714223949.png|400]]
volume 부분에 path가 잘못 지정되어있음을 확인

![[Pasted image 20240714224041.png|500]]
실제 mounts한 path는 `/etc/kubernetes/pki`


### Worker Node Failure

check node status
![[Pasted image 20240714224415.png|600]]
status, message를 통해 노드의 상태를 알수 있다.

Status는 True , False , Unknown

OutOfDisk: 디스크 공간 부족
DiskPressure: 메모리 부족
PIDPressure: 디스크 용량부족
PIDPressure: 프로세스 초과

crash로 인해 워커 노드가 마스터와의 통신을 중단하면 이러한 상태는 Unknown으로 설정된다. 
LastHeartbeatTime 필드를 확인하여 노드가 crash된 시간 체크.


check node

node가 온라인 상태이거나 crash된 경우
노드 자체의 상태 확인한다.
노드에 가능한 cpu 메모라와 디스크 공간이 있는지 확인
![[Pasted image 20240714221028.png|500]]

check kubelet status
![[Pasted image 20240714221052.png|500]]

check certificates
![[Pasted image 20240714221104.png|500]]


#### test
1.Fix the broken cluster

![[Pasted image 20240714225219.png|400]]
![[Pasted image 20240714225205.png]]
kubelet 셋팅과정 중에 notready

`ssh node01`
노드 접속 

![[Pasted image 20240714225452.png]]
kubelet status 확인시 inactive 상태임을 확인
(`service kubelet status`)

kubelet start 해줌
#### service, systemctl 주요 차이점

- **시스템 호환성**: `systemctl`은 `systemd`를 사용하는 최신 시스템에서 사용됩니다. `service`는 SysVinit과 Upstart를 사용하는 시스템에서 사용될 수 있습니다.
- **정보의 상세성**: `systemctl`은 서비스 상태에 대한 자세한 정보를 제공하며, 로그와 상태에 대한 심층 분석이 가능합니다. `service`는 보다 기본적인 상태 정보만 제공합니다.
- **기능**: `systemctl`은 서비스의 활성화, 비활성화, 재시작, 상태 확인 등 다양한 기능을 지원하며, `service`는 주로 상태 확인과 시작, 중지 등의 기본적인 기능만 지원합니다.

현대적인 Linux 배포판에서는 `systemctl`을 사용하는 것이 더 일반적이며, `service` 명령어는 이전 시스템에서 사용될 수 있습니다.

2. The cluster is broken again. Investigate and fix the issue.

![[Pasted image 20240714225645.png]]

![[Pasted image 20240714225814.png]]
code=exited 임을 볼 수 있다.

`journalctl -u kubelet`
로그확인
![[Pasted image 20240714230109.png|400]]
![[Pasted image 20240714230755.png]]

`vi /var/lib/kubelet/config.yaml`
파일의 CA path 부분 수정.

![[Pasted image 20240714231000.png]]


3.The cluster is broken again. Investigate and fix the issue.

![[Pasted image 20240714232148.png]]
control plane port는 6553이 아닌 6443

kubelet이 사용하는 config 파일에 포트 수정.
```
cat /etc/kubernetes/kubelet.conf
vi /etc/kubernetes/kubelet.conf
service kubelet start
```


### Network Truobleshooting

**3. Calico :**

   To install,

   `curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml -O`

  _Apply the manifest using the following command._

      `kubectl apply -f calico.yaml`

CKA 및 CKAD 시험에서는 CNI 플러그인을 설치하라는 메시지가 표시되지 않는다.
그러나 만약 요청을 받으면 설치할 수 있는 정확한 URL을 제공받게 된다.


#### Troubleshooting issues related to coreDNS

1. CoreDNS pod가 pending 상태 -> 네트워크 플러그인 확인
2. CoreDNS pod가 CrashLoopBackOff 또는 Error 상태 -> 설치된 OS와 도커의 버전 및 권한 문제
	- Docker 새 버전으로 업그레이드
	- SELinux 비활성화
	- coredns deployment의 allowPrivilegeEscalation을 true로 변경
```
kubectl -n kube-system get deployment coredns -o yaml | \
sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \
kubectl apply -f -
```

d)Another cause for **CoreDNS** to have CrashLoopBackOff is when a **CoreDNS** Pod deployed in Kubernetes detects a loop. ????
- Add the following to your kubelet config yaml: 
  `resolvConf: <path-to-your-real-resolv-conf-file>`
  This flag tells **kubelet** to pass an alternate **resolv.conf** to Pods.
  For systems using **systemd-resolved**, **/run/systemd/resolve/resolv.conf** is typically the location of the **_"real" resolv.conf_**, although this can be different depending on your distribution.
- Disable the local DNS cache on host nodes, and restore **_/etc/resolv.conf_** to the original. 
- A quick fix is to edit your **Corefile**, replacing forward **_. /etc/resolv.conf_** with the IP address of your upstream DNS, for example forward **. 8.8.8.8**. But this only fixes the issue for **CoreDNS**, **_kubelet_** will continue to forward the invalid **_resolv.conf_** to all default dnsPolicy Pods, leaving them unable to resolve DNS.

3. CoreDNS 와 관련된 pod가 모두 정상 -> 서비스 엔드포인트, selector, port를 확인

`kubectl -n kube-system get ep kube-dns`

서비스에 대한 엔드포인트가 없는 경우, 서비스 selector, port를 확인


#### Kube-Proxy

kube-proxy는 클러스터의 각 노드에서 실행되는 네트워크 프록시이며, 노드에서 network rules를 유지 관리한다. 
이러한  network rules은 클러스터 내부 또는 외부 파드로의 네트워크 통신을 허용한다.
kubeadm으로 구성된 클러스터에서는 **_daemonset_ 으로 kube-proxy를 찾을 수 있다.
kubeproxy는 각 서비스와 관련된 서비스 및 엔드포인트를 watching 한다.

If you run
`kubectl describe ds kube-proxy -n kube-system`
you can see that the **kube-proxy** binary runs with following command inside the kube-proxy container.

```shell
Command:
	/usr/local/bin/kube-proxy
	--config=/var/lib/kube-proxy/config.conf
	--hostname-override=$(NODE_NAME)
```

`/var/lib/kube-proxy/config.conf` 에서 config 가져오고

we can override the hostname with the node name of at which the pod is running.

In the config file we define the **clusterCIDR, kubeproxy mode, ipvs, iptables, bindaddress, kube-config** etc.

#### Troubleshooting issues related to kube-proxy
- kube-proxy 파드 확인
- kube-proxy 로그 확인
- `/var/lib/kube-proxy/config.conf` 파일 확인
- configmap 확인
    - 만약 kube-proxy의 configmap에서 지정한 경로와 kube-proxy 파드 내 경로가 다르다면 수정 필요
- kube-proxy가 컨테이너 안에서 실행하고 있는지 확인
    - `netstat -plan | grep kube-proxy`


#### test


`**Troubleshooting Test 1:**` A simple 2 tier application is deployed in the `triton` namespace. It must display a green web page on success. Click on the app tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

![[Pasted image 20240715000132.png|300]]
![[Pasted image 20240715000153.png]]

pod describe
![[Pasted image 20240715000122.png]]
network weave가 failed 임을 확인.

```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

![[Pasted image 20240715000339.png]]
--> **Network Plugin 이슈**\


`**Troubleshooting Test 2:**` The same 2 tier application is having issues again. It must display a green web page on success. Click on the app tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

![[Pasted image 20240715000508.png]]
![[Pasted image 20240715000539.png]]

kube-proxy 가 cashloopbackoff 상태임을 볼 수 있다.

```
k describe po kube-proxy-n2dvv -n kube-system
```
![[Pasted image 20240715000711.png]]

![[Pasted image 20240715000811.png]]
로그를 살펴봤을때 config 를 지정한 path에 그런 파일이 없다고 뜬다.

실제 configmap을 살펴보니
![[Pasted image 20240715000939.png]]
config map에는 kubeconfig.conf로 되어있지만,
정작 kube-proxy command 에는
![[Pasted image 20240715001145.png|400]]
configuration.conf로 잘못지정되어있었다.

```
k edit daemonset.apps/kube-proxy -n kube-system
```

-->**Kube Proxy이슈**