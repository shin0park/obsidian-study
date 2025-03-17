
### AWS Site-to-Site VPN architectures

- Virtual Private Gateway (VGW)를 사용한 구성
- Transit Gateway (TGW)를 사용한 구성
- 여러 지점 office와 연결되는 다중 Site-to-Site VPN 연결
- 고가용성(High Availability)을 위한 Redundant VPN 연결
-  [AWS Site-to-Site VPN user guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/s2s-vpn-user-guide.pdf)

![](images/Pasted%20image%2020250316202404.png)

![](images/Pasted%20image%2020250316202413.png)
- 다중 vpc와 연결을 하고자 한다면 TGW 유리
![](images/Pasted%20image%2020250316202423.png)
- 단일 vgw가 제공하는 총 bandwidth 대역폭은 1.25GPS 이다.
- 2개이상의 vpn 터널을 생성해도 이 limit을 초과할 수는 없다.
- 즉, 여러 VPN 연결을 설정해도 전체 VGW에 대한 총 대역폭 제한이 존재한다.

![](images/Pasted%20image%2020250316202436.png)
- TGW 의 VPC 간 트래픽의 대역폭 100GPS
- TGW에서 생성된 VPN 연결도 VGW와 동일한 **1.25Gbps**의 제한을 가짐 (개별 VPN)
- https://docs.aws.amazon.com/vpc/latest/tgw/transit-gateway-quotas.html
![](images/Pasted%20image%2020250316202447.png)
- HA를 위해 redundant vpn 구성
- vpn connection 이중
- 각 vpn connection당 터널 수 이중으로 총 4개의 터널 존재

---

### AWS Site-to-Site VPN Cloud Hub

VPN CloudHub – Routing between multiple customer sites
![400](images/Pasted%20image%2020250316204441.png)
- 여러 고객 site(지점) 간에 라우팅을 지원하는 VPN 솔루션

![400](images/Pasted%20image%2020250316204534.png)
- VPN 게이트웨이를 detached mode 로 사용 가능 - vpc에 연결하지 않음.
- 각 고객 게이트웨이는 고유한 BGP ASN을 사용해야 함 + BGP enabled
- Site 는 IP range를 겹쳐서는 안된다.
- 최대 10개의 고객 게이트웨이 연결 가능
- 온프렘 지점간의 failover(장애조치) 연결로도 사용할 수 있다.
	- 기존의 Direct Connection 또는 WAN이 장애 발생했을때 백업경로로 사용할 수 있음
	- BGP가 적용되어있으므로, 자동 경로 전환(failover) 가능

![400](images/Pasted%20image%2020250316204549.png)
- vpc도 도입해서 aws와의 연결할수도 있다.

---
### Amazon EC2 based VPN

#### VPN termination on EC2
own vpn 솔루션을 ec2에 배포하여 직접 관리
![300](images/Pasted%20image%2020250316205444.png)
- AWS 관리형 VPN이 아닌 **EC2 인스턴스를 VPN 종단점**으로 사용
- VPW를 vpc에 연결하는대신 EC2 인스턴스 중 하나에서 VPN 연결을 종료할 수 있다.
- EC2 인스턴스에는 VPN 소프트웨어 존재, 외부와의 연결도 되어있어야한다.
- Why would you do that?
    - IPSec 프로토콜 사용하기 싫을때
	    - IPSec 이외의 VPN 프로토콜(GRE, DMVPN 등) 필요
    - 중복 CIDR을 사용할 수 있음.
	    - 기본적으로 VGW, TGW를 통한 VPN 연결에서는 중복 CIDR 허용하지 않는다.
	    - VGW 및 TGW는 **기본적으로 라우팅 테이블을 기반으로 네트워크를 연결**하며, 동일한 CIDR이 두 개 이상 존재할 경우 **라우팅 충돌**이 발생한다.
	    - EC2 기반 VPN은 가능
		    - NAT 또는 정책 기반 라우팅(Policy-Based Routing)을 활용할 수 있기 때문에
			    - EC2 기반 VPN을 NAT 게이트웨이로 설정하여 **중복된 IP 주소를 변환하여 통신** 가능
			    - EC2 기반 VPN에서는 **라우팅 정책을 수동으로 정의**하여 **특정 트래픽이 특정 인터페이스 또는 VPN 터널을 통해 전달되도록 구성 가능**
				    - EC2 VPN을 통해 **VPN 터널 1 → 온프레미스 A, VPN 터널 2 → 온프레미스 B** 식으로 정책을 정의
					- 특정 트래픽이 특정 터널을 통해 전달되도록 설정하면 중복 CIDR 문제 해결 가능
    - AWS 내부에서 transitive routing 활성화 싶은 경우
	    - site-to-site vpn은 transitive routing 지원하지 않음
	    - EC2의 경우 EC2에서 VPN 트래픽이 종료되므로, 즉 VPC 안의 ENI에서 종료되는 것
		    - 따라서 vpc로 부터 IGW or NAT or Peering 으로 트래픽이 전달 될 수 있음
    - 고급 보안 기능(Advanced Threat Protection) 적용 필요
    - 1.25 Gbps 이상의 대역폭이어야한다.
	    - EC2 는 EC2의 최대 대역폭으로 제한이 되어있을 것

#### Considerations for EC2 based VPN
    
- VPN 종단 EC2 인스턴스에서 **source/destination check** 비활성화 필요
	- EC2 인스턴스는 NAT로 동작할수도 있음
