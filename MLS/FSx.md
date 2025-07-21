### Amazon FSx

- **Amazon FSx**는 AWS에서 고성능 제3자 파일 시스템을 실행할 수 있도록 해주는 완전관리형 서비스이다
    
- 지원하는 파일 시스템 종류:
    - **FSx for Windows File Server**
    - **FSx for Lustre**
    - **FSx for NetApp ONTAP**
    - **FSx for OpenZFS**
        

---

### Amazon FSx for Windows File Server

- Windows 기반의 파일 공유 시스템을 AWS에서 완전관리형으로 제공
- **SMB(Server Message Block) 프로토콜, Windows NTFS ** 지원
	- SMB
		- 파일, 프린터, 포트, 시리얼 등 리소스를 네트워크를 통해 공유하기 위한 Windows 기반 프로토콜
		- Microsoft에서 개발했고, **Windows 컴퓨터 간 파일 공유의 핵심 프로토콜**
	- NTFS = New Technology File System
		- Microsoft Windows의 기본 파일 시스템 포맷 (Windows 2000 이후부터 기본)
			- **파일 및 폴더에 대한 권한 설정 (ACL)**
		    - **파일 압축, 암호화, 대용량 파일 지원**
		    - **트랜잭션 로그로 장애 복구 기능**
- **Microsoft Active Directory**와 통합되어 ACL(Access Control List) 및사용자 쿼터 설정 지원
- **Linux EC2 인스턴스**에서도 마운트 가능하여 다양한 컴퓨팅 환경에서 활용 가능
- Microsoft의 분산 파일 시스템 **DFS Namespaces** 지원 → 여러 FS를 하나의 논리적 그룹으로 연결
	- DFS = Distributed File System
	- Namespace = 논리적 경로 구조
	- 여러 파일 서버에 흩어진 폴더를 **하나의 폴더 구조로 통합해서 보여주는 기능**
	- 사용자 입장에서는 `\\회사서버\공유폴더`처럼 보이지만, 실제로는 **여러 서버의 공유폴더가 논리적으로 묶여 있음**
- 성능 확장: **수십 GB/s** 처리량, **수백만 IOPS**, **수백 PB** 지원
	- IOPS = Input/Output Operations Per Second
	- 초당 입출력 작업 처리 횟수 - "스토리지가 초당 몇 번이나 파일 읽기/쓰기 요청을 처리할 수 있느냐?"
    
- **스토리지 옵션** - 워크로드의 특성에 따라 최적의 선택 가능
    - **SSD**: 지연 시간에 민감한 워크로드(데이터베이스, 미디어 처리, 데이터 분석 등)에 적합
	    - 전자식 (플래시 메모리)
	    - 빠른 반응속도, 낮은 지연, 비쌈
    - **HDD**: 광범위한 워크로드(홈 디렉터리, CMS 등)에 적합
	    - 기계식 (디스크 회전)
	    - 용량당 가격이 저렴, 속도 느림
        
- **온프레미스에서도 VPN 또는 Direct Connect를 통해 액세스 가능**
	- 하이브리드 클라우드 환경 구축에 용이
- **Multi-AZ 배포 가능** (고가용성)
- 데이터는 매일 S3로 백업
	- 데이터 안전성

-> 기존 Windows 기반 애플리케이션이 클라우드로 이동할 때 코드 변경이나 복잡한 재구성이 거의 필요 없도록 Windows의 핵심 기능들을 지원함, 고객의 마이그레이션 노력을 크게 줄여줌
    
---

### Amazon FSx for Lustre

- **병렬 분산 파일 시스템**으로 대규모 연산에 최적화
- “Lustre” 이름은 "Linux"와 "cluster"에서 유래
- 주요 활용 분야:
    - 머신러닝, 고성능 컴퓨팅(HPC), 비디오 처리, 금융 모델링, 전자 설계 자동화(EDA)
	    - 극도로 높은 성능과 처리량을 요구하는 것들
- 성능: 최대 수백 GB/초의 처리량, **수백만 IOPS**, **1ms 미만의 지연** 제공
    
- **스토리지 옵션**
    - **SSD**: 낮은 지연, IOPS 집약적 워크로드, 작은 파일/랜덤 액세스에 적합
    - **HDD**: 처리량 중심, 크고 순차적인 파일 작업에 적합
        
- **S3와의 통합 지원**
    - FSx를 통해 S3를 파일시스템처럼 읽을 수 있다.
    - 결과 데이터를 다시 S3로 출력 저장 가능하다.
	    - 대규모 데이터 레이크와의 연동이 매우 효율적
	    - S3는 저렴하고 확장성이 뛰어난 객체 스토리지로, 대규모 데이터 레이크의 기반으로 널리 사용
	    - FSx for Lustre가 S3를 파일 시스템처럼 읽고 쓸 수 있다는 것은, S3에 저장된 방대한 데이터를 Lustre의 고성능 파일 시스템으로 빠르게 가져와 처리하고, 그 결과를 다시 S3에 저장할 수 있음을 의미
        
- 온프레미스 서버에서 VPN 또는 Direct Connect를 통해 사용 가능**

#### FSx for Lustre – 파일 시스템 배포 옵션

- **Scratch File System**
    - 임시 스토리지로 사용
    - 데이터가 복제되지 않음
	    - 파일 서버가 실패하면 데이터가 지속되지 않는다
    - 높은 버스트 성능을 제공
	    - Persistent 파일 시스템보다 6배 더 빠름
	    - TiB당 200MBps의 처리량 제공
    - 주로 단기 처리 및 비용 최적화에 사용
        
- Persistent File System**
    - 장기 스토리지에 사용
    - 데이터가 동일한 가용 영역 내에서 복제된다
    - 장애가 발생한 파일을 몇 분 내에 교체할 수 있다
    - 주로 장기 처리 및 민감한 데이터에 사용

---

### Amazon FSx for NetApp ONTAP

- AWS에서 제공하는 **NetApp ONTAP 파일 시스템의 관리형 서비스**
- 지원 프로토콜: **NFS(Network File System), SMB, iSCSI(Internet Small Computer Systems Interface)**
- ONTAP 또는 NAS(Network Attached Storage) 워크로드를 AWS로 손쉽게 이전 가능
- **호환 환경**: Linux, Windows, MacOS, VMware Cloud on AWS, AppStream 2.0, WorkSpaces, EC2, ECS, EKS
- **스토리지 자동 확장/축소 지원**
- 주요 기능:
    - 스냅샷, 복제, 낮은 비용, 압축 및 데이터 중복 제거 기능을 제공 -> NetApp ONTAP의 풍부한 데이터 관리 기능을 클라우드에서 활용 가능
    - **Point-in-time Clone 기능** → 테스트에 유용
	    - 새로운 워크로드 테스트에 유용한 특정 시점(point-in-time) 즉각적인 Clone 기능도 제공하여 개발 및 테스트 주기를 단축 가능

---

### Amazon FSx for OpenZFS

- AWS에서 제공하는 **OpenZFS 파일 시스템의 관리형 서비스**
- NFS (v3, v4, v4.1, v4.2) 프로토콜과 호환되는 파일 시스템
- 기존 ZFS 기반 워크로드를 AWS로 이전 가능
- **호환 환경**: Linux, Windows, MacOS, VMware Cloud on AWS, AppStream 2.0, WorkSpaces, EC2, ECS, EKS
- 성능: **최대 100만 IOPS**, **0.5ms 미만 지연**
- 기능:
    - 스냅샷, 압축 및 낮은 비용 기능을 제공
    - **Point-in-time Clone 기능** 

