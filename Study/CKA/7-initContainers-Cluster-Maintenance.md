
### initContainers
https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
Pod의 container들이 실행되기 전에 실행되는 특수한 초기 컨테이너이다. 
예를들어 웹 어플리케이션이 시작되기전에 logging agent를 실행하거나 웹어플리케이션에는 없는 유틸리티 또는 설정을 실행시키는 경우에 사용된다.

한개또는 여러개의 Container를 initContainer로 추가 할 수 있으며, 순차적으로 실행된다. - one at a time in swquential order.
initContainer 중 하나라도 실패하면 pod는 재실행된다. initContainer들이 모두 succeed 될때까지.

## Cluster Maintenance
### OS Upgrade

만약 노드 하나가 다운된다면 어떻게 될까?

노드가 다운되었을때 5분이상이 지나면 쿠버네티스는 노드가 다운 되었다는 것으로 여기고 
pod를 terminated 시킨다.
`kube-controller-manager --pod-eviction-timeout=5m0s`
로 5분이 기본값으로 설정되어있는데 노드가 다운된지 5분뒤에 pod를 evict 시킨다는 의미이다.
노드가 오프라인 될때마다 마스터 노드는 최대 5분까지 기다립니다.

이때, replica set으로 설정된 pod는 다른 노드에 다시 recreate 될테지만,
replica set으로 설정되어있지 않은 pod는 노드가 다운되어 파드가 다 내려간 다음 다시 노드가 online 상태가 되어도 다시 recreate될 수 없다. 해당 pod는 사라지게 되는 것이다.

따라서 노드를 다운시킬때 안전한 방법은 아래와 같다.

![[Pasted image 20240421185638.png|500]]

*drain* 명령어로 노드에 배치되어있는 파드들을 정상종료하고 다른 노드에 recreate한다.
정상적으로 모두 recreate 되었다면, 노드를 재부팅한다.
이떄, *cordon* 이란 명령어도 존재한다. 이 명령어는 기존의 파드를 이동시키진 않고 단순히 해당 노드에 새포드가 스케쥴링 되지 않도록 설정한다.
재부팅 이후, 노드가 다시 온라인 상태로 돌아오면 이제 파드를 다시 배치될  있도록
*uncordon* 명령어로 해지시켜주면 다시 파드가 배치될 수 있는 상태로 돌아오게 된다.
*이때 이전에 다른 노드에 생성된 파드가 다시 돌아가지는 않는다. 이제부터 새로 생성될 파드가 배치되는 것일 뿐이다.*

![[Pasted image 20240421191352.png]]
daemonset은 노드가 처음뜰때 뜨는 파드들로 cannot delete error가 발생한다. - `--ignore-daemonsets`
*Cannot delete pods not managed by replicationController, ReplicasSet, Job, DamonSet or StatefulSet*
replicaset으로 관리되지 않는 pod들도 cannot delete 에러가 발생한다.

#### 현업발생이슈
프로젝트에서 eks 내부통신을 위한 service ipv4 range: 172.a 대역으로 설정된 상태에서 
따로 karpenter clusterDNS를 설정하지 않아서.
karpenter default clusterDNS값인 172.20으로 노드가 올라가 다시 삭제후 생성해야 되는 상황 발생.

-> cordon, drain 명령어를 사용하여 안전하게 노드 재부팅

```
kubectl cordon (node명)
```
해당 노드의 STATUS: SchedulingDisabled 상태
해당 노드로 더이상 스케쥴 되지 않도록 막는 것.
```
kubectl drain (node명) --delete-local-data --ignore-daemonsets
```
--delete-local-data: 노드마다 로컬 데이터를 삭제하여 종속된 부분이 없도록 조치. 
--ignore-daemonsets: daemonsets은 노드가 처음 뜰때 데몬셋으로 뜨는 파드로, 다른 노드로 옮길 필요가 없다. daemonsets은 각 노드들에 하나씩 실행되어야 하는데 다른 노드로 옮겨 생성하면 중복생성으로 에러가 발생하기 때문에 해당 옵션을 주지 않으면 에러가 발생할 것이다.

