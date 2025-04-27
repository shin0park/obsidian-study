
### What is load balancing?

로드 밸런서는 여러 downstream 서버(예: EC2 인스턴스)로 트래픽을 forward 하는 서버이다 
이를 통해 단일 서버에 트래픽이 몰리는 것을 방지하고, 애플리케이션의 가용성과 확장성을 확보할 수 있다.

---
### Why use a load balancer?

- 여러 다운스트림 인스턴스 간 부하 분산
- 애플리케이션에 대한 a single point of access 단일지점(DNS) 제공    
- 다운스트림 인스턴스 장애를 원활하게 처리 (장애시 정상 인스턴스로 트래픽 재분배)
- 인스턴스 상태를 주기적으로 health checks 
- 웹사이트에 대한 SSL termination (HTTPS) 제공
- 쿠키를 통한 고정성(stickiness) 적용
- zone 간 고가용성(HA) 제공
- public 트래픽과 private 트래픽 분리
---
### Why use an Elastic Load Balancer?

- ELB는 **managed load balancer**이다.
	- AWS가 로드 밸런서의 작동을 보장하며, 업그레이드, 유지보수, 고가용성을 관리
	- 간단한 설정만으로 사용 가능
- **비용 효율성**: 자체 로드 밸런서를 구축하는 것보다 비용이 적으며, 관리 부담 감소
- **AWS 서비스 통합**
	- EC2, EC2 Auto Scaling Groups, Amazon ECS와 통합
	- AWS Certificate Manager(ACM), CloudWatch
	- Route 53, AWS WAF, AWS Global Accelerator와 연동

---
### Health Checks
![](images/Pasted%20image%2020250416230537.png)
- 로드 밸런서가 트래픽을 전달할 인스턴스의 가용성을 확인
- 로드 밸런서는 인스턴스가 정상인지 확인하기 위해 주기적으로 헬스 체크를 수행
- 지정된 port와 path(/health 등)로 확인하며, 응답이 200(OK)이 아니면 비정상(unhealthy)으로 판단
---
### Types of load balancer on AWS

- **Classic Load Balancer (CLB, 2009)** - 구세대
    - Layer 4 및 7에서 작동
    - 지원 프로토콜: HTTP, HTTPS, TCP, SSLsecureTCP)
- **Application Load Balancer (ALB, 2016)** - 신세대
    - Layer 7에서 작동
    - 지원 프로토콜: HTTP, HTTPS, WebSocket
- **Network Load Balancer (NLB, 2017)** - 신세대
    - Layer 4에서 작동
    - 지원 프로토콜: TCP, TLS(secureTCP), UDP
- **Gateway Load Balancer (GWLB, 2020)**
    - Layer 3 (네트워크 계층)에서 작동
    - 지원 프로토콜: IP - Protocol

- **권장 사항**: 신세대(ALB, NLB, GWLB)가 더 많은 기능을 제공하므로 사용 권장
- 일부 로드 밸런서는 - internal (private) or external (public) ELBs 로 설정 가능

---
### Load Balancer Security Groups
![](images/Pasted%20image%2020250416231105.png)
- 로드 밸런서 보안 그룹: HTTP/HTTPS 트래픽을 모든 곳(0.0.0.0/0)에서 허용 (port 80,443)
- 애플리케이션 보안 그룹: 로드 밸런서로부터 오는 트래픽만 허용 -> 더 높은 보안 설정 가능

---
### Classic Load Balancer

- Layer 4 및 7에서 작동
- 지원 프로토콜: HTTP, HTTPS, TCP, SSL/TLS
- EC2 인스턴스를 CLB에 직접 등록 (no Target Groups)
- Health Checks: HTTP, HTTPS, TCP 지원
- EC2-Classic networks 지원

![](images/Pasted%20image%2020250416233323.png)

---
### Classic Load Balancer – SSL considerations
![](images/Pasted%20image%2020250416233809.png)
- **Backend Authentication**
    - CLB와 EC2 인스턴스 간 HTTPS/SSL 활성화 시 "백엔드 인증" 설정
    - EC2 인스턴스에 인증서 설치 필요.
- TCP => TCP passes all the traffic to the EC2 instance (no termination)
    - TCP 리스너는 트래픽을 종료하지 않고 EC2로 전달
	    - 트래픽을 암호화 상태 그대로 EC2로 전달
    - Mutual SSL 인증(2-way Mutual SSL)을 지원하는 유일한 방식
