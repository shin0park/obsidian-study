
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
- secretsmanager에 auto-rotate을 설정한 다음, 람다 함수를 invoke 시켜 cloudfront의 custom http header와 waf의 rule을 new value로 주기적 업데이트 한다.
---

### CloudFront and HTTPS

![300](images/Pasted%20image%2020250331211614.png)

- **Viewer Protocol Policy** - client가 edge location에 연결하는 방법
    - HTTP, HTTPS 모두 지원
    - Redirect HTTP to HTTPS -> 추천
    - HTTPS Only
- **Origin Protocol Policy (HTTP or S3)**:
    - HTTP Only (default for S3 Static Website)
    - HTTPS Only
    - Or Match Viewer (HTTP => HTTP, HTTPS => HTTPS)
- **주의사항**:
    - S3 버킷의 "Static Website"는 HTTPS를 지원하지 않음
    - CloudFront와 오리진 간에는 유효한 SSL/TLS 인증서가 필요하며, 자체 서명 인증서는 사용할 수 없음
---
#### Alternate Domain Names

![400](images/Pasted%20image%2020250331213223.png)
- CloudFront가 제공하는 기본 도메인 대신 사용자 지정 도메인 사용 가능
	- ex: http://d11111abcdef8.cloudfront.net/cat.jpg => http://www.example.com/cat.jpg

- 인증된 CA에서 발급한 유효한 SSL/TLS 인증서 필요
	- 인증서는 다음을 포함해야 함
	    - Your domain name  
	    - All Alternate Domain Names you added to your distribution
- wildcards 사용가능  in Alternate Domain Names (e.g., *.example.com)


#### CloudFront – SSL Certificates

- Default CloudFront Certificate 
	- `*.cloudfront.net`
	- 기본 CloudFront 도메인 이름과 함께 사용 
		- `https://d11111labcdef8.cloudfront.net/catjpg`

- Custom SSL Certificate
	- 사용자 도메인 이름 사용 시 필요 - Alternate Domain Names 
		- (e.g., https://www.example.com)
	- HTTPS 요청 제공 방식
		- Server Name Indication (SNI) – Recommended
		- 각 Edge Location의 전용 IP 주소 (expensive)
        
- 인증서 옵션
    - AWS Certificate Manager (ACM)에서 제공하는 인증서
    - 3rd party certificates uploaded to ACM or IAM Certificate Store (manually rotate when expired)   
- 인증서는 US East (N.Virginia) 리전에서 created/imported 필요.
- Security Policy 보안 정책 지정 가능 (최소 SSL/TLS 프로토콜 및 사용할 암호화 방식 설정)
---

#### End-to-End Encryption: CloudFront, ALB, EC2
![100](images/Pasted%20image%2020250331214347.png)

- **CloudFront**
    - Origin protocol policy: HTTPS Only
    - Install an SSL /TLS certificate on your custom origin (위 그림에서는 origin:ALB)
    - 인증서는 Origin domain field(CloudFront에 설정된)를 포함하거나 "Host" 헤더에 도메인을 포함해야 함
    - self-signed certificate 사용 불가
- **Application Load Balancer (ALB)**
    - AWS Certificate Manager(ACM) 제공 또는 imported into ACM 인증서 사용
- **EC2 인스턴스**
    - ACM은 EC2에서 지원되지 않음
    - third-party SSL certificate (any domain name) 사용 가능
    - self-signed certificate 사용 가능 (ALB는 인증서 자체를 검증하지 않음)

---

#### CloudFront – Restrict Content Geographically

- 특정 지역의 사용자가 content/distribution 에 접근하지 못하도록 제한

![300](images/Pasted%20image%2020250331214841.png)
- 국가 단위로 제한 (3rd party GeoIP database)
    - Allow list: 승인된 국가에서만 접근 허용
    - Block list: 차단된 국가의 사용자 접근 차단
- **적용 범위**: Applied to an entire CloudFront distribution
- Use case: 저작권법에 따라 콘텐츠 접근 제어

---
### CloudFront – Customization At The Edge

- Many modern applications은 edge에서 일부 로직 실행한다.
- **Edge Funtion**
	- CloudFront distributions에 연결하는 사용자 작성 코드
	- 사용자 근접 실행으로 지연 시간 최소화 
	- 캐시 없음, 요청/응답 변경만 가능
- CloudFront provides two types: CloudFront Functions & Lambda@Edge
- Use cases
	- HTTP 요청 및 응답 조작
	- 애플리케이션 도달 전 request filtering 구현
	- 사용자 인증 / 인가
	- edge에서 HTTP 응답 생성
	- A/BTesting
		- 두 가지 버전(A와 B)을 비교하여 어느 것이 더 나은 성과를 내는지 확인하는 실험 방법
	- Bot mitigation at the edge - 봇(bot) 활동을 탐지하고 차단하거나 완화, 보안과 성능 향상
		- 악의적인 트래픽이나 비정상적인 요청이 애플리케이션의 오리진 서버에 도달하기 전에 사전에 방어
- You don’t have to manage any servers, deployed globally

#### CloudFront Functions & Lambda@Edge

![](images/Pasted%20image%2020250331214917.png)
- CloudFront Function
	- Edge Location 레벨
- Lambda@Edge
	- Regional Edge Cache 레벨


#### CloudFront – CloudFront Functions

![200](images/Pasted%20image%2020250331222536.png)

- JavaScript로 작성된 Lightweight 함수
- 고규모, 지연 시간에 민감한 CDN customizations
- 밀리초 미만 시작 시간, 초당 수백만 요청 처리
- RuN at Edge Locations
- 프로세스 기반 격리
- **Viewer requests and responses 변경**
	- Viewer Request: CloudFront가 뷰어로부터 요청 수신 후
	- Viewer Response: CloudFront가 뷰어에게 응답 전달 전
- CloudFront 내에서 코드 완전 관리


#### CloudFront – Lambda@Edge

- Node.js 또는 Python으로 작성
- 초당 수천 요청 처리 가능
- VM 기반 격리
- **CloudFront  requests and responses 변경**
    - Viewer Request – after CloudFront receives a request from a viewer
    - Origin Request – before CloudFront forwards the request to the origin
    - Origin Response – after CloudFront receives the response from the origin
    - Viewer Response – before CloudFront forwards the response to the viewer
- 한 AWS 리전(us-east-1)에서 함수 작성 후 CloudFront가 위치에 복제

| **항목**     | **Process-based (CloudFront Functions)** | **VM-based (Lambda@Edge)** |
| ---------- | ---------------------------------------- | -------------------------- |
| **격리 수준**  | 프로세스 수준 (경량)                             | VM 수준 (완전 격리)              |
| **실행 환경**  | 호스트 OS 공유                                | 독립적인 가상 머신                 |
| **시작 속도**  | 밀리초 미만 (초고속)                             | 상대적으로 느림 (몇 밀리초)           |
| **리소스 접근** | 제한적 (네트워크/파일 시스템 불가)                     | 가능 (외부 호출, SDK 사용)         |
| **보안**     | 보통 (커널 공유)                               | 높음 (완전 격리)                 |

![](images/Pasted%20image%2020250331223434.png)

*참고*: Viewer requests and responses)에서는 CloudFront Functions와 Lambda@Edge를 결합할 수 없음

