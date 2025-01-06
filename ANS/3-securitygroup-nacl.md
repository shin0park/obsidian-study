## VPC Firewal -  Security Group
![500](images/Pasted%20image%2020241021103701.png)
- sg - ec2 인스턴스 level
- nacl - subnet level
	- so, inbound traffic은 NACL을 먼저 거치게 된다.

### Security Group
![500](images/Pasted%20image%2020241021141604.png)
![500](images/Pasted%20image%2020241021141905.png)

- access to Ports
- authorised IP ranges - IPv4 IPv6
- control of inbound/outbound network
- SG are *stateful*
- reference another sg 

중요: *SG는 stateful 하다.*
sg rule 에 트래픽 반환(return traffic)에 대한 룰까지 추가할 필요가 없다.
트래픽 반환은 by default allow

*reference another sg*
![500](images/Pasted%20image%2020241021143139.png)
- inbound를 추가한다고 했을때, ip를 추가하는 대신 다른 ec2에 적용한 sg를 소스로 추가할 수 있다.
- why? 웹계층과 앱계층이 있다고 했을때, 웹계층에서 어떤 앱계층과도 통신하고 싶다고 하자,
  그렇다면 웹계층의 sg에 앱계층에 있는 모든 인스턴스의 ip를 inbound로 허용해줘야한다. 하지만 reference를 사용하면, 단지 앱계층의 sg만 웹계층의 sg의 소스로 등록하면 앱게층의 모든 앱과만 통신하도록 설정할 수 있게 되는 것이다.

Security Group
- 여러개의 인스턴스에 attach 할 수 있다.
- 만약, 애플리케이션에 access할 수 없다면(timeout) -> sg issue
- 만약 애플리케이션에 connection refused 에러라면, 그건 sg는 허용하지만 에플리케이션 에러나 아직 running state가 아닌 경우일 수 있다.
- sg를 생성하면 *All inbound traffic is blocked by default*
- *All outbound traffic is authorised by default*
![400](images/Pasted%20image%2020241021150349.png)


## VPC Firewal - Network Access Control List(NACL)
![500](images/Pasted%20image%2020241021150750.png)
![400](images/Pasted%20image%2020241021152247.png)
- sg와 반대로 nacl은 *stateless* 하다. return traffic에 대한 rule도 추가해줘야 통신할 수 있다. 조금 더 까다로움
-  기본적으로 인터넷 ec2에서 인터넷 통신을 하기위해 outbound에 0.0.0.0/0 대역을 설정해놨어도,
  현재 적용된 inbound 통신이 이뤄지려면 해당 1.2.3.4/32 로의 outbound 까지 존재해야 통신이 가능하다. *nacl은 stateless 하므로*
- 단 이때 새로운 연결을 맺을때마다 pc의 port는 변경될 것이므로 *range* 로 설정해줘야한다.

- Allow, Deny rules (sg에서는 deny rule을 통해 트래픽을 거부할 수 없지만 nacl은 가능)
- sg에서는 모든 rules이 함께 적용되지만, nacl에서는 *rule number 순으로 적용된다.*
- rule은 보통 10, 100단위가 보편적 (중간에 어떤 rule을 추가할지 모르니)

![600](images/Pasted%20image%2020241021153153.png)


## Default VPC
모든 aws region에는 하나의 default vpc가 존재한다.
삭제 가능하지만 편의성을 위해 기본적으로 리전별 하나씩 생성되어있다.

![600](images/Pasted%20image%2020241021153338.png)
(뭄바이)
- 삭제가능하며, 다시 생성도 가능
- 하지만 custom vpc를 사용하기를 권장

## Hands On: Creating VPC with Public subnet

![500](images/Pasted%20image%2020241021153430.png)
#### Exercise 1
1. Delete default VPC (We will create our own VPC)
2. Create VPC
    a. Go to VPC service => Your VPCs => Create VPC (Name: MyVPC, CIDR: 10.100.0.0/16) => Create
3. Create Internet Gateway  
	a. Internet Gateways => Create internet gateway
4. Attach Internet Gateway to VPC  
	a. Select Internet gateway => Actions => Attach to VPC => Select your VPC

5. Create Subnet
	a. Subnets => Create subnet (Name: MyVPC-Public,VPC: MyVPC, AZ: Select first AZ - ap-south-1a, CIDR: 10.100.0.0/24)
	b. Select Subnet => Action => Modify Auto Assign Public IP => Enable => Save
	-> *인스턴스 실행되면 자동으로 공용IP갖도록*
        
6. Create Route table
    a. Route Tables => Create Route Table (Name: MyVPC-Public,VPC: MyVPC)
    b. Select Route table => Routes => Edit => Add another route (Destination: 0.0.0.0/0,Target: Internet
        gateway => igw-xxx) => Save
    
7. Associate Route table with Subnet to make it Public subnet  
	a. Select Route table => Subnet Associations => Edit => Check the MyVPC-Public subnet => Save

8. Launch EC2 instance in newly created Public Subnet
	a. Go to EC2 Service => Instances
	b. Launch EC2 Instance => Select Amazon Linux 2 => Select t2.micro
	c. Configure Instance Details:
	    i. Network: MyVPC  
	    ii. Subnet: MyVPC-Public (rest all defaults)
	d. Add storage (all defaults)
	e. Add Tags
		i. Key=Name,Value=EC2-A 
	f. Configure Security Group
		i. Add rule for SSH port 22 for source as MyIP 
	g. Review and Launch

9. Connect to EC2 instance (Public IP) from your desktop/laptop using Putty or terminal (ec2-user)

![500](images/Pasted%20image%2020241021155149.png)
1. Create a Private subnet  
	a. Create subnet (Name: MyVPC-Private,VPC: MyVPC, AZ: Select different AZ (ap-south-1b), CIDR:10.100.1.0/24)
	
2. Create Private route table  
	a. Route Tables => Create Route Table (Name: MyVPC-Private,VPC: MyVPC)
	
3. Associate Route table with Subnet to make it Private subnet  
	a. Select Route table => Subnet Associations => Edit => Check the MyVPC-Private subnet => Save

4. Launch another EC2 instance in sameVPC but in newly created Private subnet.
    a. Tag this instance with Name=EC2-B
    b. New security group    
    - Add rule SSH for CIDR of Public Subnet source CIDR -> *for ssh*
    - Add rule All-ICMP IPv4 for Public Subnet source CIDR -> *ping 하기 위함*
            
5. Note down EC2-B private IP address

6. Try to ping EC2-B Private IP from EC2-A instance => Should work
    
7. Try to connect to EC2-B instance from EC2-A (Permissions denied..Why?) -> pem키 필요
	a. $ssh ec2-user@10.100.1.x (Replace this ip with your EC2-B IP address)
	
8. Get your ssh .pem file on EC2-A instance
	a. Open local .pem file with nodepad and copy the content (CTRLA => CTRL+C)  
	b. On EC2 A terminal => vi key.pem => enter => press i => paste using right click => esc => :wq => enter  
	c. chmod 600 key.pem  
	d. ssh -i key.pem ec2-user@10.100.1.x => should be able to connect

6. Try to ping google.com from EC2-B instance  
	a. ping google.com (You should not be able to ping.Why?)