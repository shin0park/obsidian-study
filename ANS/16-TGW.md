# Transit Gateway 

Amazon VPC Transit Gateways는 가상 프라이빗 클라우드(VPCs)와 온프레미스 네트워크를 상호 연결하는 데 사용되는 네트워크 전송 허브이다. **네트워크 아키텍처를 단순화** 할 수 있는 기능이다.

Amazon VPC Transit Gateways란 무엇입니까?
https://docs.aws.amazon.com/ko_kr/vpc/latest/tgw/what-is-transit-gateway.html


![600](images/Pasted%20image%2020250202184710.png)

![600](images/Pasted%20image%2020250202184723.png)

## AWS Transit Gateway

- 수천 개의 **VPC(가상 사설 클라우드)** 및 온프레미스 네트워크를 상호 연결할 수 있도록 지원하는 AWS 네트워크 서비스
    
- Transit Gateway attachments
    - One or more VPCs
    - Peering connection with another Transit Gateway
    - A Connect SD-WAN(Software-Defined Wide Area Network)/ third-party network appliance (타사 네트워크 어플라이언스) -> 온프레미스와 클라우드 환경을 보다 효율적으로 통합 가능
	    - SD-WAN: **소프트웨어 기반으로 WAN(광역 네트워크)의 트래픽을 최적화하고 관리하는 네트워크 솔루션**
		    - 기존의 물리적인 라우터 기반 네트워크와 달리, **클라우드 기반으로 WAN 트래픽을 유연하게 조정 가능**
		- AWS Transit Gateway는 **Cisco, Palo Alto**와 같은 서드파티 네트워크 어플라이언스를 AWS 환경에 통합할 수 있도록 지원
    - VPN
    - Direct Connect Gateway (DX)
        
- **Transit Gateway features
    - Multicast support
	    - 멀티캐스트는 데이터의 단일 스트림을 여러 수신 컴퓨터에 동시에 전달하는 데 사용되는 통신 프로토콜
	    - Transit Gateway는 연결된의 서브넷 간에 멀티캐스트 트래픽 라우팅을 지원 - 멀티캐스트 라우터 역할
	    - https://docs.aws.amazon.com/ko_kr/vpc/latest/tgw/tgw-multicast-overview.html

    - MTU(Maximum Transmission Unit) 조정 - 8,500 바이트 MTU

    - Appliance Mode 지원
	    - **Appliance Mode**는 **가상 방화벽, IDS/IPS, NAT, VPN 게이트웨이 등과 같은 네트워크 보안 장비를 통합**할 때 사용된다.
	    - 해당 모드를 사용하면,TGW가 트래픽을 항상 같은 AZ 내의 동일한 보안 장비로 보내도록 보장한다. 
	    - 대부분 보안장비(Firewall, NAT) 는 일반적으로 Stateful 방식으로 작동하기에, 트래픽의 요청과 응답이 같은 경로로 돌아와야 하므로, Appliance Mode를 지원.

    - 가용 영역(AZ) 고려 사항
	    - TGW에 vpc를 연결할때 , 반드시 한 개 이상의 AZ를 선택해야 한다.
	    - 선택된 AZ마다 **하나의 서브넷을 지정해야 하며**, **Transit Gateway가 해당 서브넷에 네트워크 인터페이스를 생성**하여 IP 주소를 할당한다.
	    - 한 번 AZ가 활성화되면, **VPC 내 모든 서브넷과의 트래픽 라우팅이 가능하다.** (TGW와 하나의 서브넷만 연결되면 나머지 서브넷끼리는 VPC 내부통신을 하면 되므로)
	    - 특정 AZ에서만 TGW가 활성화된 경우, 그 **AZ 안에 있는 리소스들만 TGW에 직접 접근 가능하다.**
	    - 그렇다고 **한 개의 AZ에서 TGW가 활성화되었다고 해서 해당 AZ 안의 서브넷만 연결되는 것은 아님** → VPC 내 모든 서브넷과 통신 가능하다. 
	    - Cross-AZ 트래픽 라우팅: TGW와 직접 연결 안된 서브넷에서 직접 연결된 서브넷으로 VPC 내부 통신한 다음, 해당 서브넷에서 TGW로 통신하면 되므로 결국 모든 서브넷과는 통신 가능 
	      -> 이 과정에서 추가적은 TGW 요금이 발생하는 건 아님.
	      
	    - 다중 AZ 활성화 권장: 가용성을 위해서는 여러개의 AZ를 활성화하는 것이 좋음.
	    - https://docs.aws.amazon.com/vpc/latest/tgw/how-transit-gateways-work.html#tgw-az-overview

    - Transit Gateway Sharing
	    - **AWS Resource Access Manager(RAM)**을 사용하여 **Transit Gateway를 여러 AWS 계정과 공유 가능**
	    - 한 개의 AWS 계정에서 생성한 Transit Gateway를 여러 계정에서 사용할 수 있도록 공유할 수 있음

