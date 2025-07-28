
### Glue Data Catalog

- AWS 환경 내 모든 데이터 테이블의 메타데이터를 저장하는 중앙 집중식 리포지토리이다.
	- 데이터 레이크의 핵심 구성 요소로, 다양한 AWS 서비스들이 데이터를 쉽게 발견하고 쿼리할 수 있도록 돕는다.
- 모든 테이블의 **스키마 정보 및 버전 관리** 수행
	- 스키마 변경 사항을 추적하고 관리하여 데이터의 변화에 유연하게 대응할 수 있도록 스키마 버전 관리를 지원
- **자동 스키마 추론** 기능 내장
	- 데이터를 스캔하여 자동으로 스키마를 추론하고 카탈로그에 등록하는 
	- 데이터 탐색 및 준비 시간을 크게 단축시키는 데 기여
- **Athena, Redshift Spectrum**과 연동하여 **스키마 및 데이터 자동 인식**
	- Glue Data Catalog에 저장된 스키마 정보를 활용하여 Amazon Athena나 Amazon Redshift Spectrum과 같은 *쿼리 엔진*이 Amazon S3와 같은 데이터 레이크의 데이터를 직접 쿼리할 수 있도록 통합되어 있다. 
	- 데이터 검색 및 분석을 용이
- **Glue Crawler**를 통해 자동으로 카탈로그 생성 가능
    

---

### Glue Data Catalog – Crawlers

- **크롤러(Crawler)** 는 데이터를 스캔하여 스키마와 파티션 정보를 자동 추출하고 Glue Data Catalog에 메타데이터를 생성
- 지원 데이터 포맷: JSON, Parquet, CSV, 관계형 DB 등
- 지원 대상(데이터 소스):
    - Amazon S3
    - Amazon Redshift
    - Amazon RDS
- **예약 실행(Schedule)** 또는 **수동 실행(On-Demand)** 가능
- IAM Role을 통해 접근 권한 부여 필요
    

---

### Glue and S3 Partitions

- **S3 디렉터리 구조에 따라 Glue가 파티션 자동 추출**
- 데이터 레이크 쿼리 용도에 따라 S3 경로 구조를 미리 잘 설계해야 함
- 예시:
    - 시간 기준 조회 → `s3://bucket/dataset/yyyy/mm/dd/device`
	    - 주로 시간 범위별로 데이터를 쿼리한다면 S3 버킷을 연/월/일 순서로 구성하는 것이 효율적
    - 디바이스 기준 조회 → `s3://bucket/dataset/device/yyyy/mm/dd`
	    - 주로 장치별로 쿼리한다면 장치 ID를 먼저 배치하는 것이 유리
- Amazon S3 데이터 레이크에서 데이터 파티션 구성 방식은 AWS Glue Data Catalog를 통한 쿼리 성능 및 비용 효율성에 직접적인 영향을 미친다.
	- 쿼리 엔진은 S3에 저장된 데이터를 쿼리할 때, 파티션 정보를 활용하여 스캔 범위를 줄인다. 
	- 스캔하는 데이터 양이 줄어들면 쿼리 실행 시간이 단축되고
	- 비용 절감으로 이어진다
        
---

### Glue ETL

- 데이터 분석 전에 데이터를 변환, 정제 및 보강하는 데 사용되는 서버리스 추출, 변환, 로드(ETL) 서비스이다.
- ETL 코드를 **Python 또는 Scala**로 자동 생성 (사용자 수정 가능)
- **Spark, PySpark 스크립트 직접 작성** 가능
    
- 결과 저장 대상:
    - Amazon S3
    - JDBC 대상 (RDS, Redshift 등)
    - Glue Data Catalog 
        
- Glue ETL은 서버를 프로비저닝하거나 관리할 필요가 없는 완전 관리형 서비스
	- 사용한 리소스에 대해서만 비용을 지불하므로 비용 효율적
- **ETL 작업은 서버리스 Apache Spark 플랫폼에서 실행 → 인프라 관리 불필요
- **Glue Scheduler**와 **Trigger**로 예약 실행 또는 이벤트 기반 자동 실행 가능

-> AWS Glue ETL의 "완전 관리형" 및 "서버리스 Spark 플랫폼"이라는 특성은 데이터 엔지니어링 팀의 운영 부담을 획기적으로 줄여주며, 인프라 관리 대신 데이터 변환 로직 자체에 집중할 수 있도록 하여 개발 속도와 효율성을 크게 향상시킨다.
전통적인 ETL 환경에서는 Spark 클러스터 설정, 유지보수, 장애 처리, 스케일링 등 인프라 관리에 상당한 시간과 인력이 소요된다.

---

### Glue ETL – Transformations

- 기본 제공 변환 기능:
    - `DropFields`, `DropNullFields` → 특정 필드 또는 null 값을 가진 필드를 제거
    - `Filter`, `Join`, `Map` → 필터링, 조인, 필드 추가/삭제 및 외부 조회 등
        
