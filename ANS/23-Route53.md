
### What is DNS?

- **Domain Name System**의 약자로, 사람이 읽기 쉬운 도메인이름 (예: `www.google.com`)을 **기계가 인식 가능한 IP 주소**(예: `172.217.18.36`)로 변환
- DNS의 중요성: 
	- DNS is the backbone of the Internet
	- 인터넷의 핵심 인프라로, 웹사이트 및 기타 인터넷 리소스에 접근할 수 있도록 한다.
- **계층적 네이밍 구조**(hierarchical naming structure) 사용: DNS는 계층적 구조를 통해 도메인 이름을 체계적으로 관리한다.
    - `api.example.com` → 서브도메인
    - `example.com` → SLD
    - `.com` → TLD (최상위 도메인)

### DNS Terminologies

- **도메인 등록기관(Domain Registrar)**: Amazon Route 53, GoDaddy 등
- **DNS Records**: A, AAAA, CNAME, NS 등
- **Zone File**: DNS 레코드를 저장하는 파일
- **Name Server**: DNS 질의를 해석 (Authoritative / Non-Authoritative)
- **TLD**: 최상위 도메인 - .com, .us, .gov, .org 
- **SLD**: 2차 도메인 - amazon.com, google.com 
- **FQDN**:  `http://api.www.example.com` (서브도메인, SLD, TLD 포함)

![400](images/Pasted%20image%2020250518152529.png)

---
### How DNS Works

![](images/Pasted%20image%2020250518152806.png)

1. 사용자가 웹 브라우저에 `example.com` 입력
2. 로컬 DNS 서버가 질의 → 루트 DNS → TLD DNS → SLD DNS 순서로 질의 전달
3. 최종적으로 **도메인의 IP 주소를 반환**
4. TTL(Time to Live) 동안 결과는 캐시됨

---
### Amazon Route 53

![300](images/Pasted%20image%2020250518152836.png)

- 고가용성 highly available, 확장성 scalable, fully managed and Authoritative DNS
	- Authoritative = 사용자(you)가 DNS 레코드를 업데이트 가능,  you have full control over this DNS
- Route 53 is also a Domain Registrar
- 리소스의 health check 가능
- AWS 유일의 **100% 가용성 SLA** 제공
- "Route 53"의 '53'은 traditional DNS 포트 번호 53에서 유래

#### Route 53 – Records

- **DNS 트래픽을 라우팅하는 방법**을 정의
- 각 레코드는 다음 요소 포함:
    - Domain/subdomain Name – e.g., example.com
    - Record Type – e.g., A or AAAA
    - Value – e.g., 123.456.789.123
    - Routing Policy – how Route 53 responds to queries (단순, 가중치, 지연 시간 기반 등)
    - TTL – amount of time the record cached at DNS Resolvers
        
- 주요 지원 레코드 유형:
    - (must know) A /AAAA / CNAME / NS
    - (advanced) CAA/ DS/ MX/ NAPTR/ PTR/ SOA/ TXT/ SPF/ SRV


#### Route 53 – RecordTypes
- A – maps a hostname to IPv4
- AAAA – maps a hostname to IPv6
- CNAME – maps a hostname to another hostname
	- target은 A 또는 AAAA 레코드가 있는 domain name이어야 한다.
	- DNS namespace의 최상위 노드에는 CNAME 레코드를 만들 수 없다. (Zone Apex)
		- Zone Apex란
			- 특정 DNS Hosted Zone의 **최상위 노드**를 의미한다
				- `example.com.` 이라는 Hosted Zone을 만들었다면,
					- 그 호스팅 존의 **Zone Apex**는 **`example.com.`**
				- `mycompany.co.kr.` 호스팅 존이라면
				    - Zone Apex는 **`mycompany.co.kr.`**
		- 왜 CNAME을 Zone Apex에 쓸 수 없을까?
			-  CNAME 레코드는 하나의 레코드만 존재할 수 있음
				- domain name에는 여러 타입의 레코드를 가질 수 있다.
					- 일반적인 A 레코드 예시
						-  ```
						    example.com.  IN  A     192.0.2.1
							example.com.  IN  MX    mail.example.com.
							example.com.  IN  TXT   "v=spf1 ~all"
						- A, MX, TXT 등 다양한 레코드 공존 가능.
					- CNAME은 예외
						- ```
						    example.com.  IN  CNAME   myloadbalancer.elb.amazonaws.com.
							example.com.  IN  MX      mail.example.com.   ← X **규칙 위반*
						- **CNAME은 '이 이름은 다른 도메인 이름의 별명(alias)이다'** 라고 선언하는 레코드이다. 따라서 **그 이름 자체에 대해 다른 어떤 정보도 존재할 수 없다고 DNS 규격(RFC 1034, RFC 1912)에서 명시**하고 있다.
			- Zone Apex는 기본적으로 **NS 레코드, SOA 레코드** 등 필수 레코드를 포함해야 하므로, CNAME을 사용하면 **DNS 표준을 위반**하게 됨
		- Zone Apex의 경우 AWS의 Alias 레코드를 사용하여 CNAME 처럼 동작할 수 있다.
	- Example: `www.example.com`은 가능, `example.com`은 불가능.
