
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

| 항목                    | 설명                            |
| --------------------- | ----------------------------- |
| **CNI 플러그인**          | Pod에 IP 할당, ENI 연결 관리         |
| **Pod 간 통신**          | NAT 없이 가능, Pod IP 직접 사용       |
| **IPv6 지원**           | Nitro/Fargate, 단일 스택만 가능      |
| **ENI 한계**            | 인스턴스 타입별 ENI 및 IP 수로 Pod 수 제한 |
| **Prefix Delegation** | IP 수 확장 기능, Nitro에서만 가능       |

---
### Amazon EKS Pod Networking – traffic between Pod and external network

- Pod 간 통신 (Pod to Pod communication)
	- 동일한 VPC 내에 있는 Pod들은 별도의 변환 없이 각자의 Pod IP 주소를 사용하여 직접 통신할 수 있다.
- Pod와 외부 네트워크 간 트래픽 (Pod and external network traffic)
	- = Pod가 VPC의 CIDR 블록에 속하지 않는 외부 IPv4 주소와 통신할 때
	- **기본 동작 (IPv4):** *Amazon VPC CNI 플러그인*은 Pod의 IPv4 주소를 해당 Pod가 실행 중인 노드의 Primary ENI의 Primary private IPv4 주소로 변환(SNAT)한다.
	- **IPv6:** IPv6 주소는 네트워크 주소 변환(NAT)을 하지 않으므로 이 동작이 적용되지 않는다.
- 외부 통신을 위한 Source NAT(SNAT) 설정
	- EKS에서는 `AWS_VPC_K8S_CNI_EXTERNALSNAT` 설정을 통해 Pod의 외부 통신 방식을 제어할 수 있다.
	- 퍼블릭 서브넷 (Public Subnet) 내 노드
		- ![](images/Pasted%20image%2020250630221553.png)
		- `AWS_VPC_K8S_CNI_EXTERNALSNAT=false`
		- VPC 피어링 또는 VPN/Direct Connect와 같은 VPC 내 통신에서는, 소스 IP는 노드 ENI의 기본 IP로 유지된다
		- 인터넷으로 나가는 트래픽은 노드의 ENI에 할당된 Public IP 로 변환된다.
		- Problem:
			- 외부에서 봤을때 노드IP 만 볼 수 있고, Pod IP는 가려지게 된다. 즉, Pod IP가 외부에 노출되지 않는다.
				- 따라서 외부에서 Pod IP로 직접 접근할 수가 없다. 
			- 일반적으로 Worker Node는 Private Subnet에 위치해야한다.
		- IPv6는 NAT 변환 없이 **직접 통신 가능**
		- `kubectl set env daemonset aws-node -n kube-system AWS_VPC_K8S_CNI_EXTERNALSNAT=false`
	- 프라이빗 서브넷 (Private Subnet) 내 노드
		- ![](images/Pasted%20image%2020250630222035.png)
		- `AWS_VPC_K8S_CNI_EXTERNALSNAT=true`
			- 개별 Pod IP가 보이도록
		- VPC 피어링 또는 VPN/Direct Connect와 같은 VPC 내 통신에서는, 트래픽의 소스 IP는 Pod의 IP 주소로 유지
		- 인터넷으로 나가는 트래픽은 Public Subnet에 위치한 NAT 게이트웨이를 통해 라우팅
			- 이때 소스 IP는 NAT 게이트웨이의 Elastic IP로 변환
		- `kubectl set env daemonset aws-node -n kube-system AWS_VPC_K8S_CNI_EXTERNALSNAT=true`
	- 외부 네트워크에서 Pod로의 통신 (External network to Pod communication)
		- ![](images/Pasted%20image%2020250630222310.png)
		- `AWS_VPC_K8S_CNI_EXTERNALSNAT`가 `true`로 설정된 경우, 외부 트래픽은 Pod의 프라이빗 IPv4 주소로 도달할 수 있다. 
			- 이 통신은 노드가 퍼블릭 서브넷에 있을 경우 퍼블릭 ENI를 통해, 
			- 노드가 프라이빗 서브넷에 있고 트래픽이 인터넷에서 오는 경우 NAT Gateway를 통해, 또는 Transit Gateway 연결, VPC 피어링, VPN/Direct Connect를 통해 발생할 수 있다.
	- `EXTERNALSNAT` 설정은 단순히 IP 변환 여부를 결정하는 것을 넘어선다
		- `false` 설정은 Pod의 외부 통신이 노드의 IP로 통합되므로, 외부에서는 Pod 개별 IP를 알 수 없게 되어 보안 측면에서 일종의 추상화 계층을 제공한다. 
			- 장점: 외부 공격 표면을 줄이고, 노드 수준에서 보안 그룹 관리를 간소화할 수 있다. 
			- 단점: Pod별 트래픽 식별이 어려워질 수 있다. 
		- `true` 설정은 Pod IP를 보존하여 VPC 내부 또는 연결된 네트워크에서 Pod에 대한 직접적인 가시성을 제공한다. 
			- 장점: `EXTERNALSNAT=true`는 Pod별 보안 정책 적용이나 감사 추적에 유리
			- 단점: Pod IP가 노출될 수 있어 추가적인 보안 고려사항(예: 네트워크 ACL, Pod 보안 그룹)이 필요하다. 
