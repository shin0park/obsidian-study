
### Design a kubernetes cluster
#### Purpose
- Education
	- Minikube
	- single node cluster with kubeadm/GCP/AWS
- Dev/test
	- Multi-node cluster with single master and multiple workers
	- setup using kubeadm tool or GCP/AWS/AKS
- Production
	- Multipel Master node (High Availability)
	-  kubeadm tool or GCP/AWS/AKS
	- upto 5000 node
	- upto 150,000 pod in cluster
	- upto 300,000 total containers
	- upto 100 pods per node
#### storage
- High performance - SSD Backed Storage에 의존
- Multiple Concurrent connections - Network based storage
- Persistent shared volumes for shared access across multiple pods
- label node with specific disk type
- use node selectors to assign application to nodes with specific disk types

#### Node
- virtual or physical machine
- master vs worker nodes
- Linux X84_64 운영체제 사용해야함
- master node can host workload 지만, best practice는 workload로 사용하지 않는 것이다. 클러스터 관리용으로만 사용하는 것이 좋다.
- 마스터 노드와 ETCD 노드를 따로 분리하여 운영하는 경우도 있다.

## Choosing kubernetes infrastructure

쿠버네티스는 다양한 시스템에 다양한 방식으로 배포될 수 있다.
노트북에서 시작해서 회사의 물리적 or 가상 서버 클라우드에서 가능하다.
요구사항과 생태계 배포하고자하는 앱 종류에 따라 이 중 하나를 선택하여 infrastructure를 구성하면 된다.

#### 로컬
- Linux 컴퓨터 - 수동으로 바이너리 설치하여 로컬 클러스터 설정가능(이건 너무 수고스러움)-자동화하는 솔루션에 의존하기- minikube
- Window 컴퓨터-바로 설정할 순 없음. 바이너리가 없기때문에. Hyper-v or VMware, Workshop box 같은 가상 소프트웨어를 사용해야함. 
  LInux VM을 생성하고 거기에 쿠버네티스 실행가능.
  Windows VM에서 Docker 컨테이너로서의 도커로 쿠버네티스 구성 요소를 실행할 수도 있음(도커이미지는 리눅스기반)
*Minikube vs Kubeadm*
- Minikube
	- Deploy VM(VM이 자체를 구성)
	- single node cluster
- Kubeadm
	- Require VM to be ready(이미 프로비전된 VM)
	- single/mutil node cluster

#### Private or Public 클라우드
*Turnkey Solution vs Hosted Solution(Managed Solution)*
![[Pasted image 20240623191140.png]]

- Turnkey Soluction
	- 우리가 요구되는 VM을 Provision함
	- 툴이나 스크립트로 쿠버네티스 클러스터를 구성해야함.
	- 가장 중요한것은 VM을 관리하고 패치, 업그레이드를 직접 책임지고 해야함
	- KOPS 같은 도구 사용하면 더 쉽게 할  수 있음
- Managed Solution
	- Provider가 VM Provision하고 구성해줌.
	- VM이 Provider에 의해 유지된다.
	- 알아서 관리해줌

![[Pasted image 20240623191407.png|500]]
OpenShift 
- Red Hat의 쿠버네티스 플랫폼. 
- 오픈소스 컨테이너 애플리케이션 플랫폼으로 쿠버네티스 환경 위에 만들어져있음. 
- 추가적인 tool 집합과 GUI 제공. 
- 쿠버네티스의 구성과 CICD 파이프라인을 쉽게 통합할 수 있게 해준다.

Cloud Foundry Container Runtime
- Cloud Foundry의 오픈소스프로젝트
- 고가용성 쿠버네티스 클러스터 배포 및 관리

VMware Cloud PKS
- VMware 환경을 활용하고 싶은 경우

Vargrant
- 유용한 스크립트 set 제공하여 서로 다른 클라우드 서비스 provider에 클러스터 배포

![[Pasted image 20240623213743.png|500]]

## Config High Availability

쿠버네티스의 고가용성(HA)