---
### Application Load Balancer

![400](images/Pasted%20image%2020250416235011.png)

- Operates at Layer 7
- 지원 프로토콜: **HTTP, HTTPS, WebSocket, HTTP/2, gRPC**
- 다양한 HTTP 기반 애플리케이션으로의 로드 밸런싱
    - 여러 인스턴스
    - 단일 인스턴스의 다양한 port (ex: 컨테이너)
- cutom HTTP responses 지원
- **HTTP redirect 지원
	- HTTP → HTTPS 자동 리디렉션


- **Target Groups**
    - EC2 Instances (can be managed by an ASG(auto scaling group)) – HTTP
    - ECS Tasks (managed by ECS itself) – HTTP
    - Lambda functions – (HTTP 요청 → JSON 이벤트로 변환)
    - IP Addresses - private IP 만 가능 
	    - (e.g., EC2 instances in peered VPC, on-premises servers accessed over AWS Direct Connect or VPN connection)

- **Supports Weighted Target Groups**
	- 가중치 기반 타겟 그룹(블루/그린 배포 지원)

- Health Checks can be HTTP or HTTPS (WebSocket is not supported)
- 서브넷당 최소 /27 크기 및 **8개의 여유 IP 필요**
- Across all subnets, ALB 전체에서 최대 100개의 IP 사용 가능
- **Routing to different target groups**
    - Routing based on **URL Path**
	    - ![](images/Pasted%20image%2020250417000226.png)
	    - `example.com/users, example.com/posts`
    - Routing based on **Hostname**
	    - `one.example.com,other.example.com`
	- Routing based on Query String, HTTP Headers, Source IP Address 
		- ![](images/Pasted%20image%2020250417000235.png)
		- `example.com/users?id=123&order=false`
        
- 마이크로서비스 및 컨테이너 기반 애플리케이션에 적합하다
	- (e.g., Docker & Amazon ECS)
- port mapping을 통해 유연한 서비스 제공 가능



- **인증 기능**
	- 라우팅 전에 authenticate users 기능 제공
	    - Amazon Cognito User Pools and Identity Providers
	    - Microsoft Active Directory, OIDC, SAML, LDAP, OpenID
	    - Social Identity Providers such as Amazon, Facebook, Google

- TLS Certificates (multiple listeners & SNI)
	- ![](images/Pasted%20image%2020250417000435.png)

---
#### ALB Listener Rules

![300](images/Pasted%20image%2020250417000655.png)

- 우선순위대로 동작 (last is Default Rule)
- Supported Actions (forward, redirect, fixed-response)
- Rule Conditions
	- host-header
	- http-request-method
	- path-pattern  
	- source-ip
	- http-header
	- query-string
---
#### Target Group Weighting

- 단일 리스너 Rule 내에서 각 타겟 그룹에 **가중치 설정**
- 트래픽을 비율에 따라 나눌 수 있음
	- ![](images/Pasted%20image%2020250417000826.png)
> Blue/Green 배포 전략, 카나리 릴리스 등에 적합

---
### Network Load Balancer

- Operates at Layer 4
- 지원 프로토콜: **TCP, UDP, TLS**
- 초당 **수백만 건**의 요청 처리 가능
- **Static IP** 제공 (AZ별 1개) 및 **Elastic IP 할당 가능 (Internet-facing NLB)**
    - 화이트리스트 설정에 유리
- 컨테이너 및 ECS 환경에서 port 단위 분산 처리 가능
- **지연 시간 매우 낮음 (~100ms)** → ALB(~400ms)보다 빠름
- WebSocket 지원
- 주 용도:
    - 초고성능 TCP/UDP 트래픽 처리
    - **AWS PrivateLink** 와의 연동 - privately expose an internal service

- **Target Groups**
    - EC2 Instances - can be managed by an ASG
    - ECS Tasks - managed by ECS itself
    - IPAddresses
        - 사설 IP만 가능
        - Inter-region VPC peering도 가능
        - **다른 VPC의 EC2 인스턴스는 ID로 등록 불가**, IP 주소로만 가능
            
- **Health Checks**
    - 지원 프로토콜: **HTTP, HTTPS, TCP**
    - **Active Health Check**: 주기적으로 타겟에 요청 전송
    - **Passive Health Check**: 타겟의 응답을 관찰, unhealthy targets 조기 감지 (비활성화/수정 불가)