---
### Multi-homed Pods with Multus CNI

![500](images/Pasted%20image%2020250630225409.png)

- **Multus CNI**는 Pod에 여러 개의 네트워크 인터페이스를 연결할 수 있게 해주는 CNI 플러그인이다.
- 이를 통해 단일 Pod가 여러 개의 네트워크 인터페이스를 가질 수 있는 멀티호밍(multi-homed) Pod를 생성할 수 있다.
- AWS는 VPC CNI를 통해 Multus에 대한 지원을 제공한다.
- Multus를 사용하지 않는 일반적인 Pod는 하나의 `eth0` 인터페이스를 가지며, 이는 `aws-cni`에 의해 관리된다. 그러나 Multus를 사용하는 Pod는 `eth0`, `net0`, `net1` 등 여러 인터페이스를 가질 수 있다
- EKS 환경에서 고급 네트워킹 시나리오를 가능하게 하여, 통신, 미디어, 금융 등 고급 네트워킹이 필요한 산업에서 유용
	- 다만 여러 CNI를 함께 쓰면 네트워크 구성이 복잡해지므로 **신중한 설계와 운영 지식이 필요**

---
### Security Groups in EKS

![](images/Pasted%20image%2020250630225901.png)
- EKS 클러스터를 생성하면 기본 보안 그룹이 함께 생성된다.
- EKS associates this SG with:
    - ENIs created by EKS in Customer VPC
    - ENIs of the nodes in Managed Node group
- 기본 규칙
	- **인바운드:** 자기 자신(Self)으로부터의 모든 프로토콜, 모든 포트 허용.
    - **아웃바운드:** 모든 대상(0.0.0.0/0 또는 ::/0)으로의 모든 프로토콜, 모든 포트 허용.
- Pod 보안 그룹의 문제점
	- 기본적으로 보안 그룹(Security Group)은 개별 Pod가 아닌 노드의 ENI에 할당된다
	- 따라서 같은 ENI를 공유하는 모든 Pod들은 동일한 보안 그룹 규칙을 적용받게 된다.
	- 이는 Pod별로 세분화된 네트워크 보안 정책을 적용하기 어렵게 만드는 단점이 있다
	- 이러한 문제를 해결하기 위한 대안으로 Calico와 같은 네트워크 정책 엔진을 사용하여 `iptables` 기반의 Pod 간 트래픽 제한을 구현하거나, EKS native 옵션인 Trunk 및 Branch ENI를 사용하는 방법이 있다
	  
### Pod 보안 그룹 해결책: Trunk & Branch ENI

  ![400](images/Pasted%20image%2020250630232528.png)

- 각 Pod에 전용 ENI(Branch ENI)를 할당하여 독립적인 보안 그룹을 적용할 수 있게 한다.
- `kubectl set env daemonset aws-node -n kube-system ENABLE_POD_ENI=true` 명령어로 활성화
    
