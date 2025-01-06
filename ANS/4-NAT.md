### NAT(Network Address Translation) 네트워크 주소 변환
- Private Subnet에서 인터넷에 나갈경우 사용(without IGW)
- 아웃바운드 인터넷 트래픽을 NAT에게 보내면 공용IP(or EIP)로 변환하여 인터넷과 통신한다.
- SNAT 
  : private subnet에 위치한 ec2에서 인터넷으로 트래픽이 나간다고 했을때, src와 des ip 중에서 nat는 src ip를 natting한다.(ec2 ip가 아닌 public ip로 변환) 인터넷과 통신 후 nat로 다시 응답이 오면, nat는 다시 원래의 ec2 ip로 응답을 전달한다.
- Private subnet에서는 route table에 인터닛(0.0.0.0/0)으로 가는 트래픽의 target을 nat로 지정해야, nat를 타고 인터넷으로 나갈 수 있게 된다.
- Public subnet에 위치 - 인터넷과 직접 통신해야하므로
- AZ level

### NAT Gateway and NAT Instance
- NAT GW
	- Managed by AWS - higher bandwidth, better availability, no admin
	- 비용 up - 비용을 결정하는 요인은 두가지존재 -> 데이터 처리와 시간당 요금
	- bandwidth  scaling - 5Gbps ~ 100 Gbps
	- HA within AZ
	- shoud allocate EIP 
	- no security group to manage/required : NAT로는 들어오는 트래픽을 제어할 수 없다.
	- NACL at subnet level applies to NAT GW
	- support protocols: TCP, UDP and ICMP
	- Uses ports 1024-65535 for outbound connection
- NAT Instance
	- NAT EC2 - using Amazon LInux NAT AMI
	  EC2 인스턴스를 NAT로 구성
	- EIP or Public IP
	- NAT GW보다 비용절감
	- 동일하게 private subnet 라우팅을 추가해야한다. set target: NAT instance ID(ENI)
	- Disable Source/Destination check on instance
	  ec2인스턴스는 by default로 Source/Destination 검사를 수행한다. 즉, 인스턴스는 받거나 주는 모든 트래픽의 Source/Destination이 자신이어야한다. 하지만 NAT는 Source/Destination이 자신이 아닐때 트래픽을 주고 받을 수 있어야 하기때문에 해당 검사 기능을 비활성화 해야한다.

- NAT GW 실습
	- eip 생성 후 create nat gateway할때 지정
	- private subnet에 라우팅 nat를 타겟으로 추가(0.0.0.0/0)

### NAT Gateway with High Availability
- AZ 하나에만 NAT를 구성하여 인터넷으로 나가게 설정했다면, 해당 AZ하나 고장시에 연결한 사설 대역의 인터넷 연결이 모두 끊어지게 된다.
- NAT GW는 단일 AZ내에서 작동, NAT GW가 실패하면 해당 NAT GW를 사용하는 리소스와의 연결도 실패된다.
  따라서, 각 AZ에 하나의 NAT GW를 배포하고 동일한 AZ내에서 트래픽 라우팅 하는 것이 좋다.
- HA를 위해서 mutiple AZ 구성 필요

대규모로 여러 Amazon VPC에 NAT 게이트웨이 사용
https://aws.amazon.com/blogs/networking-and-content-delivery/using-nat-gateways-with-multiple-amazon-vpcs-at-scale/

![600](images/Pasted%20image%2020241027193342.png)


Exam Essentials

- Maximum size of VPC/Subnet is /16 which contains 65536 IP addresses
- Minimum size of VPC/Subnet is /28 which contains 16 IP addresses
- In every subnet 5 IPs are un-usable – first 4 and last IP in the subnet
- Subnet is associated with AZ. One subnet can not span across AZs. However one AZ may contain any number of subnets.
- All subnets by default follow VPC main route table unless explicitly attached to custom route table
- All route tables have a default local route which you can’t remove.This route enables inter VPC communication between all subnets.
- You can’t block any IP with Security group. Use Network ACL.
- Security groups is stateful - No need to explicitly add outbound rule for incoming return traffic or inbound rules for z return traffic
- Network ACL is stateless – Rules must be created for both side of the traffic
- NAT Gateway must be created in each AZ for High Availability.
- NAT Gateways must be created in Public Subnet
- NAT Gateway does not have security group hence traffic should be controlled by using Network ACL
- You don’t get access to NAT Gateway machine. It’s managed by AWS.
- NAT Instance may be cost effective but may not be as performant and resilient as NAT Gateway.  
- For NAT Instance, Source/Destination check must be disabled


Question 2:

In order to decrease the number of instances that have inbound web access, your team has recently placed a Network Address Translation (NAT) instance on Amazon Linux in the public subnet. The private subnet has a 0.0.0.0/0 route to the elastic network interface of the NAT instance. Users are complaining that web responses are slower than normal. What are practical steps to fix this issue?

- Replace the NAT instance with a NAT gateway

Which Amazon Virtual Private Cloud (Amazon VPC) feature allows you to create a dual homed instance?
- ENI(Elastic network instance)
Dual Homed Instance in AWS
https://medium.com/@vinoth186/dual-homed-instance-in-aws-5208e8e8c1ad