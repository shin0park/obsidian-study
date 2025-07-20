
### Amazon Kinesis Data Streams

![](images/Pasted%20image%2020250720232117.png)

- **실시간 스트리밍 데이터 수집 및 저장** 서비스
- 데이터 생산자(Producer) → 소비자(Consumer) 구조
- Click Stream, IoT 기기, 지표 및 로그와 같은 실시간 데이터의 Producer로부터 데이터를 받는다.
- 다양한 Consumer: 애플리케이션, Lambda, Flink, Firehose 등 연계 가능
    

**특징:**

- 최대 **365일 데이터 보관 가능**
	- 이러한 장기 보관 기능은 컨슈머가 과거 데이터를 재처리할 수 있도록 지원
- 컨슈머가 데이터를 재처리 가능
	- 애플리케이션 오류 발생 시 데이터 손실 없이 복구
	- 새로운 분석 모델을 과거 데이터에 적용
- Kinesis에 한 번 저장된 데이터는 설정된 보존 기간이 만료될 때까지 삭제될 수 없다.
	- 데이터의 불변성 특성은 데이터의 무결성을 보장하며, 감사 추적(auditing) 및 규제 준수(compliance) 요구사항을 충족하는 데 기여한다.
- 일반적으로 1MB 이하의 작고 빈번한 실시간 데이터 처리에 최적화
- 동일한 "Partition ID"를 가진 데이터 레코드에 대해서는 데이터 정렬을 보장
- 암호화: **At-rest: KMS**, **In-transit: HTTPS**
	- 저장 데이터(at-rest)는 AWS KMS를 통한 암호화 지원
	- 전송 중 데이터(in-flight)는 HTTPS 프로토콜을 통해 암호화되어 안전한 데이터 전송을 보장
- Kinesis Producer Library (KPL)/ Kinesis Client Library (KCL) **라이브러리**로 최적화된 Producer/Consumer 을 제공

#### Kinesis Capacity Modes

- **Provisioned 모드:**
    - 사용자가 직접 샤드(shard)의 개수를 선택하고 관리
    - 샤드는 Kinesis Data Streams의 기본 처리량 단위
    - 초당 1MB의 데이터 입력 (또는 초당 1000 레코드) 및 초당 2MB의 데이터 출력 용량을 제공
    - 워크로드의 처리량이 예측 가능하고 안정적인 경우에 적합
    - 수동 스케일링 
    - 비용: 프로비저닝된 샤드 수에 따라 시간당 부과
	    - 예측 용이
        
- **On-demand 모드:**
    - 용량을 프로비저닝하거나 관리할 필요가 없다.
    - 자동 스케일링, 사전 설정 필요 없음
    - 기본 처리량: 4MB/s (또는 초당 4000 레코드)
    - 최근 30일간 피크 처리량에 따라 자동 조절
    - 예측 불가능하거나 급변하는 워크로드에 매우 유리
    - **과금 방식**: 스트림 단위 시간당 요금 + 데이터 입출력량(GB 기준)에 대한 요금
        
---
### Amazon Data Firehose

![](images/Pasted%20image%2020250720232659.png)

- 이전 명칭: Kinesis Data Firehose
- 스트리밍 데이터를 다양한 스토리지 및 분석 서비스로 손쉽게 로드하는 데 중점을 둔 서비스
- Fully Managed Service (Serverless)
	- 사용자가 서버 프로비저닝, 패치, 스케일링 등 인프라 관리에 신경 쓸 필요 없이, 데이터 수집 파이프라인 구축에만 집중할 수 있음을 의미. 
	- 사용한 만큼만 비용을 지불하는 종량제 모델
- 이 서비스의 주요 목적은 스트리밍 데이터를 Amazon S3, Amazon Redshift, Amazon OpenSearch Service와 같은 AWS 서비스뿐만 아니라 Splunk, MongoDB, Datadog, New Relic 등 주요 서드파티 파트너 목적지, 그리고 사용자 지정 HTTP 엔드포인트로 손쉽게 로드하는 것이다. 
- Automatic scaling, serverless이며 사용한 만큼만 지불한다.
	- Firehose는 수신되는 데이터 처리량에 따라 자동으로 확장되므로, 예측 불가능한 트래픽 스파이크에도 안정적으로 데이터를 수집할 수 있다.
