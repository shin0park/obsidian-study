
## VPC Traffic Monitoring with VPC flow logs

### VPC Flow Logs

- **VPC Flow Logs**는 **네트워크 인터페이스**에서 들어오고 나가는 **IP 트래픽 정보를 캡처**하는 기능
- 캡처할 수 있는 Flow Logs 유형:
    - **VPC Flow Logs**
    - **Subnet Flow Logs**
    - **Elastic Network Interface (ENI) Flow Logs**
- 주요 기능:   
    - 네트워크 연결 문제를 **모니터링**하고 **문제 해결** (sg/nacl로 인해 통신이 막혔는지, 악성ip가 접근하려고 하는지)
    - Flow Logs 데이터를 **S3**, **CloudWatch Logs**, **Kinesis Data Firehose**에 전송하여 분석 가능.
    - **AWS 관리형 서비스**에서도 네트워크 정보를 캡처 가능:
        - ELB (Elastic Load Balancer), RDS, ElastiCache, Redshift, Amazon WorkSpaces 
          (해당 managed service 생성할때 aws가 우리의 vpc에 ENI를 생성. 따라서 flow logs가 eni에서 로그를 캡처함)
    - VPC Flow Logs 활성화가 **네트워크 성능에 영향을 미치지 않음**.


#### Publishing VPC flow logs

![[Pasted image 20250101185029.png|600]]
- vpc level
- subnet level
- eni level
위 레벨 별로 flow logs 존재

이 로그들을 cloudwatch, s3, kinesis 에 전송하여 분석가능



### VPC Flow Logs default format

- **default format**
    `<version> <account-id> <interface-id> <srcaddr> <dstaddr> <srcport> <dstport> <protocol> <packets> <bytes> <start> <end> <action> <log-status>`
    
    - **필드**:
        - **srcaddr, dstaddr**: 문제를 일으키는 소스 및 목적지 IP 주소 식별
        - **srcport, dstport**: 문제를 일으키는 소스 및 목적지 포트 식별
        - **action**: 요청이 성공(success)했는지 또는 보안 그룹/네트워크 ACL에 의해 실패(failure)했는지 여부
        - protocol: 네트워크 트래픽에서 사용된 프로토콜 번호
	        - 프로토콜 번호는 **IANA(Internet Assigned Numbers Authority)**가 정의한 프로토콜 번호 표준을 따름.
		        - `6` → **TCP**
				- `17` → **UDP**
				- `1` → **ICMP**
				- `41` → **IPv6**
				- `50` → **ESP (Encapsulating Security Payload)**
				- `58` → **ICMPv6**
        - packets: 전송된 패킷의 총 수
        - start, end: 해당 트래픽 흐름의 **시작 시간**과 **종료 시간** - `end` - `start` = duration
        - log-status: Flow Logs 데이터가 얼마나 성공적으로 기록되었는지를 나타낸다.
	        - **OK**:
			    - 트래픽이 성공적으로 기록
			- **NODATA**:
			    - 해당 시간 간격 동안 기록된 트래픽이 없음
			    - 이 값은 특정 인터페이스에 트래픽이 없을 때 반환
			- **SKIPDATA**:
			    - 기록 중 일부 로그가 누락
			    - 주로 **IAM 권한 문제**나 네트워크 **트래픽 과부하**로 인해 발생
    - 사용 패턴 분석하고 악의적 행위 탐지 및 보안 모니터링 가능

- **version**
    - Flow Logs는 **버전 2, 3, 4, 5**를 지원하며, 기본 버전은 **2**
    - 어떤 버전이냐에 따라 다른 포맷을 가질 수 있음

![[Pasted image 20250101185656.png|600]]

### VPC Flow Logs custom format

- **vpc-id**: 트래픽이 발생한 VPC의 ID
- **subnet-id**: 트래픽이 발생한 서브넷의 ID
- **instance-id**: 트래픽을 송수신한 EC2 인스턴스의 ID
- **type**: 트래픽의 IP 유형 (IPv4 또는 IPv6).
- **pkt-srcaddr, pkt-dstaddr**: 실제 소스 및 목적지 IP 주소 (nat로 Ip가 변환됐다면 실제 ip)
- **pkt-src-aws-service, pkt-dst-aws-service**: 소스 및 목적지가 AWS 서비스인지 여부와 서비스 이름
- **region**: 트래픽이 발생한 AWS 리전
- **az-id**: 트래픽이 발생한 가용 영역 ID
- **tcp-flags**: 트래픽에서 사용된 TCP 플래그 
- 
- **traffic-path** : 트래픽이 이동한 경로 (Local(vpc내부), IGW, NATGW, TGW)
- **traffic-flow-direction**: 트래픽의 흐름 방향:
    - Ingress: 내부로 들어오는 트래픽.
    - Egress: 외부로 나가는 트래픽.
- +more

![[Pasted image 20250101190922.png|300]]
![[Pasted image 20250101190913.png|600]]

srcaddr - s3 public ip
dstaddr - ec2 instance private ip

-> https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/flow-log-records.html#flow-logs-fields

TCP 플래그의 비트 마스크 값:
- FIN — 1
- SYN — 2
- RST — 4
- SYN-ACK — 18

