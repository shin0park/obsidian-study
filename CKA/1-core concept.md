
# kubernetes Architecture

![[Pasted image 20240310201619.png|500]]

크게 Master 와 Worker 노드로 나뉜다. 쿠버네티스의 궁극적인 목표는 컨테이너로 호스팅하는 것이다. 이렇게 호스팅된 여러 컨테이너들을 잘 관리하기 위해서는 중앙에서 관리하는 부분이 필요하며 이를 Master 노드에서 이뤄지게 된다. Worker 노드는 말그대로 어떠한 애플리케이션이 컨테이너 형태로 호스팅되는 노드를 뜻하며 마스터 노드는 이를 관리한다. 

마스터 노드(Control Plane) 에는 관리에 필요한 여러 기능들이 존재한다. 
**ETCD 클러스터** 
: key-value 데이터베이스 (어떤 컨테이너가 어느 노드에 있고 언제 적재되었는지 등의 데이터들을 저장)

**kube-scheduler**: 어떤 노드에 파드를 놓을건지 결정하 스케쥴러이다. 컨테이너 리소스 요구사항이 워커노드 용량, 다른 정책이나 제약들을 보고 판단하여 가장 올바른 노드에 배치하도록 한다.

kube Controller Manager: 
-  Controller Manager
- Node Controller : 새 노드를 클러스터에 온보딩, 노드의 사용불가능 destroy들을 컨트롤
- Replication Controller: replica set으로 지정한 수만큼의 파드가 적재되어 실행되고 있는지 컨트롤

how to comunicate?
=> **kube apiserver**
kube 클러스터 내의 모든 작업을 오케스트레이션 하는 역할
컨트롤러, 스케쥴러, etcd등 모두 kube apiserver와 통신하며 작업한다.

**kubelet**
worker노드에서는 각각 노드에 kubelet이 존재하고, 이 kubete이 kube apiserver와 통신하며 마스터 노드에서 내려온 명령들을 실제 워커노드에서 실행한다. pod를 생성하거나 파괴하는 등의 작업을 한다.

**kube-proxy**
하나의 워커노드 안에서도 여러 파드가 떠있을텐데 이 파드들간의 통신은 어떻게 할까 만약 어떤 애플리케이션이 컨테이너형태로 띄워져있고 디비가 다른 컨테이너로 띄워져 있다면 말이다. 이때 kube-proxy를 통해 노드안의 파드끼리 통신을 하게 된다.

# Docker vs Containerd

![[Pasted image 20240310204101.png]]
쿠버네티스 -도커만 지원하다 다른 컨테이너 런타임과도 작업이 필요해졌다. 
따라서 **CRI** container runtime interface를 발표.
CRI 는 어떤 벤더든 쿠버네티스의 컨테이너 런타임으로 작업하게 해준다. OCI 표준을 준수하는 한.
OCI: open container Initiative
- imagespec : 이미지를 어떻게 만들어야 하는지.
- runtimespec: 이미지 빌드 방식
이런 OCI 표준을 만들어놓고 누구나 쿠버네티스와 작업할 수 있는 컨테이너 런타임을 만들 수 있도록 했다.

**dockershim**: 
그러나 docker는 CRI 표준을 지원하려고 만든게 아니였지만 docker는 예전부터 쿠버네티스와 같이 사용됐기때문에 이 또한 지원을 해야했다. 이를 위해 나온것이 dockershim.
CRI 밖에서도 도커를 지원하는 임시방편.

docker 요소 중. => containerd -> CRI 호환 가능. 쿠버네티스와 직접적으로 작업가능. 컨테이너는 docker와 별도로 런타임으로 사용가능.

![[Pasted image 20240310204435.png]]
-> 컨테이너는 containerd 런타임과 별도 docker 를 지원하다. dockershim의 지원을 유지하는 노력으로 문제가 더 커지자 도커 자체는 컨테이너 지원 런타임으로 제거되고, containerd 만 지원하는 것으로 변경되었다. 
**도커 자체를 지원  안해도 Docker가 만든 이미지들은 OCI 표준의 imagespec을 따르기 때문에 계속 도커로 만든 이미지는 쿠버네티스에서 사용 가능하다**