- **동작 방식:**
    - `amazon-vpc-resource-controller-k8s`라는 addon 이 기능을 관리
    - VPC 리소스 컨트롤러는 "aws-k8s-trunk-eni"라는 특수 네트워크 인터페이스를 생성하여 노드에 연결한다. 
    - 컨트롤러는 또한 "aws-k8s-branch-eni" 인터페이스를 생성하고 이를 트렁크 인터페이스와 연결한다.
      
- Trunk 및 Branch ENI는 EKS 환경에서 애플리케이션의 보안 태세를 크게 강화할 수 있는 중요한 기능이다
- 그러나 이 기능을 활성화하면 ENI 수가 증가하고, 각 Pod에 대한 보안 그룹을 개별적으로 관리해야 하므로, 자동화된 보안 정책 관리 도구(예: Kubernetes Network Policies, GitOps 기반 보안 정책 관리)의 도입이 필수적이다. 이는 보안 강화와 운영 복잡성 증가 사이의 균형점을 찾는 중요한 결정이 된다.
        
- **제약사항:**
    - Windows 노드에서는 사용할 수 없다.
    - IPv6 클러스터에서는 Fargate 노드에서만 작동한다.
    - 대부분의 Nitro 기반 인스턴스에서 지원되지만, t 계열 인스턴스 등 일부는 지원되지 않는다

---

### Exposing EKS services

- Pod의 IP를 직접 사용하는 것은 비추천(anti-pattern)이다.
	- Pods are non-permanent objects
	- Pods may be created and destroyed  
	- 스케일링, 노드 교체 등으로 노드간에 파드가 이동될 수도 있다.
- 즉, Pod는 일시적인 리소스이므로, 안정적인 서비스 접근을 위해 Kubernetes Service 는 Pod 집합을 네트워크 서비스로 노출하는 방법을 제공한다
- Service Type
	- **ClusterIP**: 클러스터 내부에서만 접근 가능한 가상 IP를 제공
	- **NodePort**: 노드의 고정 포트를 통해 외부에서 접근 가능
	- **LoadBalancer**: CLB/NLB(네트워크 로드 밸런서, Layer 4)를 통해 외부 접근 제공
	- **Ingress**: ALB(애플리케이션 로드 밸런서, Layer 7)를 통해 외부 접근 제공


![600](images/Pasted%20image%2020250701002758.png)

- #### ClusterIP
	- 기본 서비스 유형으로, 클러스터 내부에서만 접근 가능한 가상 IP를 할당한다. 이 IP는 클러스터 외부에는 노출되지 않는다.
	- 가상 IP는  `kube-apiserver`의 `--service-cluster-ip-range` 파라미터로 구성된 풀에서 할당된다.
		- 명시적으로 구성되지 않으면, EKS는 기본적으로 `10.100.0.0/16` 또는 `172.20.0.0/16` 대역에서 이 IP를 할당한다.
	- 각 노드의 `kube-proxy`데몬이 `iptables` 규칙을 사용하여 ClusterIP와 실제 Pod IP 간의 매핑을 정의하여 트래픽 라우팅 하도록 한다.
	- `<service-name>.<namespace-name>.svc.cluster.local` 형태의 프라이빗 DNS 이름으로 접근가능 하다.
	  
- #### NodePort
	- NodePort는 Kubernetes 서비스를 클러스터 외부에서 접근 가능하게 하는 데 사용된다. 
	- 서비스는 각 워커 노드의 IP에 정적 포트(NodePort)로 노출되며, 포트 범위는 `30000-32767`이다.
	- NodePort는 내부적으로 ClusterIP를 사용하여 NodeIP/Port 요청을 ClusterIP 서비스로 라우팅한다.
	- 클라이언트가 노드의 IP를 직접 알아야 하므로 외부 서비스 노출용으로는 비효율적일 수 있다.
	  
