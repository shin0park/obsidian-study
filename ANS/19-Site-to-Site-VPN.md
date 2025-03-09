
# Site-to-Site VPN

#### VPN 기본 개념
- **VPN이란**: VPN(가상 사설망)은 인터넷과 같은 신뢰할 수 없는 중간 네트워크를 통해 호스트가 암호화된 형태로 privately한 통신을 할 수 있게 함
- **AWS 지원**: AWS는 Layer 3 VPN을 지원하며, Layer 2 VPN은 지원하지 않음
- **VPN 형태**:
    - **Site-to-Site VPN**: 두 개의 서로 다른 네트워크를 연결
    - **Client-to-Site VPN**: 클라이언트 장치(예: 노트북)를 사설 네트워크에 연결
- **VPN 유형**:
    - **IPSec(IP Security) VPN**: AWS managed VPN에서 지원 - 두 네트워크를 point to point 로 연결
    - **기타 VPN**: GRE(라우팅 캡슐화), DMVPN(dynamic multi point vpn/hub and spoke vpn 과 비슷) 등은 AWS managed VPN에서 지원되지 않음


#### AWS에서 Site-to-Site VPN 작동 방식
![](images/Pasted%20image%2020250309190753.png)
- site-to-site vpn 의 경우 한 site는 aws가 될 것 (AWS managed vpn 이므로)
- **VPN 연결 종료**: AWS 측에서는 Virtual Private Gateway(VGW)에서 VPN 연결이 종료됨
- **고가용성**: VGW는 고가용성을 위해 서로 다른 가용 영역(AZ)에 두 개의 터널 엔드포인트를 생성


#### Virtual Private Gateway (VGW)

- VPC에 대한 Managed gateway endpoint
- **제한**: VPC당 하나의 VGW만 연결 가능
- **라우팅 지원**: 정적 라우팅과 BGP(Border Gateway Protocol)를 사용한 동적 라우팅 모두 지원
- **BGP 설정**: VGW에 64512~65534 범위의 private ASN(Autonomous System Number)을 지정 가능. 
  지정하지 않으면 default ASN(64512)이 할당되며, 한번 지정된 ASN은 수정 불가
- **보안**: AES-256 and SHA-2 암호화와 데이터 무결성(data integrity) 지원

#### Hands On

![](images/Pasted%20image%2020250309201934.png)

VPC-DC의 public 서브넷에 위치한 EC2에서 VPC-AWS 의 EC2-A 로 VPN을 생성하여 연결
TIP
- public 서브넷 ec2 생성시, auto public ip 자동할당 설정 (별도 설정하지 않고도 자동 할당됨)
- Download the configuration file 
	- Select VPN connection you created above -> Download configuration -> Select vendor as Openswan -> Download
	- VPN 설정하기 위한 내용들이 나와있음. 이를 따라 ec2에 접속 후 IPSec 설정하여 vpn 연결시도 


---
#### VPN NAT-Traversal (NAT-T) 지원

- NAT-Traversal (NAT-T)란
	- IPSec VPN 연결이 NAT(Network Address Translation) 장치(예: 라우터, 방화벽) 뒤에 있는 환경에서도 정상적으로 작동하도록 지원하는 기술이다.
	- NAT-T는 특히 Public IP 주소가 부족하거나 사설 네트워크에서 VPN을 설정해야 할 때 유용하다.