- **Transit Gateway architectures**
    - Centralized traffic inspection 중앙 집중식 트래픽 제어 
    - egress 아웃바운드 인터넷 트래픽 관리
	    - VPC에서 외부 인터넷으로 나가는 트래픽을 **중앙에서 제어(Egress Traffic Control)**
	    - 모든 VPC가 직접 인터넷에 연결되지 않고, **Transit Gateway를 통해 중앙 NAT Gateway 또는 인터넷 게이트웨이(IGW)를 공유**
		    - 인터넷 트래픽 중앙 집중식으로 관리
		    - 비용 절감: 모든 VPC에 개별적으로 NAT 게이트웨이를 배포하는 대신, **공유 NAT 게이트웨이**를 사용
		    - 보안 강화: 외부 인터넷으로 나가는 모든 트래픽을 중앙 게이트웨이를 거치도록 설정
    - interface endpoints
	    - TGW + Interface Endpoint를 함께 사용
	    - 각 VPC마다 interface endpoints(s3, dynamodb 연결을 위한)를 만들지 않고, TGW를 통해 중앙 VPC에서만 interface를 설정하여, 모든 VPC가 해당 TGW를 통해 해당 interface endpoint를 사용하도록 설정 가능. -> 단순화


![600](images/Pasted%20image%2020250202184748.png)
- 서로 다른 **AWS 리전에 위치한 Transit Gateway 간의 연결**을 제공
- 글로벌 규모의 네트워크 연결이 필요할 때 사용됨

![600](images/Pasted%20image%2020250202184800.png)
- VPN 연결을 통해 여러 VPC 및 온프레미스 네트워크를 통합 가능

![600](images/Pasted%20image%2020250202184811.png)

![600](images/Pasted%20image%2020250202184823.png)
https://docs.aws.amazon.com/ko_kr/prescriptive-guidance/latest/inline-traffic-inspection-third-party-appliances/vpc-to-vpc-traffic-inspection.html
VPC-to-VPC 트래픽 검사
"VPC-to-VPC traffic inspection occurs when traffic originates from one VPC and is destined for another VPC. The traffic is redirected to an appliance VPC for traffic inspection before arriving at the destination VPC."
어느 한 vpc에서 시작하여 다른 vpc에 도달할때 발생. 목적지 vpc에 도착하기 전 appliance vpc에 리다이렉트되어 트래픽 검사를 함.

## Transit Gateway VPC attachments

![500](images/Pasted%20image%2020250202191748.png)
#### Transit Gateway Attachment란?

-> Transit Gateway에 연결되는 네트워크 엔드포인트로, TGW에 연결된 네트워크 리소스(VPC, VPN, Direct Connect Gateway 등)를 의미한다.

- TGW를 사용하여 다중 VPC나 아키텍처를 연결하기 위해서는 우선 Attachment를 생성해야한다.
- attachment를 생성할 때는 어느 리소스를 연결할 것인지 즉, VPC인지 VPN인지 등을 선택할 수 있다.
	-  VPCs should not have overlapping CIDRs
- **고유 ID**: 각 Attachment는 고유한 ID를 가지며, TGW와 연결된 네트워크를 식별한다.


![](images/Pasted%20image%2020250202191917.png)
#### Transit Gateway Route Table이란?

-> TGW에서 트래픽을 올바른 목적지로 보내기 위한 라우팅 설정하기 위한 라우팅 테이블

- TGW 에는 **VPC, 온프레미스 네트워크, VPN, Direct Connect 등의 경로를 관리하기 위한 별도 Route Table 이 존재**
- 즉, **어떤 네트워크(Attachment)에서 온 트래픽을 어떤 네트워크(Attachment)로 보낼지**를 결정하는 역할
- TGW를 생성하면 default TGW Route Table이 생성된다.
- Route Table은 연결된 네트워크를 통해 트래픽이 흘러가는 것을 제어한다.

Amazon VPC Transit Gateway의 작동 방식
라우팅, 라우팅 테이블
https://docs.aws.amazon.com/ko_kr/vpc/latest/tgw/how-transit-gateways-work.html#tgw-route-tables-overview

---

#### Association 연결

- Association은 **TGW 라우팅 테이블**과 **Attachment** 간의 연결 관계를 의미한다.
- "Association을 통해 특정 Attachment의 트래픽이 어떤 라우팅 테이블을 사용할지 결정"
	- **트래픽 경로 결정**: Association을 통해 해당 Attachment로 들어오는 트래픽이 어떤 라우팅 테이블을 참조할지 결정
