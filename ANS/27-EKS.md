
## Kubernetes Architecture

![500](images/Pasted%20image%2020250622221049.png)

- Kubernetes Cluster consists of Control Plane and Data Plane
- **Control plane**은 마스터 노드에 위치한 여러 제어 프로세스로 이루어져 클러스터 상태를 관리한다.
- **Data Plane**은 **Node**라고 불리는 워커 노드들로 구성되어 있다.
- Nodes host the Pods
- **Pod**는 하나 이상의 애플리케이션 컨테이너로 구성되어 있다.

![](images/Pasted%20image%2020250622221416.png)

- 사용자는 쿠버네티스 클러스터와 어떻게 통신하는가
	- kube-api
		- Expose된 api 서버를 통해 kubectl 같은 명령 도구를 사용한다.
- Control Plane을 통해 Data Plane을 관리하며 사용자와 통신한다.

---
### Control Plane Components

![400](images/Pasted%20image%2020250622221938.png)

- **kube-apiserver**
    - Kubernetes API를 외부에 노출하는 프론트엔드 역할을 한다 
    - 사용자는 이 API를 통해 클러스터를 관리하고 상호작용할 수 있다.
        
- **etcd**
    - Kubernetes 클러스터의 모든 데이터를 저장하는 **Key-Value 저장소**
    - 클러스터 상태 정보의 중심 저장소 역할을 한다.
        
- **kube-scheduler**
    - 새로 생성된 Pod 중 노드가 할당되지 않은 것을 감지하고, 적절한 노드를 선택해 한다.
        
- **kube-controller-manager**
    - 여러 컨트롤러를 실행하는 컴포넌트이다
    - 주요 컨트롤러: Node controller(노드가 다운됐는지 확인), Replication controller(replication 수 유지되도록), Namespace controller, Job Controller, EndpointSlice controller etc.
        
- **cloud-controller-manager**
    - Kubernetes 클러스터를 클라우드 서비스와 연결
        
    - 예시:
        - 클라우드에서 노드(인스턴스)가 삭제되었는지 확인하는 **노드 컨트롤러**
        - 클라우드 로드 밸런서를 생성하는 **서비스 컨트롤러** 등
---

### Kubernetes Data plane components

![500](images/Pasted%20image%2020250622223217.png)

- **Node (노드)**
    - hosts the pods (applications)
        
- **kubelet**
    - 각 노드에서 실행되는 에이전트로, 해당 노드의 **Pod 안에 있는 컨테이너들이 제대로 실행 중인지 지속적으로 확인**
        
- **kube-proxy**
    - 클러스터 **내부 또는 외부의 네트워크 세션을 통해 Pod들과의 통신이 가능하도록 네트워크 라우팅을 지원**
        
- **Container Runtime (컨테이너 런타임)**
	- 컨테이너를 실행시키기 위해 노드 안에 컨테이너 런타임 엔진이 필요하다.
    - Kubernetes는 여러 컨테이너 런타임을 지원
        - `containerd`
        - `CRI-O`
        - Kubernetes CRI(컨테이너 런타임 인터페이스)를 구현한 기타 런타임
---

![](images/Pasted%20image%2020250622223526.png)

---

### Amazon EKS Architecture

![](images/Pasted%20image%2020250622223605.png)
- HA를 위해 AZ별로 구성
- control plane node는 aws가 관리

![](images/Pasted%20image%2020250622223610.png)
- EKS를 사용하면 control plane은 신경쓰지 않아도 된다. - managed control plane

![](images/Pasted%20image%2020250622223616.png)
- 노드는 일반적으로 EC2 인스턴스 기반
- Data plane에는 노드를 시작하는 방법에 여러 옵션이 존재한다.
	- self-managed nodes
		- 사용자가 직접 ec2 인스턴스 생성하고 관리
		- 원하는 AMI 선택
		- 수동으로 스케일링, 업데이트, 보안패치 적용 등을 처리
		- 유연성은 높지만 관리 부담이 큼
	- managed node group
		- AWS가 제공하는 **자동화된 EC2 노드 운영 방식**
		- AWS에서 안정성과 보안을 검증한 EKS 최적화 AMI를 사용
		- 다음 기능이 자동으로 제공됨:
		    - EC2 인스턴스 **프로비저닝**
		    - **스케일 인/아웃 자동 처리**
		    - **버전 업그레이드** 및 노드 교체
	- aws fargate
		- 서버리스: **노드 관리 불필요, 과금도 Pod 기준**
		- Pod 단위로 **CPU 및 메모리 자원을 정의**하여 실행
		- **단점**
			- 커널 수준 권한이 필요한 작업이나 특정 네트워크 설정이 제한적일 수 있음
			- 가격이 EC2 기반 노드보다 높을 수 있음

---
## Amazon EKS Cluster Networking
### EKS Networking

![500](images/Pasted%20image%2020250622223641.png)

- **Control Plane**
    - EKS Control Plane은 **AWS가 관리하는 계정과 VPC 내에 배포**된다.
    - 사용자는 직접 Control Plane을 관리하지 않으며, AWS에서 완전히 운영한다.
        
- **Data Plane**
    - 워커 노드(노드 그룹)는 **사용자 계정과 VPC 내에서 실행**된다.
    - 즉, 실제 애플리케이션이 실행되는 환경은 고객 소유 VPC에 위치한다.
        
- **ENI(Elastic Network Interface) 설정**
    - EKS는 Control Plane과 고객 VPC 간 통신을 위해 고객 VPC 내에 **2~4개의 ENI(탄력적 네트워크 인터페이스)**를 자동으로 생성한다.
    - 이 ENI들은 **Control Plane의 통신 경로** 역할을 하며, 각 ENI에는 **보안 그룹(SG)**이 자동으로 할당된다.
    - 이 보안 그룹은 **EKS가 소유한 ENI**와 **Managed Group Node**에도 적용된다. 
    - ENI를 위한 서브넷은 **일반 노드용 서브넷과 분리하여 구성**하는 것이 권장된다
    - 각 서브넷에는 **최소 6개의 IP** (16 recommended)
        
