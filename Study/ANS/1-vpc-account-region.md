
## Introduction to AWS Networking
### VPC
- vpc: virtual private cloud
- AWS region 에 있는 하나의 개인 주소 공간이다. ex) 10.0.0.0/16
- vpc는 가상 프라이빗 클라우드이기 때문에, internet gateway를 열지 않으면, 아무도 vpc에 접속할 수 없다.
- region level (모든 리전은 2개이상의 az를 사용할 수 있다.)

### Subnet
- ec2 같은 리소스를 어디에 배치할 것인가는 어느 Subnet에 위치하느냐에 따라 다르며, 따라서 이전에 Subnet 디자인이 필요하다.
- vpc 내부 AZ 내부에 존재.
- public vs private (public은 subnet의 route table에 igw로의 route가 있는 경우, public subnet이라고 정의된다. 인터넷과 직접 통신할 수 있다는 뜻)
- Subnet 디자인시, HA(high availability)를 위해 적어도 두 개 이상의 AZ 사용 권장

### AWS 3 tier architecture
![[Pasted image 20241007120847.png|500]]
- ALB를 통해 Web server에 부하분산 처리
- HA를 위해 적어도 두 개이상의 리소스 필요. (DB replication)
- 웹서버에서 인터넷을 통해 패키지 다운 필요하다면? 
	- 웹서버가 인터넷에 도달해야하는데 현재 위 그림에서는 private subnet에 위치해 있기 때문에 통신할 수 없다. 
	- 즉 웹서버에서는 인터넷에 도달할 수 있어야 하며, 인터넷에서는 직접 웹서버에 도달할 수 없어야 한다. 이 웹서버에 도달할 수 있는 유일한 방법은 ALB를 통하는 것이여야 한다. 
	- -> *AWS NAT GW* 네트워크 주소 변환 게이트로 사설 서브넷 인스턴스에서 인터넷으로의 아웃바운드 트래픽은 NAT로 관리된다. 
	- ![[Pasted image 20241007121838.png|500]]

![[Pasted image 20241007122813.png|600]]
### VPC간의 통신
- vpc 내부의 앱과 통신하고 싶지만 인터넷을 거치고 싶지 않은 경우 
- *VPC Peering* 
	- 1 to 1
	- 비공개 연결로 신뢰성, 일관성 높일 수 있다.
	- full mesh 구조
- *Transit Gateway*
	- 여러개의 vpc를 한번에 연결. 중앙집중식
	- 하나의 route table로 관리할 수 있는 장점 존재.
	- 복잡성을 단순화시킴
	- 문제 발생시 관련없는 곳에도 영향이 갈수 있다는 단점 존재

### AWS 관리서비스와의 통신
- *VPC Endpoint*
- ex) s3, dynamoDB
- region 하위에 생성.
- 앱에서 통신하기 위해 *같은 리전에 존재*한다면, 인터넷을 거칠 필요가 없다
- vpc endpoint가 없다면 인터넷 게이트웨이를 통해 통신이 될 것이다. -> 추가비용 대기시간 안전성의 단점 존재


![[Pasted image 20241007123409.png]]
### PrivateLink
- 인터넷에 노출하지 않고, vpc와 aws 서비스간의 프라이빗한 연결을 설정.
- 마치 VPC에 있는 것처럼 서비스에 비공개로 연결하는데 사용할 수 있는 기술.
- 인터넷 게이트웨이, NAT 디바이스, 퍼블릭 IP 주소, AWS Direct Connect 연결 또는 AWS Site-to-Site VPN 연결을 사용하지 않아도 프라이빗 서브넷에서 서비스와 통신가능.
- ex) 로드밸런서를 통해 서비스를 노출 -> privateLink -> vpc endpoint
- 안전하게 AWS 서비스 엑세스 가능하며, 중요 데이터를 안전하고 확장 가능한 비공개방식으로 전송
### Route53
- IP 주소 대신 DNS 사용

### CloudFront(CDN)
- 사용자와 가까운 곳에 고정 콘텐츠 캐싱
- AWS 전용 네트워크를 통해 VPC와 연결
- 최종 사용자의 지연 감소

### Flow Logs
- vpc 에 in/out 트래픽의 패킷 캡처
- 로그들 분석을 위해 s3, cloudwatch로 전송

하이브리드 모드 일부는 온프렘인 경우 
how to connect 온프렘
### VPN
- on-prem과의 연결
- site to site
- VPC - VGW - VPN - onprem
- ex) 재택근무

### Direct Connect
- AWS로 전용 네트워크 연결 생성 - 전용선 서비스
- AWS 리소스에 대한 최단 경로
- 연결에 안전성 보장, 병목현상 지연시간 증가하는 가능성이 줄어듬


## VPC
실제 전형적인 물리적 it network를 그대로 VPC에도 매핑시킬 수 있으며,
vpc 는 가상 프라이빗 클라우드일뿐 실제 내부적으로는 물리적으로 구조되어있다.
igw를 연결하지 않는한 기본적으로 폐쇄망이다.

![[Pasted image 20241007233943.png|500]]

### AWS Account -> Region & AZ -> VPC
![[Pasted image 20241007234643.png|500]]
- 자연재해가 발생해도 고가용성을 위해 AZ
- vpc는 region level

![[Pasted image 20241007234755.png]]
- AZ 별로 subnet 생성 필요

![[Pasted image 20241007234927.png]]
