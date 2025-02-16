## Transit Gateway Connect Attachment
### TGW
- AWS Transit Gateway는 여러 VPC 및 온프레미스 네트워크를 단순화하여 연결하는 중앙 허브 역할을 한다
- 개별적인 VPC Peering 연결을 설정할 필요 없이 다수의 VPC를 단일 Transit Gateway를 통해 연결 가능하다.
- 온프레미스 네트워크와도 AWS Direct Connect 및 VPN을 통해 연결할 수 있다.

### Transit Gateway - Connect Attachment

- **Transit Gateway Connect Attachment**는 Transit Gateway와 VPC에서 실행 중인 SD-WAN(Software-Defined Wide Area Network) 어플라이언스와 같은 서드파티 가상 어플라이언스 간의 연결을 설정하기 위해 사용된다.
	- SD-WAN: **소프트웨어 기반으로 WAN(광역 네트워크)의 트래픽을 최적화하고 관리하는 네트워크 솔루션**
		- 기존의 물리적인 라우터 기반 네트워크와 달리, **클라우드 기반으로 WAN 트래픽을 유연하게 조정 가능**
    
- Connect Attachment는 기존의 VPC 또는 AWS Direct Connect Attachment를 기본 전송 메커니즘으로 사용한다.
- GRE(Generic Routing Encapsulation) 터널 프로토콜을 사용하여 고성능 연결 제공
- BGP(Border Gateway Protocol)를 사용하여 동적 라우팅 지원

![](images/Pasted%20image%2020250209204918.png)
- vpc attachment를 통해 vpc와 tgw 연결
- connect attachment를 통해 appliance가 있는 vpc와 연결하는데,
- 이때 연결하면, 고성능의 GRE Tunnel이 생성되며, 이를 통과하는 BGP session 맺어져 동적라우팅을 지원
- TGW 쪽에 TGW BGP Peer IP , appliance 쪽에 BGP Peer IP를 부여

---
### VPC Transport Attachment를 통한 Transit Gateway Connect Attachment

![](images/Pasted%20image%2020250209205223.png)
Document: https://aws.amazon.com/blogs/networking-and-content-delivery/simplify-sd-wan-connectivity-with-aws-transit-gateway-connect/

- AWS 클라우드에서 VPC Attachment를 통해 Connect Attachment를 생성
- Connect VPC와 온프렘의 SD-WAN을 DX를 통해 연결하거나 인터넷 기반 연결을 사용할 수 있지만, 궁극적으로 온프레미스 네트워크를 VPC 로 **확장** 하는 것이다.
- GRE 터널과 BGP 피어링을 사용하여 연결을 설정
    
### DX Transport Attachment를 통한 Transit Gateway Connect Attachment

![](images/Pasted%20image%2020250209205307.png)

- Transit Gateway Connect은 온프레미스 네트워크에서 실행되는 서드파티 브랜치 또는 고객 게이트웨이 어플라이언스와 연결할 수 있으며, AWS Direct Connect를 전송 메커니즘으로 사용한다.

### Transit Gateway Connect Attachment의 주요 사항

- Connect Attachment는 static routing 을 지원하지 않으며, BGP가 최소 요구사항이다.
- 각 GRE 터널당 최대 5Gbps의 대역폭을 지원하며, 5Gbps 이상의 대역폭은 동일한 Connect Attachment에 대해 여러 GRE 터널(Connest Peer)을 통해 동일한 prefix를 광고함으로써 달성할 수 있다.
- 각 Connect Attachment당 최대 4개의 Connect peer를 지원하며, 이를 통해 총 20Gbps의 대역폭을 제공할 수 있다.

---

## Transit Gateway with VPN

Site-to-Site VPN를 사용하여 연결할 경우는 아래와 같다.

#### **AWS Site-to-Site VPN란 무엇인가요?**
https://docs.aws.amazon.com/ko_kr/vpn/latest/s2svpn/VPC_VPN.html