### Kubernetes Release
 ![[Pasted image 20240421193838.png|500]]
 다른 애플리케이션처럼 쿠버네티스도 표준 소프트웨어 release 버전 관리 절차를 따른다.
 몇달에 한번씩 소규모 release를 통해 새로운 기능을 선보인다.
또한 새로운 버그를 고치거나 했을때, alpha -> beta 순으로 태그를 붙여 main 릴리즈까지 가기도 한다.

### Cluster Upgrade Process
![[Pasted image 20240421194858.png|500]]
외부 구성요소를 제외한 핵심 control plane 컴포넌스만 봤을때,
모두 같은 버전을 갖는게 의무인가? No
단, kube-apiserver는 다른 컴포넌트들과 모두 통신하기 때문에, 다른 컴포넌트들은 kube-apiserver의 버전보다 높아서는 안된다.
(kubectl은 x+1 >x-1 까지 가능)

![[Pasted image 20240421195258.png|500]]
다음과 같이 1.13까지 버전이 나오게 되고 1.10이 un-supported 될 예정일때, 버전 업그레이드를 해야될텐데,
이때 한번에 13까지 업그레이드 하는 것이 아닌 한단계씩 이루어져야 한다.

#### 현업발생이슈
EKS 업그레이드
- AWS 콘솔
  AWS Console의 EKS 대시보드에서도 클러스터 버전을 업그레이드 할 수 있다. 단점으로는 업그레이드 중 진행되는 내용을 확인할 수 없다는 점과 한 단계씩 순차 업그레이드(1.25에서 1.27로 업그레이드가 바로 불가능하고, 1.25에서 1.26을 거쳐 1.27로 업그레이드해야한다.)만 가능하다.
- 새로운 버전의 EKS를 생성하고 마이그레이션
  원하는 버전의 클러스터를 미리 생성하기 때문에 업그레이드를 가장 안전하게 할 수 있다. 또한, 장애 발생 시 트래픽을 원복하면 되기 때문에 원복도 간단하고 빠르게 할 수 있다. 새로운 버전의 클러스터를 생성하는 것이기 때문에 순차 업그레이드뿐만 아니라 여러 버전을 거쳐 업그레이드가 가능하다. (1.25에서 1.27로 업그레이드가 가능.)
  고려해야 할 부분으로는 haproxy, nginx, route53 등을 통해 트래픽을 전환해야 하며, 새로운 버전의 클러스터로 들어오는 트래픽의 정상 여부를 어떻게 판단할지 고민해야한다.
- eksupgrade CLI
  eksupgrade CLI는 파이썬으로 개발되었으며, pip 명령어를 통해 CLI를 쉽게 설치할 수 있다.
  2023년도 2월에 AWS에서 공개한 AWS의 관리형 서비스인 EKS의 업그레이드를 CLI 형태로 제공하는 eksupgrade CLI
  - *이슈*
    karpenter로 생성된 node에서 업그레이드 에러 - ASG(Auto Scaling Group)에 포함된 값이 없기 때문.  
	karpenter는 ASG를 통해 노드를 프로비저닝 하는 방식이 아닌 karpenter가 노드를 바로 프로비저닝 하기 때문. 실제 eksupgrade CLI 파이썬 코드에서도 ASG 참조해 node group 업그레이드하는 것을 알 수 있다. 


#### kubeadm을 활용한 cluster upgrade

*먼저 마스터 노드 업그레이드 후 워커노드 업그레이드*
마스터 노드 업그레이드되는 동안 control plane의 구성요소는 잠시 다운될 것이다.
마스터 노드가 다운된다고 워커노드의 파드들이 죽는 것 아닐 것이다. just, 관리기능들이 다운된 것이다.
kubectl 을 통해 클러스터에 엑세스 할 수 없거나 새로운 파드 배포 및 삭제 등은 불가능 할것이다.
다시 마스터노드가 올라오면 다시 기능이 원상복구된다.

**worker node upgrade**
![[Pasted image 20240421201354.png|500]]
한번에 다 업그레이드 한다. 파드들이 모두 다운되면 사용자는 앱에 접속할 수 없으며, 업그레이드 되는 시간동안에는 순단이 발생한다. 

