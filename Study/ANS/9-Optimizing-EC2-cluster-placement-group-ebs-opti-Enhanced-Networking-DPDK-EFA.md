## Optimizing Network Performance

### EC2 Network optimization 
• Cluster Placement Groups 
• EBS Optimized Instances 
• Enhanced Networking
-> 네트워크 처리량을 늘리는 방식들

---
### Placement Groups - Cluster
![400](Pasted%20image%2020241222200648.png)
- **클러스터 배치 그룹**은 **단일 가용 영역** 내에서 인스턴스를 논리적으로 그룹화하는 방법
- **물리적으로 가까운 인스턴스를 그룹화**하여 네트워크 지연(Latency)을 최소화하고 대역폭을 최대화할 수 있는 환경을 제공
- **HPC(고성능 컴퓨팅)**처럼 **낮은 지연 시간**이 요구되는 분산 애플리케이션에 사용
- **저지연 10Gbps 네트워크**를 제공하여 높은 네트워크 성능을 지원

tip: 시험에서 ec2인스턴스 간의 최대 네트워크 처리량에 관한 질문 -> ec2 placement group

### EBS 최적화된 인스턴스
![400](Pasted%20image%2020241222200904.png)
- **EBS(Elastic Block Store)**는 물리적인 드라이브가 아닌 **네트워크 드라이브**로, EC2 인스턴스와 네트워크를 통해 통신
- **네트워크 통신을 통해** 데이터를 처리하기 때문에 약간의 **지연(latency)**이 발생할 수 있음
- 이로 인해 EBS의 **입출력(I/O)** 성능이 네트워크 성능에 영향을 미칠 수 있음
- Amazon EBS-optimized instances는 EC2와 EBS 간에 **전용 대역폭**을 제공하여, EBS의 **입출력**과 다른 트래픽 간의 경쟁을 최소화
---

### Enhanced Networking

#### Enhanced Networking – What is it?
**고속 네트워크 성능**을 제공하는 기능
일반적으로 AWS EC2에서 사용, 온프레미스나 AWS S3에서도 사용
- **1M(일백만) PPS** 성능을 지원
- **인스턴스 간 지연 시간** 감소
- SR-IOV with PCI passthrough를 사용하여 **하이퍼바이저의 개입을 최소화**하고 **일관된 성능** 제공 (본질적으로 패킷이 한 시스템에서 시스템으로 이동할때 multiple 홉을 제거한다는 것 )
- **Intel ixgbevf(intel virtual function) 드라이버**나 **Elastic Network Adapter (ENA)**를 사용하여 활성화


#### SR-IOV and PCI for Enhanced Networking
- **SR-IOV**와 **PCI passthrough**는 **device virtualization** 방법으로, **더 높은 I/O 성능**과 **낮은 CPU 활용**을 제공
- **SR-IOV**는 **단일 물리적 NIC**(네트워크 인터페이스 카드)가 여러 개의 **vNIC**(가상 네트워크 인터페이스 카드)로 자신을 나타내도록 함.
- **PCI passthrough**는 **ENI**(Elastic Network Interface)와 같은 **PCI 장치**가 하이퍼바이저를 우회하여 **게스트 운영 체제**에 물리적으로 연결된 것처럼 보이게 함
- 이 두 기술이 결합되어 **낮은 지연 시간**과 **고속 데이터 전송**(1M PPS 이상)을 가능하게 함

#### Enhanced Networking pre-requisites
- Instance Type에 따라, Enhanced Networking은 다음 **네트워크 드라이버** 중 하나를 사용하여 활성화할 수 있다.
    - **옵션 1**: **Intel 82599 VF** (최대 10Gbps, ixgbevf 드라이버 사용)
    - **옵션 2**: **Elastic Network Adapter (ENA)** (최대 100Gbps)
- Enhanced Networking을 사용하려면 **EC2 운영 체제(AMI)** 와 **Enhanced Networking을 지원하는 Instance Type**이 필요

#### Supported Instance types  
- **Elastic Network Adapter (ENA)**를 지원하여 **최대 100Gbps** 속도를 제공하는 인스턴스:
    - A1, C5, C5a, C5d, C5n, C6g, F1, G3, G4, H1, I3, I3en 등
- **Intel 82599 VF** 인터페이스를 지원하여 **최대 10Gbps** 속도를 제공하는 인스턴스:
    - C3, C4, D2, I2, M4(단, m4.16xlarge 제외), R3 등