- 기본적으로 Amazon VPC에서 인스턴스를 실행하면 **사용자의 원격 네트워크(온프레미스 네트워크)와 직접 통신할 수 없다**.
- AWS Site-to-Site VPN **연결을 생성하고, 트래픽이 해당 연결을 통해 전달될 수 있도록 라우팅을 구성하면** VPC에서 원격 네트워크로의 접근을 활성화할 수 있.
- **VPN 연결이란 VPC와 온프레미스 네트워크 간의 연결을 의미한다**. 
- Site-to-Site VPN은 **IPsec(Internet Protocol Security) 기반의 VPN 연결을 지원합니다**.

- **Customer gateway**: Customer gateway device에 대한 정보를 AWS에 제공하는 AWS 리소스
- **가상 프라이빗 게이트웨이(Virtual private gateway)** VGW: 가상 프라이빗 게이트웨이는 단일 VPC에 연결할 수 있는 사이트 간 VPN 연결의 Amazon 측 VPN 엔드포인트이다.

#### Single Site-to-Site VPN Connection over Virtual Private Gateway (VGW)
![](images/Pasted%20image%2020250209210519.png)
-  VGW, Single Site-to-Site VPN Connection, Customer Gateway를 통해 1:1 연결을 할 수 있다.

#### Connecting multiple branch offices over Site-to-Site VPN
![](images/Pasted%20image%2020250209210557.png)
- 여러 지사 사무실을 Site-to-Site VPN으로 연결할 수 있다.

#### Connecting multiple VPCs over Site-to-Site VPN connections
![](images/Pasted%20image%2020250209210534.png)
- 여러 VPC를 여러개의 VGW와 Site-to-Site VPN 연결로 연결할 수 있다.

#### Simplify Site-to-Site VPN network with Transit Gateway
Transit Gateway를 사용하여 Site-to-Site VPN 네트워크를 단순화할 수 있다.

![](images/Pasted%20image%2020250209210654.png)
- VGW 대신, VPC attachment를 사용하면 된다.

![](images/Pasted%20image%2020250209210715.png)

- 여러 VPC attachment 그리고 여러 VPN attachments, VPN Connection를 통해 다중 연결이 가능하다.
- 데이터 센터 네트워크 간의 하이브리드 연결을 구현할때 좋다.


### Accelerated Site-to-Site VPN Network
좀더 가속화된 연결 가능
![](images/Pasted%20image%2020250209210735.png)
- AWS Global Accelerator를 사용하여 고객 게이트웨이에서 가장 가까운 AWS edge location 으로 트래픽을 라우팅할 수 있다.
	- 즉, 온프레미스 네트워크의 트래픽이 가장 가까운 edge location으로 이동해 그곳에서부터 AWS backbone 네트워크에 연결할 수 있다.
	- VPN connection은 인터넷을 통해 연결되는데 이를 사용하면 일부만 인터넷을 타게되고, 나머지는 모두 AWS backbone망에서 흐르게 된다.
- 이는 Transit Gateway VPN Attachment에서만 사용할 수 있으며, VGW에서는 지원되지 않는다.

---
#### ECMP (Equal-Cost Multi-Path)를 통한 고성능 VPN

#### Limited n/w throughput with multiple VPN Connections over VGW
original 네트워크에서는 아래와 같다.
![](images/Pasted%20image%2020250209212835.png)
- VGW과 VPN Connection 처리량 limit이 1.25Gbps 이다.
- 따라서 처리량을 늘리기 위해 VPN connection을 2개로 늘려도 2배가 되지 못한다.
- 다중 연결의 트래픽을 로드밸런싱 하기 위해서는 ECMP가 필요하다.
- VGW는 ECMP를 지원하지 않는다.

#### ECMP over Transit Gateway & Site-to-Site VPN for higher aggregate throughput
![](images/Pasted%20image%2020250209210808.png)

- Transit Gateway 를 사용하면 BGP와 ECMP를 사용할 수 있다.
- Transit Gateway와 Site-to-Site VPN을 통해 ECMP를 활성화하면 더 높은 집계 처리량을 달성할 수 있다.
- 2개의 VPN 연결과 각 연결당 2개의 터널을 사용하면 약 5Gbps의 처리량을 달성할 수 있다.
- 한개의 flow의 limit은 여전히 1.25Gbps 이다.