- AWS VPN은 고객 측 NAT 뒤에서의 VPN termination을 지원. 
- IPSec VPN은 두 가지 주요 프로토콜인 **AH**와 **ESP**를 사용하여 데이터를 보호한다. 이들은 NAT 환경에서 서로 다른 문제를 일으킬 수 있으며, NAT-T가 이를 해결하는 데 중요한 역할을 한다.
	- AH (Authentication Header)
		- 데이터의 무결성과 출처 인증을 보장. 데이터를 암호화하지는 않지만, IP 헤더와 페이로드를 포함한 전체 패킷에 대한 인증을 제공.
		- **NAT와의 문제**: AH는 IP 헤더의 소스 및 목적 IP 주소를 인증에 포함한다. NAT가 IP 주소를 변경(예: 사설 IP를 공인 IP로 변환)하면 AH의 해시 값이 달라져 인증이 실패한다. 
		  즉, AH는 기본적으로 NAT와 호환되지 않는다.
	- ESP (Encapsulating Security Payload)
		- 데이터 암호화, 무결성, 인증을 제공. ESP는 IP 헤더를 인증에 포함시키지 않으므로 NAT와 더 잘 호환된다.
		- **NAT와의 문제**: ESP 자체는 NAT와 비교적 호환되지만, ESP 패킷이 UDP나 TCP 같은 전송 계층 프로토콜에 캡슐화되지 않은 상태로 전송되면 NAT 장치가 이를 처리하지 못할 수 있다. 
			- ESP는 기본적으로 전송 계층 프로토콜(UDP/TCP)이 아닌 IP 프로토콜(프로토콜 번호 50)을 사용
			- NAT 장치는 포트 번호를 기반으로 트래픽을 매핑함
		- **NAT-T를 통한 해결**: NAT-T는 ESP 패킷을 **UDP 헤더**로 캡슐화하여 NAT 장치가 패킷을 올바르게 처리할 수 있도록 한다. 
		- 이렇게 캡슐화된 ESP 패킷은 NAT 환경을 통과하며 IP 주소 변환을 해도 정상적으로 작동한다.
- NAT-T가 활성화되면 IPSec VPN은 **UDP 포트 4500**을 사용하여 통신한다.
- 따라서, NAT-T를 사용하기 위해서는 고객 측 방화벽에서 UDP 포트 4500을 열어야 한다.
	- IPSec는 기본적으로 **UDP 포트 500**을 사용하여 IKE(Internet Key Exchange) 협상을 수행한다. 
	  그러나 NAT 환경에서 IKE 패킷과 ESP 패킷이 NAT 장치를 통과할 때, NAT가 이를 올바르게 매핑하지 못할 수 있다.
    - NAT-T는 이 문제를 해결하기 위해 IKE와 ESP 트래픽을 UDP 4500으로 이동시켜 캡슐화한다.
- 고객 측 방화벽에서 UDP 4500을 열지 않으면 NAT-T 트래픽이 차단되어 VPN 연결이 실패한다.
- **즉, AWS Site-to-Site VPN은 NAT-T를 지원하며, 고객 게이트웨이(Customer Gateway, CGW)가 NAT 뒤에 있을 때 방화벽에서 UDP 4500을 열어야 연결이 유지된다.**

![](images/Pasted%20image%2020250309201955.png)

---

## VPN Route Propagations
### VPN Static and Dynamic routing

- **정적 라우팅**: VPN 양쪽의 CIDR 범위를 미리 정의해야 하며, 네트워크 범위 추가 시 자동 전파되지 않음
- **동적 라우팅**: 양쪽에서 새로운 네트워크 변경 사항을 자동으로 학습. AWS 측에서 경로가 라우팅 테이블에 자동 전파됨
- **제한**: AWS 라우팅 테이블은 최대 100개의 전파된 경로만 허용. 초과 시 더 큰 CIDR 범위로 통합 필요.

![](images/Pasted%20image%2020250309202228.png)
- **정적 라우팅**:
    - 회사 데이터 센터에서 목적지 10.0.0.1/24를 CGW로 라우팅.
    - AWS에서 목적지 10.3.0.0/20을 VGW로 라우팅
- **동적 라우팅 (BGP)**:
    - BGP를 사용해 경로 자동 공유(eBGP for internet 사용)
    - 라우팅 테이블 수동 업데이트 불필요, CGW와 VGW의 ASN만 지정 

---
### VPN Transitive Routing

#### Site to Site VPN and Internet Access
![](images/Pasted%20image%2020250309202319.png)
- VGW은 결국, Gateway 디바이스로 AWS가 provision하는 ENI가 아니다.
- ENI는 VPC의 일부이므로 서브넷의 일부이다. 즉, 해당 서브넷에 도달하는 트래픽에 대한 것이라고 볼 수 있고 ENI는 서브넷의 라우팅 테이블 rule을 따르게 되어 IGW로의 라우팅이 가능할 수 있지만
- VGW는 ENI가 아닌 게이트웨이이므로 VGW에서 IGW로 트래픽을 직접 라우팅할 수가 없다.

![](images/Pasted%20image%2020250309202326.png)
- NAT 게이트 웨이도 결국 managed service이고 NAT의 어느 구성도 변경하지는 못하도록 제한되어있기 때문에 트래픽이 VGW로 들어와도 NAT를 통한 인터넷 접속이 불가능하다.