---
#### Network Load Balancer TCP & UDP - based Traffic
![](images/Pasted%20image%2020250417002816.png)

---
#### Client IP Preservation

- client IP is forwarded to targets
    - instance ID / ECS Tasks: 보존 활성화.
    - IP address TCP & TLS: 기본 **비활성화**, 
	    - 비활성화인 경우 Proxy Protocol v2 사용 가능 (헤더에 정보 추가)
    - IP address UDP & TCP_UDP: 기본 활성화

- 리스너 및 타겟 모두 다양한 프로토콜 조합 가능
    - TCP → TCP
    - UDP → UDP
    - TCP_UDP → TCP_UDP
    - TLS → TLS (SSL 종료)
    - ![](images/Pasted%20image%2020250417003142.png)
    - 리스너에 인증서를 설치하여 TLS 종료 가능
    - 주의: **UDP는 SSL 종료 지원 안 함**

---
#### Network Load Balancer – Availability Zones

![](images/Pasted%20image%2020250417003328.png)

- AZ를 **명시적으로 활성화해야** 트래픽 전달 가능
- 생성 이후에는 **비활성화 불가**
- **Cross-Zone 로드 밸런싱**은 활성화된 AZ 간에서만 동작
---
#### Network Load Balancer – Zonal DNS Name

![400](images/Pasted%20image%2020250417003344.png)

- Regional DNS: 모든 AZ의 IP 반환 
	- ex. `my-nlb-123456.elb.us-east-1.amazonaws.com`
    
- Zonal DNS: 특정 AZ의 NLB 노드 IP 확인 가능  
	- ex. `us-east-1a.my-nlb-123456.elb.us-east-1.amazonaws.com`  
	- 지연 시간 최소화 및 비용 절감
    
- You need to implement app specific logic
	- **애플리케이션이 스스로 클라이언트 위치를 판단하고 Zonal DNS를 선택하는 로직**을 구현해야 사용 가능하다.

---
#### Network Load Balancer – Good To Know

- 한 번 활성화한 AZ는 제거 불가
- 각 AZ 별로 NLB를 위해 생성된 ENI는 수정 불가 (view only)
- ENI에 할당된 IP(EIP 포함) 변경 불가
	- lb를 다시 만들지 않는 한
- 최대 44만 connections/minute per target 지원
    - 초과 시 port allocation errors 발생 → 해결 방법: 타겟 수 늘리기
        
- For internet-facing load balancers, 서브넷에 최소 **8개의 사용 가능한 IP 주소 필요**  
	- For internal load balancers, AWS가 서브넷에서 private ipv4 address를 할당하는 경우에 해당 
---
### Connection Idle Timeout

![300](images/Pasted%20image%2020250417004138.png)

- **Idle Timeout period for ELB’s connections**
	- **ELB와 클라이언트 또는 타겟 간 연결에서** 일정 시간 동안 데이터가 주고 받지 않으면, 해당 연결은 **자동으로 종료**됨
	- 이 유휴 제한 시간 내에 **1바이트라도 전송되면 연결 유지**
- Supported for CLB,ALB,and NLB
	- CLB & ALB: 기본 60초(설정 가능)
	- NLB: TCP 350초, UDP 120초(설정 불가)
	- 파일 업로드 시 시간 초과 방지에 유용
- EC2 인스턴스의 웹 서버 설정에서 HTTP keep-alive 활성화하여 백엔드 연결 재사용하는 것을 권장
	- Idle Timeout 설정은 연결 안정성을 유지하며, keep-alive는 성능 최적화에 기여

---
===
### Request Routing Algorithms

- Least Outstanding Requests
	- 미완료(보류 중인) 요청 수가 가장 적은 인스턴스를 선택하여 다음 요청을 라우팅
	- **지원 로드 밸런서**:
	    - Application Load Balancer (ALB)
	    - Classic Load Balancer (CLB, HTTP/HTTPS 리스너)
- round Robin
	- 타겟 그룹 내 타겟에 요청을 순차적으로 균등 분배
	- **지원 로드 밸런서**:
	    - Application Load Balancer (ALB)
	    - Classic Load Balancer (CLB, TCP 리스너)