#### ECMP over Transit Gateway with dual VPN
![](images/Pasted%20image%2020250209210819.png)
- CIDR 의 반은 첫번째 VPN connection으로, 나머지 반은 두번째 VPN Connection으로 사용된다.
- static 라우팅은 사용할 수 없으며 BGP 라우팅을 구성해야한다.

---

## Transit Gateway with Direct Connect

#### Connecting without Transit Gateway
![](images/Pasted%20image%2020250209214703.png)
(DX에 대한 자세한 내용은 추후 챕터에서)
- Using Direct Connect Gateway over a Private VIF
- Allows connecting maximum 10 VPCs

즉 100개의 VPC를 연결하고자 할때는 위 방식으로는 불가하다.
이때 TGW를 통한 DX 연결을 고려해 볼 수 있다.

---
### Direct Connect Gateway with Transit Gateway

![](images/Pasted%20image%2020250209214756.png)
### Transit Gateway와 Direct Connect
 
- Transit Gateway와 Direct Connect Gateway를 함께 사용하면 단순하고 확장 가능한 멀티 리전, 멀티 계정 연결을 제공한다.
- TGW는 한 리전에서 수백 수천개의 VPC와 연결 가능하기때문에 이 아키텍처는 매우 scalable하게 될 수 있다.
- 단일 AWS Direct Connection은 최대 4개의 Transit VIF를 지원하며, 단일 Direct Gateway는 최대 6개의 Transit Gateway에 연결할 수 있다.
- 따라서 하나의 AWS Direct Connection에 최대 24 TGW가 연결 가능하다. (리전 당)
    

### IPSec VPN to Transit Gateway over Direct Connect

![](images/Pasted%20image%2020250209214914.png)
- Public VIF는 모든 AWS public IP들에 대해 연결을 허용하며, TGW는 TGW VPN endpoints 에  a pair fo public IPs 를 제공한다.
- Direct Connect를 통해 Transit Gateway와 IPSec VPN을 설정할 수 있으며, 이는 Layer 3에서 트래픽을 암호화한다.

---
## Multicast with Transit Gateway


### Multicast
https://en.wikipedia.org/wiki/Multicast
![300](images/Pasted%20image%2020250209221317.png)
- **멀티캐스트**는 여러 컴퓨터에 동시에 단일 데이터 스트림을 전달하는 통신 프로토콜이다.
	- ex) OTT, 스트리밍, 금융 거래, 실시간 데이터 공유, 이메일 전송
- 멀티캐스트는 단일 또는 다수의 소스와 목적지를 가질 수 있다.
- 목적지는 멀티캐스트 그룹 주소로, Class D 범위인 `224.0.0.0`에서 `239.255.255.255`까지 사용된다.
- 멀티캐스트는 connectionless UDP 기반 전송을 사용하며, 단방향 통신이다.

#### **Multicast** 구성 요소
- **멀티캐스트 도메인**: 멀티캐스트 트래픽이 전달되는 논리적 영역
- **멀티캐스트 그룹 및 멤버**: 멀티캐스트 그룹은 특정 IP 주소를 가지며, 멤버는 이 그룹에 가입하여 데이터를 수신한다.
- **멀티캐스트 소스 및 수신자**: 데이터를 보내는 소스와 데이터를 받는 수신자
- **IGMP (Internet Group Management Protocol)**: 멀티캐스트 그룹 멤버십을 관리하는 프로토콜
    
#### Transit Gateway에서의 Multicast 지원

![300](images/Pasted%20image%2020250209221330.png)

- Transit Gateway는 연결된 VPC의 서브넷 간 멀티캐스트 트래픽을 라우팅할 수 있다.
- Transit Gateway는 멀티캐스트 트래픽을 지원하며, IPv4 및 IPv6 주소를 지원한다.
- 외부 application과의 하이브리드 통합을 지원한다.
-  static (API based) 와 IGMP프로토콜을 통한 Dynamic group membership 을 지원한다. (supports IGMPv2) 
	- static (API based): API를 통해 멀티캐스트 그룹 수동을 구성
	- IGMP를 통해 자동으로 멀티캐스트 그룹에 참여 및 탈퇴 가능

### Multicast traffic in a VPC

![600](images/Pasted%20image%2020250209222154.png)

