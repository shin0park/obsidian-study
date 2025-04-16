
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