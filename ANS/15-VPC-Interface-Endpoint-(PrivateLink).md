 
## VPC Interface Endpoint (PrivateLink)

일반적으로 interface endpoint는 AWS PrivateLink에 의해 구동되도록 설정되어있다.
PrivateLink는 근본 개념으로, 이를 통해서 AWS Service뿐만 아니라 고객 서비스에도 접근 할 수 있으며, Interface endpoint는 이러한 연결을 가능하기 한다.
### VPC Interface Endpoint – Accessing AWS or Customer Services
![600](images/Pasted%20image%2020250119230340.png)


#### Let’s first create VPC interface endpoint
![400](images/Pasted%20image%2020250119230439.png)
- VPC 생성 및 퍼블릭 서브넷과 프라이빗 서브넷 구성
- 두 서브넷에 EC2 인스턴스 시작
- 프라이빗 서브넷에 **SQS 서비스**를 위한 interface endpoint 생성
	- VPC - Enable DNS Support & DNS hostname enable 한다.
	  -> AWS는 서비스에 대해 **DNS 레코드**를 제공하며, VPC 내에서 이를 활용하려면 DNS 해석 및 DNS 호스트 이름 설정이 필요하다. 
	- vpc endpoint 정책 - all allow
- Private EC2에 로그인하여 interface endpoint를 통해 SQS에 메시지 전송(PutMessage)
	- Vpce의 Private Dns name 사용: `sqs.ap-south-1.amazonaws.com`
	- SQS에 PutMessage 하기위한 IAM 생성: trust relation: ec2, role: SQSFullAccess
	- SQS url로 send message CLI 명령어 실행
	- vpce의 sg의 inbound rule을 삭제하면 위의 통신 불가 테스트

### VPC Interface Endpoint - Features

- Interface endpoints VPC 내 **ENI(Elastic Network Interface)**를 사용해 local IP 주소를 생성한다.
- 가용 영역(AZ)별, interface endpoint 하나씩 생성하여 고가용성 보장
- 시간당 약(~$0.01/hr per AZ) and data processing cost (~$0.01/GB)
- Uses Security Groups – inbound rules 을 사용하여 접근 제어
- AWS가 **resional and zoneal DNS entries**를 생성하여 interface endpoint를 프라이빗 IP 주소로 해석(resolve) 한다.
- IPv4 트래픽과 TCP 트래픽만 지원.

### VPC Interface endpoint – Accessing Customer service

#### **AWS PrivateLink (VPC Endpoint Services)**

![](images/Pasted%20image%2020250120220316.png)

AWS PrivateLink 는 수천 개의 VPC(동일 계정 또는 다른 계정)에 서비스를 노출하는 가장 안전하고 확장 가능한 방법이다.

- Does not require VPC peering, internet gateway, NAT, routetables ...  
- Service VPC에 **네트워크 로드 밸런서(NLB)** 필요
- Customer VPC에 **ENI(Elastic Network Interface)** 필요
- **고가용성**:
	- NLB와 ENI가 여러 가용 영역(AZ)에 배치되면 솔루션은 **장애 복원력**을 제공한다

---

### Interface VPC Endpoint – Accessing Customer VPC services

![](images/Pasted%20image%2020250120221143.png)- PrivateLink를 사용해 Customer VPC에서 Provider VPC 서비스에 안전하게 접근 가능

### Interface VPC Endpoint – Accessing Customer On-premises services
**온프레미스 서비스 접근**

![](images/Pasted%20image%2020250120221319.png)
- PrivateLink를 통해 온프레미스 네트워크의 서비스에 접근 가능

---
#### **VPC Endpoint Service**

- **VPC Endpoint Service**는 **AWS PrivateLink**를 사용해 다른 VPC 또는 AWS 계정, 혹은 온프레미스 네트워크에 **자신의 서비스**를 노출하는 데 사용된다.
- 서비스 제공자(service Provider)가 설정하며, **서비스 소비자(service Consumer)는 VPC Endpoint를 생성하여 연결**한다.
##### **Feature**

- **구성 요구 사항**:
    - 서비스 제공자(Provider)는 **네트워크 로드 밸런서(NLB)**를 통해 서비스를 노출한다.
    - 소비자(Consumer)는 Interface Endpoint를 통해 서비스에 접근한다.
- **주요 역할**:
    - 서비스 제공자(Provider) 역할
    - 사용자 정의 애플리케이션, API, 서비스 등을 PrivateLink로 노출한다.
- **승인 메커니즘**:
    - 서비스 제공자(Provider)는 소비자 계정(Consumer Account)을 **whitelist 에 추가**하여 접근을 제어한다.

Document: 엔드포인트 서비스 구성
https://docs.aws.amazon.com/ko_kr/vpc/latest/privatelink/configure-endpoint-service.html 


#### **Accessing Customer Service - Demo**

![](images/Pasted%20image%2020250120221736.png)
##### **1) 사전 준비**
- **EC2 AMI 생성**:
    - httpd webserver 가 설치된 EC2 AMI 생성
    - 이 AMI를 사용해 VPC-B에서 더미 서비스를 호스팅할 EC2 인스턴스를 실행

