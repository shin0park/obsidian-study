
### Domain Registar vs. DNS Service

- 도메인은 GoDaddy, Amazon Registrar 등 등록기관에서 구매함
- 등록기관은 기본적으로 DNS 관리 서비스도 제공하지만, Route 53 같은 별도 DNS 서비스 사용도 가능
- ex) 도메인은 GoDaddy에서 구매하고 DNS는 Amazon Route 53으로 설정

![](images/Pasted%20image%2020250608195402.png)
- 도메인은 GoDaddy에서 등록 → Route 53에서 Public Hosted Zone 생성하여 DNS 관리

#### 3rd Party Registrar with Amazon Route 53

- 3rd party registrar에서 도메인을 구매해도 Route 53을 DNS Service provider로 사용 가능
    1. Route 53에 Hosted Zone 생성
    2. 도메인 등록기관의 NS 레코드를 Route 53 NS로 변경

#### Making Route 53 the DNS service for a domain that is in use (users are accessing it)

1. 현재 DNS configuration 가져오기(복제할 레코드)
2. Route 53에 public hosted zone 생성
3. 새로 생성된 영역에 모든 레코드 생성
4. NS 레코드의 TTL 설정을 15분으로 낮춤 (롤백 가능하도록)
5. 새 NS 레코드 TTL이 전파될 때까지 2일 대기 (default 설정이 2일)
6. NS 레코드를 Route 53 name server 로 업데이트 
7. 도메인 트래픽 모니터링
8. Route 53에서 NS 레코드 TTL을 더 높은 값(2일)으로 변경
9. (선택사항) domain registration을 Amazon Route 53으로 이전

---

### Route 53 Scenarios

- EC2 (IP 연결): `example.com → 54.55.56.57 (A레코드)`
  
- EC2 (DNS 이름): `app.example.com → ec2-xx.compute.amazonaws.com (CNAME)`
	- ec2는 alias 미지원
  
- ALB 연결: 도메인을 AWS provided ALB DNS name에 Alias 설정
	- `example.com` => `my-load-balancer-1234567890.us-west-2.elb.amazonaws.com` (Alias) 
		- apex type일때 alias 사용. (apex일 경우 CNAME 사용 불가)
	- `lb.example.com` => `my-load-balancer-1234567890.us-west-2.elb.amazonaws.com` (Alias or CNAME)
		- apex type이 아닐때, Alias or CNAME 사용
	
- CloudFront Distribution: 콘텐츠 경로에 따라 ELB, S3로 라우팅
	- ![](images/Pasted%20image%2020250608200621.png)
	  
- API Gateway: API Gateway Regional/Edge Optimized DNS name에 Alias
	- `example.com` => `b123abcde4.execute-api.us-west-2.amazonaws.com` (Alias)
    
- RDS: `db.example.com → RDS DB instance DNS name (CNAME만 가능)`
	- `db.example.com` => `myexampledb.a1b2c3d4wxyz.us-west-2.rds.amazonaws.com` (CNAME)
		- alias 미지원
    
- S3 Bucket: `example.com → S3 website endpoint (Alias)`
	- You must create an Alias record for S3 endpoints 
	- **Bucket name must be the same as domain name** (매우 중요)
	- ex) `example.com` => `s3-website-us-west-2.amazonaws.com` (Alias)
		- 그렇다면, s3 버킷명이 `example.com`여야 하는가. -> yes
			- https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/bucketnamingrules.html (s3 범용 버킷 이름 지정 규칙)
			- ![](images/Pasted%20image%2020250608204343.png)
			- static web site 이외의 용도에는 권장하지 않는다.
    
- PrivateLink (VPC Interface Endpoint): 도메인을 VPC Interface Endpoint (AWS PrivateLink)에 Alias 연결
	- `example.com` => `vpce-1234-abcdev-us-east-1.vpce-svc-123345.us-east-1.vpce.amazonaws.com` (Alias)
	- ![](images/Pasted%20image%2020250608201355.png)

---
### Route 53 – Hosted Zones

![](images/Pasted%20image%2020250608201548.png)

- Route 53은 Hosted Zone 생성 시 NS와 SOA 자동 생성
- 동일 네임스페이스가 겹치는 Public/Private Zone은 가장 구체적인 경로로 트래픽 전달

---
### Route 53 – Routing Traffic For Subdomains

![400](images/Pasted%20image%2020250608205253.png)

- Create a Hosted Zone for the Subdomain
- Known as, either:  
	- 서브도메인을 Hosted zone, 즉 다른 name server에 위임한다.
	- “Delegation Responsibility for a Subdomain to a Hosted Zone” 
	- “Delegating a Subdomain to Another Name Servers”
    
