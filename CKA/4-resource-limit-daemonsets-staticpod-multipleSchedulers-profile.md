
## Resource requirement and limit

pod가 노드에 배치될때 그 노드의 리소스를 사용하게 되는데 노드의 리소스는 한정적이기때문에 kube-scheduler가 pod를 배치할때 노드의 남은 리소스양을 고려하여 배치한다. 모든 노드의 리소스가 사용중일 경우 pod는 배치되지 못하고 pending 상태로 남게되며 event: insufficient cpu 등의 문구를 볼 수 있을 것이다.

### Resource requests
pod를 생성할때 최소한의 CPU, Memory를 지정할 수 있다. -> reource requests
컨테이너에 대한 리소스 요청으로 최소한의 cpu와 memory를 요청한 것이다. 스케줄러는 파드를 노드에 배치할때 이를 고려하여 배치한다.  

pod 정의시, `spec.containers.resources.requests.cpu` 에 정의하면 된다.
`cpu:1`의 의미는 무엇일까?
- 1 AWS vCPU
- 1 GCP Core
- 1 Azure Core
- 1 Hyperthread

`memory`
1G = 1,000,000,000 bytes
1M = 1,000,000 bytes
1K = 1,000 bytes
1Gi = 1,073,741,824 bytes
1Mi = 1,048,576 bytes
1Ki = 1,024 bytes

### Resource limit
기본값으로는 컨테이터는 노드에서 소비할 수 있는 리소스의 한계는 없다. 필요한만큼 리소스를 소비할 수 있다. 그럴경우 계속해서 사용량이 증가하여 노드를 죽일 수도 있게 된다. 따라서 limit을 걸어 최대 리소스를 제한 할 수 있다.
즉, 리소스 한도를 지정할 수 있다. 
requests와 마찬가지로 `pod 정의시, spec.containers.resources.limits.cpu`에 정의하면 된다.
containter이 여러개라면 각각 지정해 설정한다.

*cpu*: 컨테이너는 제한된 cpu 이상의 리소스를 쓸 수 없다.
*memory*: 하지만 메모리는 한도 이상의 리소스를 쓸 수 있는데, 이때 계속해서 한도보다 초과하면 *OOM*(out of memory)로 Terminate 된다.

#### default behavior 
기본적으로 default로 리소스의 request, limit이 없기때문에 따로 설정을 하지 않은 상태로 과도한 리소스를 사용하면 다른 파드나 프로세스를 죽일 수도 있으니 주의해야한다.

(CPU)
- no request, no limits -> 노드의 리소스를 모두 사용하고 다른 파드에도 영향을 미칠 수 있다.
- no request, limits -> request = limit
- request, limit -> request만큼 보장되면서 최대 limit까지만 가능하다. 
- request, no limit -> request만큼 보장되지만, 리소스 제한이 되진 않는다.
  -> 노드의 모든 파드에 requests가 설정되어있어야, 다른 파드에 대한 제한들이 없을때도 파드가 자원을 보장 받을 수 있다. 즉, 한 파드가 과도한 cpu를 사용하여 다른 파드의 requests 만큼에 위협을 끼치게 되면 더이상의 CPU를 쓸 수 없도록 조절하게 된다.
  
(Memory)
다른 부분은 동일하지만,
- request, no limit -> request만큼 보장되지만, 리소스 제한이 되진 않는다.
  -> 노드의 모든 파드에 requests가 설정되어있어도 CPU와 달리 Memory는 조절할 수 없기 때문에 하나의 파드가 과도한 메모리 리소스를 써서 다른 파드의 requests를 위협한다면 그 파드를 terminate되는 방법밖에 없다.

### LimitRange
`kind: LimitRange`
기본값을 정의할 수 있고, 범위로 limit을 제한 할 수 있다.
*namespace 레벨에서도 적용 가능하다.*
pod가 생성될때 적용 -> 즉, 값을 중간에 변경해도 이미 파드가 생성되어있으면, 파드는 그 영향을 받지 않는다.

### Resource Quotas
`kind: ResourceQuota`
namespace 레벨에서 엄격한hard request, limits을 제한하는 방법.