- **기본 라우팅 테이블**: TGW를 생성하면 기본 라우팅 테이블이 자동으로 생성되며, 새로운 Attachment는 기본적으로 이 라우팅 테이블과 연결된다.
- 추가 route table 을 생성할 경우 Attachment를 Association 해야 한다.

![](images/Pasted%20image%2020250202191923.png)
#### **Propagation(전파)**

Route Propagation(라우팅 전파)은 특정 **Attachment가 자신의 네트워크 경로를 TGW에 자동으로 등록하도록 설정하는 기능**이다.

VPC, VPN 연결 또는 Direct Connect 게이트웨이는 라우팅을 TGW 라우팅 테이블에 동적으로 전파할 수 있으며, 이 기능을 통해 네트워크 관리자는 수동으로 경로를 추가하지 않고도, 연결된 네트워크 간의 통신을 자동으로 설정할 수 있다.

- 경로 정보 수집: TGW는 연결된 네트워크로부터 경로 정보를 수집한다.
- 경로 전파 활성화: TGW 라우팅 테이블에서 해당 Attachment의 **Route Propagation**을 활성화하면, TGW는 연결된 네트워크로부터 학습한 경로를 해당 라우팅 테이블에 자동으로 추가한다.
	- Propagation은 Attachment 별로 활성화 또는 비활성화할 수 있다.
	- ex) VPC 연결에서 경로 전파를 활성화하면, 해당 VPC의 CIDR 블록이 TGW 라우팅 테이블에 추가된다.
- 동적 라우팅: Route Propagation은 동적 라우팅 프로토콜(BGP 등)을 통해 경로를 학습할 수도 있다.
	- ex) VPN 연결이나 Direct Connect Gateway를 통해 on-premise 네트워크의 경로를 TGW 라우팅 테이블에 전파할 수도 있다.
- 유연성: 네트워크 구성이 변경되더라도 TGW가 자동으로 경로를 업데이트하므로, 관리 부담이 줄어든다.

![](images/Pasted%20image%2020250202191932.png)
- VPC와 서브넷에도 이에 상응하는 Route table이 존재해야 한다.
- 경로는 양쪽에서 모두 관리해야한다.
- 각 VPC 라우팅 테이블에도 해당 CIDR 대역은 TGW로 전달되도록 경로를 추가한다.

---


![](images/Pasted%20image%2020250202213322.png)
1. **3개의 VPC 및 프라이빗 서브넷 생성**
	- VPC A: 퍼블릭 서브넷(we need that for jump host) + 프라이빗 서브넷
	- VPC B, VPC C: 프라이빗 서브넷
2. Create Transit Gateway
3. Create 3 VPC attachments for the Transit Gateway
4. Modify all Private subnet route table and add route for 10.0.0.0/8 via the Transit gateway attachment
5. SSH to VPC A Jump host -> SSH to EC2-A -> Ping to EC2-B or EC2- C using a Private IP
---


![](images/Pasted%20image%2020250202213307.png)
1. **3개의 VPC 및 프라이빗 서브넷 생성**
	- VPC A: 퍼블릭 서브넷(we need that for jump host) + 프라이빗 서브넷
	- VPC B, VPC C: 프라이빗 서브넷
2. Create Transit Gateway 
   (Do not enable default route table association and default route table propagation)
3. 각 VPC에 대한 Transit Gateway 연결(Attachment) 생성
4. **각 VPC에 맞는 개별 Transit Gateway 라우트 테이블 생성 및 연결**
5. For att-a route table add the route propagation for att-b
6. For att-b route table add the route propagation for att-a and att-c
7. For att-c route table add the route propagation for att-b
8. SSH to VPC A Jump host -> SSH to EC2-A -> Ping to EC2-B or EC2-C using a Private IP. **EC2-B는 연결 가능, EC2-C는 연결 차단되어야 함**
9. Similarly from EC2-B try to SSH/Ping to EC2-A and EC2-C. 
   **EC2-A 및 EC2-C 모두 연결 가능해야 함**

---

## Transit Gateway – VPC Network patterns

![](images/Pasted%20image%2020250202221121.png)
TGW에 연결된 모든 VPC가 서로 통신할 수 있는 구조

![](images/Pasted%20image%2020250202221130.png)
- vpc끼리 통신하는 것은 원하지 않으며, VPC에서 온프렘으로 연결을 원하는 경우
- vpc route table에 온프렘으로 가는 경로를 TGW를 타도록 설정.


---
## VPC & Subnets design for Transit Gateway

![](images/Pasted%20image%2020250202221206.png)
#### Availability zone considerations

- VPC를 Transit Gateway(TGW)에 연결할 때, 하나 이상의 가용 영역(AZ)을 활성화해야 한다.
- 각 **가용 영역을 활성화하려면 해당 AZ에서 하나의 서브넷을 지정해야한다** 
  (typically /28 range to save IPs for workload subnets)
