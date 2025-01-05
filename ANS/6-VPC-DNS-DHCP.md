#### AWS DNS & DHCP
![500](Pasted%20image%2020241110203009.png)
- 처음만 위와 같은 통신을 통해 ip를 가져오며, 이후의 연결부터는 캐시된 정보를 사용
- DNS is the backbone of the internet - DNS 작동이 안되면 인터넷 서버에 연결할 수 없음

TODO-learn
- Amazon VPC DNS server(Route53 DNS Resolver)
- DHCP Option sets
- EC2 DNS name - internal and external - EC2 인스턴스가 어떻게 DNS Name을 얻는가
- VPC DNS 속성 -enableDnsSupport, enableDnsHostname
- intruduction to Route53 resolver endpoints

#### Amazon VPC DNS server(Route53 DNS Resolver)
Amazon Route 53 Resolver 는 
퍼블릭 레코드, Amazon VPC별 DNS name 및 Amazon Route 53 프라이빗 호스팅 영역에 대한 AWS 리소스의 DNS 쿼리에 반복적으로 응답한다.

 - vpc를 생성할때 마다 default DNS server인 Route53 DNS Resolver를 생성한다.
 - Amazon은 VPC+2 IP 주소로 Route 53 Resolver에 VPC 연결한다. 
   (예약된 주소: VPC의 DNS 서비스에 사용
	   - `10.0.0.1` - **기본 게이트웨이** IP로 예약됨 
	     (VPC의 인터넷 게이트웨이나 NAT 게이트웨이를 통해 외부와의 연결에 사용)
	- `10.0.0.2` - **Route 53 Resolver**의 DNS IP로 예약됨
	  이를 통해 외부 네트워크와의 연결 및 DNS resolver 기능 사용.) 
 - 또한 VPC내에서는 가상 ip인 169.254.169.253 으로도 접근가능.(오직 VPC내에서만 가능)


![500](Pasted%20image%2020241110221740.png)
Resolves DNS request from:
- Route53 Private Hosted Zone 
	- vpc 내에서만 유효한 도메인 이름을 설정. 내부도메인
	- 예를 들어, 특정 VPC에 연결된 리소스에 대해 `internal.example.com`과 같은 내부 도메인 이름을 정의가능
	- 프라이빗 호스팅 영역에 설정된 도메인은 지정된 VPC 내에서만 조회할 수 있고, 외부에서는 접근이 불가능
	- create record - private ip 주소와 매핑 가능
- VPC internal DNS 
	- AWS가 각 vpc 리소스에 대해 자동으로 할당하는 기본 DNS name. ex.ec2 dns name
	- 예를 들어, EC2 인스턴스가 `ip-10-0-0-1.ec2.internal`와 같은 형식으로 자동 생성된 DNS 이름을 갖게된다.
	- VPC별 DNS name은 특별히 Route 53 프라이빗 호스팅 영역을 설정하지 않아도 VPC 내에서만 접근할 수 있는 기본적인 DNS name
- Forwards other requests to Public DNS (including Route 53 Public Hosted Zone)
	- 퍼블릭 호스팅 영역을 포함한 외부 퍼블릭 DNS에 대한 요청을 포워딩 (재귀적 DNS 검색)
	- ex. google.com, amazon.com
	- Amazon services public endpoint
		- AWS 서비스가 인터넷을 통해 접근할 수 있도록 제공하는 URL
		- 퍼블릭 엔드포인트는 일반적으로 인터넷을 통해 접근 가능하므로, 퍼블릭 엔드포인트를 사용하는 서비스는 누구나 인터넷을 통해 접근할 수 있다. 
		  다만, AWS 서비스는 접근 제어와 보안 설정을 통해 사용자가 해당 서비스에 접근할 수 있도록 제한할 수 있다.
		- sqs.ap-south-1.aws.amazon.com
		- s3.ap-south-1.amazonaws.com
		-> *tip. aws dns에 레코드가 없으면 public dns로 가는것이 아니라, 알맞는 hosted zone이 없을때 public dns 로 나가는 것.*

![500](Pasted%20image%2020241110222700.png)
so many domain register