- #### LoadBalancer
	- AWS의 로드 밸런서를 프로비저닝하여 서비스를 외부에 노출
	- **Kubernetes Controller Manager(legacy)** - 쿠버네티스에 내장된 컨트롤러로, CLB(기본값) 또는 NLB를 '인스턴스(instance)' 모드로 배포한다.
		- CLB는 Layer 4/Layer 7 트래픽(TCP, SSL/TLS, HTTP, HTTPS)을 지원 
		- NLB는 Layer 4 트래픽(TCP, UDP, TLS)을 인스턴스 모드에서만 지원. 
		- LoadBalancer 서비스는 NodePort 서비스 위에 구축된다.
	- **AWS Load Balancer Controller (권장):**
	    - NLB를 '인스턴스(instance)' 또는 'IP' 모드로 프로비저닝할 수 있다.
		- 각 서비스마다 전용 NLB가 필요하여 서비스 수가 많아지면 확장성 및 관리에 어려움이 있을 수 있다.

- #### Ingress
	- ![500](images/Pasted%20image%2020250701003621.png)
	- HTTP 및 HTTPS 경로를 기반으로 외부 트래픽을 클러스터 내부 서비스로 라우팅한다.
		- 트래픽 라우팅은 Ingress 리소스에 정의된 규칙에 의해 제어된다.
	- **AWS Load Balancer Controller**를 사용하여 ALB를 프로비저닝한다
	- Ingress는 ALB Target Group을 사용하여 단일 ALB 뒤에 여러 서비스를 추가할 수 있어 비용과 복잡성을 절감한다.

#### AWS Load Balancer Controller
-  AWS Load Balancer Controller는 Ingress 컨트롤러의 AWS 구현체이다.
	- 이는 Ingress rule, 파라미터, 어노테이션을 ALB 구성으로 변환하여 리스너와 Target Group을 생성하고 백엔드 서비스에 연결한다. 
	- Target으로 인스턴스 또는 Pod IP를 지원하며 , `kubernetes.io/ingress.class: alb` 어노테이션이 사용된다. 
	- 여러 서비스와 ALB를 공유하려면  `alb.ingress.kubernetes.io/group.name: my-group` 어노테이션을 사용한다.
	- IPv6 트래픽은 IP 대상에 대해서만 지원되며,  `alb.ingress.kubernetes.io/ip-address-type: dualstack` 어노테이션을 사용한다.
	- IPv6 트래픽은 IP 타겟에서만 지원된다.

![](images/Pasted%20image%2020250701003611.png)

- 각 Kubernetes 서비스 유형은 특정 사용 사례와 트래픽 패턴에 최적화되어 있다. 
- ClusterIP는 내부 통신에, NodePort는 개발/테스트 환경의 임시 노출에 적합하며, LoadBalancer는 L4 트래픽의 외부 노출에, Ingress는 L7 트래픽의 고급 라우팅 및 비용 효율적인 노출에 사용된다.
- AWS Load Balancer Controller와 Ingress 리소스의 조합은 단일 ALB로 여러 서비스를 관리할 수 있게 하여 운영 효율성과 비용 절감이라는 중요한 이점을 제공한다. 
- 서비스 유형의 선택은 단순한 기술적 결정이 아니라, 애플리케이션 아키텍처와 비즈니스 요구사항을 반영하는 전략적 결정이다. 
	- 예를 들어, 내부 마이크로서비스 간 통신에는 ClusterIP가 최적이며, 웹 애플리케이션의 외부 노출에는 Ingress가 가장 효율적이다. 
	- LoadBalancer 서비스는 NLB를 통해 고성능 L4 트래픽을 처리하는 데 유용하지만, 서비스별 NLB는 관리 오버헤드를 증가시킬 수 있다. 

