## VPC Building blocks - core components
![600](images/Pasted%20image%2020241013155837.png)
#### VPC
지역 선택 
-> 사용할 AZ들 선택 
-> IPv4 or IPv6 주소범위 지정 (CIDR)
#### Subnet
-> Subnet 생성 (VPC 주소 범위를 subnet으로 더 작게 네트워크를 나누는 것) - AZ 지정 필요 - 주소 대역 지정(VPC 범위에 포함되며 더 작은 주소범위)
-> subnet 레벨에서 route table을 가질 수 있으며, subnet의 용도에 따라 이를 컨트롤 할 수 있다.
-> route table - vpc와 subnet을 위한 라우팅 구성을 정의

#### 인터넷
-> 인터넷 통신 필요한 경우 - Internet Gateway
-> 인터넷 통신이 필요한 subnet의 route table에 igw로 향하는 route 설정 필요

#### 방화벽
AWS는 기본적으로 두 가지 방화벽 설치
- Security Group
	- *Resource level 예를 들어, EC2 instance level 작동*
	- EC2로 드나드는 모든 트래픽을 security group rule 로 제어
- Network ACL
	- 네트워크 엑세스 제어 목적, *Subnet level 작동*
	- 해당 rule이 지정된 subnet 내에서 시작하는 모든 리소스에 대해 적용되는 것

#### DNS
- route53

## VPC Addressing (CIDR)
![500](images/Pasted%20image%2020241013160827.png)

![500](images/Pasted%20image%2020241013161020.png)
이전에는 Class A, B, C class로 나누어 사용했지만 이는 -> *CIDR로 대체됨*

192.168.0.0/16
16의 의미?
16비트의 Network Address는 Fixed 되고 나머지 (32-16)비트는 해당 네트워크 내의 Host Address를 할당


![500](images/Pasted%20image%2020241013161037.png)

+ subnetting 
192.168.0.0/16 -> 2^16  = 65536 개의 ip 주소 가능 

192.168.0.0/24
192.168.0.0/28 
등등 VPC 보다 더작은 범위로 subnetting 가능

![500](images/Pasted%20image%2020241013161059.png)


## VPC Route Table

![|500](images/Pasted%20image%2020241013162913.png)
- subnet 내부에 ec2인스터스를 생성하면 subnet ip 범위 내에서 ip를 하나 지정된다.
- Can these instances communicate with each other? yes
- 로컬 라우터를 통해 같은 VPC 내라면 통신이 가능하다.
- root route table에는 default로 vpc 대역이 destination으로 Local이 target인 route가 지정되어있다.
- A에서 B로 통신하고자 할때 root route table을 보고 local router를 통해 instance B 로 통신한다.
- => *VPC 내의 모든 서브넷 사이에서는 항상 네트워크 연결이 되어있다.* (단, 방 화벽 레벨에서 막지 않았다는 조건 하에)

![500](images/Pasted%20image%2020241013163326.png)
- igw를 연결하지 않는한 기본적으로 vpc는 폐쇄망이다.
- *IGW를 생성했다면 인터넷 통신이 가능한가? NO*
- 기본적으로 instance A에서 인터넷 통신을 원한다면, 일단 private ip(사설IP)만이 아닌 public ip(공용IP)가 필요할 것이다. 오직 private ip만 존재한다면 인터넷 통신을 할 수 없다.
  *공용IP가 존재한다고 가정하면 인터넷 통신이 가능한가? NO*
- main route table을 봤을때 결국 IGW로 가는 route가 존재해야 인터넷 통신이 가능하다. 경로가 있어야 트래픽을 전송할 수 있기 때문에. 
  -> *0.0.0.0/0 대상으로 iGW로 가는 경로를 추가하고 나서야 인터넷 통신이 가능해질 것 이다.*

#### 하나의 인스턴스만 인터넷 통신이 가능하고, 다른 인스턴스는 불가능하게 하는 경우
![500](images/Pasted%20image%2020241013163338.png)
- 위 처럼 root table에 igw로 가는 경로를 추가하여 인터넷 통신을 가능하게 한다면, 두 인스턴스 모두 인터넷 통신이 가능해진다. *서브넷은 기본값으로 main route table을 따르기 때문이다.*
- *서브넷 별로 트래픽을 다르게 통제하고 싶다면, public subnet  or private subnet*
- main route table(기본 라우팅 테이블)을 follow하게 하는거 대신 custom route table(사용자 지정 라우팅 테이블)을 생성하여 사용할 수 있다.
- custome route table을 생성하여 subnetA에 추가하는 순간. 
  *해당 서브넷은 main route table을 follow하는 것을 중단한다.*
  subnet B만 main route table을 follow 하게 되는 것.
- 각각 custom route table을 생성하여 서브넷에 add 한다면 서브넷 별로 route table을 갖게되는 것이고, 각각 트래픽을 통제할 수 있는 것
- *custome route table 생성도, by default, vpc 대역에 대해 Local로 가는 경로가 자동 추가된다.* 해당 route를 제외하고는 route를 추가하고 제거 할 수 있다.

![500](images/Pasted%20image%2020241013165431.png)

### Subnet
![500](images/Pasted%20image%2020241013165547.png)

![500](images/Pasted%20image%2020241013165614.png)

![600](images/Pasted%20image%2020241013165841.png)
0 - Network address
1 - inter vpc for communication
2- 국가안보부
3- future use
255- network broadcast 용이지만 aws는 broadcast를 원하지 않기때문에 사용하지 않는다.

## IP Addresses - Private vs Public vs Elastic & IPv4 vs IPv6

- 인터넷 통신을 위해서는 Public IP(공용IP)가 필요
- AWS 공개 IP 풀에서 제공하기때문에 Public IP를 우리가 지정할 수는 없음
- 만약 ecw instance를 stop한다면?
  -> private ip는 인스턴스와 함께 남지만, public ip는 사라지게 된다.
  그리고 다시 Start하면?
  -> 다시 AWS 공용IP 풀에서 새로운 public ip를 받게된다.
  -> 즉, private ip가 변경되게 된다는 뜻 
  -> 사용자가 이전 주소로 재접속 할 수 없게됨
  -> 해결방법: *Elastic IP (EIP)*
-  EIP: AWS 계정에 위치한 Static Public IP이다. 
  해당 EIP를 배포하지 않는한, 동일한 IP로 instance가 사용할 수 있도록 한다.

![600](images/Pasted%20image%2020241013170411.png)
![600](images/Pasted%20image%2020241013170425.png)
- EIP 비용: EC2 인스턴스와 연결되어 사용되고 있는한 이에 대한 요금 청구를 하지 않는다. ip주소를 사용하고 있으니,
  하지만, 인스턴스를 stop하면 *ip 주소를 사용하지 않고 낭비하고 있는 것으로 요금을 청구하니 명심.*
  

![500](images/Pasted%20image%2020241013170447.png)

![600](images/Pasted%20image%2020241013170459.png)

*IPv6 중요 요소*
- IPv6는 public ip만 제공한다는 것
- 인스턴스를 stop, start든 계속 남아있음
- AWS ip 풀에 존재해서 그 범위를 만들거나 선택할 수 없음.
