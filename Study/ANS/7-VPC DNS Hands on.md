![500](Pasted%20image%2020241124173342.png)
- name server는 AmazonProvidedDNS 
- DHCP option set의 name server를 수정할 필요 없음.
- (optional) DHCP option set의 Domain name을 `corp.internal` 로 설정하여, vpc와 연결
- use Route53 Private Hosted Zone - 호스트 존 생성 후 레코드 생성

만약 DHCP option set의 domain name을 별도 설정하지 않은 경우,
예를 들어 ec2 내부에서 `ping db.corp.internal` 동작하지 않음.
-> Private Hosted Zone은 VPC 내에서만 동작하도록 설계되어 있지만, 
EC2 인스턴스는 `/etc/resolv.conf`에 설정된 DNS `search domain`에 의존하기 때문에, 
이를 올바르게 설정하지 않으면 Hosted Zone의 레코드를 해석할 수 없다. 
따라서 **EC2와 같은 일반적인 컴퓨팅 리소스에서는 DHCP 옵션 세트를 통해 도메인 이름(`domain-name`)을 설정해줘야 질의가 가능하다.**


![500](Pasted%20image%2020241124172845.png)
![500](Pasted%20image%2020241124175734.png)
- 아마존 DNS 서버를 사용하지 않고 custom dns server 사용
- DHCP option set 수정 필요 (name server를 custom dns server 주소로 지정해야함)
- app, db, dns 용 ec2 3대 구축 (dns용 ec2의 경우, dns package 다운)
![400](Pasted%20image%2020241124180524.png)
![400](Pasted%20image%2020241124180547.png)
Restart named service 

- Create new DHCP Option set
- Reboot DNS server and App server (to update DHCP)

### Route 53 Resolver Endpoints
![600](Pasted%20image%2020241124180818.png)
- 아마존에서 온프렘의 dns server 질의
- 사내 pc에서 aws dns 질의
- 양방향, 일부는 aws, 일부는 온프렘
그럴 경우 두 네트워크가 VPN이나 DX를 통해 연결되어 있다고 가정

하지만 온프레미스 네트워크와 AWS VPC 간에 **VPN** 또는 **Direct Connect(DX)**를 통해 네트워크 연결이 이루어진 경우에도, 리졸버는 온프레미스 DNS 서버의 요청을 처리할 수 없다. 즉, 자동으로 온프램에서 aws route53 dns resolver에 도달할 수 없다.
- 이유는 `vpc + 2` 리졸버가 **VPC 내부 IP 범위**에서만 접근 가능하도록 제한되기 때문
- 따라서 route53 resolver Endpoint를 제공
- 온프레미스에서 AWS 리졸버를 사용하려면 **Route 53 Resolver Inbound Endpoint**를 설정해, 온프레미스 DNS 쿼리를 처리할 수 있도록 해야한다.

### Route53 Resolver Endpoint
- Officially named the .2 DNS resolver to Route 53 Resolver
- Endpoint는 **Elastic Network Interface (ENI)** 형태로 VPC에 생성되며, VPN 또는 DX를 통해 연결된 온프레미스 네트워크에서 접근 가능
- Provides Inbound & Outbound Route53 resolver endpoints
	- Inbound -> On-premise forwards DNS request to R53 Resolver 
	- Outbound -> Conditional forwarders for AWS to On-premise

[Route 53 Resolver란 무엇인가요?](https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/resolver.html)


![600](Pasted%20image%2020241124180856.png)
- HA를 위해 다중 서브넷에 endpoint 생성

![600](Pasted%20image%2020241124180907.png)
- conditional forwarding 설정하여 해당하는 질의는 Outbound Endpoint(ENI)를 통해 온프레미스 DNS 서버로 요청을 보냄


### VPC DNS & DHCP exam essentials

- VPC에는 기본 DNS 서버로 **AmazonProvidedDNS**가 제공
- AWS 제공 DNS 서버 IP
	- 이 DNS 서버는 **VPC의 기본 CIDR 블록 + 2**의 IP 주소에서 동작
	- 또는 VPC 내에서 **169.254.169.253**의 가상 IP를 사용하여 DNS 서버에 질의
- AmazonProvidedDNS는 아래의 순서대로 DNS 질의
	- AWS Route53 Private Hosted Zone
	- VPC internal DNS
	- Public DNS 
	  -> *tip. R53에 레코드가 없으면 public dns로 가는것이 아니라, 해당 되는 hosted zone이 없을때 public dns 로 나가는 것.*
	 즉, **Route 53 Private Hosted Zone에 Hosted Zone은 있지만 레코드가 없는 경우**, 
	 DNS 질의는 퍼블릭 DNS로 넘어가지 않고, 질의에 대해 응답이 없다는 상태를 반환.
	 이는 Private Hosted Zone이 존재할 때 해당 호스트 이름을 처리할 수 있는 권한을 가지며, Hosted Zone이 해당 도메인 이름에 대해 권한(authoritative)으로 간주되기 때문
- VPC의 DNS 설정 변경
	- VPC의 DNS 설정은 **DHCP 옵션 세트**를 사용하여 변경 가능
- DHCP 옵션 세트 특징
	- DHCP Option set can not be edited.
	- Create new one and associate it with VPC.
	- only one DHCP option set associated with a VPC at a time.
- For hostname resolution, we should enable both enableDnsSupport and enableDnsHostname
	- Route 53 Private Hosted Zone 사용
	- EC2 인스턴스가 퍼블릭 IP를 가지며, 퍼블릭 DNS 이름(`ec2-xx-xx-xx-xx.compute.amazonaws.com`)으로 접근 가능해야 할 때 두 설정이 필요
- 온프레미스와 VPC 간 Hybrid DNS Resolution
	- use Route53 Resolver endpoints.

