
AWS 클라우드 환경에서 기계 학습 및 데이터 관련 작업을 수행하는 데 필수적인 핵심 개념들을 체계적으로 정리

#### AWS Step Functions

- 분산 애플리케이션의 구성 요소를 시각적인 워크플로우의 일련의 단계로 조정하는 서버리스 함수 오케스트레이터
- 복잡한 코드를 이해하기 쉬운 상태 및 다이어그램으로 변환하여 워크플로우를 정의하고, 필요에 따라 업데이트 및 수정할 수 있다.
	- **워크플로우 설계 도구**로 사용됨
	- 시각적 흐름도 제공
	- 코드 외부에서의 **오류 처리 및 재시도** 기능
		- 실패한 작업을 자동으로 재시도
		- 특정 유형의 오류에 다르게 반응하며, 지정된 정리 및 복구 코드로 정상적으로 복구
		- 비즈니스 로직(예: 기계 학습 모델 훈련 코드)과 오류 처리 로직을 분리하는 데 중요한 역할을 함
		- 오류 발생 시 전체 워크플로우가 중단되는 대신, Step Functions가 정의된 정책에 따라 자동으로 재시도하거나 대체 경로를 실행
			- 견고성과 안정성 향상
	- 실행 이력 추적(Audit 가능)
		- 워크플로우 실행의 전체 이력을 감사 
		- 실시간 진단 및 대시보드를 제공
		- AWS CloudWatch 및 AWS CloudTrail과 통합하여 모든 실행 상태를 로깅
	- 지정한 시간만큼 **대기(wait)** 가능
	- State Machine 최대 실행 시간: **1년**
		- 장기 실행 프로세스에 적합

#### Step Functions 활용 예시

- **머신러닝 모델 훈련**
	- ![](images/Pasted%20image%2020250804202008.png)
	- ML모델 훈련 워크플로우를 효율적으로 오케스트레이션할 수 있다.
	- 상태 머신은 JSON 기반 Amazon States Language (ASL)를 사용하여 정의되며, 선택된 샘플 프로젝트에 의해 코드가 사전 구성되어 생성 후 편집할 수 있다
	- Flow
		- `StartAt`: "Generate dataset" (데이터셋 생성으로 워크플로우 시작).
		- `Generate dataset`: Lambda 함수(`GENERATE LAMBDA FUNCTION ARN`으로 지정)를 사용하여 데이터셋을 생성하는 `Task` 상태이다. 이 단계가 완료되면 "Train model (XGBoost)" 상태로 전환
		- `Train model (XGBoost)`: Amazon SageMaker를 사용하여 훈련 작업을 생성하는 `Task` 상태. 이 상태는 다음과 같은 파라미터를 포함함
		    - `AlgorithmSpecification`: 훈련 이미지(`SAGEMAKER_TRAINING IMAGE`) 및 입력 모드(`File`)를 정의
		    - `OutputDataConfig`: 모델 출력이 저장될 S3 버킷 경로(`s3://<S3_BUCKET>/models`)를 지정
		    - `StoppingCondition`: 훈련 작업의 최대 실행 시간(86400초, 즉 1일)을 설정
		    - `ResourceConfig`: 훈련 작업에 사용될 인스턴스 수(1) 및 인스턴스 유형(`ml.m4.xlarge`)을 설정
			- 이후 워크플로우는 "Save Model" 및 "Batch transform" 단계를 거쳐 "End" 상태로 진행
- **모델 튜닝 및 하이퍼파라미터 최적화**
	- 동일하게 step function을 통해 튜닝 및 하이퍼파라미터 최적화를 오케스트레이션 할 수 있다.
- **배치 작업 처리** 등 다양한 ML 및 데이터 워크플로우 자동화 가능
    

---

### Data Engineering Pipeline
#### Real-Time Layer
![](images/Pasted%20image%2020250804202809.png)
- 실시간 계층은 데이터가 도착하는 즉시 처리하여 실시간 기계 학습 및 분석을 가능하게 하는 데 중점을 둠
- **Amazon Kinesis** (Data Streams, Analytics)로 실시간 데이터 처리
- **Amazon Kinesis Data Firehose:** 스트리밍 데이터를 데이터 레이크, 데이터 스토어 및 분석 서비스로 안정적으로 로드. Kinesis Data Streams에서 데이터를 받아 Amazon S3로 전달하는 데 사용될 수 있음
- **Amazon S3, EC2, SageMaker**로 저장·분석
	- **S3 (Parquet, ORC, JSON):** Kinesis Data Firehose를 통해 전달된 데이터는 Parquet, ORC, JSON과 같은 다양한 형식으로 S3에 저장될 수 있음
	- **EC2:** Kinesis Data Streams와 연동하여 실시간 처리를 수행할 수 있는 컴퓨팅 자원을 제공
	- **SageMaker (REAL-TIME ML):** 실시간 기계 학습 처리
- **Elasticsearch, Redshift**로 검색·집계
	-  처리된 데이터는 Redshift(데이터 웨어하우징) 또는 Elasticsearch Service(검색 및 분석)로 로드되어 추가적인 분석에 활용
- **AWS Lambda**를 통한 이벤트 기반 처리

#### Video Layer

![](images/Pasted%20image%2020250804202823.png)

