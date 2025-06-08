
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
		    - AWS KMS(키 관리 서비스)의 비대칭 CMK(고객 마스터 키)를 기반으로 함
		    - https://docs.aws.amazon.com/ko_kr/kms/latest/cryptographic-details/basic-concepts.html (aws kms 기본개념)
	    - AWS가 관리: Zone-signing Key (ZSK)
        
- 활성화하면 Route 53은 호스팅 영역의 모든 레코드에 대해 1주의 TTL을 적용

#### Route 53 – Enable DNSSEC on a hosted zone