- NS – Name Servers for the Hosted Zone
	- 트래픽 라우팅 제어

---
#### Route 53 – Hosted Zones

- **도메인 및 서브도메인 트래픽 라우팅 방식을 정의**하는 레코드의 컨테이너
- 두 가지 유형:
    - **Public Hosted Zone**: 인터넷에서 라우팅(`application1.mypublicdomain.com`)
    - **Private Hosted Zone**: VPC 내에서 라우팅 (`application1.company.internal`)
- 비용: 호스팅 존당 월 $0.50


#### Route 53 – Public vs. Private Hosted Zones

![](images/Pasted%20image%2020250518152931.png)

| 항목    | Public Hosted Zone | Private Hosted Zone       |
| ----- | ------------------ | ------------------------- |
| 대상    | 인터넷 상의 클라이언트       | VPC 내부 인스턴스               |
| 예시    | `www.example.com`  | `db.internal.example.com` |
| 연결 방식 | 공개 IP 또는 AWS 리소스   | VPC 내 사설 IP               |

---
#### Route 53 – RecordsTTL (TimeTo Live)

![500](images/Pasted%20image%2020250518152945.png)

- TTL은 DNS 결과를 **얼마나 오래 캐시할 것인지**를 정의
- **High TTL – e.g., 24 hr**:
    - Less traffic on Route 53
    - 레코드 변경 반영이 느림
- **Low TTL – e.g., 60 sec.**:
    - 빠른 반영 가능
    - More traffic on Route 53 → 비용 증가
- 모든 레코드에 TTL 필요 (**Alias 레코드 제외**)

---
#### CNAME vs Alias

- AWS Resources (Load Balancer, CloudFront...) expose an AWS hostname:
	- `lb1-1234.us-east-2.elb.amazonaws.com` and you want `myapp.mydomain.com`

- CNAME:  
	- Points a hostname to any other hostname. (app.mydomain.com => blabla.anything.com)
	- ONLY FOR NON ROOT DOMAIN (aka. something.mydomain.com)
- Alias:  
	- Points a hostname to an AWS Resource (app.mydomain.com => blabla.amazonaws.com)
	- Works for ROOT DOMAIN and NON ROOT DOMAIN (aka. mydomain.com)  
	- Free of charge  
	- Native health check

#### Route 53 – Alias Records

![300](images/Pasted%20image%2020250518164929.png)
- host name 을 AWS 리소스에 매핑하는 DNS 확장 기능
- 리소스의 IP 변경이 발생해도 자동으로 추적됨
- Zone Apex에도 사용 가능
- Alias Record is always of type A/AAAA for AWS resources (IPv4 / IPv6)
- TTL 설정 불가 (자동 처리)

#### Route 53 – Alias Records Targets

Alias 레코드는 다음 **AWS 리소스**에만 설정 가능:
- **Elastic Load Balancer**
- **CloudFront Distributions**
- **API Gateway**
- **Elastic Beanstalk environments**
- **S3 Websites**
- **VPC Interface Endpoints**
- **AWS Global Accelerator**
- **동일 호스팅 영역 내의 Route 53 레코드**
- **You cannot set an ALIAS record for an EC2 DNS name**