- **API 서버 접근**
    - **기본 설정으로 Kubernetes API 서버는 인터넷을 통해 접근 가능하다**
    - 필요 시 퍼블릭 접근을 제한하고, 프라이빗 네트워크로만 연결하도록 설정할 수 있다
        
- **Pod IP 할당**
    - EKS는 Pod에 대해 **IPv4 또는 IPv6 주소를 할당**할 수 있다
    - 단, **IPv4와 IPv6를 동시에 사용하는 듀얼스택(Dualstack)은 지원하지 않는다



![](images/Pasted%20image%2020250622230653.png)
- 누구나 컨트롤 플레인에 접근가능하다는 의미
- CIDR 블록을 사용하여 화이트리스트로 제한 할 수도 있다.

![](images/Pasted%20image%2020250622230713.png)


![](images/Pasted%20image%2020250622230723.png)
- public access 완전히 비활성화 하고 오직 private하게 

![](images/Pasted%20image%2020250622230734.png)
- PrivateLink(VPC 인터페이스 엔드포인트)를 통해 API 트래픽을 AWS 네트워크 내에서 유지
- vpn이나 DX의 타켓을 VPC interface Endpoint로만 잡으면 됨.

![](images/Pasted%20image%2020250622231900.png)

- **Public Subnet**
	- **Elastic Load Balancer (ELB)** 배포 용도로 사용
	    - 외부에 Kubernetes 서비스를 노출할 때 필요
	- **NAT Gateway** 배포 용도로 사용
	    - **IPv4 기반 인터넷 접근** 허용
	    - IPv6의 경우 **Egress 전용 IGW** 사용

---

### Amazon EKS Pod Networking - CNI

### Kubernetes Network Model (CNCF 표준) - CNI

- 각 **Pod는 고유한 IP 주소**를 할당받음
- 같은 Pod 내부의 컨테이너들은 동일한 IP를 공유
- 모든 Pod는 NAT 없이 서로 통신 가능
- 모든 노드는 NAT 없이 모든 Pod와 통신 가능
- Pod가 인식하는 자신의 IP와 외부에서 보는 IP는 동일함 (즉, pod 접속하고자하면 해당 pod ip로 접속하는 것이 동일하다는 의미)

> → **CNI (Container Network Interface)** 표준 기반의 네트워크 모델

### Amazon VPC CNI plugin

![500](images/Pasted%20image%2020250622233807.png)

- **Amazon VPC CNI**:
    - **ENI**를 워커 노드에 생성 및 연결
    - Pod에 **ENI의 secondary IP**를 할당
        
- EKS는 Amazon VPC CNI를 기본 지원함
    
- 그 외 호환 CNI 플러그인:
    - **Calico**, **Cilium**, **Weave Net**, **Antrea**

---
### Maximum Pods per node

![](images/Pasted%20image%2020250622234009.png)
- 파드에 secondary IP**를 할당하기 때문에 -> ips per network interface 에서 -1
- +2: 노드마다 추가 파드들 : kubelet, kube-proxy

---
### Increased available IP addresses for Pods

![](images/Pasted%20image%2020250622234145.png)

- AWS Nitro 기반 인스턴스에서만 지원
- **Prefix Delegation** 방식: 각각의 ENI에 IP주소 대신 CIDR 블록을 할당 할 수 있음
	- 각 ENI에 `/28 (IPv4)` → 16개 IP
	- `/80 (IPv6)` → 대량 IP 가능
- 쿠버네티스 자체에도 일종의 제약이 있기 때문에 최대 몇개의 파드를 둘 것을 권장하고 있다. = 110
	- 노드의 상태를 관리하고 파드 스케쥴링 등의 작업들을 고려

---
### Assigning IPv6 addresses to pods and services

![500](images/Pasted%20image%2020250622234202.png)

**Supported with AWS Nitro-based instances & Fargate**

- 기본은 IPv4지만, 설정 시 IPv6 사용 가능
- **EKS는 dual-stack(IPv4+IPv6) 미지원**
	- 하나의 Pod 또는 서비스가 동시에 IPv4와 IPv6를 사용하는 것은 불가능
- **Amazon EC2 인스턴스에서 IPv6 사용 시 조건**:
    - **Amazon VPC CNI add-on**을 설치 및 구성해야 함
    - **IP Prefix Delegation** 기능을 함께 활성화해야 함
- **IPv4 주소도 반드시 설정되어야 함**
	- VPC와 서브넷에 **IPv4 주소가 필수 조건**
 - **서브넷은 반드시 IPv6 자동 할당(auto-assign)을 활성화**해야 함
- **Windows Pod 및 서비스는 IPv6 미지원**

---
### Pod to Pod communication

![](images/Pasted%20image%2020250622234228.png)

- 동일 VPC 내에 존재하는 경우, **Pod IP를 통해 직접 통신** 가능
- 별도의 NAT, 게이트웨이, 라우팅 없이도 VPC 내에서 **Pod ↔ Pod 통신 가능**

---

|항목|설명|
|---|---|
|**CNI 플러그인**|Pod에 IP 할당, ENI 연결 관리|
|**Pod 간 통신**|NAT 없이 가능, Pod IP 직접 사용|
|**IPv6 지원**|Nitro/Fargate, 단일 스택만 가능|
|**ENI 한계**|인스턴스 타입별 ENI 및 IP 수로 Pod 수 제한|
|**Prefix Delegation**|IP 수 확장 기능, Nitro에서만 가능|