#### test

k pod edit으로 resource 수정 불가능.
![[Pasted image 20240401001000.png]]

## DaemonSets
![[Pasted image 20240401001245.png]]
daemonsets은 복제집합이라 볼 수 있다. 여러개의 파드를 배포하도록 도와주는데 클러스터 내의 모든 노드에 배포하고 싶은 파드가 있는 경우 daemonsets을 사용한다. 그럴 경우, 클러스터에 새로운 노드가 생성되면 해당 파드가 복제되어 추가된다. 노드가 제거될경우, 파드도 자동으로 제거된다.
죽, daemonset은 파드의 복제본을 클러스터 내 모든 노드에 항상 존재하도록 한다.

![[Pasted image 20240401001252.png]]
사용예시로는 모니터링 에이전트나 log collector를 들 수 있다. 파드들을 잘 모니터링 할 수 있도록 모든 노드마다 배치해야하기 때문이다. 새로운 노드가 뜨면 자동으로 생성되어야하기 때문이다. 노드가 제거될때도 따로 제거할 필요없이 daemonsets이 이를 알아서 해준다.

![[Pasted image 20240401001304.png]]
또한 쿠버네티스 아키텍처에서 클러스터 내 모든 노드에 kube-proxy가 필요한데 이또한 daemonsets이 사용된다.
ex) networking

![[Pasted image 20240401001314.png]]
replicaset의 yaml 방식과 유사하다.
selector와 template.metadata.labels 를 통해 daemonset과 pod를 연결한다.

`kubectl get daemonsets`

![[Pasted image 20240401002211.png]]
1.12 버전까지만 해도 daemonsets의 작동 방식은 파드에 배치될 노드의 속성을 지정하여 각 노드에 직접 파드가 배치되었다면,
그 이후부터는 파드 스케줄링을 위해
NodeAffinity와 default scheduler를 사용하여 파드가 배치되는 방식을 사용한다.

## Static Pod

![[Pasted image 20240401003137.png|500]]
Kubelet 은 kube-apiserver 에 명령에 따라 노드에l 파드을 load하는데, 이는 kube-scheduler 의 결정에 근거한것으러, 이에 대한 데이터는 ETCD에 저장된다.

![[Pasted image 20240401003201.png|600]]
만약 이러한 마스터노드가 없다면 kubelet은 파드를 실행시킬 수 있을까? 가능하다.
kubelet은 독립적으로 노드를 관리할 수 있다.
node에는 kubelet이 설치되어있고 컨테이너를 작동할 docker가 존재한다.
그렇다면 api-server가 없는데 어떻게 파드의 상세정보를 전달할까? 파드를 생성하기 위해선 yaml으로 파드의 상세정보가 필요하다.
kubelet은 파드의 정보를 저장하는 서버 디렉토리 -> `/etc/kubernetes/manifests` 파드 정의파일을 읽을 수 있다. 이 디렉토리에 파드 정의파일을 넣으면 가능하다.
kubelet은 주기적으로 이 디렉토리의 파일을 확인하고, 파드를 생성한다. 생성뿐만 아니라 해당 파일이 update되면 수정하여 재배포 하고, 죽을 경우 재시작을 하는 등의 관리또한 kubelet이 한다 해당 디렉토리에서 파일을 삭제하면 파드 또한 자동으로 제거된다.
이렇게 kubelet이 직접만든 이 파드들은 api-server의 간섭이나 나머지 클러스터 구성요소들의 간섭을 받지 않는다.
이를 -> *static pod*라고 한다.
![[Pasted image 20240401004442.png|600]]
`config` 옵션을 사용해 파일을 지정하고 파일 내 `staticPodPath`를 정의하는 방법이 있다

- kube-apiserver는 static pod를 인식하기 때문에 `kubectl get po`를 해도 static pod 목록을 볼 수 있다.
- apiserver가 수정 삭제할 수는 없고 직접 manifest 파일 위치에 가서 파일을 수정해야만 가능하다.