1. 멀티캐스트 도메인을 생성하고 참여하는 서브넷을 추가한다.
2. 멀티캐스트 그룹을 생성하고 그룹 멤버십 IP(예: `224.0.0.100`)를 연결한다.
3. CLI/SDK를 사용하여 정적으로 또는 IGMPv2를 사용하여 동적으로 그룹 멤버십을 구성한다.
4. 소스에서 멀티캐스트 그룹 IP로 트래픽을 전송한다.
5. 그룹의 모든 멤버가 멀티캐스트 트래픽을 수신한다.
6. 멀티 VPC 및 멀티 account 아키텍처에서도 유사한 네트워크 흐름이 작동한다.

---
#### Integrating external multicast services
외부 Multicast 서비스 통합

![](images/Pasted%20image%2020250209222704.png)

- TGW는 DX나 Connect attachment를 통한 multicast를 지원하지 않는다.
- 따라서 온프렘 데이터 센터를 VPC와 연결하고, transport mechanism을 기반으로 한다. 연결은 VPN or DX or Internet
- 연결이 됐다면 virtual router를 vpc 내부에 구성할 수 있다. 그리고 이 virtual router는 온프렘 router와의 GRE 터널을 생성할 수 있다. 
- GRE 터널을 활성화 했다면, 이제 TGW를 통해 ec2 인스턴스와 내부 VPC의 ENI를 연결한다. 
- multicast domain을 생성하고, multicast group을 생성한다.
- -> 온프렘의 multicast router로 부터 VPC 내부의 virtual router로 트래픽이 전송된 다음, TGW로 전송되어 TGW는 multicast group의 모든 멤버에게 이를 multicast한다.
- Direct Connect, VPN 또는 인터넷을 통해 외부 멀티캐스트 라우터와 PIM(Protocol Independent Multicast) neighbour 관계를 설정할 수 있다. 

---

### Multicast considerations for TGW

- 서브넷은 하나의 멀티캐스트 도메인에만 속할 수 있다.
- 서브넷 내의 호스트(ENI)는 멀티캐스트 도메인 내에서 하나 이상의 멀티캐스트 그룹에 속할 수 있다.
- 멀티캐스트 그룹 멤버십은 VPC 콘솔, AWS CLI 또는 IGMP를 통해 관리된다.
- **IGMPv2**를 지원하는 경우, 멤버는 JOIN 또는 LEAVE 메시지를 보내 그룹에 가입하거나 탈퇴할 수 있다.
- Transit Gateway는 2분마다 모든 멤버에게 IGMPv2 QUERY 메시지를 보내고, 각 멤버는 IGMPv2 JOIN 메시지로 응답하여 멤버십을 갱신한다.
- IGMP를 지원하지 않는 멤버는 Amazon VPC 콘솔 또는 AWS CLI를 통해 직접 그룹에 추가하거나 제거해야 한다.
- **igmpv2Support**: 이 속성이 활성화되면 멤버가 JOIN 또는 LEAVE 메시지를 보낼 수 있다.
- **StaticSourcesSupport**: 이 속성은 그룹에 static multicast sources가 있는지 아닌지의 multicast domain 속성을 결정한다.

- **Nitro 인스턴스가 아닌 경우**: Nitro 인스턴스가 아닌 경우 멀티캐스트 sender가 될 수 없다. 
  만약 Nitro 인스턴스가 아닌 receiver를 사용하는 경우, Source/Destination check를 비활성화해야 한다.
    
- **지원되지 않는 연결**: 멀티캐스트 라우팅은 AWS Direct Connect, Site-to-Site VPN, TGW 피어링 Attachment, Transit Gateway Connect Attachment에서는 지원되지 않는다.
    
- **보안 그룹 및 NACL 구성**: IGMP 호스트(인스턴스)의 보안 그룹 및 서브넷의 ACL 구성은 이러한 IGMP 프로토콜 메시지를 허용해야 한다.

![](images/Pasted%20image%2020250209224816.png)
- **IGMP 쿼리 패킷**: IGMP 쿼리 패킷의 소스 IP는 `0.0.0.0/32`, 목적지 IP는 `224.0.0.1/32`이며, 프로토콜은 IGMP(2)