##### **2) VPC-B 설정**
- 프라이빗 서브넷 두 개를 가진 VPC-B 생성
- 위에서 만든 AMI를 사용해 EC2 인스턴스를 프라이빗 서브넷에 시작
- 다른 프라이빗 서브넷에 NLB(Network Load Balancer) 생성
- NLB 뒤에 EC2 인스턴스를 등록

##### **3) VPC Endpoint Service 생성**
- Provider: NLB와 연결된 VPC Endpoint Service 생성
	- Acceptance required 체크박스: 승인여부, 누군가 vpc endpoint 서비스에 연결하고자 시도한다면 받을거라는 의미 
		- disable: 제공자는 별도의 승인이 필요하지 않으며, 소비자가 Interface Endpoint를 생성하면 승인 절차 없이 서비스에 연결
		- enable: 승인절차를 거쳐서, 승인(Approve) 또는 거절(Reject)를 해야함.
	- Enable Private DNS name 설정도 가능
- VPC-A의 AWS Account(Consumer) 를 Whitelist 추가 (두 VPC가 다른 AWS 계정에 속한 경우 필요) 

##### **4) VPC-A 설정**
- 퍼블릭 서브넷과 프라이빗 서브넷이 있는 Service Consumer VPC(VPC-A) 생성
- Consumer: VPC Endpoint를 생성하고, 이전에 만든 Endpoint Service와 연결

##### **5) 서비스 접근 테스트**
- VPC-A의 프라이빗 EC2 인스턴스에 로그인
- VPC Endpoint DNS를 통해 서비스 접근 테스트

---
### How to create AMI

- 아래의 userdata를 사용하여 기본 VPC(Default VPC)에서 EC2 인스턴스를 시작한다
	- userdata는 EC2 인스턴스를 부팅할 때 자동으로 실행
- EC2 인스턴스가 실행된 후, 해당 인스턴스의 **퍼블릭 IP(Public IP)**를 사용해 웹 페이지에 접근할 수 있는지 확인한다
- 위에서 생성한 EC2 인스턴스를 기반으로 **AMI(Amazon Machine Image)**를 생성한다.
- 이 AMI는 동일한 설정의 EC2 인스턴스를 재사용하거나 복제하는 템플릿으로 활용된다.

```
#!/bin/bash

yum install -y httpd

systemctl start httpd

systemctl enable httpd

echo THIS IS A SERVICE HOSTED BEHIND AWS PRIVATELINK > /var/www/html/index.html
```

---

### VPC Interface Endpoint – DNS

#### VPC Interface Endpoint

- VPC Interface Endpoint는 Private endpoint interface hostname 을 가지는 **ENI(Elastic Network Interface)를 생성한다.

- **Private DNS settings for Interface endpoint**
	- The public hostname of a service will resolve to the private Endpoint Interface hostname
    - **퍼블릭 호스트 이름**: AWS 서비스의 퍼블릭 호스트 이름(예: `athena.us-east-1.amazonaws.com`)은 **Interface Endpoint**의 프라이빗 호스트 이름으로 해석된다.
	    - 이는 퍼블릭 인터넷을 거치지 않고 AWS의 프라이빗 네트워크를 통해 서비스에 안전하게 접근할 수 있도록 보장한다.

- **VPC Setting**
    - **Enable DNS hostnames**와 **Enable DNS Support** 설정이 활성화되어야 한다.
	    - **Enable DNS hostnames**: VPC 내 리소스가 DNS 호스트 이름을 사용할 수 있도록 허용
	    - **Enable DNS Support**: DNS 쿼리를 처리할 수 있도록 지원
		    - 이 두 설정이 `True`로 설정되지 않으면 퍼블릭 DNS 이름이 프라이빗 DNS로 해석되지 않으며, Interface Endpoint를 통한 접근이 제한될 수 있다.

- Interface Endpoint는 다양한 DNS 이름을 제공

1. **Regional(리전 단위)**:
    - `vpce-0b7d2995e9dfe5418-mwrths3x.athena.us-east-1.vpce.amazonaws.com`
    - 리전 단위의 프라이빗 엔드포인트를 나타냄

2. **Zonal(가용 영역 단위)**:
    - `vpce-0b7d2995e9dfe5418-mwrths3x-us-east-1a.athena.us-east-1.vpce.amazonaws.com`
    - `vpce-0b7d2995e9dfe5418-mwrths3x-us-east-1b.athena.us-east-1.vpce.amazonaws.com`
    - 가용 영역별로 제공되는 엔드포인트 DNS

3. **서비스 DNS(기본 이름)**:
    - `athena.us-east-1.amazonaws.com`
    - 서비스의 기본 DNS 이름을 통해 Interface Endpoint를 통해 연결 가능

![500](images/Pasted%20image%2020250120231145.png)
**Private DNS enable**: AWS 서비스의 기본 퍼블릭 DNS 이름(예: `kinesis.us-east-1.amazonaws.com`)이 Interface Endpoint의 프라이빗 IP 주소(ENI)로 해석