- TGW는 해당 서브넷에 네트워크 인터페이스를 배치하며, 해당 서브넷의 IP 주소 하나를 사용한다.
- AZ를 활성화하면, **VPC 내 모든 서브넷과의 트래픽 라우팅이 가능하다**
  (TGW와 하나의 서브넷만 연결되면 나머지 서브넷끼리는 VPC 내부통신을 하면 되므로)
  
- 특정 AZ에서만 TGW가 활성화된 경우, 그 **AZ 안에 있는 리소스들만 TGW에 직접 접근 가능하다.**
- 그렇다고 **한 개의 AZ에서 TGW가 활성화되었다고 해서 해당 AZ 안의 서브넷만 연결되는 것은 아님** 
  → VPC 내 모든 서브넷과 통신 가능하다. 
- 단지, TGW attachment가 없는 AZ의 리소스들은 직접 TGW와 통신할 수 없는 것이다.
- Cross-AZ 트래픽 라우팅: TGW와 직접 연결 안된 서브넷에서 직접 연결된 서브넷으로 VPC 내부 통신한 다음, 해당 서브넷에서 TGW로 통신하면 되므로 결국 모든 서브넷과는 통신 가능 
  -> 이 과정에서 추가적은 TGW 요금이 발생하는 건 아님.
  
- 다중 AZ 활성화 권장: 가용성을 위해서는 여러개의 AZ를 활성화하는 것이 좋음.
- https://docs.aws.amazon.com/vpc/latest/tgw/how-transit-gateways-work.html#tgw-az-overview
---
## Transit Gateway AZ affinity & Appliance mode

![](images/Pasted%20image%2020250202221241.png)
- source and destination in same AZ
- TGW는AZ Affinity(선호)가 있다. 통신을 가능한 한 AZ에서 하려고 한다.
- TGW는 목적지에 도달할 때까지 동일한 원래의 AZ 에 트래픽을 유지하려고 노력한다.

![](images/Pasted%20image%2020250202221251.png)
- 만약 출발지와 목적지의 AZ가 다를 경우
- 응답 트래픽에서 B인스턴스는 자신의 AZ에 위치한 TGW ENI로 트래픽을 보낼것이고, 해당 트래픽이 TGW를 지날때 TGW는 원래의 AZ인 AZ2로 유지하려고 할 것이다.

### Stateful Appliance

#### Appliance Mode Disable
![](images/Pasted%20image%2020250202221301.png)
- stateful Appliance: 트래픽의 요청과 응답이 같은 경로로 돌아와야한다.
- B Appliance는 해당 트래픽에 대한 State를 갖고 있지 않기 때문에 이 트래픽을 단순히 삭제해 버린다.
#### Appliance Mode Enable
![](images/Pasted%20image%2020250202221315.png)
- Appliance Mode를 활성화 하면, TGW는 Appliance VPC의 Single Network ENI를 사용한다.
	- fixed only one ENI for the flow
- flow hash 알고리즘을 사용, 오직 특정 ENI(origin의 traffic이 지나간)로만 traffic을 보내도록 한다.
- AWS SDK, CLI : Use --option `ApplianceModeSupport=enable` attachment 생성이나 수정 시

---

- **Appliance Mode**는 **가상 방화벽, IDS/IPS, NAT, VPN 게이트웨이 등과 같은 네트워크 보안 장비를 통합**할 때 사용된다.
	    - 해당 모드를 사용하면,TGW가 트래픽을 항상 같은 AZ 내의 동일한 보안 장비로 보내도록 보장한다. 
	    - 대부분 보안장비(Firewall, NAT) 는 일반적으로 Stateful 방식으로 작동하기에, 트래픽의 요청과 응답이 같은 경로로 돌아와야 하므로, Appliance Mode를 지원.
---

## Transit Gateway Peering

![](images/Pasted%20image%2020250202225803.png)
- TGW는 **regional router** 로, 같은 region의 VPC와 연결할 수 있다.
- across the region의 연결을 위해서는 다른 리전의 TGW와 peer 연결할 수 있다.
	-  vpc peering과 유사하며 한 TGW가 request, 다른 하나의 TGW가 accept가 된다.
	- across AWS Account 도 가능.
- Peering connection을 위해서는 Static routes가 필요하다 (no BGP)

![](images/Pasted%20image%2020250202225811.png)
- 리전 간의 트래픽은 AWS global network에 의해 모두 암호화된다. 
- 해당 연결은 private, 인터넷으로 노출되지 않는다.
- 지원하는 Bandwidth 은 50Gbps
- Use unique ASNs(Autonomous System Number)

Transit Gateway Connect attachments and Transit Gateway Connect peers in Amazon VPC Transit Gateways
https://docs.aws.amazon.com/ko_kr/vpc/latest/tgw/tgw-connect.html?utm_source=chatgpt.com