![](images/Pasted%20image%2020250309202344.png)
- own instance에 구축한 NAT라면 NAT 인스턴스에 대한 모든 컨트롤이 가능하고 게이트웨이 제약사항을 제거할 수 있다.
#### Site to Site VPN and VPC Peering
![](images/Pasted%20image%2020250309202406.png)
- vpc peering은 transitive routing을 지원하지 않음
#### Site to Site VPN and VPC gateway endpoint
![](images/Pasted%20image%2020250309202422.png)
- vpc gateway endpoint도 마찬가지로 VGW로 들어오는 트래픽은 VPC에서 발생한 트래픽이 아니기때문에 transitive될 수 없다.
- vpc gateway endpoint는 VPC에서 발생한 트래픽에 대해서 전달
#### Site to Site VPN and VPC Interface endpoint
![](images/Pasted%20image%2020250309202440.png)
- ENI를 생성해서 aws endpoint와 연결하는 vpc interface endpoint 는 가능하다.
#### Site to Site VPN and on-premises Internet Access
![](images/Pasted%20image%2020250309202455.png)


- **온프레미스에서 AWS로 (VGW 경유)**:
    - VPC의 IGW를 통해 인터넷 접근 불가
    - 퍼블릭 서브넷의 NAT GW를 통해 인터넷 접근 불가
    - VPC Peering 을 통해 피어링된 VPC 리소스 접근 불가
    - VPC Gateway endpoint를 통해 S3, DynamoDB 접근 불가
    - VPC Interface endpoint를 통해 AWS 서비스(예: API Gateway, SQS) 및 PrivateLink 기반 고객 서비스 접근 가능
    - 퍼블릭 서브넷의 NAT EC2 인스턴스를 통해 인터넷 접근 가능
- **AWS에서 온프레미스로 (CGW 경유)**:
    - CGW의 라우팅 규칙에 따라 인터넷 및 기타 네트워크 엔드포인트 접근 가능

---
### VPN Tunnels and Routing Active/Active and Active/Passive

aws vpn은 고가용성을 위해 두 개의 터널을 제공한다. 이 터널을 어떻게 설정할지는 사용자가 설정하기 나름이다.
#### 정적 라우팅 - Active/Active 터널
두 터널이 모두 active인 상태
![](images/Pasted%20image%2020250309210737.png)
- Active/Active 터널은 비대칭 라우팅(Asymmetric routing)을 유발할 수 있으며, CGW에서 비대칭 라우팅 활성화 필요
	- 터널 1로 보낸 트래픽이 반환시 터널 2로 올 수 있음.
	- stateful한 방화벽이 CGW에 있은 경우, 요청의 상태가 아니기 때문에 이를 거부하게 된다.
- AWS로의 시작된 트래픽은 무작위로 터널 선택

#### 정적 라우팅 - Active/Passive 터널

![](images/Pasted%20image%2020250309210715.png)
- 하나의 터널만 UP 상태일 때 양방향 트래픽에 사용
- **장애 시**: 터널 다운 시 CGW에서 다른 터널 활성화 필요. 고가용성을 보자아지 않기 때문에 관리가 필요하다.

#### 동적 라우팅 - Active/Active 터널

![](images/Pasted%20image%2020250309210725.png)
- **터널 우선순위 설정**:
    - Advertise a more specific prefix on the tunnel
	    - **BGP의 경로 선택 알고리즘**
			- BGP는 동일한 목적지로 가는 여러 경로 중에서 "최적 경로(Best Path)"를 선택한다. 이 과정에서 가장 먼저 확인하는 요소는 Prefix Length이다.
			- 예를 들어, 10.0.0.0/16과 10.0.1.0/24가 동일한 목적지를 가리킬 때, BGP는 더 구체적인 프리픽스인 /24(더 긴 서브넷 마스크)를 선택한다. 이는 더 좁은 범위를 나타내며, BGP가 "더 정확한 경로"로 인식하기 때문이다.
			- 더 구체적인 프리픽스를 특정 터널에서 광고하면, BGP는 그 터널을 더 "정확한 경로"로 판단하고 해당 터널로 트래픽을 우선적으로 보낸다.
    - BGP 사용 시 ASPATH 조정(짧은 ASPATH 우선)
    - 동일 ASPATH일 경우 낮은 MED(Multi-Exit Discriminator) 값 우선