- AWS account, OU inside its organization or across organization 에서 multicast domain을 공유할 수 있다.
- multicast domain 공유는 또한 AWS RAM과 통합할 수도 있다.

---

## Transit Gateway - Centralized egress to internet

### Centralized routing using Transit Gateway

![](images/Pasted%20image%2020250209225029.png)
- 인터넷으로의 egress를 중앙 집중 통제하는 아키텍처

### Centralized egress to internet with NAT gateway

![](images/Pasted%20image%2020250209225050.png)

![](images/Pasted%20image%2020250209225118.png)

- **Transit Gateway**를 사용하여 중앙 집중식 라우팅을 구성할 수 있다.
- 각 가용 영역(AZ)에 NAT Gateway를 배치하여 고가용성과 AZ 간 데이터 전송 비용을 절감할 수 있다.
- NAT Gateway는 최대 55,000개의 동시 연결을 지원하며, 5Gbps에서 100Gbps까지 확장할 수 있다.

- Transit Gateway 라우팅 테이블에 **black hole**를 추가하여 VPC 간 트래픽을 제한할 수 있다.

(black hole 제거 했을 경우)
![](images/Pasted%20image%2020250209230908.png)
- vpc1 -> vpc2로 갈때
- vpc1 라우팅 테이블에 10.2.0.0/16 대역에 해당하는 라우트가 0.0.0.0/0 밖에 없으므로, tgw로 전달된 다음 tgw 라우팅 테이블에 의해 egress-vpc-att 로 전달되고 nat gateway로 전달된다음, nat gateway의 라우팅 테이블의 10.0.0.0/8 대역은 tgw으로 가라는 라우트에 의해 다시 tgw로 간다.
- 그다음 tgw의 돌아오는 테이블에 의해  tgw-att-vpc-2를 타고 vpc2에 도달
- 매우 비효울적인 루트이다.

![](images/Pasted%20image%2020250209230924.png)
- 따라서 black hole을 없애면, vpc1, vpc2로 가는 라우트를 tgw에 추가해줘야 바로 vpc1 -> vpc2로의 이동이 가능해진다.
---
- 각 AZ에 NAT Gateway를 배치하여 고가용성을 보장하고, 하나의 AZ가 완전히 실패할 경우 다른 AZ의 NAT Gateway를 통해 트래픽을 전달할 수 있다.
- 이 아키텍처는 VPC당 NAT Gateway 비용(e.g. ~$0.045/hr + ~$0.045/GB) 대신 Transit Gateway Attachment 및 데이터 처리 비용을 추가(~$0.05/hr per VPC attachment + ~$0.02/GB) 할 수 있으므로 비용 절감 효과가 없을 수 있다.

---
## Centralized traffic inspection with Gateway Load Balancer & Network Appliance

![](images/Pasted%20image%2020250216224145.png)
- **Gateway Load Balancer (GWLB)**
	- 방화벽, 침입 탐지 및 방지 시스템, 심층 패킷 검사 시스템과 같은 가상 어플라이언스를 배포, 규모 조정 및 관리할 수 있다. 
	- 여러 가상 어플라이언스에 트래픽을 분산하는 동시에, 수요에 따라 규모를 늘리거나 줄일 수 있는 단일 게이트웨이를 제공한다.
    
- GWLB는 **GENEVE** 프로토콜을 사용하여 트래픽을 캡슐화하고, 네트워크 어플라이언스로 전달한다.
- GWLB는 간단하고, 낮은 지연 시간, 고가용성, 확장성을 제공한다.
- **GWLB endpoint(GWLBE)**: 트래픽을 GWLB로 전달하는 역할 수행

    
![](images/Pasted%20image%2020250216224029.png)

![](images/Pasted%20image%2020250216224123.png)
- 인터넷으로 나갈때 동일하게 GWLBE까지 온다음 GWLB를 타고 어플라이언스 검사를 한 다음, 다시 GWLBE로 도달. 마치 아무일이 일어나지 않고 GWLB에 머물러 있는 것처럼 동작
- GWLB RT에 따라 0.0.0.0/0은 Nat GW로 전달 그다음, IGW를 통해 인터넷으로 나감