- 사용 사례:
    - 다른 팀이 관리하는 다른 서브도메인
    - IAM 권한을 사용하여 액세스 제한 (Route 53 레코드에 대한 액세스 제어에는 IAM을 사용할 수 없음)
      
- Using Route 53 as the DNS Service for a Subdomain without Migrating the Parent Domain
	- ![](images/Pasted%20image%2020250608205608.png)

---
### DNS Poisoning (Spoofing)

![](images/Pasted%20image%2020250608205636.png)

- DNS는 UDP 기반 → 보안에 취약 
- 암호화된 인증 절차가 없음

#### Route 53 – DNS Security Extensions (DNSSEC)

- DNS 트래픽을 보호하기 위한 프로토콜, DNS 데이터 무결성 및 origin 검증
- Public Hosted Zone 에서만 작동
- Route 53은 Domain Registration 및 Signing 모두에 대해 DNSSEC 지원
- DNSSEC Signing
	- DNS 응답이 실제 Route 53에서 생성된 것이며, **변조되지 않았음을 검증**
	- Route 53은 Hosted Zone 내 **각 레코드에 대해 암호화된 서명**을 생성
	- 두 가지 키:
	    - 사용자가 관리: Key-signing Key (KSK) 
		    - AWS KMS(키 관리 서비스)의 비대칭 CMK(**Customer managed key**)를 기반으로 함
		    - https://docs.aws.amazon.com/ko_kr/kms/latest/cryptographic-details/basic-concepts.html (aws kms 기본개념)
		    - CMK 용어를 KMS key 로 대체
	    - AWS가 관리: Zone-signing Key (ZSK)
		    - https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/dns-configuring-dnssec-zsk-management.html (Route 53에서의 KMS 키 및 ZSK 관리)
        
- DNSSEC가 활성화되면, Route 53은 **Hosted Zone 내 모든 레코드에 대해 최소 1주일의 TTL(Time To Live)**을 강제 적용한다.
	- 새로 생성되는 서명에 대해 최소 TTL을 1주로 강제
		- TTL이 너무 짧으면 **캐시가 자주 무효화되고**, 리졸버들이 자주 DNSSEC 검증을 반복하게 되어 **서버 부하가 증가**하기 때문.
	- **이미 TTL이 1주보다 짧은 레코드는 DNSSEC 서명의 TTL 제한 대상이 아님.** (즉, 기존 TTL 그대로 유지됨)
	
#### Route 53 – Enable DNSSEC on a hosted zone

- Step 1: DNSSEC 서명을 위한 준비
	- **DNS 영역(Zone)의 가용성**을 확인
	    → 고객 피드백 등을 통해 장애 가능성 모니터링
	- Lower TTL for records (recommended 1 hour)
	- Lower SOA(권한시작) minimum for 5 minutes
		- DNS '권한 시작'(SOA) 레코드는 관리자의 이메일 주소, 도메인이 마지막으로 업데이트된 시간, 새로 고침 사이에 서버가 대기해야 하는 시간 등 도메인 또는 영역에 대한 중요한 정보를 저장
		  
- Step 2: DNSSEC 서명 기능 활성화 및 KSK 생성
	- Route 53 콘솔 또는 CLI를 통해 **DNSSEC를 활성화**
	- **KSK(Key Signing Key)** 를 Route 53 콘솔에서 생성하고,  
	    → 고객이 관리하는 AWS KMS의 CMK(비대칭 키)와 연결
    
- Step 3: 신뢰 체인(chain of trust) 구축
	- **hosted zone과 상위 존(Parent Zone)** 사이에 **신뢰 체인을 설정**
	- 이를 위해, **Delegation Signer (DS) 레코드**를 **부모 존에 생성**한다.
	    - 이 DS 레코드는 **DNS 레코드를 서명하는 공개키의 해시값**을 포함하고 있어 신뢰 검증 가능
	- 도메인 등록기관은 AWS Route 53 or **타사 등록기관**도 사용 가능
    
- Step 4: (선택 사항) 오류 감지를 위한 CloudWatch Alarms 모니터링
	- 다음과 같은 이벤트에 대해 **CloudWatch 알람을 생성**하면 오류 탐지에 유용하다.
	    - `DNSSECInternalFailure`
	    - `DNSSECKeySigningKeysNeedingAction`

#### DNSSEC – Chain of Trust

![](images/Pasted%20image%2020250608215733.png)


---
### Route 53 – Hybrid DNS