#### Summary

- **정적 라우팅 Active-Active**
    - AWS에서 트래픽 시작 시 터널 무작위 선택
    - CGW에서 비대칭 라우팅 활성화 필요
- **정적 라우팅 Active-Passive**
    - UP 상태인 터널이 양방향 트래픽에 사용
- **동적 라우팅 (BGP) Active-Active**
    - 선호 터널 설정 방법: 구체적 프리픽스, 짧은 ASPATH, 낮은 MED 값

---

### VPN Dead Peer Detection (DPD)

#### Dead Peer Detection (DPD)

- IPSec VPN 연결의 활성 상태를 감지하는 방법
- VPN peer은 "IKE Phase I"에서 DPD 사용 여부 결정한다. 
- DPD 활성화 시 AWS는 10초마다 DPD 메시지(R-U-THERE)를 CGW로 전송, 3회 연속 응답 없으면 타임아웃
![](images/Pasted%20image%2020250309210652.png)
- **기본 동작**: 
	- DPD 발생 시, gateway는 보안 연결(Security Associations, SA) 삭제한다. 
		- DPD가 발생하면, 게이트웨이(AWS 쪽 또는 고객 쪽)는 "이 터널은 더 이상 믿을 수 없어!"라고 생각하고 기존 보안 연결을 삭제한다.
	- 가능하면 대체 IPSec 터널 사용
		- 주 터널에서 문제가 생겨(DPD 발생) 보안 연결이 삭제되면, 가능하면 대체 터널로 전환
		- 물론 대체 터널이 제대로 작동 중이어야 가능
- **타임아웃**: 기본 30초, 30초 이상으로 설정 가능
- **포트**: UDP 500 또는 4500 사용
- **타임아웃 액션**:
    - **Clear**: IPSec IKE 세션 종료, 터널 중지, 경로 삭제 (기본값)
    - **Restart**: AWS가 IKE Phase I 재시작
    - **None**: 조치 없음
- **요구사항**: 동적 라우팅(BGP) 사용 시 고객 라우터는 DPD를 지원해야 한다
	- 즉, AWS와 고객 네트워크 간에 BGP로 경로를 자동으로 관리하려면, 고객 측 라우터가 DPD 기능을 제대로 처리할 수 있어야 한다
	- 만약 VPN 터널이 끊기면 BGP 세션도 끊어지고, 경로 정보 업데이트가 중단된다. 이 경우 네트워크 트래픽이 제대로 흐르지 않을 수 있다.
	- 따라서, DPD는 터널이 끊겼는지 빠르게 감지해서 BGP 세션이 유지되도록 돕거나, 대체 터널로 전환하게 해줘야한다.
	- AWS는 DPD를 지원하는만큼 고객 라우터도 AWS가 죽었는지 확인하려면 DPD를 보내고 응답받아야하므로, 양쪽이 지원해야 원활한 통신이 가능하다.
	- 동적 라우팅은 네트워크의 고가용성(HA)을 목표로 하며, DPD는 이 고가용성을 실현하는 데 핵심적인 역할을 한다.
- **연결 유지**: 유휴 연결 종료 방지를 위해 ICMP(ping) 요청(5~10초 간격) 설정 가능


---

### VPN Monitoring

#### CloudWatch를 통한 VPN 모니터링
![](images/Pasted%20image%2020250309210638.png)
- **TunnelState**: 터널 상태. 0은 DOWN, 1은 UP. 0~1 사이 값은 최소 1개 터널이 DOWN 상태임을 의미.
- **TunnelDataIn**: VPN 터널로 수신된 바이트.
- **TunnelDataOut**: VPN 터널로 전송된 바이트.
- **구성 요소**: AWS Site-to-Site VPN 터널, Amazon CloudWatch, 알람, Amazon SNS(Simple Notification Service)

#### VPN Monitoring with Health Dashboard

- AWS Site-to-Site VPN은 자동으로 AWS Personal Health Dashboard(PHD)에 알림을 보낸다.
- Tunnel endpoint replacement notification  
- 단일 터널 VPN 알림 (하루 중 한 개 터널이 1시간 이상 다운된 경우)