![](images/Pasted%20image%2020250216224134.png)
- 인터넷에서 들어올 경우, IGW으로 트래픽이 들어온 다음 Nat GW로 도달. Nat GW RT에 의해 목적지인 10.0.0.0/8은 vpce로 가라했으니, GWLBE에 도달
- GWLBE RT에 의해 tgw로 전달
- TGW RT에 10.2.0.0/16 목적지인 경우 tgw-att-vpc-2 로 전달하라고 되어있으니 해당 TGW ENI 타고 B 인스턴스에 도달
---

#### Centralized inspection with AWS GWLB: Important to know:

- GWLB Endpoint는 AWS PrivateLink를 사용하여 GWLB로 트래픽을 안전하게 전달한다. 추가 구성 없이 가능.
    
- **GENEVE 캡슐화**: GWLB는 원본 IP 트래픽을 GENEVE 헤더로 캡슐화하고, UDP 포트 6081을 통해 네트워크 어플라이언스로 전달한다.
    
- GWLB는 트래픽 flow를 유지하는 것이 중요하다
- **세션 고정성**: GWLB는 IP 패킷의 5-tuple 또는 3-tuple을 사용하여 특정 네트워크 어플라이언스에 트래픽을 고정시킨다. 이는 어플라이언스의 전반적인 flow에 고정적인 세션을 생성하여 방화벽 같이 stateful 해야하는 어플라이언스에 필수적이다.
    
- Transit Gateway의 appliance 모드와 결합하여, 출발과 목적지 AZ에 관계없이 세션 고정성을 제공한다.
    
- Refer to this blog for further details: https://aws.amazon.com/blogs/networking-and-content-delivery/centralized-inspection-architecture-with-aws-gateway-load-balancer-and-aws-transit-gateway/
---
## Centralized VPC interface endpoints

![](images/Pasted%20image%2020250216231047.png)
- **VPC 인터페이스 엔드포인트**는 AWS 서비스(예: KMS, SQS, EC2)에 대한 프라이빗 연결을 제공한다.
- VPC 내에서 **Private Link**를 사용하여 AWS 서비스와 통신 가능하다. 인터넷 통신, Nat  없이
- ENI를 서브넷에 프로비전하고 ENI를 통해 프라이빗하게 aws service endpoint에 도달 가능하다.

![](images/Pasted%20image%2020250216231056.png)
- Transit Gateway를 사용하면 여러 VPC에서 중앙 집중식으로 인터페이스 엔드포인트를 공유할 수 있다.
- 각 VPC는 Transit Gateway를 통해 중앙 VPC의 인터페이스 엔드포인트에 접근할 수 있다.
- ![300](images/Pasted%20image%2020250216231904.png)
  마치hub and spoke 구조와 같으며 각 a,b,c,d VPC는 spoke. central 한 vpc가 위치

### DNS resolution

![](images/Pasted%20image%2020250216232704.png)
- central vpc에서는 dns resolution이 정상 작동하지만, spoke vpc 에서는 동작하지 않는 문제가 있다.

sqs DNS 질의를 해보자면
- central vpc에서는 vpc endpoint 생성시 **private DNS**를 enable한다. 그럼 PHZ(private host zone)이 생성되고(api endpoint와 동일한 host로: sqs.ap-south-1.amazonaws.com) 해당 vpc와 연결된다.
  그럼 해당 vpc 내부의 ec2에서 sqs api를 질의한다면 성공적으로 질의될 것이다. 자동으로 sqs.ap-south-1.amazonaws.com에 vpc endpoint eni의 private ip가 등록되기 때문이다. 
- spoke vpc에서 만약 sqs api를 질의한다면, 인터넷으로 나가 질의하기에는 인터넷 통신구간이 없으며 그렇다고 sqs에 직접 연결되어있지 않기도 하다. 이때 결국 sqs.ap-south-1~ 도메인을 질의했을때, vpc endpoint private ip가 질의되어야 가능하다.

![](images/Pasted%20image%2020250216232717.png)
- 해결 방안
- private DNS 를 disable 한다.
- own **Private Hosted Zone (PHZ)**을 생성하고, interface endpoint의 DNS에 대한 Alias 레코드를 설정한다.
- PHZ를 모든 Spoke VPC와 연결하여 DNS resolution을 가능하게 한다.