- VPC 내 Route 53 Resolver가 내부/외부 도메인을 동시에 질의 응답
	- Local domain names for EC2 instances
	- Records in Private Hosted Zones
	- Records in public Name Servers
	  
- 온프레미스 네트워크와 AWS VPC 간 DNS 질의 가능
	- VPC itself / Peered VPC
	- On-premises Network (connected through Direct Connect or AWS VPN)
  
- ![400](images/Pasted%20image%2020250608215901.png)

---
### Route 53 – Resolver Endpoints

- #### Inbound Endpoint
	- 온프레미스 DNS Resolver가 AWS의 Route 53 Resolver로 DNS 요청 전달
	- EC2 인스턴스 또는 Private Hosted Zone 내 리소스를 외부에서 질의 가능

- #### Outbound Endpoint
	- Route 53 Resolver가 조건에 따라 외부 DNS Resolver로 질의 포워딩
	- Resolver Rules를 사용해 DNS 요청을 목적지 Resolver로 전달

- Associated with one or more VPCs in the same AWS Region  
- 고가용성을 위해 two AZs 에 배치 가능
- Each Endpoint IP당 초당 10,000개 쿼리 처리 가능

---

#### Route 53 – Resolver Inbound Endpoints

![](images/Pasted%20image%2020250608220811.png)
- 온프레미스 DNS → VPC 내 Resolver Inbound Endpoint로 질의 전송

#### Route 53 – Resolver Outbound Endpoints

![](images/Pasted%20image%2020250608220821.png)
- AWS VPC 내부 → 온프레미스 DNS 서버로 질의 전달

#### Route 53 – Resolver Rules

- 특정 DNS 요청을 포워딩할 대상을 정함
  
- #### Conditional Forwarding Rules (Forwarding Rules)
	- 지정된 도메인 및 모든 하위 도메인에 대한 DNS 쿼리를 타켓 IP 주소로 전달
	- ex) `acme.example.com` → 특정 타켓 IP 로 forward
    
- #### System Rules
    - 포워딩 규칙의 동작을 **선택적으로 재정의** 가능
    - ex) `acme.example.com` 하위 도메인은 포워딩에서 제외
      
- #### Auto-defined System Rules
	- AWS가 사전에 정의한 규칙으로, **특정 도메인의 질의가 어떻게 처리될지 자동으로 결정**
	- ex) AWS 내부 도메인(`ec2.internal`, `compute.amazonaws.com`), Private Hosted Zone 등에 적용
        
- **최적합(Most Specific Match) 원칙** 적용
	- **여러 규칙이 일치할 경우**, Route 53 Resolver는 **가장 구체적으로 일치하는 규칙**을 우선 적용
    
- 규칙 공유 및 중앙 관리
	- Resolver Rule은 **AWS RAM(Resource Access Manager)**을 통해 여러 계정 및 VPC 간에 공유 가능
	- 중앙 계정에서 규칙 집중 관리 가능
		- **하나의 중앙 계정에서 규칙을 생성·관리**하고, 여러 VPC에서 해당 규칙을 사용 가능
	- 규칙에 정의된 **타겟 IP 주소로 여러 VPC에서 DNS 요청을 전송**할 수 있음

![400](images/Pasted%20image%2020250608220848.png)

---

#### Route 53 – DNS Query Logging

- #### public DNS 쿼리 로깅
	- Route 53 Resolver가 수신하는 public DNS 쿼리 정보 기록
	- **Only for Public Hosted Zones
	- **로그 전송 옵션**:
	    - CloudWatch Logs로 전송 (기본)
	    - 필요 시 S3로 추가 내보내기(Export) 가능
	- ![](images/Pasted%20image%2020250608224053.png)
    
- #### VPC 내부 DNS 쿼리 로깅
	
	- VPC 내 모든 리소스의 DNS 쿼리 기록
	- **포함 대상**:
	    - 프라이빗 호스팅 영역(Private Hosted Zones)
	    - 리졸버 인바운드/아웃바운드 엔드포인트
	    - DNS 방화벽(Resolver DNS Firewall) 활동
	        
	- **로그 저장 옵션**:
	    - CloudWatch Logs (실시간 모니터링)
	    - S3 버킷 (장기 보관)
	    - Kinesis Data Firehose (스트림 처리)

#### **계정 간 공유**:
- AWS RAM(Resource Access Manager)을 통해 로깅 설정 공유
- 중앙 관리 계정에서 다중 계정 로그 집계 가능

#### 주요 특징
- **보안 감사**: 의심스러운 DNS 요청 추적
- **트러블슈팅**: DNS 확인 문제 진단 지원
    
- **규정 준수**: 감사 요구사항 충족