#### DHCP Option sets
- *DHCP 옵션 세트는 EC2 인스턴스와 같은 VPC의 리소스가 가상 네트워크를 통해 통신하는 데 사용하는 네트워크 설정 그룹*
- 각 리전에 기본 DHCP 옵션 세트가 있으며, 각 VPC는 해당 리전의 기본 DHCP 옵션 세트를 사용
- 단, 사용자 지정 DHCP 옵션 세트를 만들어 VPC에 연결하거나 DHCP 옵션 세트 없이 VPC를 구성한 경우는 예외
- 여러 VPC와 DHCP 옵션 세트를 연결할 수 있지만 각 VPC에는 하나의 DHCP 옵션 세트만 연결되어 있어야 한다.
- **DHCP 메시지의 옵션 필드**에는 **Domain name**, **Domain name server**, **NTP 서버**, **NetBIOS 노드 유형**과 같은 네트워크 구성 매개변수가 포함
- AWS는 **VPC 생성 시 자동으로 DHCP 옵션 세트를 생성하고 연결** (특별히 별도 DHCP option set 지정하지 않는 한, 기본 DHCP option set로 연결). 이 때 설정되는 기본 값들은 다음과 같다.
    1. **domain name servers**: 기본값은 **AmazonProvidedDNS**. 
       이는 AWS의 기본 DNS 서비스인 **Route 53 Resolver**를 사용하여 DNS 쿼리를 처리하게 된다는 의미.
    2. **domain name**: 기본값은 **해당 리전의 내부 Amazon 도메인 이름**으로 설정 
       도메인 이름 시스템(DNS)을 사용하여 호스트 이름을 확인하는 경우 클라이언트가 사용해야 하는 도메인 이름이다.
       예를 들어, `ap-south-1` 리전에서는 기본적으로 `*.ap-south-1.compute.internal` 형식의 내부 도메인 이름이 사용
       
document reference
https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/DHCPOptionSetConcepts.html

![600](Pasted%20image%2020241110224734.png)

![600](Pasted%20image%2020241110224842.png)
nameserver가 vpc +2ip인 10.10.0.2 인 것을 볼 수 있다.
option set에 보면 name-server=AmazonProvidedDNS 를 사용한다고 나와있으므로

#### AWS assigned domain names for EC2
- **내부 DNS**:
    
    - `ip-<private-ipv4-address>.ec2.internal` (미국 동부 리전 US-East-1)
    - `ip-<private-ipv4-address>.region.compute.internal` (기타 모든 리전)

    EC2 인스턴스의 **사설 IP 주소**에 대해 내부 DNS가 할당된다. 
    이 도메인 이름은 인스턴스가 동일한 VPC 내에서만 접근할 수 있도록 내부적으로 사용가능하다.
    
- **외부 DNS** (인스턴스에 퍼블릭 IP가 할당된 경우):
    - `ec2-<public-ipv4-address>.compute-1.amazonaws.com` (미국 동부 리전 US-East-1)
    - `ec2-<public-ipv4-address>.region.amazonaws.com` (기타 리전)
    
    퍼블릭 IP가 할당된 EC2 인스턴스는 외부 DNS 이름을 가질 수 있으며,
    이를 통해 인터넷에서 해당 인스턴스에 접근할 수 있다.

![600](Pasted%20image%2020241110230051.png)

- DHCP Option set는 수정할 수 없으며, 새로운 것을 만들고 VPC와 연결해서 바꿔야한다. (몇시간이 걸린다.)
- 만약 바로 적용하고 싶다면, os command를 사용하여 직접 DHCP 을 변경.
  ex. linux: `$sudo dhclient -r eth0` (DHCP 임대 해제)
- DHCP Option set가 설정되어있지 않다면 VPC에서 DNS Server로 접근 할 수가 없다.
### **VPC DNS 속성**

1. **enableDnsSupport**
    - **기본값: True**
    - VPC가 Amazon에서 제공하는 DNS 서버를 통해 DNS 확인을 지원하는지 여부를 결정
    - 만약 `True`로 설정되면, EC2 인스턴스는 **169.254.169.253** (AWS DNS 서버)을 통해 DNS Resolution 요청할 수 있다. 이 IP는 **VPC+2**로 예약된 주소
2. **enableDnsHostname** (= DNS 호스트 이름 설정)
    - **기본값: False** (새로 생성된 VPC에 대해서)
    - **기본 VPC**에서는 기본값이 `True`
    - `enableDnsSupport`가 `True`일 경우에만 작동
    - 이 설정이 `True`일 경우, 
     VPC가 퍼블릭 IP 주소가 있는 인스턴스에 퍼블릭 DNS 호스트 이름을 할당
     (**`enableDnsHostnames`** 설정은 이 내부 DNS 호스트 이름의 할당에는 영향을 미치지 않으며, 주로 **퍼블릭 DNS 호스트 이름**과 관련된 기능을 제어)
1. **Route 53 프라이빗 호스팅 영역 사용 시**:
    - **사용자 정의 DNS 도메인 이름**을 Route 53 프라이빗 호스팅 영역에서 사용하려면, 
      **`enableDnsSupport`**와 **`enableDnsHostname`** 둘 다 `True`로 설정해야한다.
aws document
https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/AmazonDNS-concepts.html#vpc-dns-support

