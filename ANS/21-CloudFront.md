
## **AWS CloudFront - CDN 서비스**

![400](images/Pasted%20image%2020250324215143.png)
- AWS CloudFront는 콘텐츠 전송 네트워크로 Content Delivery Network (CDN) 서비스이다.
- 콘텐츠를 엣지(Edge)에서 캐싱하여 읽기 성능을 향상시킨다.
- **글로벌 배포**: 전 세계적으로 225개 이상의 PoP(Point of Presence)를 보유 
	- 215+ Edge Locations & 13 Regional Edge Caches
- **보안**: 네트워크 및 애플리케이션 계층 공격(예: DDoS 공격)으로부터 보호
- **통합**: AWS Shield, AWS WAF, Route 53과 통합 가능
- **HTTPS 지원**: 외부 HTTPS 노출 및 내부 HTTPS 백엔드와 통신 가능
- WebSocket 프로토콜 지원

---
### Edge Locations & Regional Edge Caches
![400](images/Pasted%20image%2020250324215208.png)

- Edge Locations
	- Serve content quickly/directly to users
	- Cache more popular content

- Regional Edge Caches
	- Serve content to Edge Locations
	- Cache less popular content that might suddenly find popularity
	- Larger cache than Edge Location (objects remain longer)
	- 성능 향상 및 오리진 부하 감소
	- 동적 콘텐츠는 통과하지 않음(오리진으로 직접 전송)

---
### CloudFront Components

- Distribution
	- 도메인으로 식별 (e.g., d11111abcdef8.cloudfront.net)
	- this distribution domain name을 사용해 웹사이트 접근 가능
	- Route53 CNAME (non-root) or Alias (root & non-root) 사용하여 도메인과 연결 가능
- Origin
	- 실제 콘텐츠가 저장된 위치(예: S3 버킷, ALB, HTTP 서버, API Gateway 등)
- Cache Behavior
	- Cache configurations (e.g., Object Expiration, TTL, Cache invalidations 캐시무효화)

---
### CloudFront Origins

- S3 Bucket  
	- 파일 배포 용도로 사용
	- **Origin Access Control(OAC, 이전 OAI)**로 보안 강화
		- CloudFront를 통해서만 S3 오리진 접근을 허용하도록 하여 S3 오리진을 보호
		- https://aws.amazon.com/ko/blogs/korea/amazon-cloudfront-introduces-origin-access-control-oac/
	- CloudFront를 통해 S3로 파일 업로드(인그레스) 가능
- S3 Bucket configured as a website  
	- 버킷에서 Static Website hosting을 먼저 활성화해야 함
- MediaStore Container & MediaPackage Endpoint  
	- AWS 미디어 서비스를 활용해 Video On Demand(VOD) 또는 라이브 스트리밍 비디오 제공
- Custom Origin (HTTP)
	- EC2 instance
	- Elastic Load Balancer (ALB)  
	- API Gateway (for more control... otherwise use API Gateway Edge)
	- Any HTTP backend you want

---
#### CloudFront Origins – S3 as an Origin

![500](images/Pasted%20image%2020250324220049.png)
- S3를 오리진으로 설정하면 콘텐츠를 CloudFront를 통해 배포 가능
- OAC 와 s3 bucket policy 결합하여 cloudfront를 통해서만 접근 가능하도록 설정 가능

#### CloudFront Origins – ALB or EC2 as an origin

![600](images/Pasted%20image%2020250324220055.png)
- EC2
	- Cloudfront로 부터의 ec2 접근이므로 must be public
	- ec2 SG에 edge location 의 public ip를 허용해줘야 접근 가능할 것인데, 
		- URL 사용하여 public ip list를 허용해줄 수 있다.
- ALB
	- ALB must be public
	- ALB Sg에 edge location 의 public ip를 허용 필요.
	- ALB 뒷단에 Private 한 EC2를 위치시킬 수도 있다.
---