#### Lab - Analyze VPC Flows logs with CloudWatch Insights
전제조건
- cloudwatch logs group 생성
- VPC flow logs 서비스(trust-relationships: vpc-flow-logs.amazonaws.com) 에 IAM role 설정 (grant permission to push logs to cloudwatch)

- VPC > Create vpc flow logs
- Cloudwatch > Logs Insights
- 쿼리를 실행할 log groups 선택
```
fields @timestamp, @message
| sort @timestamp desc
| limit 10
```
@message 컬럼을 보면 위에서 다룬 기본 format들을 확인할 수 있음
```
 
stats sum(packets) as packetsTransferred by srcAddr, dstAddr
| sort packetsTransferred  desc
| limit 15
```
Sum of packets by src and dest
```
fields @timestamp, interfaceId, srcAddr, dstAddr, action
| filter (interfaceId = 'eni-xxxxxxxxx' and action = 'REJECT')
| sort @timestamp desc
| limit 5
```

```
fields @timestamp, srcAddr, dstAddr
| sort @timestamp desc
| limit 5
| filter srcAddr like "172.31."
```
List  IP addresses trying to connect to specific IP or CIDR

```
fields @timestamp, @message
| stats count(*) as records by dstPort, srcAddr, dstAddr as Destination
| filter dstPort="80" or dstPort="443" or dstPort="22" or dstPort="25"
| sort HitCount desc
| limit 10
 
fields @timestamp, @message
| stats count(*) as records by dstPort, srcAddr, dstAddr as Destination
| filter dstPort="80" or dstPort="443" or dstPort="22" or dstPort="25"
| sort HitCount desc
| limit 10
 
 
```


#### How to troubleshoot SG vs NACL issue?
![[Pasted image 20250101194704.png|300]]
“ACTION” Field

- For incoming requests
	Inbound REJECT: NACL or SG  
	Inbound ACCEPT, outbound REJECT: NACL

- For outgoing requests
	Outbound REJECT: NACL or SG  
	Outbound ACCEPT, inbound REJECT: NACL

#### Flow Logs limitation

아래와 관련된 트래픽은 기록하지 않음
- AWS DNS 서비스
	- ex. 도메인 이름을 ip 주소로 변환
- AWS EC2 Metadata 서비스
	- Metadata 서비스는 ec2 내부에서만 사용되며, 네트워크 외부로의 영향을 주지 않음
- DHCP(Dynamic Host Configuration Protocole) 서비스
	- VPC에서 인스턴스가 IP를 자동 할당받을때 사용하는 DHCP 트래픽은 기록하지 않는다. - 내부에서 자동으로 처리되며 외부 트래픽으로 간주되지 않기 때문
- Windows license activation server 
	- Windows 인스턴스가 AWS에서 제공하는 **Windows 라이선스 활성화 서버**와 통신하는 트래픽은 기록하지 않음.

## VPC Traffic Mirroring

- VPC Traffic Mirroring은 Amazon EC2 인스턴스의 네트워크 인터페이스(ENI)에서 발생하는 네트워크 트래픽을 복사하여, 이를 보안 및 모니터링 도구로 전송하는 기능
- 트래픽 미러링은 **컴플라이언스 요구 사항**을 충족하는 데 필요
- 실시간으로 네트워크 트래픽을 분석하여 악위적 행위나 비정상적인 트래픽을 탐지하기 위해 사용된다. (IDS, IPS)
- 전송하기 위해 네트워크 성능 걱정할 필요없이 미러링하여 전송 가능
- How to set up Traffic Mirroring (via AWS VPC console)
    • Create the Mirror Target (ENI or NLB)
    • Define the Traffic Filter (protocol, port, CIDR range)
    • Create Mirror Session (소스(트래픽 복사 대상)와 타켓(복사된 트래픽 전송대상) 연결)

#### VPC Traffic Mirroring 대상
1. **소스(Mirror Source)**:
    - Amazon EC2 인스턴스의 **Elastic Network Interface(ENI)**.
2. **타겟(Mirror Target)**:
    - **다른 ENI**: 같은 또는 다른 EC2 인스턴스의 ENI.
    - **네트워크 로드 밸런서(NLB)**:
        - UDP 포트 **4789**를 사용!

![[Pasted image 20250101201903.png|400]]
![[Pasted image 20250101201912.png|400]]

#### VPC Traffic Mirroring filter
- **filter parameter**로 원하는 트래픽만 캡처
    - **트래픽 방향**:
    - **Action**:
        - **Accept**: 허용된 트래픽만 복사
        - **Reject**: 거부된 트래픽만 복사
    - **프로토콜**: Layer 4 프로토콜(TCP, UDP 등)
    - **포트 범위**: 소스/목적지 포트
    - **CIDR 블록**: 소스/목적지 IP 범위

#### VPC Traffic Mirroring – Good to know
**source and target의 위치**:
- source와 target은 **같은 VPC**, 또는 **다른 VPC**(동일 리전 내 VPC 피어링이나 Transit Gateway로 연결된 경우)에서 위치 가능
- **다른 AWS 계정**에 위치한 리소스도 타겟으로 설정 가능