- Flow Hash
	- protocol, source/destination IP address, source/destination port, and TCP sequence number를 기반으로 타겟 선택
	- 각 TCP/UDP 연결은 연결 지속 시간 동안 단일 타겟으로 라우팅
	- **지원 로드 밸런서**:
	    - Network Load Balancer (NLB)
	- ![](images/Pasted%20image%2020250421225541.png)

---

### Sticky Sessions (Session Affinity)

- **동일한 클라이언트가 항상 동일한 백엔드 인스턴스로 지속적으로 라우팅되도록** 설정
- 세션 데이터 유지를 위한 용도 (예: 로그인 상태)
- ALB, CLB, NLB에서 모두 지원됨
- CLB 및 ALB: 쿠키를 사용하며, 쿠키 만료 날짜 설정 가능
- 단점: 백엔드 인스턴스 간 부하 불균형 초래할 수 있음
- 스티키 세션은 사용자 경험을 향상시키지만, 부하 균형에 영향을 줄 수 있음

#### Sticky Sessions – Cookie Names

- Application-based Cookies
	- Custom cookie
		- 타겟(애플리케이션)에서 생성
		- 애플리케이션 요구에 맞는 사용자 지정 속성 포함 가능
		- 타겟 그룹별로 쿠키 이름 개별 지정해야한다.
		- 예약 이름(AWSALB, AWSALBAPP, AWSALBTG) 사용 불가
	- Application cookie
		- 로드 밸런서에서 생성
		- 쿠키 이름: AWSALBAPP
		- 모든 계층에서 스티키 세션 필요한 경우 사용
- Duration-based Cookies
	- 로드 밸런서에서 생성
	- Cookie name: ALB는 AWSALB, CLB는 AWSELB

---

### Cross-Zone Load Balancing

![](images/Pasted%20image%2020250421230621.png)
-> 여러 가용 영역(AZ)에 걸쳐 트래픽을 **균등 분산**
- **비활성화 시** → 로드 밸런서 노드가 속한 AZ 내 인스턴스만 사용
- **활성화 시** → 모든 AZ에 등록된 인스턴스들에 트래픽 분산

|로드 밸런서 유형|기본 설정|비용 여부|
|---|---|---|
|ALB|활성화|❌ (AZ 간 요금 없음)|
|NLB / GWLB|비활성화|✅ (AZ 간 트래픽 요금 발생)|
|CLB|비활성화|❌ (요금 없음)|

-> 교차 영역 로드 밸런싱은 고가용성과 부하 분산을 향상시키지만, NLB/GWLB는 비용 고려 필요

---
### Elastic Load Balancer – SSL Certificates

#### SSL/TLS - Basics

- SSL(TLS) Certificate은 **클라이언트와 로드 밸런서 간 통신을 암호화** 
	- (in-flight encryption 전송중 암호화)
- TLS는 SSL의 최신 버전으로, 현재 주로 사용
- 대부분 TLS를 사용하지만 “SSL”이라는 용어가 아직도 자주 사용됨
- Public SSL 인증서는 인증기관(CA)에서 발급 
	- (Comodo, Symantec, GoDaddy, GlobalSign, DigiCert, Let’s Encrypt...)
- 인증서는 만료 날짜가 있으며, 갱신 필요
---
#### Elastic Load Balancer – SSL Certificates

![](images/Pasted%20image%2020250421233239.png)

- **X.509 certificate** 사용 (SSL/TLS server certificate)
- ACM (AWS Certificate Manager)로 관리 가능
- 직접 업로드한 인증서도 사용 가능
    
- **HTTPS listener**:
    - default certificate 지정해야함
    - 여러 도메인 지원을 위한 인증서 목록 추가 가능
    - **SNI(Server Name Indication)** 사용 시 다중 도메인 지원
    - **보안 정책(Security Policy)** 설정 가능 (컴플라이언스, 호환성 등)

---
#### SSL – Server Name Indication (SNI)

![400](images/Pasted%20image%2020250421233647.png)

- SNI는 단일 웹 서버에서 다중 SSL 인증서를 로드하여 여러 웹사이트를 제공하는 문제를 해결
- 클라이언트가 첫 SSL 핸드셰이크 중 **접속하려는 타겟 호스트 이름을 전달**
- 서버는 해당 호스트에 맞는 인증서를 응답
- 지원 대상: **ALB, NLB**
- CLB는 **SNI 미지원 → 호스트당 별도 CLB 필요**
- SNI는 다중 도메인 환경에서 인증서 관리를 간소화