## containerd

이전에는 도커의 일부였지만 현재 독립된 프로젝트이다. CNCF 회원

이젠 docker 설치없이 컨테이너 자체 설치 가능. docker안의 기능이 필요한게 아니라면 컨테이너만 설치해도 가능한 것.
### CLI 
ctr : only for debug, 사용자 친화적이지 않다.

![[Pasted image 20240310205546.png]]

nerdctl
도커와 유사.
도커 지원하는 대부분의 옵션 지원. 
supports newest features in containerd

![[Pasted image 20240310205731.png]]

crictl
- 컨테이너 런타임과 상호작용하는데 사용
- 별도 설치
- work across different runtime. 
- used to inspect and debug container runtimes
	- not to create containers ideally -> crictl로 컨테이너를 만들면 kubelet이 자신이 만든게 아니면 모르기 때문에 삭제할 것이기 때문에
	- 
![[Pasted image 20240310210547.png|600]]
docker와의 차이점. crictl은 pod를 인식하기 때문에 Pods 명령어 가능.
과거에 워커노드에서 docker로 했던 작업들, 이젠 crictl 사용하면 됨. 


## ETCD
![[Pasted image 20240310212413.png]]
![[Pasted image 20240310212640.png]]
ETCD Server using 2 API versions - Version 2 and Version 3.
To set the right version of API set the environment variable ETCDCTL_API command
`export ETCDCTL_API=3`
## ETCD in kubernetes

쿠버네티스 환경에서 실행하며 얻게되는 모든 정보를 저장하는데 이러한 정보들을 etcd Server로 부터 온다. 
변화가 발생할때마다 etcd에 저장된 데이터도 업데이트된다.

클러스터 사용방식에 따라 etcd 배포방식이 두가지로 나뉜다. 
1. setup - manual
2. kubeadm

![[Pasted image 20240310213943.png]]
직접 바이너리 설치
마스터 노드에 직접 서비스로 구성.
etcd server ip and port: 2379

![[Pasted image 20240310213949.png]]
kube-system ns 안에 그외 기타 서버까지 배포

![[Pasted image 20240310214433.png]]
HA 환경(가용성)
에서는 마스터노드도 여러개가 될것이고 이에 따라 etcd도 여러개가 되지만,
etcd 인스턴스 들이 서로 알도록 구성한다.


## Kube API Server
![[Pasted image 20240311011941.png|555]]
kubectl 명령어 사용시 kube-apiserver에 명령이 도달하고 Authenticate User, Validate request 단계를 거쳐 사용자를 인증한다.
클러스터에서 변경을 위한 모든 작업의 중심에 있다.
![[Pasted image 20240311012346.png|555]]
인증서 옵션을 포함하는 것을 볼 수 있다.
![[Pasted image 20240311012446.png|555]]
![[Pasted image 20240311012509.png|555]]
프로세스 실행 확인

## Kube Controller Manager
![[Pasted image 20240311012619.png|555]]
지속적으로 모니터링 하고 시스템 전체를 정의한대로 유지시키도록 만드는 것. 
node controller: 정의된 노드들이 정상작동을 지속적으로 하도록 도와주는 역할
replication controller: 정의한 복제본 만큼의 pod가 유지되도록 유지
이외에도 많은 controller로 쿠버네티스 클러스터 환경이 유지된다.

### manual
![[Pasted image 20240311012858.png|555]]
모니터링 관련 설정들을 확인 할 수 있으며
`Default: [*]` 부분 설정을 통해 원하는 리소스 controller만 작동하게 하는 것도 가능하다.
### kubeadm
![[Pasted image 20240311013017.png]]

![[Pasted image 20240311013008.png]]

