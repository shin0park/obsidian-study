
### Amazon Kinesis Data Streams

![](images/Pasted%20image%2020250720232117.png)

- **실시간 스트리밍 데이터 수집 및 저장** 서비스
- 데이터 생산자(Producer) → 소비자(Consumer) 구조
- Click Stream, IoT 기기, 지표 및 로그와 같은 실시간 데이터의 Producer로부터 데이터를 받는다.
- 다양한 Consumer: 애플리케이션, Lambda, Flink, Firehose 등 연계 가능
    

**특징:**

- 최대 **365일 데이터 보관 가능**
- 컨슈머가 데이터를 재처리 가능.
- Kinesis가 만료될 때까지는 데이터를 삭제할 수 없다.
- 1MB 이하의 데이터 단위로 처리, 주로 많은 "small" 실시간 데이터를 처리하는 데 사용된다.
- 동일한 "Partition ID"를 가진 데이터에 대한 데이터 순서 보장
- 암호화: **At-rest: KMS**, **In-transit: HTTPS**
- Kinesis Producer Library (KPL)/ Kinesis Client Library (KCL) **라이브러리**로 Producer/Consumer 최적화 가능

Kinesis 용량 모드
- **Provisioned 모드:**
    - 샤드(shard) 수 직접 설정
    - 샤드 1개당: 입력 1MB/s 또는 초당 1000 레코드, 출력 2MB/s
    - 수동 스케일링 
    - 시간당 샤드 단위 비용
        
- **On-demand 모드:**
    - 용량을 프로비저닝하거나 관리할 필요가 없다.
    - 자동 스케일링, 사전 설정 필요 없음
    - 기본 처리량: 4MB/s 또는 4000(또는 초당 4000 레코드)
    - 최근 30일간 피크 처리량에 따라 자동 조절
    - **과금 방식**: 스트림 단위 시간당 요금 + 데이터 입출력량(GB 기준)에 대한 요금
        
---
### Amazon Data Firehose

![](images/Pasted%20image%2020250720232659.png)

- 이전 명칭: Kinesis Data Firehose
- Fully Managed Service
	- 스트리밍 데이터를 Amazon S3, Amazon Redshift, Amazon OpenSearch Service 또는 타사 서비스(Splunk, MongoDB, Datadog, NewRelic 등) 또는 사용자 지정 HTTP 엔드포인트로 로드하는 데 사용되는 완전 관리형 서비스이다.
- AWS IoT, Kinesis Data Streams, Kinesis Agent, CloudWatch Logs & Events, 또는 자체 클라이언트 SDK를 통해 데이터를 수신한다.
    

**특징:**

- Automatic scaling, serverless이며 사용한 만큼만 지불한다.
- **Near Real-Time**
	- 실시간에 가까운 처리, 버퍼 기반 전송 (시간/크기 단위)
- CSV, JSON, Parquet, Avro, Raw Text, Binary 데이터 지원
- Parquet/ORC로의 변환 및 gzip/snappy 압축을 지원
- AWS Lambda를 사용하여 사용자 지정 데이터 변환(예: CSV를 JSON으로)을 수행할 수 있다.


## Kinesis Data Streams vs Firehose 비교

| 항목     | Kinesis Data Streams                   | Amazon Data Firehose                                                      |
| ------ | -------------------------------------- | ------------------------------------------------------------------------- |
| 역할     | 스트리밍 데이터 수집                            | 스트리밍 데이터를 <br>S3 / Redshift / OpenSearch <br>/ 3rd party /custom HTTP로 로드 |
| 실시간 처리 | O (Real-time)                          | O (Near Real-time)                                                        |
| 저장 기간  | 최대 365일                                | 없음                                                                        |
| 재처리 기능 | O                                      | X                                                                         |
| 프로비저닝  | 수동/자동 선택(Provisioned / On-Demand mode) | 완전 자동(Automatic scaling)                                                  |
| 코드 필요  | Producer/Consumer 직접 구현                | 불필요 (Fully managed)                                                       |

---
## Amazon Managed Service for Apache Flink (MSAF)

![](images/Pasted%20image%2020250720234824.png)

- 이전 명칭: Kinesis Data Analytics for Apache Flink
- Flink(Java/Scala/SQL) 기반 실시간 스트림 처리 프레임워크
- AWS의 관리형 클러스터에서 Apache Flink 애플리케이션을 실행
    - 프로비저닝된 컴퓨팅 리소스, 자동 스케일링, 병렬 처리 제공
    - 애플리케이션 백업(체크포인트 및 스냅샷으로 구현)을 지원
	- 모든 Apache Flink 프로그래밍 기능을 사용하여 데이터를 변환 가능
- **Data Firehose와는 연동 불가**
    
### Kinesis Data Analytics + Lambda (람다와의 통합)

- AWS Lambda를 목적지로 가능
- 데이터 후처리(post-processing)에 대한 많은 유연성을 제공
    - 행 집계, 포맷 변환, 데이터 정제, 암호화 등
        
- 다른 AWS 서비스와 연계 가능
    - S3, DynamoDB, Aurora, Redshift, SNS, SQS, CloudWatch 등


![400](images/Pasted%20image%2020250720234809.png)

- 기존 Flink 기반 분석 기능을 확장
- **이제 Python, Scala 지원**
- DataStream API와 Table API(SQL 기반) 모두 제공
- S3를 통해 사용자 정의 Flink 애플리케이션을 로드 가능
- 완전 서버리스로 동작
#### Common use-cases
- **Streaming ETL**
- **지속적인 메트릭 생성**
- **실시간 분석 응답 처리**


## Kinesis Analytics

- **Consume 리소스 기반 과금 (KPU 단위)**
    - 1KPU = 1 vCPU + 4GB 메모리
- 서버리스, 자동 확장
- IAM을 통해 스트리밍 소스/목적지 접근 제어
- 자동 스키마 인식

---
## Kinesis Video Stream

![400](images/Pasted%20image%2020250720234956.png)

- 실시간 **비디오 스트리밍 수집 및 분석**
- Producer: CCTV, AWS DeepLens, RTSP 카메라 등
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
    → SageMaker와 실시간 연동 예시 있음
    

---
### Kinesis Summary – Machine Learning

- **Kinesis Data Stream**: 실시간 머신러닝 애플리케이션 개발
- **Data Firehose**: 대량의 데이터 수집 및 S3 등으로 전달
- **Kinesis Analytics**: 실시간 ETL 및 ML 알고리즘 실행
- **Kinesis Video Stream**: 비디오 기반 ML 애플리케이션 구성