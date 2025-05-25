
## Amazon Route 53 – Routing Policies 

###  라우팅 정책이란?

- Route 53이 **DNS 질의에 응답하는 방식**을 정의
- **로드 밸런서의 트래픽 라우팅과는 다름** 
	- DNS는 트래픽을 라우팅하는게 아니라 오직 DNS 쿼리에 대한 응답만 한다.
    
- 지원되는 정책:
    1. Simple
    2. Weighted
    3. Failover
    4. Latency-based
    5. Geolocation
    6. Multi-Value Answer
    7. Geoproximity (using Route 53 Traffic Flow feature)

---
#### Routing Policies – Simple

![300](images/Pasted%20image%2020250525231736.png)
- 단일 리소스로 트래픽을 라우팅하는 기본 정책
- 동일 레코드에 여러 값을 지정 가능
- 여러 값이 반환될 경우, 클라이언트가 랜덤으로 하나를 선택
- Alias 사용 시 하나의 AWS 리소스만 지정 가능
- Can’t be associated with Health Checks

#### Routing Policies – Weighted

![300](images/Pasted%20image%2020250525231835.png)

- 각 레코드에 **가중치(Weight)** 를 부여하여 트래픽 비율 조정
- 가중치 계산: traffic(%) = (특정 레코드의 가중치) / (모든 레코드 가중치의 합).
	- Weights don’t need to sum up to 100
- **DNS record는 Record name과 type이 동일해야한다.**
- Can be associated with Health Checks
        
- Use cases:
    - 리전 간 로드 밸런싱
    - 새 애플리케이션 버전 테스트
        
- 가중치를 0으로 설정하면 해당 리소스로 트래픽 전송 중지
- 모든 레코드의 가중치가 0이면 모든 레코드가 균등하게 반환
- 가중치 정책은 트래픽 분배를 세밀하게 제어할 수 있다.


#### Routing Policies – Latency-based

- 사용자와 가장 낮은 지연 시간(Latency)을 제공하는 리소스로 반환
- Super helpful when latency for users is a priority
- 사용자와 AWS 리전 간 **지연 시간(Latency)**을 기반으로 동작
- 독일 사용자가 미국 리전에 연결될 수도 있음 (지연 시간 낮은 경우)
- Can be associated with Health Checks (failover 기능 포함)
- Use cases: 지연 시간 최적화가 중요한 애플리케이션
- 지연 시간 기반 라우팅은 사용자 경험을 향상시키기 위해 설계됨

---
### Route 53 – Health Checks

![400](images/Pasted%20image%2020250525233003.png)

- HTTP Health Checks are only for **public resources**
- Health Check => Automated DNS Failover (자동 DNS 레벨에서의 장애조치):
	1. Health checks that monitor an endpoint (application, server, other AWS resource)
	2. Health checks that monitor other health checks (Calculated Health Checks)
	3. Health checks that monitor CloudWatch Alarms (full control !!)
		1. ex) DynamoDB throttles, RDS 알람, custom metrics ... (private 리소스에 유용)
- CloudWatch 메트릭과 통합
- Health Check 는 리소스 상태를 확인하여 안정적인 DNS 라우팅을 보장한다.

#### Health Checks – Monitor an Endpoint

![400](images/Pasted%20image%2020250525233838.png)
- 15 global health checkers will check the endpoint health
    - **Healthy/Unhealthy 임계값**: 기본값 3
    - Interval – 30 sec (can set to 10 sec – higher cost)
    - Supported protocol : HTTP, HTTPS and TCP
    - 상태 코드: **2xx, 3xx** → 통과
    - **Healthy 기준**: 18% 이상의 헬스 체커가 정상 응답(2xx, 3xx 상태 코드) 시 정상으로 간주
    - 응답의 첫 5120바이트 내 텍스트를 기반으로 통과/실패 설정 가능
        
- Configure you router/firewall to allow incoming requests from Route 53 Health Checkers
	- 클라이언트 IP 대역 허용 필요  
    → [https://ip-ranges.amazonaws.com/ip-ranges.json](https://ip-ranges.amazonaws.com/ip-ranges.json)


#### Route 53 – Calculated Health Checks

![400](images/Pasted%20image%2020250525233908.png)

- Combine the results of multiple Health Checks into a single Health Check
- AND, OR, NOT 조건 사용 가능
- 최대 **256개 Child Health Checks 조합 가능
- 상위 헬스 체크가 통과하려면 몇 개의 하위 헬스 체크가 통과해야 하는지 지정
- Use cases: 웹사이트 유지보수 중 모든 헬스 체크 실패 방지
- Calculated Health Checks는 복잡한 모니터링 시나리오를 지원

#### Health Checks – Private Hosted Zones

![400](images/Pasted%20image%2020250525233934.png)

- Route 53 헬스 체커는 VPC 외부에 위치, private endpoints(private VPC or on-premises resource)에 직접 접근 불가
    - CloudWatch 메트릭과 알람을 생성한 후, 알람을 모니터링하는 헬스 체크 생성
        
- Use case: private 리소스의 상태 모니터링
- CloudWatch 알람을 활용해 private 리소스의 가용성을 간접적으로 확인

---
#### Routing Policies – Failover (Active-Passive)

![](images/Pasted%20image%2020250525234018.png)

- **Active-Passive 구성**에 사용
- **Primary 리소스 장애 발생 시**, 자동으로 secondary 트래픽 전환
- 헬스 체크와 연계하여 자동 장애 조치 구현
---