만약, 마스터노드가 한개인데 그게 고장나면 어떻게 될까?
-> 워커노드들이 동작하고 컨테이너가 살아 있는 한 애플리케이션은 작동할 것이다. 하지만 그 컨테이너나 파드가 고장나버리는 순간, replica set의 일부라면 워커노드에게 새 파드를 로드하라는 명령을 지시해야하는데, 마스터노드가 없다보니
이를 하지 못하게 될것이다. 

*따라서 마스터 노드의 고가용성을 고려해야한다.*
고가용성 구성이란 클러스터 내 모든 컴포넌트에 걸쳐 중복을 갖는 것이다. 단일 실패 지점을 피하기 위해서말이다. 

#### Master Node(control plane) HA
마스터 노드에서도 구성 컴포넌트에 따라서 고가용성을 유지했을때 일을 분담하는 방식이 다르다.
![[Pasted image 20240623215627.png|400]]
- API server
	- *병렬*적으로 동작
	- API 서버는 요청을 수신하고 프로세싱하여 클러스터에 대한 정보를 제공해야하는 책임이 있다.
	- 한번에 하나씩 요청에 따라 작업된다.
	- 따라서 모든 API 서버가 active 모드 상태로 병렬적으로 동작한다.
	- 만약 master node의 주소가 https://master1:6443 이라면 다른 마스터노드는 https://master2:6443 로 구성되어있을 것이고 이중 어디로 요청이 갈지는 앞단의 *LB*를 통해 지정된다. 
	- Nginx, HA proxy, Load balancer 사용가능하다

![[Pasted image 20240623215642.png|500]]
- Controler manager, Scheduler
	- active, standby mode로 한개만 active인 상태로 동작(병렬 실행 X)
	- 컨트롤러 매니저를 예를 들면 파드를 관찰하다가 파드가 고장나면 새파드를 생성하는 등 필요한 조치를 해야하는데 다수의 컨트롤러가 동시에 모두 active이면 *필요이상으로 파드가 증가될 수 도 있다.*
	- active or standby 를 어떻게 정하는가? *leading election*
	  controler manager가 시작되면 *kube-controller-manager-endpoint* 라는 오브젝트에 *lease*나 *lock*을 얻으려고 한다. 그 endpoint를 업데이트 하는 쪽이 lease(임대)를 획득해 활성화되고 나머지는 passive가 됩니다.
	  - --leader-elect: 리더를 뽑는 작업을 하겠다는 설정. 기본값 true
	  - --leader-elect-lease-duration: 리더로 최종 확정될떄까지의 대기 시간 설정.
	  - --leader-elect-renew-deadline: 문제가 발생된 기존 리더가 문제가 발생된 시점부터 정상수행을 할 수 있다고 알리는 기한을 설정. 따라서 lease-duration보다 작거나 같아야된다.
	  - --leader-elect-retry-period: 리더를 뽑아야하는 상황인지 체크하는 기간

![[Pasted image 20240623215700.png|400]]
![[Pasted image 20240623215709.png|400]]
- ETCD
	- etcd가 마스터노드의 일부일 경우: *Stacked control plane nodes topology*
		- 설정관리가 더 쉽고 추가적인 노드가 필요하지 않다. 하지만 마스터노드가 다운되면 etcd도 같이 분실ㄹ도니다.
	- 분리 되어 있는경우: *External ETCD Topology*
		- 위보다 덜 위험하다. control plane이 영향을 주지 않는다.
		- 셋업이 더 어렵고 노드에 대한 서버 수가 두배가 필요하다.
		
![[Pasted image 20240623221053.png|500]]
- API server는 etcd 서버에 통신하는 유일한 컴포넌트이기에.
- API server config 옵션을 보면 etcd 서버 위치를 지정하는 옵션 집합이 있댜. 
- 토폴로지에 상관없이 ETCD 서버 구성 장소가 동일 서버든 분리된 서버든 궁극적으로 API 서버가 etcd 서버의 올바른 주소를 가리키도록 해야한다. 
- 이것이 kupe-apiserver configuration에서 etcd 서버 리스트를 명시하는 이유이다.