|서비스 유형|접근성|사용 사례|로드 밸런서 유형|주요 특징/제한 사항|
|---|---|---|---|---|
|ClusterIP|클러스터 내부|내부 마이크로서비스 통신|없음 (가상 IP)|기본 유형, 클러스터 내부에서만 접근, `kube-proxy`를 통한 라우팅|
|NodePort|클러스터 외부 (노드 IP 통해)|개발/테스트 환경의 임시 노출|없음 (노드 포트)|각 노드의 정적 포트 노출, 노드 IP 변경 시 추적 필요, 외부 노출에 비효율적|
|LoadBalancer|클러스터 외부|L4 트래픽의 외부 노출|AWS CLB/NLB|CLB (L4/L7), NLB (L4, 인스턴스 모드), 각 서비스에 전용 LB 필요 가능, 관리 오버헤드|
|Ingress|클러스터 외부 (HTTP/HTTPS)|L7 트래픽의 고급 라우팅, 비용 효율적 노출|AWS ALB|단일 ALB로 여러 서비스 관리, 경로 기반 라우팅, `X-Forwarded-For` 헤더로 클라이언트 IP 보존|

---

#### 클라이언트 IP 보존 (Preserving Client IP)

![500](images/Pasted%20image%2020250701010201.png)

- **NLB (LoadBalancer 서비스):** 
	- 서비스 명세에서 `externalTrafficPolicy`스펙은 클러스터 내에서 로드 밸런싱이 어떻게 발생하는지 정의한다.
	- 즉, `externalTrafficPolicy`는 노드에 도달한 후 트래픽이 라우팅되는 방식 을 지시한다.
	- 초기 노드의 선택은 `externalTrafficPolicy` 자체에 의해 직접 제어되는 것이 아니라, 상위 로드 밸런서에 의해 결정된다. `externalTrafficPolicy`는 이후 2차 라우팅 결정 계층으로 작동한다.
		- `externalTrafficPolicy=Cluster`로 설정
			- 트래픽이 다른 노드로 전송될 수 있으며 소스 IP가 노드의 IP 주소로 변경되어 클라이언트 IP가 보존되지 않는다. 
				- 로드밸런서 자체의 알고리즘을 통해 초기 노드에 도착했지만, 대상 Pod가 해당 노드에 없을 경우, 다른 노드로 트래픽을 전달한다. 이때 소스 IP가 출발지점의 노드의 IP 주소로 변경된다 - 원본 클라이언트 IP 손실
			- 그러나 이 설정은 로드를 노드 전체에 고르게 분산시킨다.  
				- Pod가 실패하거나 다른 노드로 재스케줄링되더라도, `kube-proxy`는 `iptables` 규칙을 자동으로 업데이트하며, 트래픽은 다른 정상 Pod로 계속 흐른다.
		- `externalTrafficPolicy=Local`로 설정 
			- 트래픽이 노드 외부로 라우팅되지 않고 클라이언트 IP 주소가 최종 Pod로 전파된다. 
				- 트래픽이 노드 간에 전달되지 않으므로, 일반적으로 클라이언트 IP를 노드 IP로 대체하는 Source NAT (SNAT) 작업이 방지된다. 따라서 원본 클라이언트 IP 주소는 보존되어 대상 Pod로 직접 전달된다.
				- `Local`을 사용하는 주된 이유이다.
			- 이는 클라이언트 IP를 보존하지만, 트래픽이 고르지 않게 분산될 수 있다.
			- 수신 노드의 `kube-proxy`는 `iptables` 규칙을 구성하여 외부 트래픽이 해당 특정 노드에 상주하는 Pod로만 _엄격하게_ 전달되도록 한다.
				- 만약 노드가 서비스에 대한 트래픽을 수신했지만 해당 서비스와 연결된 로컬 Pod가 없다면, `kube-proxy`는 트래픽을 다른 노드로 전달하지 않는다. 대신, 트래픽은 드롭된다.
				- `Local` 서비스의 경우, `kube-proxy`의 헬스 체크 엔드포인트는 정상이며 해당 노드에 서비스에 대한 로컬 엔드포인트(Pod)가 하나 이상 있는 경우에만 200 OK를 반환한다. 이를 통해 로드 밸런서는 로컬 엔드포인트가 없는 노드를 대상 그룹에서 제거하여 트래픽 드롭을 완화할 수 있다.
	- 기본값인 `Cluster`는 트래픽 분산을 최적화하지만, 소스 IP가 노드의 IP로 변경되어 특정 보안 기능이나 분석 도구의 효율성을 저해할 수 있다.
	- `Local`은 클라이언트 IP를 보존하여 세밀한 제어와 로깅을 가능하게 하지만, 트래픽 불균형을 초래할 수 있어 노드 자원 활용에 비효율성을 야기할 수 있다.
    