#### Centralized VPC interface endpoints: Important to know:

- VPC interface endpoints는 regional 과 AZ 레벨의 DNS endpoint들을 제공한다.
![](images/Pasted%20image%2020250216235152.png)
- regional DNS endpoint는 모든 AZ의 endpoint에 대한 IP 주소를 제공한다.
![](images/Pasted%20image%2020250216235201.png)
- AZ를 구체화하여 질의했을때는 하나의 ip주소만이 나오는 것을 볼 수 있다.
- AZ 간의 data transfer 비용을 아끼기 위해서(spoke vpc에서 hub vpc로의), AZ specific한 DNS endpoints들을 사용할 수 있다. 

#### 비용

![](images/Pasted%20image%2020250216235535.png)
- 일반적으로 interface endpoint로만 구성했을때 비용이다

![](images/Pasted%20image%2020250216235601.png)
- tgw를 사용하여 중앙집중화를 했을경우 각 vpc와 tgw 연결마다 attachment 비용과 data transfer 비용이 든다.
- 위와 마찬가지로 interface endpoint 비용도 동일하게 들어간다.

![](images/Pasted%20image%2020250216235654.png)
- 비용적인면에서 peering를 사용하면 tgw를 사용할때의 비용은 들지 않을 것이다.
- vpc가 많으면 비용을 지불하더라도 tgw가 관리효율에 좋을 수 있기때문에 상황에 맞게 택해야한다.

---
## Transit Gateway vs VPC Peering


![](images/Pasted%20image%2020250216235834.png)
- AWS 전송 게이트웨이의 보안 그룹 참조 지원 기능 소개 https://aws.amazon.com/ko/blogs/tech/introducing-security-group-referencing-for-aws-transit-gateway/

- **DNS Resolution**: Transit Gateway에 연결된 모든 VPC에서 DNS Resolution 지원
- Supports resolving ‘public’ DNS names to Private IP’s for EC2
- TGW can be shared using Resource Access Manager (RAM) across AWS accounts
- Billed per hour, per attachment
- data processing 비용: amazon vpc or AWS Site-to-Site VPN 에서 TGW 까지 보낼때 각 GB당 비용 청구된다.
- **대역폭 제한**: VPN 터널당 1.25Gbps, ECMP를 사용하면 최대 50Gbps까지 지원
- TGW 5000 attachment 까지 지원
- **MTU 지원**: VPC, Direct Connect, Transit Gateway Connect, Peering Attachment 간 트래픽의 MTU는 8500바이트를 지원한다. VPN 연결의 MTU는 1500바이트.

---
## Transit Gateway Sharing


![](images/Pasted%20image%2020250217000746.png)

-  **Transit Gateway Sharing**: AWS Resource Access Manager (RAM)을 사용하여 TGW를 여러 AWS 계정 또는 조직과 공유할 수 있다.
- AWS Site-to-Site VPN attachment 는 TGW를 소유한 AWS 계정에서 생성해야 된다.
- **Direct Connect Gateway로의 Attachment**는 Direct Connect Gateway와 동일한 AWS 계정에서 생성할 수도 있고, 다른 계정에서 생성할 수도 있다.
- Transit Gateway가 다른 AWS 계정과 공유되면, 해당 계정은 Transit Gateway의 라우팅 테이블을 생성, 수정 또는 삭제할 수 없다.
	- 라우팅 테이블의 전파(propagation) 및 연관(association)도 관리할 수 없다.
	- Transit Gateway의 라우팅 설정을 소유 계정에서만 제어할 수 있음을 의미
    
- **Availability Zone ID 사용**: Transit Gateway와 Attachment 엔터티가 서로 다른 계정에 있는 경우, Availability Zone ID를 사용하여 AZ를 고유하고 일관되게 식별할 수 있다.
    - `us-east-1` 리전의 AZ ID인 `use1-az1`은 모든 AWS 계정에서 동일한 위치를 가리킨다.
    - 이를 통해 다른 계정 간에 AZ를 일관되게 식별하고 관리할 수 있다.
    - 즉 다른 계정 간의 attachment를 식별하기 위해 AZ ID를 사용할 수 있다.