- IP forwarding 활성화 os level
- EC2 하드웨어 장애 시 자동 복구(AWS CloudWatch status check)
- EC2 간 대역폭 제한 존재
- AWS에서는 타사 VPN 소프트웨어 제공하지 않음 (AWS Marketplace에서 Cisco, Juniper, Palo Alto 등의 솔루션 사용 가능)
- **IPSec은 TCP 또는 UDP가 아닌, 자체적인 프로토콜(ESP, Protocol 50)을 사용**하므로 NLB를 통해 정상적으로 동작할 수 없음
	- IPSec 기반 VPN을 EC2에서 운영할 때는, NLB를 사용할 수 없고, 대신 EC2 인스턴스에서 직접 VPN 터널 설정 필요하다
		- EC2 인스턴스를 VPN 종단점으로 사용하여 **IPSec을 직접 수락하도록**
	- 또는, UDP 기반의 NAT-Traversal(NAT-T) 사용
- **수직적 확장**: EC2 인스턴스 크기 증가
- **수평적 확장**: 여러 개의 EC2 인스턴스로 로드 분산

![](images/Pasted%20image%2020250316211702.png)

---

### VPN design pattern scenarios

1. **중간(moderate) 수준의 트래픽(최대 1Gbps) 요구**
    - AWS 관리형 VPN(VGW) 사용
    - 최대 1.25Gbps 대역폭 지원
2. **고대역폭(2Gbps 이상) 요구**
    - **AWS Direct Connect**가 최우선 선택
    - EC2 기반 VPN을 다중 인스턴스로 확장하여 고가용성 및 대역폭 확보
3. **IPSec 이외의 VPN 프로토콜 사용**
    - EC2 기반 VPN 사용
    - GRE, DMVPN 등의 프로토콜 지원 가능
4. **VPN 종단점에서 고급 보안 기능 필요**
    - EC2 기반 VPN 사용
    - EC2 내 보안 소프트웨어 설치 및 AWS Marketplace 솔루션 고려

---

### TransitVPC

![](images/Pasted%20image%2020250316212730.png)
- full mesh 구조

![](images/Pasted%20image%2020250316213046.png)
- vpn 소프트웨어를 포함하는 ec2를 구축하여 해결 가능 
- aws managed vpn을 사용하지만 ec2 에서 해당 vpn을 종료하는 구조
- 그다음 ec2에서 다시 vpn 연결을 함
	- 이 영역은 no mananged, 사용자 지정 vpn 솔루션이다.

#### Transit VPC의 장점
중앙집중식 네트워크를 갖고 관리할 수 있다. 
![](images/Pasted%20image%2020250316213111.png)

![](images/Pasted%20image%2020250316213121.png)
- 중복 CIDR 해결가능.

![](images/Pasted%20image%2020250316213131.png)
- transitive routing 가능
- eni 기반의 ec2 인스턴스에서 트래픽이 종료되었으므로 IGW, VPC endpoint로의 트래픽 전달이 가능하다

![500](images/Pasted%20image%2020250316213150.png)
- 다중 vpc간 다중 client 간의 연결 가능

---

**Transit VPC 시나리오**

1. 여러 VPC가 온프레미스 네트워크와 연결될 때
2. 중앙 집중식 방화벽 시스템을 통해 트래픽을 라우팅하는 고급 위협 방어
3. 겹치는 CIDR을 가진 온프레미스 네트워크와 연결 시 NAT 역할을 수행
4. Transit Hub VPC에 호스팅된 엔드포인트에 원격 네트워크(Spoke VPC 또는 온프레미스 네트워크)가 접근 가능하도록 설정
5. 클라이언트 장치가 Transit VPC의 EC2 인스턴스에 VPN 연결을 통해 접근하는 Client-to-Site VPN

**Transit VPC 아키텍처 (Transit Hub용)**
![](images/Pasted%20image%2020250316220112.png)
- Hub/Transit VPC: VPN 소프트웨어(PaloAlto, Avitarix, CheckPoint 등)를 실행하는 EC2 인스턴스, 고가용성을 위한 Multi-AZ 설정
- 각 Spoke VPC는 VPN 종료 지점으로 VGW를 사용
- 온프레미스가 Transit Hub와 VPN 연결 설정
	- Direct Connect도 사용 가능
- 이 아키텍처는 VPC 간, 온프레미스 VPN, AWS Direct Connect 간 Full Mesh 통신을 지원

---
#### **AWS Site-to-Site VPN 요약**

- VPN은 **Site-to-Site**와 **Client-to-Site**의 두 가지 카테고리로 구분
- AWS는 Site-to-Site VPN을 위한 완전 관리형 솔루션 제공, AWS 측에서 VPN은 VGW에서 종료
- AWS는 IPSec Site-to-Site VPN 지원
- AWS S2S VPN은 고가용성을 위해 2개의 터널 생성
- VPN 설정 완료 후 트래픽은 고객 측에서 시작해야 하며, VGW는 트래픽을 시작하지 않음 (2021년 2월까지 적용)
- VGW의 총 대역폭 제한은 1.25 Gbps
- AWS S2S VPN은 정적 라우팅과 BGP를 사용한 동적 라우팅 지원
- VGW로 100개 이상의 경로를 게시할 수 없음, CIDR을 통합하여 제한 해결 가능