- **ALB (Ingress 서비스):** `X-Forwarded-For` HTTP 헤더를 통해 클라이언트의 원본 IP를 확인할 수 있다
	- ALB는 클라이언트 IP를 직접 보존하지는 않는다. 
		- 대신, 클라이언트의 원본 IP 주소를 `X-Forwarded-For` HTTP 헤더에 삽입한다. 
		- Pod에서 실행되는 애플리케이션은 이 헤더를 파싱하여 실제 클라이언트 IP를 검색하도록 구성되어야 한다.
	- ALB Target Group이 노드 IP를 대상으로 구성된 경우, ALB는 라우팅 알고리즘에 따라 정상 노드에 트래픽을 분산한다. 
	- ALB Target Group이 Pod IP를 직접 대상으로 하도록 구성할 수 있다. 
		- ALB의 라우팅 알고리즘이 개별 Pod IP로 트래픽을 직접 분산한다. 
		- ALB가 트래픽을 워커 노드의 IP 주소로 보낸 다음 `kube-proxy`가 Pod로 전달하는 대신, ALB가 Pod의 IP 주소로 직접 트래픽을 라우팅
	
---
### EKS 사용자 지정 네트워킹 (Custom Networking)

![](images/Pasted%20image%2020250701010226.png)

- 제한된 IP 공간 문제점
	- VPC의 기본 IP 대역이 제한적인 경우, 실행할 수 있는 Pod의 수가 제약될 수 있다. 
		- 예를 들어 `/24` CIDR은 251개의 고유 IPv4 주소만 가진다.

- 해결책: 보조 VPC CIDR 및 Custom Networking 활성화
	- VPC에 `100.64.0.0/16`과 같은 Secondary VPC CIDR 블록을 추가한다.
		- 이 CIDR은 약 65,000개의 IP를 제공하며, VPC 내에서만 라우팅 가능하다.
	- Custom Networking 활성화
		- `kubectl set env daemonset aws-node -n kube-system AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG=true` 
		- **동작 방식:** VPC CNI 플러그인은 별도의 서브넷에 보조 ENI를 생성한다. Pod에는 이 보조 ENI에서 할당된 IP만 지정된다.
		- Custom Networking은 SNAT와 조합하여 사용할 수 있다.
			-  **Pod 시작 트래픽:** `AWS_VPC_K8S_CNI_EXTERNALSNAT=false`일 때, 소스 IP는 Primary ENI IP 로 VPC 내부, 피어링된 VPC, 또는 Transit Gateway를 통한 통신 시 보존된다. 
			- **외부 트래픽에서 Pod로:** Custom Networking 구성에서 Pod로 향하는 외부 트래픽은 일반적으로 노드의 기본 ENI IP 또는 구성된 Elastic Load Balancer를 통해 라우팅된다.

---

### EKS 네트워킹 요약

- EKS 컨트롤 플레인은 AWS 관리형 VPC에, 워커 노드는 고객 VPC에 배포된다
- EKS는 두 플레인 간의 통신을 위해 고객 VPC에 ENI를 프로비저닝한다.
- **VPC CNI**는 노드 ENI의 보조 IP를 Pod에 할당하는 역할을 한다.
- **Prefix Delegation** 기능을 사용하면 Nitro 기반 인스턴스에서 노드당 Pod 수를 늘릴 수 있다.
- **사용자 지정 네트워킹(Custom Networking)**은 보조 VPC CIDR을 활용하여 Pod를 위한 더 큰 규모의 IPv4 주소 공간을 제공한다
- **SNAT** 설정을 통해 Pod가 인터넷 게이트웨이 또는 NAT 게이트웨이를 통해 외부 인터넷에 접근하는 방식을 제어할 수 있다.