![](images/Pasted%20image%2020250331223445.png)

![](images/Pasted%20image%2020250331223555.png)


#### CloudFront Functions vs. Lambda@Edge – Use Cases

- **CloudFront Functions**:
    - Cache key normalization
	    - 요청 속성(헤더, 쿠키, 쿼리 문자열, URL) 변환으로 최적 캐시 키 생성
    - 헤더 조작
	    - HTTP 헤더 삽입/수정/삭제
    - URL rewrites or redirects
    - Request authentication & authorization
	    - 사용자 생성 토큰(JWT 등) 생성 및 검증

- **Lambda@Edge**:
    - 긴 실행 시간(수 밀리초)
    - 조정 가능한 CPU 또는 메모리
    - Your code depends on a 3rd libraries (e.g., AWS SDK to access other AWS services)
    - 외부 서비스 사용 하기 위한 네트워크 접근 가능
    - 파일 시스템 또는 HTTP request body 접근 가능


#### CloudFront Functions vs. Lambda@Edge – Authentication and Authorization

![](images/Pasted%20image%2020250331224050.png)
- CloudFront Function 
	- 가장 가까운 edge location에서 인증/인가 가능 (validate JWT tokens)
	- 만약 오류가 나면 클라이언트가 오류 메시지 직접 받게 됨 - request가 origin 에 가지 않도록
- Lambda@Edge
	- Regional Edge cache 레벨에서 실행
	- lambda function이 request 가로 챌 수 있음.
		- custom logic(3rd party api call)으로 인증/인가 체크 가능

#### Lambda@Edge use case: Loading content based on User-Agent

![](images/Pasted%20image%2020250331224110.png)
- iphone 모바일인 경우 이미지 크기를 더 작은 사이즈로 전송하도록 조작 할 수 있음.
- 즉, user-agent 에 따라 content를 조절하여 load 할 수 있음.

#### Lambda@Edge – Global Application

![](images/Pasted%20image%2020250331224127.png)
- 클라이언트가 s3 버킷에서 HTML website를 받으면 
- 동적으로 api request를 cloudfront로 보내서 cache를 활용한다.
- 람다함수를 통해 dynamodb에서 쿼리하는 것도 가능.
- fully global 하며, serverless 인 구조를 제공

---

### Global users for our application

![](images/Pasted%20image%2020250407230045.png)

- 전 세계에 있는 사용자가 배포한 애플리케이션에 접근하고자 한다.
- 이들은 public internet을 통해 접속하게 되며, 여러 중간 hops들로 인해 latency가 증가할 수 있다.
- latency를 최소화하기 위해 가능한 빠르게 AWS 내부 네트워크를 통해 전달하고자 한다.