- **Kinesis Video Streams**로 영상 수집
- **EC2, SageMaker**를 통해 영상 분석
- **Rekognition**을 통한 객체 인식
	- 이미지 및 비디오 분석(활동 감지, 사람 움직임 이해, 객체, 유명인, 부적절한 콘텐츠 인식)을 수행할 수 있음 
	- 이미지 및 비디오 인식 작업을 위해 사전 훈련되어 있어, 개발자가 딥러닝 파이프라인을 직접 구축할 필요가 없음
- **Kinesis Data Streams, Kinesis Data Firehose, Kinesis Data Analytics:** 
	- 처리된 비디오 데이터는 일반적인 스트리밍 파이프라인으로 전환되어 추가 처리 및 분석을 위해 사용될 수 있음
    
#### Batch Layer

![](images/Pasted%20image%2020250804202832.png)

- **DMS:** 온프레미스 데이터베이스에서 AWS로 데이터를 마이그레이션하는 데 사용
- 소스
	- **DynamoDB / RDS:** 데이터는 DynamoDB(NoSQL) 또는 RDS(관계형 데이터베이스)에 저장될 수 있음
- 전송
	- **Data Pipeline:** 데이터 이동 및 변환을 자동화하는 데 사용
- 변환 및 처리
	- **Glue ETL:** 데이터 변환(Extract, Transform, Load)에 사용되는 서버리스 데이터 통합 서비스. 시각적 인터페이스인 AWS Glue Studio를 통해 ETL 작업을 생성하거나, Jupyter Notebook 기반의 스크립트로 작성 가능
	- **Glue Data Catalog (Crawler):** AWS Glue 크롤러는 데이터를 검색하고 메타데이터 및 스키마로 Data Catalog를 채운다. 이는 데이터에 대한 중앙 집중식 메타데이터 저장소 역할을 한다.
	- **Batch:** 배치 컴퓨팅 워크로드를 실행하는 데 사용. 컨테이너화된 ML, 시뮬레이션 및 분석 워크로드를 계획, 예약, 실행.
- 제어
	- **Step Functions Orchestration:** 전체 배치 처리 워크플로우를 오케스트레이션하고 다양한 서비스를 조정 할 수 있음.
    

#### Analytics layer

![](images/Pasted%20image%2020250804204027.png)

- **EMR (Hadoop / Spark / Hive...):** 대규모 데이터 처리에 사용되며 Hadoop, Spark, Hive와 같은 빅데이터 프레임워크를 지원
- **Amazon Athena Serverless:** 표준 SQL을 사용하여 Amazon S3의 데이터를 직접 분석할 수 있는 서버리스 쿼리 서비스
- **데이터 웨어하우징 (Redshift / Redshift Spectrum):** 데이터 웨어하우스, 운영 데이터베이스 및 데이터 레이크 전반에 걸쳐 구조화 및 반정형 데이터를 쿼리하고 결합할 수 있도록 합니다.
- **시각화 (Amazon QuickSight):** 분석된 데이터에서 대화형 대시보드 및 시각화를 생성하는 데 사용. QuickSight는 확장 가능하고 서버리스이며 기계 학습 기반의 비즈니스 인텔리전스(BI) 서비스


---
#### AWS DataSync

- **온프레미스 → AWS 저장소로 데이터 마이그레이션** 시 사용
	- 대용량 데이터셋을 수백만 개의 파일과 함께 안전하고 신속하게 복사할 수 있도록 설계됨
	- 주요 목적은 활성 데이터를 AWS로 마이그레이션하고, 온프레미스 스토리지 용량 확보를 위해 데이터를 아카이빙
	- 비즈니스 연속성을 위해 AWS로 데이터를 복제하고, 분석 및 처리를 위해 데이터를 클라우드로 전송
- DataSync Agent는 **VM 형태**로 내부 저장소와 연결
	- 온프레미스 내부 스토리지(NFS, SMB, HDFS)에 연결
- 다양한 프로토콜을 지원: 
	- NFS(Network File System)
	- SMB(Server Message Block)
	- HDFS(Hadoop Distributed File System)
- 기능: 암호화, 무결성 검증
	- 데이터가 안전하고 온전하게 도착하도록 보장
- 대상: AWS 스토리지 서비스:
	- Amazon S3
	- Amazon EFS(Elastic File System)
	- Amazon FSx(FSx for Windows File Server, FSx for Lustre 등)

---
####  MQTT (IOT 프로토콜)

- MQTT(Message Queuing Telemetry Transport)
- IoT 기기에서 자주 사용하는 **표준 메시징 프로토콜**
- 주로 스마트 센서, 웨어러블 기기 및 기타 IoT 장치가 제한된 대역폭의 네트워크를 통해 데이터를 효율적으로 전송하고 수신하는 데 사용
- 센서 데이터를 ML 모델로 전송하는 데 적합
- AWS IoT Device SDK
	- AWS IoT Device SDK는 MQTT를 통해 AWS IoT에 연결할 수 있도록 하는 오픈 소스 라이브러리
	- SDK는 C++, Python, JavaScript, Java, Swift, Embedded C 등 다양한 언어로 제공
	- MQTT 또는 MQTT over WebSocket 프로토콜 지원