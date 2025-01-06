### Extending VPC address space

Secondary IP 설정
1. VPC에 너무 작은 CIDR를 설정하여 부족한 경우 - Secondary CIDR 설정 가능
2. 다른 VPC와 연결시, CIDR 범위가 현재 CIDR와 겹치는 경우 - 새로운 CIDR 대역인 Secondary CIDR 설정 가능

#### Secondary CIDR
- 기존의 VPC에 Secondary로 CIDR를 추가할 수 있다.
- 기존 CIDR 대역과 겹치는 대역을 지정할 수 없다.
- primary CIDR가 RFC1918 범위의 CIDR 블록이라면, Secondary CIDR는 다른 RFC1918 범위에서 올 수 없다.
  (RFC1918 범위: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`)
  https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/vpc-cidr-blocks.html#add-cidr-block-restrictions
  
- CIDR 블록은 모든 VPC 라우팅 테이블에서 경로의 대상 CIDR 범위보다 작아야 한다.
  예를 들어, 기본 CIDR 블록이 `10.2.0.0/16`인 VPC에서 가상 프라이빗 게이트웨이에 대한 대상이 `10.0.0.0/24`인 라우팅 테이블에 기존 라우팅이 있고, `10.0.0.0/16` 범위의 보조 CIDR 블록을 연결하려고 한다. 
  기존 경로 때문에 `10.0.0.0/24` 이상의 CIDR 블록을 연결할 수 없다. 그러나 `10.0.0.0/25` 이하의 보조 CIDR 블록은 연결할 수 있다.
- VPC에 총 5개의 IPv4와 1개의 IPv6 CIDR 블록을 가질 수 있다.

![400](images/Pasted%20image%2020241110190003.png)

![600](images/Pasted%20image%2020241110185744.png)

#### ENI (Elastic Network Interface)
EC2 인스턴스에 IP 주소가 할당하는것은
-> ENI를 통해 IP 주소를 할당받는 것
마치 네트워크 카드를 여러장 꽂는 것처럼
EC2 인스턴스에도 ENI를 여러개 지정할 수 있으며, ENI 하나당 IP 하나라고 보면된다.
결국 ENI가 IP를 가지기 때문에 EC2인스턴스도 IP를 할당받을 수 있는 것,

- VPC 내의 *Virtual Network Card*를 의미하는 논리 컴포넌트이다.
- ENI는 AZ 범위로 동작한다.

ENI의 구성요소
- VPC 내 Primary IPv4 주소
- VPC 내 Primary IPv6 주소
- VPC 내 하나 이상의 Secondary IPv4 주소
- Private IPv4 당 하나의 Elastic IP 주소
- 하나의 Public IPv4
- 하나 이상의 IPv6 주소
- 하나 이상의 Security Group
- 하나의 MAC 주소
    - 참고, Mac 주소에 소프트웨어 라이센스가 연결되어있으므로, 새 인스턴스에 기존의 ENI를 붙이면 같은 라이센스를 이어갈 수 있음.
- 하나의 Source/Destination Check Flag (ec2의 Source/Destination Check 기능이 ENI로 인해)
- EC2 인스턴스에 ENI가 얼만큼 붙을 수 있는지는 -> EC2 인스턴스 타입에 따라 다르다.

![400](images/Pasted%20image%2020241110190935.png)

#### ENI Use Case
- Requester managed ENIs - AWS가 다른 AWS서비스를 대신에 VPC에 ENI를 생성 (ex. RDS DB instance, Nat gateway)
- Management Network/Dual Home 인스턴스 생성
- 인스턴스 장애 시, EC2 IP 유지 - HA
- Using ENI secondary IPs for EKS Pods

#### Requester managed ENIs
- RDS DB instance는 full managed 서비스이지만 SG를 통해 DB 인스턴스로 트래픽을 통제할 수 있다.
  AWS가 RDS서비스를 대신해 ENI를 우리(고객)의 VPC에 생성하기 때문에 가능한 것
- EKS(Control Plane) Master node 가 aws mananged vpc에서 launch되지만 ENI는 고객의 vpc에 생성되기때문에 eks worker node와 통신할 수 있는 것
- Workspaces or Appstream2.0- 관리 컴퓨팅 인스턴스, VDI - Workspaces or Appstream2.0는 AWS managed VPC에 위치하고, 고객은 Customer managed VPC에 ENI 를 생성해서 연결하기 때문에 vpc 안의 애플리케이션과 통신 할 수 있는 것
- 람다 - 람다 함수는 Lambda Service VPC 내에 생성이 되지만,  ENI는 고객 VPC에 생성이 되어 접근 가능 한 것.

![500](images/Pasted%20image%2020241110195739.png)

![500](images/Pasted%20image%2020241110195745.png)
 
#### Management Network/Dual Home 인스턴스 생성
![600](images/Pasted%20image%2020241110195752.png)
하나는 인터넷 게이트웨이에 연결하여 인터넷 접근 가능
다른 하나는 프라이빗 서브넷 내의 ENI를 통해 온프렘 네트워크에 연결하여 사내망에서만 온프램 사용가능
Dual home instance 로 사용가능하다.

![600](images/Pasted%20image%2020241110195801.png)
- Primary 인스턴스가 fail되면 hot-standby 인스턴스가 ENI 에 연결
- Routing, DNS 설정에 변화 없이 가능
- 순단 연결 손실 (Brief loss of connectivity) 발생 가능

### Exam essentials
1. 인스턴스에 연결할 수 있는 ENI 수, 각 ENI에 설정할 수 있는 Secondary IP 수는 EC2 인스턴스 유형에 따라 다르다.
2. 인스턴스의 Primary 네트워크 인터페이스는 detach 할수 없다. Secondary ENI는 detach 가능하며, 동일 AZ 내 다른 인스턴스에 연결할 수 있다.
3. **보안 그룹**은 네트워크 인터페이스에 연결한다. 개별 IP 주소가 아닌
4. MAC 주소에 bound된 애플리케이션 라이선스의 경우 동일한 ENI를 사용하면 MAC 주소를 유지할 수 있다.
5. Secondary ENI - 인스턴스가 동일 AZ 내에서 여러 서브넷에 연결될 수 있으며, across aws account로 생성 가능 (requester managed ENI)
6. ENI는 네트워크 인터페이스 카드(NIC) 팀에서 사용할 수 없음 - 인스턴스 네트워크 bandwidth를 증가하는데 사용할 수 없음.
7. 동일 서브넷에서 인스턴스에 두 개 이상의 네트워크 인터페이스를연결하면 비대칭 라우팅(Asymmetric Routing)과 같은 네트워크 문제가 발생할 수 있다.
    - 가능하면 primary 네트워크 인터페이스 대신에, secondary Private IPv4 주소를 사용하는 것이 좋다.

#### BYOIP (Bring your own IP)
- 자신의 IPv4 이나 IPv6을 AWS에 할당
- why?
	- Keep your IP address reputation
	- IP 화이트 리스트 변경을 피하기 위해
	- IP 주소 변경 없이 애플리케이션을 이동하기 위해
	- AWS as a hot standby

#### Pre-requisites to BYOIP
- 주소 범위는 반드시 지역 내 internet registry RIR 에 등록되어야한다. - ARIN or RIPE or APNIC
- IP 주소 범위의 주소들은 반드시 history clean 해야하며, AWS는 평판이 좋지 않은 ip는 거부한다.
- IPv4 주소 범위는 가장 작은 범위 - `/24` 까지를 지정가능
- IPv6 주소 범위는 가장 작은 범위 - `/48` 까지를 지정가능 - Publicly Advertised CIDR 인 경우
- IPv6 주소 범위는 가장 작은 범위 - `/56` 까지를 지정가능 - Not Publicly Advertised CIDR 인 경우, 하지만 필요 시 Direct Connect 으로 전파 가능
- ROA (Route Origin Authorization)을 생성 - Amazon ASN 16509 와 14618 을 허가해야한다. 우리의 주소 범위에 유효하도록

#### Good to know about BYOIP
- BYOIP 으로 인해 IP 소유권이 AWS 으로 변경되는 건 아니다
- 당신의 주소 범위를 가져오면, AWS 에서는 그 Address Pool 인 것처럼 사용.
- EC2 Instances, Network Load Balancers, NAT GW 로 IP 주소 연결 가능.
- AWS 계정 내 각 리전마다 총 5개의 IPv4 와 IPv6 주소 범위 사용가능.