---

### Unicast IP vs Anycast IP

![300](images/Pasted%20image%2020250407230109.png)

- Unicast IP: 하나의 서버가 하나의 고유한 IP 주소를 가진다.
- Anycast IP: 모든 서버가 동일한 IP 주소를 가지며, 클라이언트는 가장 가까운 서버로 라우팅 된다.


---

### AWS Global Accelerator

![300](images/Pasted%20image%2020250407230206.png)

- AWS 내부 네트워크를 활용하여 애플리케이션으로 트래픽을 빠르게 라우팅 한다.
	- india에 있는 애플리케이션을 각 전세계에서 접속한다고 할때
- 애플리케이션에 대해 **2개의 Anycast IP**가 생성 된다.
- 그 Anycast IP는 트래픽을 **Edge Locations**으로 직접 전송
- The Anycast IP send traffic directly to Edge Locations
- 그 Edge locations은 그 트래픽을 최종적으로 여러분의 애플리케이션으로 전달 한다.

주요 특징
- **지원 서비스**:Elastic IP, EC2 인스턴스, ALB, NLB (퍼블릭/프라이빗 모두 가능)
- 일관된 성능 제공: AWS 백본 네트워크를 통해 낮은 지연 시간 보장
- **지능형 라우팅**: 최저 지연 경로로 트래픽 전송 + 빠른 리전 장애 조치(failover)
- **클라이언트 캐시 문제 없음**: IP가 변경되지 않아 DNS 캐싱 문제 방지
	- 도메인 DNS 이름을 IP 주소로 변환하여 사용하는데, 클라이언트는 이를 일정 시간동안 캐시하여 dns 조회하지 않고 바로 접속하고는 한다. 백엔드 서버의 ip가 바뀌면 캐시때문에 접속 오류가 생길 수가 있는데, AWS Global Accelerator 의 경우 ip가 바뀌지 않고 고정이므로 이러한 문제가 발생하지 않는다.
- 내부 AWS 네트워크 사용


Health Checks  
- Global Accelerator 애플리케이션 health check를 지속적으로 한다.
- 문제가 발생한 경우, 1분 이내에 다른 정상 리전으로 자동 장애 조치(failover)를 수행
- 재해 복구(Disaster Recovery)에 유리 (thanks to health check)

Security 
- 외부에서 접근 가능한 IP는 오직 2개이므로, 화이트리스트 설정이 간단함
- AWS Shield를 통해 DDoS 공격에 대한 보호 제공

#### AWS Global Accelerator vs. CloudFront

- 둘 다 AWS global network 와 전 세계 edge locations을 활용
- 두 서비스 모두 AWS Shield와 통합되어 DDoS 보호 제공

- **CloudFront**:
	- 캐시 가능한 콘텐츠(이미지, 비디오 등) 성능 향상
	- 동적 콘텐츠(API acceleration, 동적 사이트 전달 등) 지원
	- 콘텐츠가 edge에서 제공됨

- **Global Accelerator**
	- **TCP/UDP 기반 애플리케이션** 성능 향상
	-  엣지에서 패킷을 프록시하여 **한 개 이상의 AWS 리전**에서 실행중인 애플리케이션으로 전달
	- non-HTTP 워크로드에 적합 (ex. 게임(UDP), IoT(MQTT), VoIP(Voice over IP))
		- **IoT 센서**: 소량의 데이터를 빠르게 전달 → MQTT (주로 TCP, 때로는 UDP)
		- VoIP(실시간 음성통신) - UDP
		- AWS CloudFront은 **HTTP(S)** 요청만 처리할 수 있고, UDP는 지원하지 않는다.
		- Global Accelerator는 TCP와 UDP를 모두 지원
	- static IP 주소가 필요한 HTTP 사용 사례에 유용
		- 보안이나 네트워크 정책상, 외부 접근을 허용하기 위해**정해진 IP만 화이트리스트에 등록**해야 할 때가 있다.
		- 도메인 이름은 IP가 바뀔 수 있기 때문에 위험
		- Global Accelerator는 **2개의 고정 IP 제공** → 변경 없음 → 보안/정책 설정에 유리
	- 결정적이고 빠른 regional failover가 필요한 HTTP 사용 사례에 적합
		- 1분 미만으로 regional failover를 지원하기에 적합
		- 클라이언트는 같은 IP로 요청하지만 **AWS 내부에서 라우팅만 바꿔줌**
---

Hands on

- 지역별로 ec2 생성 
- AWS Global Accelerator 생성 - 앞서 생성한 ec2를 엔드포인트로 설정 - 엔드포인트 2개 
- vpn을 변경하면서 가까운 ec2로 로드되는지 테스트
- 하나의 ec2가 unhealthy 됐을때 다른 ec2로 트래픽이 가는지 테스트

