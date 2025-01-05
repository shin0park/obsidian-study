## Network Performance

- Bandwidth(대역폭): 일정 시간 동안 네트워크를 통해 전송할 수 있는 데이터의 최대 전송률
	- bandwidth 25GB -> 초당 25GB 전송가능
- Latency: 네트워크에서 두 지접 사이의 지연
	- 지연에는 신호가 매체를 통해 이동하는 **전파 지연**과 네트워크 장치가 데이터를 처리하는 **처리 지연**이 포함
- Jitter: 패킷 간 지연 시간의 변동
	- 네트워크에서 데이터 패킷이 전송되는 시간 간격의 **불규칙한 변동**
	- 성능에 영향을 미치는 중요 지표로 - 실시간 데이터 전송(예: 음성, 영상 스트리밍, 온라인 게임)에서 문제가 될 수 있음.
	- 네트워크 혼잡, 패킷 우선순위, 네트워크 장비의 성능제한, 네트워크 경로 동적 변동 등에 의해 발생
- Throughput (처리량): 성공적으로 전송된 데이터의 속도 (초당 비트 단위로 측정)
	- Bandwidth, Latency and Packet loss -> Throughput에 직접적인 영향을 미침
- Packet Per Second, PPS(초당 패킷수): 초당 처리되는 패킷의 수 
	- 네트워크 하드웨어의 처리 능력을 평가할 때 유용하며, 특히 CPU의 성능과 밀접한 관련
- Maximum Transmission Unit, MTU: 네트워크를 통해 전송할 수 있는 가장 큰 패킷 크기
	- Bandwidth, Throughput과 밀접한 관계 있음.

### MTU

![400](images/Pasted%20image%2020241222190937.png)
- 대부분의 네트워크에서 표준 MTU는 **1500바이트**입니다. 이는 일반적인 이더넷 네트워크에서 사용되는 기본 설정
- 점보 프레임(Jumbo Frames)
	- 표준 MTU보다 큰 패킷으로, **1500바이트를 초과해 최대 9001바이트**까지 설정할 수 있다 -> *AWS에서는 9001바이트*
	- 주로 데이터 센터, 고속 네트워크, 대량 데이터 전송 환경에서 사용
	- Benfits of using Jumbo frames
		- 패킷 수 감소
		- 더 높은 처리량(Throughput)
			- 패킷 사이즈가 클 수록, Header Overhead 를 줄여서 효율적으로 데이터 전송가능 (패킷 사이즈가 작으면 그만큼 패킷수가 증가하며, 각 패킷마다 헤더를 추가해야 하므로 오버헤드 발생)
		- PPS(Packets Per Second) 제한 극복
			- 네트워크 장비는 초당 처리할 수 있는 패킷 수(PPS)에 물리적인 한계가 있음.
			- MTU를 늘리면 PPS를 늘리지 않고도 처리량을 증가시킬 수 있음.