![[Pasted image 20240421201409.png|500]]
한번에 노드 하나씩 업그레이드 한다.

![[Pasted image 20240421201439.png|500]]
클러스터에 새노드를 추가한다.
클라우드 환경에서 특히 편리하다. 새노드를 프로비전하고, 이전 노드의 리소스를 모두 옮긴 다음 삭제한다.

![[Pasted image 20240421201502.png|600]]
kubelet 버전도 업그레이드는 수동으로 해야된다. *kubeadm이 해주지 x*

![[Pasted image 20240421201513.png|500]]
먼저 kubeadm 를 버전 업그레이드 한다음.
클러스터 업그레이드를 한다.
k get nodes를 보면 11버전임을 볼 수 있는데,
이는 서버자체의 버전이 아닌 Kubelet 버전으로 kubelet 버전을 업그레이드 하면된다.

Kubeadm 이경우에는 마스터 노드에 kubelet 이 위치해있기때문에 마스터 노드에서 Kubelet을 12로 버전업그레이드 한다음 restart를 시킨다.


![[Pasted image 20240421201542.png|500]]
```
apt-get upgrade -y kubeadm=1.12.0-00
apt-get upgrade -y kubelet=1.12.0-00
kubeadm upgrade node config --kubelet-version v1.12.0
systemctl restart kubelet
```

#### test
https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
![[Pasted image 20240421210454.png]]
![[Pasted image 20240421210515.png]]
![[Pasted image 20240421211329.png]]
이어서 kubelet, kubectl도 업그레이드
restart kubelet
https://v1-29.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
????

in worker node
![[Pasted image 20240421212621.png]]
이어서 kubelet, kubectl도 업그레이드
restart kubelet

### Backup and Restore
![[Pasted image 20240421212950.png|400]]
쿠버네티스 리소스를 정의하여 생성하면,
클러스터와 관련된 모든 정보는 ETCD 클러스터에 저장된다.
이를 영구적인 저장소에 저장하고 싶으면 추가적인 셋팅을 하면된다.

imperative하게 리소스를 생성하기도 declarative하게 리소스를 생성하여 github 저장소에 올려 보관하거나 한다.
github에 정의파일들이 모두 저장되어있다면 리소스가 날라가도 다시 원복할 수 있게 된다. 따라서 선언적 방법이 선호되곤 한다.
![[Pasted image 20240421213412.png|500]]
만약 누군가가 어디에 기록하지 않고 imperative한 방법으로 리소스를 생성했다면?
kube api 서버를 쿼리하여 백업을 볼 수 있다.
kubectl 을 이용해 kube API서버를 쿼리하거나 kube API 서버에 직접 access 리소스 구성을 저장 할 수도 있다.

### ETCD
ETCD는 클러스터에 저장되어있는 리소스에 관한 정보들을 저장하고 있다. 클러스터 자체의 정보나 내부에서 생성된 리소스들을 모두 포함해서말이다. ETCD를 활용해서 자체 백업하는 걸 선택할 수 있다.

![[Pasted image 20240421213710.png|600]]
마스터 노드에 ETCD는 호스트되어있고, 설정파일을 보면 어느 디렉토리에 저장되는지 볼 수 있다.

![[Pasted image 20240421214119.png|600]]
또한 빌트인으로 *스냅샷 솔루션*도 제공한다.
스냅샷을 떠서 저장할 수 있고
![[Pasted image 20240421214240.png|600]]
이를 복원하는 것도 가능하다.
복원프로세스는 etcd 클러스터를 재시작해야하는데 kube-apiserver가 depend 되어있기 때문에
kube-apiserver를 stop하여 depend 를 해제하고
restore명령어로 복원한 다음 etcd를 restart 하고, 다시 kube-apiserver를 start하면 가능하다.

*managed 쿠버네티스 환경을 사용하는 경우 ETCD 클러스터에 엑세스 하지 않기때문에 kubeapi 서버를 쿼리하는 백업이 더 나을 수 있다.*

#### etcdctl
To make use of etcdctl for tasks such as back up and restore, make sure that you set the ETCDCTL_API to 3.
```
export ETCDCTL_API=3

etcdctl snapshot save -h
etcdctl snapshot restore -h
```