- **머신러닝 기반 변환**:
    - `FindMatches ML` → 중복/유사 데이터 탐지 (고유키 없어도 가능)
	    - 고유 식별자가 없거나 필드가 정확히 일치하지 않더라도 데이터셋 내에서 중복되거나 일치하는 레코드를 식별함.
        
- **포맷 변환** 지원: CSV, JSON, Avro, Parquet, ORC, XML
- Apache Spark 기반 고급 변환도 가능 (예: K-Means)
    

---

### AWS Glue DataBrew

- **코드를 작성할 필요 없이 데이터를 시각적으로 정리(clean)하고 정규화(normalize)** 가능한 도구
- ML/분석 데이터 준비 시간을 최대 **80% 단축**
- 데이터 소스:
    - S3, Redshift, Aurora, Glue Data Catalog 등
    - 다양한 데이터 소스 지원
- 250개 이상의 내장 변환 기능 제공
	- ETL 작업도 가능
    - 이상치 필터링, 포맷 변경, 잘못된 값 수정 등
    - 변환을 통해 데이터 이상 감지, 형식 변환, 유효하지 않은 값 수정 등의 작업 자동화 가능
- DataBrew는 시각적 인터페이스와 사전 정의된 변환을 통해 이 과정을 간소화하고, 데이터 과학자가 코딩 대신 모델링에 집중할 수 있도록 한다
- https://aws.amazon.com/glue/features/databrew/
- ![](images/Pasted%20image%2020250728223916.png)

---

### AWS Data Stores for Machine Learning

- **Redshift**: Data Warehousing, SQL 분석 
	- 특히 OLAP(Online analytical processing) 분석 전용 DB
	- S3에서 데이터 로드 가능
    - Redshift Spectrum을 사용하여 S3의 데이터 직접 쿼리 가능하여, 데이터 로딩없이 분석이 가능하다.
    - column base
- **RDS, Aurora**: 관계형 DB
	- OLTP(Online Transaction Processing) 
	-  사전 프로비저닝 필요
	- row base
- **DynamoDB**: NoSQL
	- serverless
		- read/write capacity 만 정하면 됨
	- ML 모델 저장에 적합
- **S3**: 객체 스토리지
	- serverless
	- 무제한 용량
	- 대부분 AWS 서비스와 통합
- **OpenSearch**: (이전 ElasticSearch)
	- indexing of data
	- data point 검색
	- Clickstream 분석
- **ElastiCache**: 캐시용, ML 직접용은 아님

---
### AWS Data Pipeline Features

- AWS Data Pipeline은 AWS 서비스의 다양한 목적지로 데이터를 전송할 수 있다.
- 대상: S3, RDS, DynamoDB, Redshift, EMR
- **작업 간 의존성 관리** 하여 올바른 순서로 작업이 실행 되도록 보장한다.
- 실패 시 재시도 및 알림 지원
- 온프레미스 데이터 소스도 사용 가능
- 고가용성 보장
- 코드를 실행하는 컴퓨팅 리소스에 대한 더 많은 제어권을 제공하며, EC2 또는 EMR 인스턴스에 접근할 수 있다. 
	- 컴퓨팅 리소스는 사용자의 계정 내에 생성된다.

### Data Pipeline Example

![](images/Pasted%20image%2020250728223927.png)
- EC2 인스턴스가 Glue 또는 SageMaker와 함께 데이터 처리 수행

### AWS Data Pipeline vs Glue

- AWS Data Pipeline과 AWS Glue의 핵심 차이는 "제어 수준"과 "서비스의 주된 목적"에 있다
- Data Pipeline
	- 데이터 이동 및 오케스트레이션에 중점을 두며 사용자에게 컴퓨팅 리소스에 대한 더 많은 제어권을 제공
	- EC2 또는 EMR 인스턴스에 접근할 수 있도록 허용
- Glue
	- Apache Spark 코드를 실행하는 ETL(Extract, Transform, Load)에 중점을 둔다
	- 서버리스 ETL에 특화되어 인프라 관리 부담을 최소화한다. 

|특징|AWS Glue|AWS Data Pipeline|
|---|---|---|
|**주요 목적**|서버리스 ETL (데이터 변환, 정제, 보강)|데이터 이동 및 오케스트레이션 (다양한 작업 조율)|
|**컴퓨팅 환경**|완전 관리형 서버리스 Spark 플랫폼|사용자의 EC2/EMR 인스턴스에 대한 제어권 제공|
|**리소스 관리**|AWS에서 관리 (사용자는 걱정할 필요 없음)|사용자가 컴퓨팅 리소스에 대해 더 많은 제어|
|**코드 기반**|Apache Spark (Scala, Python)|다양한 스크립트 및 작업 유형 (사용자 정의 가능)|
|**주요 사용 사례**|대규모 데이터 ETL, 데이터 레이크 구축|복잡한 데이터 파이프라인 오케스트레이션, 온프레미스 통합|
|**비용 모델**|사용한 리소스에 대해서만 지불|사용자의 EC2/EMR 인스턴스 비용 + 서비스 비용|