![500](images/Pasted%20image%2020241222192634.png)
- **DF (Don't Fragment) 비트**: IP 패킷 헤더에 설정된 플래그.
	- DF=1이면 패킷 분할(Fragmentation)이 금지
	- DF=0이면 패킷이 MTU보다 클 경우 분할
- DF=1 으로 설정한 후 1500 MTU로 패킷 전송을 했을때, 
	- 경로 중간에 **MTU가 1000바이트로 제한된 링크**가 존재한다고 하자.
	  이때 DF 설정으로 인해 패킷 분할을 하지 않고, I**ICMP 메시지**를 송신 측으로 반환
	- ICMP 메시지를 수신한 송신 측은 패킷 크기를 1000바이트 이하로 줄임
	- 후 작은 크기의 패킷을 다시 전송하여 경로 상의 모든 장비가 이를 처리할 수 있도록 조정
- ICMP 허용의 중요하다.
	- ICMP가 차단되면 송신 측은 패킷 크기 문제를 감지하지 못하고, 네트워크 성능이 저하되거나 연결이 끊길 수 있다

### Jumbo Frames
- AWS 에서 점보프레임 -> 9001 MTU
- Amazon VPC(Virtual Private Cloud)에서는 **기본적으로 점보 프레임이 활성화**되어 있음
- AWS에서 점보 프레임 지원 범위
	- VPC 내부 통신에서 점보 프레임이 지원됨
	- 하지만, **VPC 외부로 나가는 트래픽**(예: IGW(인터넷 게이트웨이)나 다른 리전 간 VPC 피어링)은 **1500바이트(인터넷 기본 제한)로 제한**되며 점보 프레임을 사용할 수 없음
- 온프레미스 네트워크와의 연동
	- AWS Direct Connect를 사용하여 VPC와 온프레미스 네트워크 간에 점보 프레임을 지원 (not vpn)
- EC2 cluster placement **groups** 내에서 점보프레임 사용하면, 최대 처리랑을 달성할 수 있음.
	- **EC2 Cluster Placement Groups**는 고성능 컴퓨팅(HPC) 또는 대규모 데이터 처리 워크로드를 위해 **물리적으로 가까운 인스턴스를 그룹화**하여 네트워크 지연(Latency)을 최소화하고 대역폭을 최대화할 수 있는 환경을 제공.
	- 따라서, 클러스터 배치 그룹 내에서 제공되는 높은 대역폭을 점보 프레임으로 최대한 활용할 수 있음
- VPC 외부로 나가는 트래픽에 점보 프레임 사용 시 주의
	- 패킷 크기가 1500바이트를 초과하면 **분할(Fragmentation)**되거나,
	- I**P 헤더에 Don't Fragment 플래그가 설정된 경우** 해당 패킷이 **드롭(Drop)**될 수 있음

### MTU on EC2 instances
- MTU는 **인스턴스 유형(InstanceType)**에 따라 달라질 수 있음.
- MTU는 **ENI(Elastic Network Interface)** 레벨에서 정의됨
- **MTU 확인 및 설정 명령어**
	- **Path MTU 확인**:
	    - `tracepath` 명령어를 사용하여 기기와 대상 간의 경로 MTU 확인 가능.
	    - ex. `$ tracepath amazon.com`
	- **인터페이스 MTU 확인**:
	    - `$ ip link show eth0`
	- **인터페이스 MTU 설정 (Linux)**:
	    - `$ sudo ip link set dev eth0 mtu 9001`

### Demo: EC2에서 MTU 확인
- 퍼블릭 서브넷에서 EC2 인스턴스 두 개를 시작하고 하나에 연결.
- 퍼블릭 및 프라이빗 IP로 각각 MTU 확인:
	- `$ tracepath <퍼블릭 IP>` (인터넷 통신) - no jumbo frame - 1500
	- `$ tracepath <프라이빗 IP>` (내부 통신) - jumbo frame - 9001
- 인터페이스의 MTU 확인:
	- `$ ip link show eth0`

### AWS 네트워크에서의 MTU 지원
#### AWS 내부 통신
- **VPC 내**: 점보 프레임 지원 (MTU 9001 바이트)
- **VPC 엔드포인트를 통한 통신**: MTU 8500 바이트 - s3, dynamodb, sqs
- **인터넷 게이트웨이(IGW)**: MTU 1500 바이트
- **같은 리전 내 VPC 피어링**: MTU 9001 바이트
- **다른 리전 간 VPC 피어링**: MTU 1500 바이트

#### 온프레미스 네트워크와의 통신
- **VPN(Virtual Private Network)**:
    - **VGW(Virtual Gateway)**를 통한 VPN: MTU 1500 바이트
    - **Transit Gateway를 통한 VPN**: - Site to Site VPN에서 MTU 1500 바이트 - 점보 프레임 미지원
- **Direct Connect(DX)**:
    - 점보 프레임 지원 (MTU 9001 바이트).
    - **Transit Gateway를 통한 DX**: MTU 8500 바이트 (VPC attachments connected over the Direct Connect)