#### CloudFront – Multiple Origin

![500](images/Pasted%20image%2020250324220132.png)

- 콘텐츠 유형에 따라 서로 다른 Origin으로 라우팅 가능
- Based on path pattern: 
	- /images/* → S3 버킷
	- /api/* → API Gateway
	- /* → S3 버킷

---
#### CloudFront – Origin Groups

![](images/Pasted%20image%2020250324220155.png)

- 고가용성(HA) 및 장애 조치(failover) 향상
- Origin Group: one primary and one secondary origin
- 기본 원본(Primary Origin)이 실패하면 보조 원본(Secondary Origin)으로 전환
---

#### CloudFront – Origin Custom Headers

- CloudFront가 Origin으로 보내는 요청에 커스텀 헤더를 추가 가능
- 각 Origin 별로 커스터마이징 가능
- Supports custom and S3 origins

- Use cases:
    - CloudFront 또는 특정 distribution에서 온 요청 식별
    - 콘텐츠 접근 제어(Origin이 커스텀 헤더 포함 요청에만 응답하도록 설정)
- **예시**:
	- 클라이언트 요청: GET /beach.jpg HTTP/1.1
	- CloudFront 요청: GET /beach.jpg HTTP/1.1에 X-Origin-Verify: xxxxxxxxxxxx 헤더 추가.

#### CloudFront HTTP Headers

- viewer request 에 따라 추가되는 특정 HTTP 헤더
	- Viewer’s Device Type Headers - (based on User-Agent)
		- `CloudFront-Is-(Android/Desktop/iOS/Mobile/SmartTV/Tablet)-Viewer`
    
	- Viewer’s Location Headers (based on viewer’s IP address)
		- `CloudFront-Viewer-(City/Country/Latitude/Longitude)`
    
	- Viewer’s Request Protocol & HTTP Version  
		- `CloudFront-Forwarded-Proto`, `CloudFront-Viewer-Http-Version`

- 이를 Cache Key 에 포함(using Legacy Cache Settings or Cache Policies) 하거나 Origin에서 수신가능 (using Origin Request Policies)
---
#### CloudFront – Restrict Access to S3 Buckets

- S3 버킷의 파일에 직접 접근 방지(CloudFront를 통해서만 접근 허용)
- 1. Origin Access Control(OAC) 생성 및 distribution과 연결
- 2. S3 버킷 정책 수정하여 OAC만 접근 허용

![200](images/Pasted%20image%2020250324222954.png)
S3 버킷 정책 예시
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "cloudfront.amazonaws.com" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::mybucket/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT_ID:distribution/EDFDVBDGEXAMPLE"
        }
      }
    }
  ]
}
```

---
#### CloudFront – Restrict Access to Application Load Balancers and Custom Origins

- ALB 또는 커스텀 오리진에 직접 접근 방지(CloudFront를 통해서만 접근 허용)
- 1. CloudFront가 ALB로 보내는 요청에 custom HTTP 헤더 추가
- 2. ALB가 해당 custom HTTP 헤더 포함 요청만 전달하도록 설정
- **보안**: custom 헤더 이름과 값은 비밀로 유지해야 함
-  [CloudFront Public IP 주소 목록](https://d7uri8nf7uskq.cloudfront.net/tools/list-cloudfront-ips)

![](images/Pasted%20image%2020250324223020.png)


---
#### Solution Architecture – Enahnce CloudFront Origin Security with AWS WAF & AWS Secrets Manager


![](images/Pasted%20image%2020250324223029.png)

- ALB 앞에 WAF를 구성하여 filter rule을 설정한다.
- 해당 custome http header가 있는 경우만 통과시키도록 -> 즉, direct로 alb에 접근할 수가 없다.
- secretsmanager에 auto-rotate을 설정한 다음, 람다함수를 invoke 시켜 cloudfront의 custom http header와 waf의 rule을 new value로 주기적 업데이트한다.