- **Near Real-Time**
	- Firehose는 "실시간(Real-time)"이 아닌 "준실시간(Near Real-Time)"으로 작동
	- 데이터를 즉시 전송하는 대신, 일정량(크기) 또는 일정 시간 동안 데이터를 모아(버퍼링) 일괄 처리한 후 목적지로 전송
		- 버퍼링 기능은 목적지 시스템의 부하를 줄이고, 전송 비용 효율성을 높이는 데 기여
	- "준실시간" 처리와 버퍼링 기능은 Firehose의 핵심적인 설계 선택이며, 이는 특정 사용 사례에서 Kinesis Data Streams보다 더 효율적인 솔루션이 될 수도 있다
- CSV, JSON, Parquet, Avro, Raw Text, Binary 데이터 지원
- Parquet/ORC로의 변환 및 gzip/snappy 압축을 지원
- AWS Lambda를 사용하여 사용자 지정 데이터 변환(예: CSV를 JSON으로)을 수행할 수 있다.
	- -> Firehose는 단순한 데이터 전송 서비스를 넘어, 스트리밍 데이터에 대한 경량 ETL(Extract, Transform, Load) 파이프라인의 핵심 구성 요소이다.
	- 특히 Parquet/ORC와 같은 컬럼 기반 형식으로의 변환은 S3 기반의 데이터 레이크에서 분석 쿼리 성능을 비약적으로 향상시키는 중요한 단계
- Firehose 자체는 데이터를 저장하지 않음 
- 컨슈머가 데이터를 재처리(replay)하는 기능을 지원하지 않음
	- 데이터는 목적지로 성공적으로 전달된 후 Firehose 스트림에서 사라짐
- 모든 데이터 또는 전송에 실패한 데이터를 지정된 S3 백업 버킷에 저장하는 기능을 제공하여 데이터 손실 위험을 최소화하고 감사 목적으로 활용할 수 있음



## Kinesis Data Streams vs Firehose 비교

| 항목     | Kinesis Data Streams                   | Amazon Data Firehose                                                      |
| ------ | -------------------------------------- | ------------------------------------------------------------------------- |
| 역할     | 스트리밍 데이터 수집                            | 스트리밍 데이터를 <br>S3 / Redshift / OpenSearch <br>/ 3rd party /custom HTTP로 로드 |
| 실시간 처리 | 실시간 (Real-time)                        | 준실시간 (Near real-time) with buffering                                      |
| 저장 기간  | 최대 365일                                | 데이터 저장 안함                                                                 |
| 재처리 기능 | O                                      | X                                                                         |
| 프로비저닝  | 수동/자동 선택(Provisioned / On-Demand mode) | 완전 자동(Automatic scaling)                                                  |
| 코드 필요  | 프로듀서 & 컨슈머 코드 작성 필요                    | 완전 관리형, 서버리스                                                              |

---
## Amazon Managed Service for Apache Flink (MSAF)

![400](images/Pasted%20image%2020250720234809.png)

- 이전 명칭: Kinesis Data Analytics for Apache Flink
- Flink(Java/Scala/SQL) 기반 실시간 스트림 처리 프레임워크
- 실시간 ETL 및 스트림 데이터에 대한 복잡한 분석을 가능하게 하는 핵심 서비스
- AWS의 관리형 클러스터에서 Apache Flink 애플리케이션을 실행
    - 프로비저닝된 컴퓨팅 리소스, 자동 스케일링, 병렬 처리 제공
    - 애플리케이션 백업(체크포인트 및 스냅샷으로 구현)을 지원
	    - 장애 발생 시에도 데이터 처리의 연속성과 복구 가능성을 보장하여 시스템의 견고성을 높임
	- 모든 Apache Flink 프로그래밍 기능을 사용하여 데이터를 변환 가능
