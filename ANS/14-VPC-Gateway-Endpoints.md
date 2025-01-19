VPC Endpoint 없이 AWS 서비스에 연결하고자 한다면,

![600](images/Pasted%20image%2020250112203336.png)
![600](images/Pasted%20image%2020250112203412.png)
![600](images/Pasted%20image%2020250112203429.png)
인터넷 통신을 해야될 것이다.
인터넷을 거치지 않고 프라이빗하게 AWS Service를 사용하기 위해 Endpoint를 사용한다. (인터넷을 타고 가는 순간 보안적 성능적 비용적 이슈 존재)  

### VPC Endpoints

#### Feature

1. 퍼블릭 네트워크 대신 프라이빗 네트워크를 통해 AWS 서비스에 연결  
2. IGW, NAT GW 없이 AWS 서비스 접근 가능 -> 비용 감소
3. 수평 확장, 고가용성, 대역폭 제한 없음  

---

### VPC Endpoint Type

![600](images/Pasted%20image%2020250112204041.png)

1. **Gateway Endpoint**:
    - S3 및 DynamoDB 연결 지원
    - route table에 대상 경로 추가해야함
    - interface endpoint가 ENI에 Private IP를 직접 부여(local routing) 하는 것 대신, Gateway Endpoint는 Prefix LIst를 활용.
    - Prefix LIst는 S3 또는 DynamoDB의 IP 주소 범위를 정의한 목록  
      ex. pl-12345678
    - 따라서 S3 또는 DynamoDB와 통신하려면 **Prefix List를 대상 경로로 추가** (대상지: Gateway Endpoint ID) 해야한다.
    - Gateway Endpoint는 트래픽을 AWS 내부 네트워크를 통해 안전하게 전달
    - interface endpoint 보다 먼저 나온 서비스로 점차 interface로 가는 추세  
2. Interface Endpoint**:
    - ENI(Elastic Network Interface)를 생성하여 Endpoint 진입점 역할 수행
    - ENI에 Private IP 부여
    - `com.amazonaws.<region>.<service>`와 같은 AWS 서비스 DNS 사용 - Private IP 매핑
    - 따라서 해당 DNS를 호출하면 ENI의 IP인 Private IP가 반환되고 route table의 local 라우팅을 통해 통신되기 때문에, **별도 Route 추가가 필요 없음.**
    - 트래픽이 ENI(프라이빗 IP)로 이동한 후, Interface Endpoint를 통해 AWS 서비스로 전달.
    - 대부분의 AWS 서비스 지원

Document: https://docs.aws.amazon.com/ko_kr/vpc/latest/privatelink/privatelink-access-aws-services.html

---

### VPC Gateway Endpoint

#### Feature

1. Private connection:
    - VPC와 S3/DynamoDB 간 프라이빗 연결. 
2. Route tables 수정 필요:
    - S3 또는 DynamoDB로의 트래픽을 route table에 추가. 
3. Prefix list사용:
    - S3 IP 주소 목록(prefix list)을 생성해 route table 및 security group 에서 사용 가능 
    - The Prefix list is formatted as pl-xxxxxxxx 
    - Prefix list : 보안 그룹의 아웃바운드 규칙에 추가해야 함(기본 허용 규칙이 없을 경우)

![600](images/Pasted%20image%2020250112210042.png)
![600](images/Pasted%20image%2020250112210129.png)

---

### VPC Endpoint Security

- **세분화된 접근 제어**:
	- VPC Peering 연결처럼 광범위한 접근 권한을 부여하는 방식과 달리, VPC Endpoint는 **VPC 리소스에 대한 세분화된 접근 제어**를 제공
- **S3 접근 보안**:
	- VPC Endpoint를 통해 S3에 접근할 경우, **버킷 정책(Bucket Policy)** 과 **엔드포인트 정책(Endpoint Policy)** 을 활용해 접근을 제어
- **VPC Endpoint Policy**:
	- VPC Endpoint에 IAM Policy를 연결
	- Default Policy은 AWS 서비스에 대해 **Allow All**를 허용

#### VPC Endpoint Policy

- 특정 S3 버킷 또는 DynamoDB 테이블에 대한 접근을 제한

![](images/Pasted%20image%2020250112211008.png)

#### Resource based policy - S3 bucket policy

- 특정 VPC 엔드포인트에 대한 접근을 제한하는 S3 버킷 정책

