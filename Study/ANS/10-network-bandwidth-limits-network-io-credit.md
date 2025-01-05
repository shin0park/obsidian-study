
## Network Bandwidth Limits

- VPC Bandwidth Limits – Internet Gateway, NAT Gateway, VPC Peering
- EC2 Bandwidth Limits
- Bandwidth over a VPN Connection & AWS Direct Connect & Transit Gateway
- Network Flow  
    • Network flow 는 **5개의 구성 요소(5-tuple)**로 정의되는 **Point-to-Point) 연결**
    - Protocol, Src IP, Src Port, Dest IP, Dest Port
    • Multiple flows allow to scale the network performance
    - 여러 플로우를 동시에 활용하면 네트워크 대역폭을 효율적으로 분산하여 고성능 네트워크 통신 가능

### VPC Bandwidth limits
- No VPC specific limits
- No limit for any Internet Gateway
- No limit for VPC peering
- NAT gateway
	- **NAT 게이트웨이**는 **최대 45 Gbps**의 대역폭을 제공
	- **45 Gbps 이상**의 대역폭이 필요한 경우, multiple NAT gateways 사용하여 확장 가능

### EC2 Bandwidth limits
![400](Pasted%20image%2020241222225912.png)
- 인스턴스의 **유형(Instance Family)**, **vCPU 개수**, **트래픽의 목적지**에 따라 대역폭이 달라짐.
- Within the Region  
    • 인스턴스가 제공하는 **전체 네트워크 대역폭**을 사용
- To other Regions, an internet gateway, or Direct Connect
    - 최신 인스턴스에서, **최소 32개의 vCPU**를 사용할 경우 네트워크 대역폭의 최대 **50%**를 사용 가능
    - 이 조건에 해당하지 않을 경우, 대역폭은 **최대 5Gbps**로 제한
#### EC2 maximum bandwidth
1. **Intel 82599 VF 인터페이스**:
    - **총 대역폭**: 최대 10 Gbps
    - **단일 흐름(Flow)**: 최대 5 Gbps
2. **AWS ENA driver**:
    - **클러스터 배치 그룹 내**: 단일 흐름당 최대 10 Gbps
    - **클러스터 배치 그룹 외부**: 단일 흐름당 최대 5 Gbps
    - **총 대역폭**: **100 Gbps** (mutiple flow을 사용할 경우, VPC 또는 피어링된 VPC 내에서 총 대역폭이 **100Gbps**에 도달할 수 있음)
3. **AWS P4d 인스턴스**:
    - **UltraClusters 슈퍼컴퓨터**에서 제공하며, **400 Gbps 네트워킹**을 지원

### VPN and DX Bandwidth

![600](Pasted%20image%2020241222230233.png)
VPN 대역폭
- Virtual Private Gateway (VGW)를 사용하는 VPN 연결의 **총 대역폭**: **1.25 Gbps**
- 동일한 Virtual Private Gateway에 여러 VPN 연결이 있더라도, **총 대역폭은 1.25 Gbps로 제한**

AWS Direct Connect
- 대역폭은 사용자가 선택한 **DX Port Speed**에 따라 결정
- Direct Connect를 통한 트래픽은 **물리적 포트의 속도**에 의해 제한

Transit Gateway
- **VPN 터널당 대역폭**: **1.25 Gbps**
- **총 VPN 대역폭**: **최대 50 Gbps**

## Network I/O Credits

- **R4**, **C5**와 같은 특정 EC2 인스턴스 패밀리는 **Network I/O credit mechanism** 사용
- 이러한 메커니즘은 네트워크 성능을 일정 수준으로 유지하면서도, **피크 트래픽 요구** 시 높은 네트워크 성능을 제공
- 대부분의 애플리케이션은 지속적으로 높은 네트워크 성능을 필요로 하지 않지만, 네트워크 크레딧 메커니즘을 통해 인스턴스는 **기본 네트워크 성능(baseline)** 이상으로 성능을 일시적으로 향상할 수 있다.
- *성능 벤치마크 테스트를 수행하기 전에, 누적된 네트워크 크레딧을 고려하는 것이 중요*

### Summary
For high network bandwidth and throughput
- **Jumbo Frames** (MTU 9001로 설정)
- **EC2 Enhanced Networking** (SR-IOV, ENA, Intel VF 82599 사용)
- **Placement Groups** (클러스터 배치 그룹)
- **EBS-Optimized Instances** (EBS 최적화 인스턴스)
- **DPDK** (운영 체제 수준에서의 네트워크 최적화)
- **EFA (Elastic Fabric Adapter)**는 ENA에 추가적으로 **운영 체제 우회 기능**을 제공하여 **HPC(고성능 컴퓨팅) 워크로드**에서 더 나은 네트워크 성능을 제공

### Exam Essentials
#### 네트워크 MTU 설정

1. VPC 내부에서는 MTU를 최대 **9001 바이트**로 설정 가능
2. **MTU가 1500 바이트로 제한되는 경우**:
    - Traffic over an internet gateway
    - Traffic over an inter-region VPC peering connection 
    - Traffic over VPN connections
3. **PPS(초당 패킷 수) 병목 현상**:
    - **MTU를 증가**시키면 PPS 병목 현상을 줄이고 처리량을 높일 수 있음.

---
#### EC2 네트워크 최적화

1. **인스턴스 수준의 최적화**:
    - **Enhanced Networking** (SR-IOV, ENA, Intel VF 82599 사용).
    - **Placement Groups**를 사용해 EC2 인스턴스 간 지연 시간 감소
    - **EBS-Optimized Instances**로 EBS I/O 성능을 개선
2. **OS 수준의 최적화**:
    - **DPDK**를 활용해 패킷 처리 성능 향상

---

#### EC2 네트워크 Bandwidth

1. **인스턴스 유형 및 크기**:
    - 인스턴스 패밀리, vCPU 수, 그리고 **Enhanced Networking 지원 여부**에 따라 네트워크 대역폭이 달라짐
2. **트래픽 목적지에 따른 대역폭 제한**:
    - **리전 내 통신**:
        - EC2 인스턴스의 **전체 네트워크 대역폭** 사용 가능
        - **다중 플로우(Multiple Flows)**를 사용하면 EC2 간 또는 EC2와 S3 간 최대 **100 Gbps** 대역폭 활용 가능
    - **리전 외부, 인터넷 또는 Direct Connect 통신**:
        - **최소 32 vCPU**를 가진 최신 인스턴스는 네트워크 대역폭의 최대 **50%** 사용 가능
        - 32 vCPU 미만인 경우, 대역폭이 최대 **5 Gbps**로 제한
3. **단일 플로우(Single Flow)의 대역폭 제한**:
    - **단일 플로우** (5-tuple 트래픽)의 대역폭은 최대 **5 Gbps**
    - Cluster placement group 내에서 최대 **10 Gbps**까지 도달 가능