![[Pasted image 20240623221225.png|200]]
## ETCD in HA
ETCD
분산되고 신뢰할 수 있는 key value store로 간단하고 안전하며 빠르다.

Distribute
고가용성을 위해 분산

Consistent
어떻게 데이터 일관성을 유지하는가
ETCD는 데이터의 동일한 복사본이 모든 인스턴스에서 동시에 사용 가능하도록 보장한다.
Read: 모든 노드에서 사용가능하기 때문에 쉽다.
write: Leader election을 통해 선출된 리더가 처리한다. 한 리더만 write 수행. 쓰기를 처리하고 리더가 다른 노드들에게 데이터 복사본을 전달하여 분산되도록 보장한다.
![[Pasted image 20240623222125.png|400]]

![[Pasted image 20240623222447.png|400]]
#### 리더 선출 방법
-> RAFT
Quorum을 만족하여 시스템 유지
무작위로 타이머를 맞춰 신호를 보낸다. 타이머가 먼저 완료된 매니저가 다른 노드들에게 투표 요청을 보냅니다.
그리고 그 투표 요청을 받은 노드들은 응답을 해야만 합니다. 그리고 과반수 이상의 투표를 받게 되면 그 노드는 리더로 선출되게 됩니다.
리더로 선출된 다음 다른 노드들에게 주기적 정기적으로 자신이 리더임을 계속 공지를 보내야합니다.

어느시점에 다른 노드들이 리더로부터 알림을 못받게 된다면, 리더가 다운됐거나 네트워크가 끊긴거기 때문에 노드들 사이에서 다시 재 리더선출 프로세스를 신작합니다. 그리고 새리더가 선출되게 된다.

즉, ETCD write는 리더에 의에 처리되고 클러스터 내 다른 노드들로 복제된다. 
*다른 노드들로 복제되었을때만 쓰기가 완료된 것으로 간주된다.*

쓰기 요청이 들어왔는데 노드하나가 응답하지 않는다고 하면?
리더가 그 하나를 제외한 노드에만 쓰기를 할 수 있을 것이다. 
-> 실패로 보지 않고 대부분의 노드에 쓰기가 되었다면 완료로 간주한다.

### Quorum
N/2 + 1
quorum은 클러스터가 제대로 기능하고 성공적으로 쓰기위해 반드시 필요한 최소의 노드 수입니다. 
N이 3이면 2인 것이다.

인스턴스1의 쿼럼은 1이다.
그 말은 단일 노드 클러스터라면 이런 건 프로토콜이 적용이 안 된다는 것. 그 노드를 잃으면 모든 게 사라진다. 
두 개를 보고 같은 공식을 적용하면 쿼럼은 2이다. 
클러스터에 인스턴스가 2개라도 과반수는 여전히 2이다.하나가 실패하면 쿼럼이 없어 쓰기도 할 수 없게된다. 
인스턴스가 2개인 건 1개인 것과 같은 것이며. 쿼럼이 충족될 수 없는 어떤 실질적 가치도 제공하지 않는다. 
-> 그래서 추가적인 클러스터에 *최소 3개의 인스턴스를* 갖도록 권장하는 것이다. 

![[Pasted image 20240623223554.png|500]]
*홀수가 좋다*
5,6중에서 fault tolerance는 둘다 2이므로 같은 개수만큼 실패해도 되는 노드가 같은데, 홀수가 더 효율적인 것이 된다. 
네트워크 세분화 중 클러스터 결함이 발생하는 가능성도 존재하는데, 세그먼트화에서 작수보다 홀수가 생존할 확률이 더 높다.

![[Pasted image 20240623224444.png]]
서버에 설치하려면 최신 지원되는 바이너리를 다운로드하고 그걸 압축해 필수 디렉터리 구조를 생성하고 추가로 생성된 인증서 파일 위로 복사

#### etcdctl
 etcdctl 유틸리티는 두 가지 API 버전 존재 -V2와 V3 
 그래서 버전마다 명령이 다르며, 버전 2는 디폴트값이다
![[Pasted image 20240623224642.png|300]]

#### Number of Nodes
HA에서 필요한 최소 노드는 3개
더 원한다면 5개
그이상은 불필요