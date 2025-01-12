
## Why do we need Private Connectivity?
-> private connectivity 가 없다면 트래픽이 인터넷으로 흐를 것이다.
### Disadvantages of having Internet based traffic

1. **보안 문제**:
    - 인터넷을 통한 트래픽은 보안 수준이 낮다.
2. **대역폭 및 지연 시간**:
    - 인터넷 연결은 일관된 대역폭과 낮은 지연 시간을 보장하지 못함
3. **비용**:
    - NAT 장치는 시간당 높은 운영 비용과 데이터 처리 비용이 발생
4. **불필요한 노출**:
    - 데이터베이스, 애플리케이션 서버 등 퍼블릭 노출이 필요 없는 리소스를 인터넷에 공개해야 할 수 있다.

---

### AWS Private Connectivity

#### Topics
1. **VPC Peering**:
2. **VPC Endpoints**:
3. **VPC PrivateLink**:

---

### VPC Peering

![500](images/Pasted%20image%2020250112194857.png)
#### Feature

- VPC는 사용자의 AWS 계정 전용 가상 네트워크이다. VPC는 AWS 클라우드에서 다른 가상 네트워크와 논리적으로 분리되어있다.  
- Peering은 두 vpc 사이에 인터넷 통신을 하지 않고 private connection을 맺는다. 즉, AWS backbone 네트워크를 통해 두 vpc를 프라이빗하게 연결하는 방법.  
- 연결된 VPC는 같은 네트워크처럼 동작
- 같은 리전뿐만 아니라 다른 리전의 VPC와도 연결 가능, 다른 AWS 계정의 VPC와도 연결 가능
- 서로 다른 AWS 리전에 위치한 VPC 사이에 피어링 관계를 설정하는 경우, 상이한 AWS 리전의 VPC에 있는 리소스(예: EC2 인스턴스와 Lambda 함수)에서 게이트웨이, VPN 연결 또는 네트워크 어플라이언스를 사용하지 않고 프라이빗 IP 주소를 사용하여 서로 통신할 수 있다.   
- 트래픽은 프라이빗 IP 주소 공간 안에서 유지된다.   
- 모든 리전 간 트래픽은 암호화되며 단일 장애 지점 또는 대역폭 제한이 없다.  
- 트래픽은 항상 글로벌 AWS 백본에서만 유지되고 절대로 퍼블릭 인터넷을 통과하지 않으므로 일반적인 취약점 공격과 DDoS 공격 같은 위협이 감소  
- 리전 간 VPC 피어링은 리전 간에 리소스를 공유하는 **간단하고 비용 효율적인 방법**  

#### Cost
- VPC 피어링 연결 생성에는 요금이 부과되지 않는다.  
- 가용 영역 내에 있는 VPC 피어링 연결을 통한 데이터 전송은 (서로 다른 계정 간이라도) 모두 무료  
- 가용 영역 및 리전 간 VPC 피어링 연결을 통한 데이터 전송에는 요금이 부과  
  -> https://aws.amazon.com/ec2/pricing/on-demand/#Data_Transfer

Document: https://docs.aws.amazon.com/ko_kr/vpc/latest/peering/what-is-vpc-peering.html

#### Caveats(주의사항)

- **CIDR non-overlapping**:
    - 연결된 VPC의 CIDR 범위 non-overlapping
- Update route tables:
    - 두 VPC 간 통신을 위해 각 VPC subnet's route tables 을 업데이트해야 함
- Full mesh
	- ![400](images/Pasted%20image%2020250112194709.png)
	- https://docs.aws.amazon.com/ko_kr/vpc/latest/peering/peering-configurations-full-access.html

---

### VPC Peering Demo

![600](images/Pasted%20image%2020250112195903.png)

#### Steps

1. **VPC-A 생성**:
    - Mumbai(ap-south-1) 리전에서 CIDR: 10.10.0.0/16. 
    - 인터넷 게이트웨이를 생성하고 VPC-A에 연결
    - 퍼블릭 서브넷(CIDR: 10.10.0.0/24) 생성
    - EC2 인스턴스를 시작하고 퍼블릭 IP를 할당. 포트 22을 오픈
2. **VPC-B 생성**:
    - North Virginia(us-east-1) 리전에서 CIDR: 10.20.0.0/16. 
    - 프라이빗 서브넷(CIDR: 10.20.0.0/24) 생성
    - ICMP 및 포트 22을 VPC-A CIDR 범위에서 허용
3. **VPC Peering 설정**:
    - VPC-A에서 VPC-B로 Peering 요청을 생성
    - VPC-B에서 Peering 요청을 수락
4. Update route table:
    - 양쪽 VPC의 트래픽이 서로 통신 가능하도록 route table 수정
5. Connect:
    - VPC-A의 EC2에서 VPC-B의 EC2로 SSH 또는 ping 테스트

---

### VPC Peering Limitations

1. Must not have overlapping CIDR
   ![400](images/Pasted%20image%2020250112201340.png)
   10.10.0.0/16, 10.10.0.0/24 여도 overlap 되기 때문에 불가능
   
2. VPC Peering connection is **not transitive** (must be established for each VPC that need to communicate with one another)
	1. Full Mesh 구조
	2. 두 VPC 간 1개의 피어링 연결만 설정 가능
	3. VPC당 최대 125개의 피어링 연결 제한
	   
	not transitive 란?  
	-> A -> B로 연결된 네트워크에서 다시 B -> C로 트래픽을 전송하려고 할 때, A -> C로의 연결이 자동으로 확장되지 않는 것을 의미  

---

### VPC Peering Limitations scenarios

1. **VPN or Direct Connect**:
    - VPN or Direct Connect connection to on-premises network
    -  **not transitive**
    - 오직 VPC A에서 B로만 가는 것 만으로 끝! 더 이상 transitive 하지 않는다.
    - proxy 를 통해 연결할 수는 있겠지만 아래와 같이 간단한 구조로는 불가능하다. 프록시 서버를 별도 설정해야하고 추가 비용 및 성능 문제 그리고 관리 복잡도가 증가될 것.
    - VPC A에도 onprem으로 가는 별도의 VPN/DX를 추가로 설정하면 가능
    - TGW를 사용하면 네트워크를 중앙 집중적으로 관리하기 때문에, transitive하게 연결 가능. 즉, A -> B -> C 가능.
      ![500](images/Pasted%20image%2020250112201624.png)
      
2. IGW를 통한 트래픽**:
    - 피어링된 VPC의 IGW를 통해 트래픽 라우팅 불가  
      ![600](images/Pasted%20image%2020250112202515.png)
      
3. **S3/DynamoDB**:
    - 피어링된 VPC의 엔드포인트를 통해 접근 불가  

---