---

### AWS Batch

- 수십만 개의 컴퓨팅 작업을 AWS에서 효율적으로 실행할 수 있도록 지원하는 서비스
- AWS Batch는 배치 작업을 Docker 이미지 형태로 실행한다
- EC2 & Spot 인스턴스를 **동적 프로비저닝**
	- 작업의 볼륨과 요구 사항에 따라 최적의 수량과 유형의 리소스를 결정하여 할당한다.
- 클러스터 관리 불필요 (서버리스)
- 비용: 기본 EC2 인스턴스 사용량에 대해서만 비용을 지불
- **CloudWatch Events**를 사용하여 스케줄링 가능
- **AWS Step Functions**를 사용하여 배치 작업을 오케스트레이션할 수 있다.
    
### AWS Batch vs Glue 

- Glue는 데이터 변환(ETL)에 특화된 반면, Batch는 Docker 이미지를 기반으로 하는 "모든 종류의 컴퓨팅 작업"에 대한 범용 배치 처리 서비스이다
- AWS가 특정 도메인에 최적화된 서비스(Glue) / 광범위한 컴퓨팅 요구 사항을 충족하는 범용 서비스(Batch)
- ETL과 관련 없는 작업의 경우 Batch가 더 나은 선택일 수 있다
- Docker 이미지의 사용은 Batch가 언어나 프레임워크에 구애받지 않고 어떤 코드든 실행할 수 있는 유연성을 제공한다는 것을 의미한다

| 특징           | AWS Glue                     | AWS Batch                                     |
| ------------ | ---------------------------- | --------------------------------------------- |
| **주요 목적**    | 서버리스 ETL (데이터 변환, 정제, 보강)    | 범용 배치 컴퓨팅 (어떤 작업이든 Docker 이미지로 실행)            |
| **컴퓨팅 환경**   | 완전 관리형 서버리스 Spark 플랫폼        | 사용자의 계정에 생성된 EC2/Spot 인스턴스 (Batch가 관리)        |
| **리소스 관리**   | AWS에서 관리 (사용자는 걱정할 필요 없음)    | Batch가 인스턴스 동적 프로비저닝 및 관리                     |
| **코드 기반**    | Apache Spark (Scala, Python) | Docker 이미지 (어떤 언어/프레임워크든 가능)                  |
| **주요 사용 사례** | 대규모 데이터 ETL, 데이터 레이크 데이터 준비  | ML 모델 학습, 시뮬레이션, 과학 컴퓨팅, 이미지 처리 등 비-ETL 작업 () |
| **비용 모델**    | 사용한 리소스에 대해서만 지불             | 기본 EC2 인스턴스 사용량에 대해서만 지불                      |

---

### DMS – Database Migration Service

![300](images/Pasted%20image%2020250728225151.png)

- 데이터베이스를 AWS로 빠르고 안전하게 마이그레이션하도록 돕는 서비스
- 서비스는 탄력적으로 설계되었으며, 스스로 문제를 감지하고 복구할 수 있는 자가 치유 기능을 제공
- 마이그레이션 중에도 소스 DB 사용 가능
- 지원 유형:
    - 동종 마이그레이션 (예: Oracle → Oracle)
    - 이종 마이그레이션 (예: SQL Server → Aurora)
- **CDC(Change Data Capture)**로 실시간 복제 지원
- 복제 작업을 수행하기 위해 EC2 인스턴스를 생성해야 한다
    
### AWS DMS vs Glue

- DMS는 데이터를 '이동'하고 '복제'하는 데 특화되어 있으며 변환 기능이 없는 반면, Glue는 '변환'에 특화되어 있다. 
- DMS의 핵심 기능은 데이터베이스를 한 위치(온프레미스 또는 다른 AWS DB)에서 다른 위치(AWS DB)로 옮기거나 동기화하는 것이다. 이 과정에서 데이터의 구조나 내용은 크게 변경하지 않는다.

|특징|AWS Glue|AWS Database Migration Service (DMS)|
|---|---|---|
|**주요 목적**|서버리스 ETL (데이터 변환, 정제, 보강)|데이터베이스 마이그레이션 및 연속 데이터 복제|
|**데이터 변환**|**수행 (핵심 기능)**|**수행하지 않음** (주로 원본 데이터 유지)|
|**소스/대상**|S3, JDBC, Glue Data Catalog 등|다양한 관계형/NoSQL 데이터베이스 (동종/이종)|
|**실행 환경**|완전 관리형 서버리스 Spark 플랫폼|복제 작업을 위한 EC2 인스턴스 필요|
|**주요 사용 사례**|데이터 레이크 구축, 데이터 웨어하우스 로딩, ML 데이터 준비|온프레미스 DB를 AWS로 마이그레이션, 실시간 DB 복제|
|**상호 보완 관계**|DMS로 마이그레이션된 데이터를 변환하는 데 사용|Glue를 사용하여 변환하기 전에 데이터를 AWS로 이동|