#### Elastic Load Balancers – SSL Certificates

- CLB
	- **단일 SSL 인증서만 지원**
	- - SSL 인증서에는 여러 도메인을 위한 다중 SAN(Subject Alternate Name) 포함 가능
	    ➝ ex: `example.com`, `api.example.com`, `www.example.com` 
	- but, SAN이 **추가/수정/삭제될 때마다 인증서 자체를 교체**해야 함
	- **여러 도메인에 각각 다른 SSL 인증서**를 사용하려면 **CLB를 여러 개** 생성해야 함
		- -> 이러한 제약 때문에, **가능하면 ALB를 사용하는 것이 바람직**
- ALB
	- **다중 리스너 및 다중 SSL 인증서** 지원
	- **SNI (Server Name Indication)** 을 사용하여 **도메인 별로 다른 인증서** 매핑 가능
		- -> 하나의 ALB로 여러 도메인에 SSL 서비스를 제공할 수 있음 
			- (ex: `app.example.com`, `api.example.com`)
- NLB
	- ALB와 마찬가지로 **다중 리스너 및 다중 SSL 인증서** 지원
	- **SNI** 기반으로 클라이언트가 요청한 호스트명에 맞는 인증서 제공
		- -> 고성능이 필요한 L4 기반 암호화 트래픽 처리에 적합

---
#### HTTPS/SSL Listener – Security Policy

Application Load Balancer 보안 정책
https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/application/describe-ssl-policies.html

- SSL 프로토콜, 암호화 방식(SSL Cipher), 서버 우선순위 옵션을 결합한 정책으로 SSL 협상 시 사용
- Predefined Security Policies 사용 가능 
	- `ELBSecurityPolicy-TLS13-1-2-2021-06` 정책은 AWS Management Console을 사용하여 생성한 리스너의 기본 보안 정책
	- `ELBSecurityPolicy-2016-08` 정책은 AWS CLI를 사용하여 생성한 리스너의 기본 보안 정책
- HTTPS 리스너를 생성할 때 보안 정책을 선택해야 한다.

