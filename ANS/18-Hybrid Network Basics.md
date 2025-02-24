하이브리드 네트워크 관련 aws 서비스
- AWS Site-to-SiteVPN
- AWS DX 
![](images/Pasted%20image%2020250223210354.png)
- 하이브리드 측면을 살펴보자
- 온프렘 연결을 어떻게 할지
-  AWS Site-to-SiteVPN
	- 인터넷 통신을 하지만 IPsec 프로토콜로 안전하다
	- private ip를 통해 연결
- Client VPN endpoint: https://docs.aws.amazon.com/ko_kr/vpn/latest/clientvpn-admin/cvpn-working-endpoints.html
	- 집에서 재택시 원격 접근할때 
	- 트래픽은 인터넷을 통함. 보안적으로 안전하지만 대역폭에 제한이 있을 수 있음
- AWS DX 
	- 데이터센터와 aws 사이의 일관된 네트워크가 필요할 때 사용
	- 물리적인 전용선을 연결하기 때문에 안정적
---
### Routing – Static vs Dynamic

![](images/Pasted%20image%2020250223210944.png)
- vpn은 서로 다른 네트워크에 연결하며, 이 네트워크들은 고유한 식별자를 갖고 있다.
- 그 식별자를 Autonomous System 자울 시스템, AS라고 한다
	- 즉, AS는 단일 관리 도메인 하에 있는 IP 네트워크와 라우터들의 집합을 의미
	- 인터넷은 수많은 AS들이 서로 연결되어 구성된 거대한 네트워크이다.
	-  각 AS는 고유한 라우팅 정책을 가지고 있으며, 내부 네트워크를 관리하고 외부 AS와의 통신을 담당한다.
- unique 식별자는 ID 사용, ID는 ASN으로 autonomus system number 라고 한다.
	- 즉, 각 AS는 전 세계적으로 고유한 번호인 AS 번호(Autonomous System Number, ASN)를 가지고 있다.
- ASN은 public or private 으로 구분된다.
- public ASN - 인터넷 상에서 고유하게 식별되는 AS 번호
	- 이 ASN은 전 세계적으로 유일해야 하며, 인터넷을 통해 다른 AS와 라우팅 정보를 교환할 때 사용
	- public이면 IANA(Internet Assigned Numbers Authority)가 할당
		- IANA가 특정 공유기(라우터)에 Public ASN을 할당하면, 해당 라우터는 인터넷을 통해 다른 AS와 경로 정보를 교환한다.
		- 이를 통해 인터넷 상에서의 라우팅이 가능해진다.
- private ASN - Private ASN은 인터넷 상에서 사용되지 않고, 내부 네트워크에서만 사용되는 AS 번호
	- Public ASN과 달리 전 세계적으로 고유할 필요가 없으며, 특정 범위 내에서만 사용된다.

	
![](images/Pasted%20image%2020250223211013.png)
- static하게 라우터를 추가하는 것.
- 만약 C라는 네트워크 대역이 추가됐을때 라우터 A는 직접 추가하지 않는 한 C로 가는 라우터가 없기 때문에 C로 트래픽이 도달 할 수 없다.  
  -> static routing의 문제

![](images/Pasted%20image%2020250223211024.png)
- 라우터 B가 라우터 C에 대한 경로를 학습하면 자동으로 A에게 경로를 전파한다
- 따라서 C의 경로가 라우터 A에 자동 추가되어 트래픽 도달이 가능하다

---
## Dynamic Routing using  Border Gateway Protocol (BGP)

- BGP는 인터넷에서 AS(Autonomous System, 자율 시스템) 간의 라우팅을 관리하는 데 사용되는 프로토콜이다.
- Path-Vector protocol을 사용하여 동적 라우팅을 수행하며, 피어(peer) 또는 AS(자율 시스템) 간에 목적지까지의 최적 경로를 교환한다
- 인터넷 상에서 효율적인 라우팅이 가능해진다.

- iBGP: AS 내부에서의 라우팅
- eBGP: AS 간의 라우팅
    
- 라우팅 결정에 영향을 미치는 요소:  
	- Weight (Cisco 라우터 전용 - AS 내부에서 작동) 
		- 높은 Weight 값을 가진 경로가 우선적으로 선택된다.
		- 라우터마다 독립적으로 설정되며, 다른 외부 BGP 라우터와 공유하지 않는다.
	- AS_PATH: 목적지에 도착하기 위해 통과해야하는 홉의 횟수(AS 간에 작동)
		- BGP는 목적지까지의 경로를 AS_PATH 형태로 전달한다
		- AS_PATH가 짧을수록 우선적으로 선택
		- AS_PATH는 패킷이 통과해야 하는 AS의 목록을 포함한다.
		- **루프 방지:** AS_PATH를 통해 동일한 AS를 두 번 이상 통과하는 루프를 방지한다.
	- Local Preference (LOCAL_PREF): AS 내부에서 작동  
		- AS 내부에서 경로의 우선순위를 결정하는 속성. - 즉, outbound external BGP path를 결정한다.
		- 다른 외부 BGP 라우터와 공유하지 않는다.
		- Default value 100
		- 높은 LOCAL_PREF 값을 가진 경로가 우선적으로 선택
	- MED (Multi-Exit Discriminator): AS 간에 작동
		- AS 간에 여러 개의 연결점(Exit Point)이 있을 때, 어떤 연결점을 사용할지 결정하는 속성. 
		- AS 간에 MED를 교환한다.
		- 낮은 MED 값을 가진 경로가 우선적으로 선택
--- 
### How BGP works

- 각 AS의 라우트는 기본적으로 자기 자신에 대한 라우트를 갖고 있다. 
	- Next Hop 0.0.0.0 으로
- BGP 를 통해서 각 AS로 가는 경로들을 전파 받을 수 있다.

![](images/Pasted%20image%2020250224211435.png)
- AS200에 도달하는 경로를 생각해봤을때, 두 가지가 존재한다
	- 200,i / 400,200,i 
	- BGP 이론상 최적의 경로에 해당되는 라우트를 사용하기 때문에, (200,i) 로 가는 것이 우선순위가 높을 것이다.
	- 하지만 위 그림상 400,200,i 경로가 `10Mbps*2` bandwitdh 으로 더 높다
	- 해당 경로로 가기 위해서 **ASPATH** 를 사용할 수 있다.
- (200,i) 경로에 dummy hub를 추가할 수 있다 -> 200,200,200,i
- 그러면 최적의 경로가 400,200,i 로 선정되고 해당 경로록 라우팅할 수 있게 된다.


![](images/Pasted%20image%2020250224211504.png)
- AS 300에 도달하기 위한 경로가 위와 같이 두가지 있을때 LOCAL_PREF 가 영향을 미칠 수 있다.
- AS 내부적으로도 지역적으로 더 해당 목적지까지 도달하기에 유리한 경우가 존재할 것이다. 이에 따라 내부적으로도 우선순위를 선정하게된다.
- LOCAL_PREF=200, LOCAL_PREF=100 중에 LOCAL_PREF=200가 더 선호도 높기 때문에 400,300,i 경로가 채택되게 된다.
- 즉, 지역적 선호도가 AS의 경로에 영향을 주게된다.

![](images/Pasted%20image%2020250224211541.png)
- LOCAL_PREF이 갖고, AS_PATH가 같다고 했을때 위와 같이 두 가지 경로 중 선택해야 될 수 있다.
- MED: 목적지에 여러 연결점이 존재할텐데, 이때 어느 연결점을 사용할지 결정한다.