그렇다면 Static Pod를 왜쓸까?
![[Pasted image 20240401003257.png|600]]
master노드의 다음과 같은 구성요소도 결국 static pod로 파드정의한 파일을 해당 마스터노드의 manifests에 넣어 생성한다. 
-> 바이너리 파일을 설치할 필요도 서비스가 다운될까 걱정할 필요도 없게된다. 이중 하나가 내려가도 static pod이기 때문에 kubelet이 재시작 하게된다.

![[Pasted image 20240401005314.png|600]]

### test
![[Pasted image 20240401005549.png]]
static pod: 이름 뒤에 노드이름이 붙는다.

![[Pasted image 20240401010025.png|500]]
static pod definition files 확인
-> `/etc/kubernetes/manifests`
![[Pasted image 20240401010118.png]]

staticpod 삭제시, 실제 manifest 파일을 삭제해야지만 가능하다. 
-> `k get node` -> 해당 node로 ssh 접속하여 -> manifest 파일 삭제


## Multiple Schedulers

쿠버네티스는 한번에 여러 스케줄러를 가질 수 있다.
![[Pasted image 20240401011331.png]]
간단한 방법으로는 scheduler 바이너리 파일을 직접 설치하는 것이다.
![[Pasted image 20240401011349.png]]
하지만 대부분의 사용자들은 pod로 scheduler를 배포한다.
config 옵션으로 `--config=/etc/kubernetes/my-scheduler-config.yaml` 을 지정해주면 그 yaml에 정의된 profiles의 `schedulerName` 이 스케줄러의 이름이된다.
`leaderElection` 이란 옵션이 있는데 여러 마스터 노드에서 사용될때 고가용성을 위해 사용되는 것이다.
동일한 스케줄러의 복사본 여러개가 다른 노드에서 실행될 경우 한번에 하나만 활성화될 수 있기 때문에 리더를 선출하는 설정이다.

document
https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/

![[Pasted image 20240401011424.png]]

![[Pasted image 20240401011439.png]]

### test
![[Pasted image 20240401013257.png]]

-  new scheduler 생성중 image 부분 누락- default scheduler image 확인
  ![[Pasted image 20240401013616.png]]

## Scheduler Profiles
1. Scheduling Queue
	파드 생성이 시작되면 가장 먼저 `Scheduling Queue`에 들어간다.
	이때 파드마다 우선순위가 정해져 있는데 -> `spec.priorityClassName: ` -> `kind: PriorityClass`를 생성해서 이를 priorityClassName에 지정.
	이렇게 우선순위가 높은 파드는 가장 먼저 대기자 명단에 오르고 스케쥴링된다.
2. Filtering
   실행할 수 없는 파드는 여기서 걸러진다. (예를 들어, 노드에 리소스가 남아있지 않으면 걸러진다.)
3. Scoring
   스케줄러는 각 노드마다 점수를 매기는데, 파드를 배치하고 남을 리소스를 근거로 점수를 부여한다.
   점수가 높은 노드가 선택된다.
4. Binding
   점수가 높은 노드에 파드가 배치된다.

### Scheduling Plugins
위의 과정들은 플러그인으로 구성되어있다.

- Scheduling Queue
	- PrioritySort: 파드의 우선순위에 따라 순서대로 정렬
- Filtering
	- NodeResourceFit: 파드에서 요구하는 자원이 충분한 노드를 식별. 부족한 노드 식별
	- NodeName: pod의 설정중에 spec.NodeName으로 배치할 노드 filtering
	- NodeUnschedulable: node 설정에 Unschedulable 값이 true일 경우 스케쥴되지 않는 노드로 filtering
- Scoring
	- NodeResourceFit : 가장 적절한 노드에 점수 부여
	- ImageLocality: 사용되는 컨테이너 이미지를 가진 노드에 점수 부여
- Binding
	- DefaultBinder: binding 매커니즘 제공

![[Pasted image 20240401015203.png]]
다음과 같은 지점-Extension Points 에 플러그인을 지정할 수 있다. 플러그인을 생성하여 각각 포인트에 배치 하여 연결할 수 있다.