- S3를 통해 사용자 정의 Flink 애플리케이션을 로드 가능
	-  SQL을 사용하는 것 외에도, 직접 Flink 애플리케이션 코드를 개발하여 S3를 통해 MSAF에 로드
- DataStream API 외에 SQL 접근을 위한 Table API도 제공
- 서버리스
-  **Data Firehose와는 연동 불가**
	- Firehose의 주요 목적이 데이터를 최종 목적지로 로드하는 데 있기 때문
    
### Kinesis Data Analytics + Lambda (람다와의 통합)

- AWS Lambda를 목적지로 가능
- 데이터 후처리(post-processing)에 대한 많은 유연성을 제공
    - 행 집계, 포맷 변환, 데이터 정제, 암호화 등
        
- 다른 AWS 서비스와 연계 가능
    - S3, DynamoDB, Aurora, Redshift, SNS, SQS, CloudWatch 등
#### Common use-cases (MSAF는 다양한 실시간 데이터 처리 시나리오에 활용)
- **Streaming ETL**
	- 실시간으로 데이터를 추출, 변환, 로드하는 파이프라인을 구축하여 데이터 레이크나 데이터 웨어하우스로 데이터를 이동
- **지속적인 메트릭 생성**
	- 메트릭을 실시간으로 계산하고 생성하여 모니터링 대시보드나 알림 시스템에 제공
- **실시간 분석 응답 처리**
	-  스트림 데이터에 대한 실시간 ETL을 수행하고, 머신러닝 알고리즘을 직접 적용하여 실시간 예측 또는 이상 감지를 수행
#### Kinesis Analytics
- 소비된 리소스에 대해서만 요금이 부과
- 시간당 소비되는 Kinesis Processing Units (KPU) 단위로 청구된다
    - 1KPU = 1 vCPU + 4GB 메모리
- 서버리스, 자동 확장
- IAM을 통해 스트리밍 소스/목적지 접근 제어
- 스키마 검색 기능을 제공

---
## Kinesis Video Stream

![400](images/Pasted%20image%2020250720234956.png)

- 실시간 **비디오 스트리밍 수집 및 분석**
- Producer: CCTV, AWS DeepLens, RTSP 카메라 등
	- 비디오 스트림당 하나의 Producer가 권장
	- Producer SDK를 사용하여 데이터를 Kinesis Video Stream으로 보낸다
- Consumer: SageMaker, Rekognition, TensorFlow, MXNet 등
- **1시간~10년**까지 보관 가능
- 실시간 분석 또는 머신러닝 프레임워크와 연계 가능


### Kinesis Video Stream 활용 예시

![](images/Pasted%20image%2020250720235030.png)
https://aws.amazon.com/blogs/machine-learning/analyze-live-video-at-scale-in-real-time-using-amazon-kinesis-video-streams-and-amazon-sagemaker/

1. 실시간 스트림 수신
2. Checkpoint를 통해 처리 상태 기록
3. 영상 프레임 디코딩 후 ML 기반 추론 수행
4. 결과 게시
5. SNS 등으로 알림 발송  
    → Amazon Kinesis Video Streams와 Amazon SageMaker를 사용하여 라이브 비디오를 실시간으로 대규모 분석

---
### Kinesis Summary – Machine Learning

- **Kinesis Data Stream**: 실시간 머신러닝 애플리케이션 개발
	- 데이터의 실시간 수집, 보존 및 재처리 기능을 통해 ML 모델의 실시간 추론 및 지속적인 학습을 지원
- **Data Firehose**: 대량의 데이터 수집 및 S3 등으로 전달
	- ML 모델 학습을 위한 대규모 데이터 세트를 효율적으로 데이터 레이크나 데이터 웨어하우스로 로드하는 데 필수적
- **Kinesis Analytics**: 실시간 ETL 및 ML 알고리즘 실행
- **Kinesis Video Stream**: 비디오 기반 ML 애플리케이션 구성
