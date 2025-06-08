
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
  
- ALB 연결: 도메인을 AWS provided ALB DNS name에 Alias 설정
	• `example.com` => `my-load-balancer-1234567890.us-west-2.elb.amazonaws.com` (Alias)  
	• `lb.example.com` => `my-load-balancer-1234567890.us-west-2.elb.amazonaws.com` (Alias or CNAME)
	
- CloudFront Distribution: 콘텐츠 경로에 따라 ELB, S3로 라우팅
	- ![](images/Pasted%20image%2020250608200621.png)
	  
- API Gateway: API Gateway Regional/Edge Optimized DNS name에 Alias
	- `example.com` => `b123abcde4.execute-api.us-west-2.amazonaws.com` (Alias)
    
- RDS: `db.example.com → RDS DB instance DNS name (CNAME만 가능)`
	- `db.example.com` => `myexampledb.a1b2c3d4wxyz.us-west-2.rds.amazonaws.com` (CNAME)
    
- S3 Bucket: `example.com → S3 website endpoint (Alias)`
	- You must create an Alias record for S3 endpoints 
	- Bucket name must be the same as domain name 
	- ex) `example.com` => `s3-website-us-west-2.amazonaws.com` (Alias)
    
- PrivateLink (VPC Interface Endpoint): 도메인을 VPC Interface Endpoint (AWS PrivateLink)에 Alias 연결
	- `example.com` => `vpce-1234-abcdev-us-east-1.vpce-svc-123345.us-east-1.vpce.amazonaws.com` (Alias)
	- ![](images/Pasted%20image%2020250608201355.png)

---
### Route 53 – Hosted Zones

![](images/Pasted%20image%2020250608201548.png)

- Route 53은 Hosted Zone 생성 시 NS와 SOA 자동 생성
- 동일 네임스페이스가 겹치는 Public/Private Zone은 가장 구체적인 경로로 트래픽 전달

---