![](images/Pasted%20image%2020250120231158.png)

![](images/Pasted%20image%2020250120231209.png)


### VPC endpoints DNS Summary

- Private DNS 활성화
	- AWS 서비스(예: `ec2.us-east-1.amazonaws.com`)의 기본 퍼블릭 호스트 이름이 Interface Endpoint의 프라이빗 호스트 이름으로 자동으로 해석된다.
	- 퍼블릭 인터넷을 거치지 않고 안전한 프라이빗 네트워크에서 통신할 수 있도록 지원
- Private Hosted Zone 생성 및 연결
	- AWS는 SQS, Kinesis와 같은 서비스에 대해 **Private Hosted Zone**을 자동으로 생성하고, 이를 VPC와 연결한다.
	- 이 Hosted Zone을 통해 기본 퍼블릭 DNS 이름이 Interface Endpoint의 프라이빗 IP로 매핑된다.
- VPC Setting:  “Enable DNS hostnames” and “Enable DNS Support” must be 'true’


---

## VPC Interface Endpoint – Remote Access


Interface Endpoint는 다음과 같은 네트워크 연결을 통해 접근할 수 있다.
- Direct Connect
- AWS Managed VPN  
- VPC peering connection


![](images/Pasted%20image%2020250120232918.png)
https://d1.awsstatic.com/whitepapers/building-a-scalable-and-secure-multi-vpc-aws-network-infrastructure.pdf

- 중앙화된 VPC에 Interface Endpoint를 설정하고, 이 Endpoint를 다른 VPC 또는 온프레미스 네트워크에서 접근할 수 있도록 구성한다.

- **VPC Peering을 통한 접근**
    - 피어링된 VPC에서 Interface Endpoint의 프라이빗 DNS를 resolve 할 수 있다.
    - **Route 53 Private Hosted Zone**에 attached

- **온프레미스 DNS resolver**
    - 온프레미스 네트워크에서 DNS 쿼리를 처리하기 위해 **Route 53 Resolver**으로 포워딩한다.

---
### AWS PrivateLink vs VPC Peering

#### **VPC Peering**

- 피어링된 VPC 간 다수의 리소스가 통신해야 할 경우 유용하다.
- CIDR 범위가 겹치는 경우 VPC 피어링 연결을 생성할 수 없다.
- 최대 125개의 피어링 연결 생성 가능
- 양방향 트래픽 기원이 허용됨

#### **AWS PrivateLink**

- 피어링 없이 VPC 간에 단일 애플리케이션에 접근을 제공해야 할 때 유용하다.
- CIDR이 겹치는 경우에도 지원
- 연결 개수에 제한이 없음
- 트래픽은 소비자(Consumer) VPC에서만 기원할 수 있음

---

### **Summary**

1. **VPC Peering**
    - 동일 리전 또는 교차 리전의 두 VPC 간 통신을 가능하게 함

2. **VPC Endpoint**
    - IGW, NAT device, VPN 연결 또는 Direct Connect 없이도 AWS 서비스 또는 PrivateLink 기반 서비스를 프라이빗하게 연결 가능
    - VPC 내 인스턴스 Public IP가 필요하지 않음
    - 트래픽은 Amazon 네트워크를 벗어나지 않음

3. **Gateway VPC Endpoint**
    - 서비스에 연결할 수 있지만, 네트워크 구성 요소로 존재하지 않음

4. **Interface VPC Endpoint**
    - ENI, 프라이빗 IP 주소 및 DNS 이름을 사용하여 AWS 클라우드 서비스를 VPC 내에서 프라이빗하게 접근 가능
    - AWS PrivateLink의 확장으로, 사용자 정의 엔드포인트를 생성하거나 다른 사용자가 생성한 엔드포인트를 사용 가능

---
### **Exam Essentials**

1. **VPC Peering**
    - **not support transitive**
        - IGW, NAT와 같은 다른 VPC 리소스에 접근 불가
    - Security Group Chaining
        - 피어링된 VPC의 보안 그룹을 인바운드 규칙의 소스로 사용 가능
    - 최대 125개의 피어링 연결 생성 가능

2. **Gateway Endpoint**:
    - S3 또는 DynamoDB에 프라이빗 연결 제공
    - IGW 또는 NAT 필요 없음
    - 트래픽 라우팅을 위해 서브넷의 라우트 테이블 수정 필요
    - Direct Connect, VPN 또는 VPC Peering을 통해 접근 불가

3. **Interface Endpoint**:
    - ENI(탄력적 네트워크 인터페이스)를 서브넷에 생성
    - 리전 및 가용 영역별 DNS 이름을 제공
    - Route 53 Private Hosted Zone을 사용하여 커스텀 DNS 구성 가능
    - Direct Connect, AWS Managed VPN 및 VPC Peering 연결을 통해 접근 가능.
    - 트래픽은 VPC 내 리소스에서 시작하며, endpoint service는 요청에만 응답 가능하다. endpoint service에서 요청을 시작할 수는 없다.