![500](Pasted%20image%2020241222214859.png)
- 기본적으로는 5Gbps 대역폭을 지원
- 가상레이어를 거쳐 네트워크 트래픽 처리

Intel VF
![500](Pasted%20image%2020241222214906.png)
- intel VF는 SR-IOV 기술을 기반으로 하나의 물리적 NIC를 여러개의 가상 vNIC로 분할하여 사용
- Bypassing virtualization layer: 이 가상 NIC가 하이퍼바이저를 우회하고 직접 guest os 연결된 것 처럼 작동
- 10Gbps까지 지원가능

ENA
![500](Pasted%20image%2020241222214915.png)
- **ENA**는 AWS에서 제공하는 **고성능 네트워크 인터페이스 카드**(NIC)로, 최대 **100Gbps**의 전송 속도를 지원
- 최대 100Gbps까지 지원가능하나, 단일 Flow만으로는 해당 대역폭을 채우기 어렵다.
	- flow란, 
		- 네트워크에서 **두 머신 간의 연결**을 의미하는 기본 단위
		- 두 서버 간에 발생하는 **네트워크 트래픽의 흐름**을 **flow** 라 한다.
		- 하나의 **flow**는 하나의 네트워크 세션을 의미하고, **패킷의 흐름**이 어떻게 처리될지 결정
	- 네트워크 **flow**가 하나의 **세션**을 담당하는데, 이 하나의 flow만으로는 대용량 데이터를 빠르게 전송하기 위해 필요한 대역폭을 다 사용할 수 없기 때문.
- Multiple search flow를 사용하면 100Gbps 가능
	- **여러 개의 flow**를 동시에 사용하여 **대역폭을 확장**
	- 병렬로 여러 flow 발생
	- 여러 흐름을 통해 대역폭 고르게 분배되어 100Gbps와 같은 속도 달성가능

---

## Additional network optimization techniques - DPDK & EFA

#### AdditionalTuning & Optimization - DPDK
- DPDK (Intel Data Plane Development Kit)
- **DPDK**는 **고속 패킷 처리**를 위한 라이브러리와 드라이버 세트 
- 이는 **운영 체제 내부**의 패킷 처리 성능을 최적화하는 데 사용
- **Enhanced Networking와의 차이점**
    - **Enhanced Networking**과 **SR-IOV**는 **인스턴스와 하이퍼바이저 간의 패킷 처리 오버헤드**를 줄이는 데 초점이 맞춰져 있으면
    - **DPDK**는 그보다 한 단계 더 나아가, **운영 체제 내부의 패킷 처리 오버헤드**를 줄이는 데 중점을 둠
- **DPDK provides**
    - **낮은 지연 시간(Lower Latency)**: **커널 우회(Kernel Bypass)**를 통해
    - **패킷 처리 제어 향상: 패킷 처리를 세밀하게 관리하고 최적화할 수 있음
    - 낮은 CPU 오버헤드

##### Packet processing without DPDK
![500](Pasted%20image%2020241222222539.png)
- DPDK가 없으면 패킷 처리가 운영 체제의 **커널**을 거치게 되며, 이로 인해 **추가적인 오버헤드**와 지연이 발생할 수 있다.

##### Packet processing with DPDK
![500](Pasted%20image%2020241222222642.png)
- Application이 DPDK 라이브러리를 통해 개발되면, 커널을 거치지 않고 패킷 처리 가능.

---

### EFA – Elastic Fabric Adapter

- **EFA**는 **ENA (Elastic Network Adapter)**에 추가 기능을 더한 네트워크 어댑터로, 
  특히 **고성능 컴퓨팅(HPC)** 애플리케이션에 최적화되어 있음
- EFA는 ENA와 비교해 **더 낮은 지연 시간(Lower Latency)**과 **더 높은 처리량(Higher Throughput)**을 제공
    - 리눅스 환경에서 **운영 체제 커널을 우회(OS Bypass)**하여 네트워크 성능을 극대화

![300](Pasted%20image%2020241222223009.png)
- **HPC 애플리케이션 지원**
    - **MPI (Message Passing Interface)**를 통해 **Libfabric API**와 상호 작용하여, 
      커널을 우회하고 **EFA 장치와 직접 통신**함으로써 네트워크에 패킷을 전송
    - 이를 통해 **고성능 병렬 작업**에서 최상의 성능을 제공
- Windows 인스턴스에서는 EFA가 ENA와 동일한 방식으로 동작
    - **OS Bypass 기능**은 **리눅스 환경**에서만 활성화

---