- For ALB and NLB
	- ALB는 사용자 지정 보안 정책을 지원하지 않는다.

	- **프론트엔드 연결**: Predefined Security Policies - 사전 정의된 보안 정책 사용 가능
		- (ex: `ELBSecurityPolicy-TLS-1-2-2017-01`, `ELBSecurityPolicy-FS`...)
	- 백엔드 연결: 
		- HTTPS 리스너 중 하나라도 TLS 1.3 보안 정책을 사용하면 `ELBSecurityPolicy-TLS13-1-0-2021-06` 보안 정책이 사용됨
			- 그러지 않으면, `ELBSecurityPolicy-2016-08` 보안 정책은 백엔드 연결에 사용됨
			- **이유**: `ELBSecurityPolicy-2016-08`은 TLS 1.0 이상(TLS 1.0, 1.1, 1.2)을 지원하며, 다양한 백엔드 타겟(EC2 인스턴스, 컨테이너, 온프레미스 서버 등)과의 호환성을 보장하는 정책이다. 이는 운영 간소화와 안정성을 위해 TLS 1.3이 아니라면 이 정책으로 사용된다.
		- **FIPS 보안 정책 사용 시**: 백엔드 연결에 `ELBSecurityPolicy-TLS13-1-0-FIPS-2023-04` 사용
			- ```Federal Information Processing Standard(FIPS)는 미국 및 캐나다 정부 보안 표준으로서, 기밀 정보를 보호하는 암호 모듈의 보안 요건을 규정하고 있습니다.```
			- AWS 클라우드 보안 규정 준수 페이지 [Federal Information Processing Standard(FIPS) 140](https://aws.amazon.com/compliance/fips/)

- Use **ELBSecurityPolicy-TLS**: 
	- 특정 TLS 프로토콜 버전을 비활성화해야 하는 규정 준수 및 보안 표준을 충족하거나 더 이상 사용되지 않는 암호가 필요한 레거시 클라이언트를 지원하려면 `ELBSecurityPolicy-TLS-` 보안 정책 중 하나를 사용할 수 있다.
- Use **ELBSecurityPolicy-FS**: FS 지원 정책
	- Forward Secrecy 제공
	- FS(순방향 비밀성) 지원 보안 정책: 고유한 랜덤 세션 키를 사용하여 암호화된 데이터를 도청할 수 없도록 추가적인 보호 기능을 제공

- 보안 정책은 보안 및 규제 요구사항을 충족하며, FS 정책은 추가 보안 제공


---
**출제 가능성 높음**
#### Connection Draining / Deregistration Delay 

연결 종료 처리
- 사용자 경험을 유지하며, 요청 손실 방지

- **CLB**: Connection Draining
- **ALB/NLB**: Deregistration Delay
    
- 인스턴스가 등록 해제되거나 비정상 상태일 때 진행 중인 요청("in-flight requests")을 완료할 시간 제공
- **신규 요청은 다른 인스턴스로 전달됨**
    
- 설정 범위: **1~3600초 (기본값 300초)**
	- 비활성화 가능(0으로 설정).
	- 짧은 요청의 경우 낮은 값 설정 권장

![400](images/Pasted%20image%2020250422002415.png)

---
#### X-Forwarded Headers (HTTP)

![300](images/Pasted%20image%2020250422002432.png)

- ELB가 클라이언트 정보를 타겟에 전달하기 위해 사용하는 비표준 HTTP 헤더(X-Forwarded 접두사)
- **지원 로드 밸런서**
    - Classic Load Balancer (CLB, HTTP/HTTPS)
    - Application Load Balancer (ALB)
- **헤더 종류**
    - **X-Forwarded-For**: 클라이언트 IP 주소 포함(프록시 경유 시 다중 IP 주소 포함, 왼쪽 첫 번째가 클라이언트 IP)
    - **X-Forwarded-Proto**: 클라이언트와 ELB 간 프로토콜(HTTP/HTTPS)π
    - **X-Forwarded-Port**: 클라이언트가 ELB에 연결한 대상 포트
- **사용 사례**: 서버에서 클라이언트 요청 로깅

---
### Proxy Protocol

![300](images/Pasted%20image%2020250428000405.png)

- **Proxy Protocol**은 source(클라이언트)에서 destination(서버)으로 연결 요청 시, 원래의 IP 주소와 포트 정보를 전달하는 인터넷 프로토콜이다.
- 로드 밸런서가 연결을 종료(Terminate)할 경우, **클라이언트의 실제 IP를 유지할 수 없기 때문에** Proxy Protocol을 사용해 보완한다.
	- 즉, Proxy Protocol은 소스/목적지 IP 주소와 포트 번호를 전달하기 위해 사용된다.
- 로드 밸런서가 TCP Data 에 **Proxy Protocol 헤더**를 추가하여 전달
- 지원 대상:
	- **Classic Load Balancer (TCP/SSL 리스너)**
		- Proxy Protocol 버전 1 사용
	- **Network Load Balancer**
		- Proxy Protocol 버전 2 사용
- For Network Load Balancer
	- **타겟이 인스턴스 ID나 ECS 태스크일 경우**: 클라이언트 IP가 자동으로 보존된다.
	- **타겟이 IP 주소일 경우**:
	    - **TCP/TLS**: 클라이언트 IP 보존되지 않음 → **Proxy Protocol 활성화 필요**
	    - **UDP/TCP_UDP**: 클라이언트 IP 보존 가능
- *주의사항*
	- 로드 밸런서가 **또 다른 프록시 서버 뒤에 배치되면 안 됨** → 중복 정보로 인해 오류 발생 가능
	- **Application Load Balancer(ALB)**는 Proxy Protocol이 필요 없음  
	    (ALB는 자체적으로 **X-Forwarded-For** HTTP 헤더를 삽입)

#### hands-on

target 을 ec2의 ip 주소로 지정한 NLB 생성.
질의한 결과 client Ip가 아닌 NLB ip가 출력된 것을 확인 할 수 있음.
client 로 부터 출발된 트래픽인데, 출력된 것으로는 aws 내부에서 트래픽이 시작된 것 처럼 보임
따라서 proxy protocol 를 활성화 할 필요 있음
->
Traffic configuration > check Proxy protocol v2 and Preserve client IP address

---
### gRPC & ALB


---
### Hybrid Connectivity


---
### Security Groups Outbound Rules & Managed Prefixes


---
