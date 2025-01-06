
## VPC Network Analysis
- Reachability Analyzer
  src, dst 를 입력하면 해당 목적지에 도달 할 수 있는지 알려준다.
  ![300](images/Pasted%20image%2020250107001006.png)
- Network Access Analyzer
  src에서 dst로 가는 모든 경로가 존재하는지를 분석 - 해당 경로가 보안 정책에 부합하는지
  모든 경로를 분석하므로, 내가 원하지 않았던 경로가 존재했는지를 체크할 수 있다.
  
  ![300](images/Pasted%20image%2020250107001020.png)

### VPC Reachability Analyzer
: VPC Reachability Analyzer는 AWS 리소스 간 연결성을 테스트하고 문제를 진단하는 도구
#### feature
- **경로 분석**: 소스 리소스에서 대상 리소스까지의 가상 네트워크 경로(hoy by hop details)를 단계별로 분석하여 연결성 확인한다.
- **차단 요소 식별**: 트래픽이 차단된 경우 문제를 일으키는 구성 요소를 식별 할 수 있다.
- **패킷 Does not send**: 실제 패킷을 전송하지 않고 네트워크 구성을 기반으로 연결 가능성을 평가한다.

##### Use case
1. **연결 문제 해결**: 네트워크 구성 오류로 인해 발생하는 연결 문제를 신속히 식별하고 해결
2. **구성 변경 자동 검증**: 네트워크 구성 변경 후 연결 상태를 자동으로 확인 가능


![500](images/Pasted%20image%2020250107002220.png)
#### Supported Source & Destination: 
• Instance
• Internet Gateway  
• Network Interfaces  
• Transit Gateway  
• Transit Gateway Attachments 
• VPC endpoints  
• VPC peering connections  
• VPN gateways

#### Intermediate components:
• ALB and NLB  
• NAT gateways  
• TGW, TGW attachment, VPC peering


The source and destination resources (제약 조건)
- Must be in the same Region
- Must be in the same VPC or VPCs connected through a VPC peering or Transit Gateway
- Can be across AWS accounts in the same AWS organization

![600](images/Pasted%20image%2020250107002311.png)


실습
- 프라이빗 서브넷에 EC2 인스턴스를 생성하고 SSH(22번 포트)로 접근
- 서브넷 route 수정하여 퍼블릭 서브넷으로 전환
- NACL의 인바운드 규칙을 제거한 후 테스트
- NACL 인바운드 규칙을 추가하고, 보안 그룹의 SSH(22)를 허용하는 인바운드 규칙을 제거하고 테스트

https://docs.aws.amazon.com/vpc/latest/reachability/what-is-reachability-analyzer.html
## VPC Network Access Analyzer
:AWS 네트워크에서 의도하지 않은 네트워크 접근을 식별하고 문제를 해결하기 위한 도구

#### feature
- 네트워크 분리 확인
	- dev 와 prod **환경 VPC** 간의 통신이 차단됐는지 보안 분리 확인
- 인터넷 접속 확인
	- 필요한 리소스만 인터넷 통신이 가능하도록 허용
	- 불필요한 인터넷 노출 방지
- 신뢰할 수 있는 네트워크 경로 
	- NAT gateway or firewalls 경로를 포함한 트래픽 경로의 안전성 확인
- 신뢰할 수 있는 네트워크 접근 
	- 특정 리소스, IP 범위, port, protocol 에서만 접근 허용
	- 네트워크 접근 범위를 정의하고, 규정 준수 여부 체크

#### Walkthrough
- Verify that 2 VPCs are isolated:
    - 두 개의 VPC가 서로 통신하지 않도록 완전히 분리되어 있는지 확인
- Setup VPC peering and check again:
    - 두 VPC 간 피어링 설정 후 네트워크 접근성을 다시 분석하여 설정이 적절한지 검토

* Pricing: $0.002 per ENI analysed for network assessment

https://docs.aws.amazon.com/ko_kr/vpc/latest/network-access-analyzer/what-is-network-access-analyzer.html