## Kube scheduler
스케쥴러는 어떤 포드를 어느 노드에 배치할것인지만 결정할 뿐 실제 pod 생성은 해당 노드에 있는 kubelet 이 하는 것이다.

아래의 여러 조건에 따라 가장 적절한 노드에 파드를 배치한다. (1. filter nodes, 2. Rank nodes)
• Resource Requirements and Limits
• Taints and Tolerations
• Node Selectors/Affinity

## Kubelet

![[Pasted image 20240311013343.png|555]]
Master 노드와의 연락망으로 노드마다 kubelet이 존재하며, kube api server와 통신하며 파드 생성 및 삭제 등의 작업들을 진행한다.

![[Pasted image 20240311013350.png|555]]
**클러스터 배포를 위해 kubedam 사용하면 자동으로 kubelet 생성해주지 않는다. 워커 노드에 수동으로 설치해야한다.**


## Kube Proxy
![[Pasted image 20240311013914.png|555]]
클러스터의 각 노드에서 실행되는 프로세스
새 서비스가 생성될때마다 각 노드에 적절한 rule을 만들어 그 서비스로 트래픽을 전달할 수 있게 한다.
-> iptables 사용


## Pod
![[Pasted image 20240310233100.png|600]]
보통 하나의 pod에 하나의 container로 1:1이 일반적이다.  
![[Pasted image 20240310233200.png|600]]
helper container라던가 lifecycle으ㄹ 함께 하고 싶은 죽으면 같이 죽고 생성되면 같이 생성되고 싶은 경우 하나의 파드에 두 컨테이너를 같이 위치시키기도 한다.

파드에 어떤 컨테이너로 구성할것인지 정의하면 그 컨테이너는 기본적으로 같은 네트워크 같은 저장소 네임스페이스를 갖는다.

### imperative 명령형
![[Pasted image 20240310233404.png|600]]
다음 kubectl 명령어로 파드생성이 가능한데 docker hub 에서 이미지를 다운하여 파드에 컨테이너를 띄운다.

### declarative 선언형
![[Pasted image 20240310234115.png|555]]
항상 4개의 필드를 포함한다. 
어떤 리소스를 생성할 것인가에 따른 올바른 api 버전을 지정해야한다.

```
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod myapp
```

## test

해당 pod가 어느 노드에 위치해 있는가?
```
kubectl describe po 
-> 보다 -o wide 가 더 간단.
kubectl get pods -o wide
```

pod yaml 파일 간단 생성 방법
```
kubectl run redis --image=redis123 --dry-run=client -o yaml > redis.yaml
```
 --dry-run=client -o yaml 
 명령어 사용 실제 생성하지 않으면서 yaml 파일 생성 하도록.


## Replica Sets

 pod가 비정상적일때도 사용자가 접속해야 하므로 **고가용성**을 위해 포드의 다중 인스턴스를 실행하도록 복제본을 생성한다. 
### **replication controller**
pod를 한개 갖고 있어도 pod 비정상적일때, replication controller는 복제본을 불러온다. 
항상 1개 이상의 pod 가 실행되는 것을 보장한다. 
![[Pasted image 20240311005003.png|400]]
하나의 노드에서만이 아니라 여러 노드도 포함한다.

### **replication controller**, replica set
용도는 같지만 같은 것은 아니다.

![[Pasted image 20240311005247.png|555]]
![[Pasted image 20240311005352.png|333]]

### Replica set

![[Pasted image 20240311005637.png|333]]
**replication controller 차이점**
- replica set의 경우 apiVersion: apps/v1
- selector 지정 - 매치되는 라벨을 갖는 파드에 한하여 복제본 생성 가능

![[Pasted image 20240311005847.png]]
라벨을 통해 replica set은 어느 파드를 모니터링 할지 알 수 있다.
라벨 이외에 템플릿도 정의해줘야하는 이유는: 추후 비정상 pod가 생길경우 새로 pod를 띄워야하는데 이때 해당 템플릿에 정의된 대로 파드를 생성하기 위함이다.

![[Pasted image 20240311010300.png|555]]