![600](images/Pasted%20image%2020250112211207.png)


---

### VPC Gateway Endpoint - Demo

![600](images/Pasted%20image%2020250112211435.png)
#### Steps

1. 퍼블릭 및 프라이빗 서브넷을 가진 VPC 생성.
2. 두 서브넷에 EC2 인스턴스 생성.
3. 프라이빗 서브넷의 EC2에 S3 접근 권한 부여 - IAM role. 
4. S3용 Gateway Endpoint 생성. 
5. 프라이빗 서브넷의 Route table 수정 - dest: prefix list, target: vpce. 
6. 퍼블릭 EC2를 통해 프라이빗 EC2에 SSH로 연결  
7. S3 업로드/다운로드 테스트  

---

## VPC Gateway endpoint and S3 policies

### VPC Endpoint Policy & S3 bucket policy

**S3 bucket policy may have**:
- **`aws:sourceVpce`**:
	- 특정 VPC 엔드포인트(`vpce-1a2b3c4d`)를 통해서만 접근 허용한다. 
	- 해당 조건을 통해 특정 VPC 엔드포인트에서 오는 요청이 아닐 경우 트래픽을 차단(더 높은 보안 제공)
- **`aws:sourceVpc`**:
	- 특정 VPC(`vpc-111bbb22`)를 통해서만 접근 허용한다.
	- **vpc endpoint**에서만 적용된다. - vpc endpoint를 사용하지 않고 이 조건을 입력하면 적용이 안된다는 것.
	  ex) ec2가 인터넷 통신하여 s3로 접근할때는 작동하지 않고, vpc endpoint를 사용할때만 적용된다.
	- 여러 엔드포인트를 사용하는 경우 이를 활용해 모든 엔드포인트에서 S3 버킷 접근을 관리 가능하다

- **`aws:SourceIp`**: S3 버킷 정책은 해당 조건으로 **특정 public IP주소**나 **elastic IP 주소**로 부터 접근 제한 가능하다 -> public 네트워크(인터넷)를 통해 들어올 때 유효한 조건
- private IP 주소**를 기반으로 접근을 제한할 수는 없다
- 그러므로, **`aws:SourceIp`** 조건은 VPC endpoint에 대해 적용되지 않는다. 
  (vpc endpoint 를 통한 요청은 AWS 네트워크 내부에서 처리되며, 인터넷을 거치지 않는다.)

### Example S3 bucket policies

1. **특정 VPC endpoint 에 대한 접근 제한**
    - 하나의 VPC endpoint를 통해서만 S3 버킷 접근을 허용하는 정책
      ![500](images/Pasted%20image%2020250119223402.png)
2. **전체 VPC에 대한 접근 제한**
    - 특정 VPC에 속한 여러 VPC endpoint에서 S3 버킷에 접근을 허용하는 정책
      ![500](images/Pasted%20image%2020250119223419.png)

![600](images/Pasted%20image%2020250119223532.png)
ec2 - check SG outbound- 기본값으로는 outbound allow 지만, custom 한 경우 체크

check vpc DNS settings, DNS resolution must be enabled
-> 그래야 모든 S3 API 호출이 이 VPC endpoint Gateway를 통해 나가 s3로 갈 수 있으므로


### **VPC Gateway endpoint access from remote networks**
VPC Gateway Endpoint 원격 네트워크로 부터의 접근
![600](images/Pasted%20image%2020250119223605.png)  
- **Gateway Endpoint**를 통해 원격 네트워크에서 접근하는 기능은 지원되지 않음
- Gateway Endpoint는 **VPC 내부의 리소스에서 시작된 트래픽만 허용**하며, 온프레미스 트래픽은 해당 조건을 충족하지 않는다.

#### 이유
Gateway Endpoint는 보안 및 네트워크 안정성을 유지하기 위해 VPC 내부의 트래픽만 처리하도록 설계되었기 때문.
외부로 들어오는 트래픽을 허용할 경우, AWS 내부 네트워크로의 접근 경로가 추가되어 보안상 위험이 증가 할 수 있음.

Document: https://docs.aws.amazon.com/ko_kr/vpc/latest/privatelink/vpc-endpoints-s3.html
![](images/Pasted%20image%2020250119224927.png)


Amazon S3용 VPC 엔드포인트 유형
Document:
https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/privatelink-interface-endpoints.html#types-of-vpc-endpoints-for-s3