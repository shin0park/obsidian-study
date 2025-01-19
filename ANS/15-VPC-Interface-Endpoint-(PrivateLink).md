
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
	- VPC - DNS resolution enable & DNS hostname enable 한다.
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
- AWS가 **리전별 및 영역별 DNS entries**를 생성하여 interface endpoint를 프라이빗 IP 주소로 해석한다.
- IPv4 트래픽과 TCP 트래픽만 지원.
