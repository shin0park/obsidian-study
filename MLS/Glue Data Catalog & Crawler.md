
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

- **코드를 작성할 필요 없이 데이터를 정리하고 정규화** 가능한 도구
- ML/분석 데이터 준비 시간을 최대 **80% 단축**
- 데이터 소스:
    - S3, Redshift, Aurora, Glue Data Catalog 등
- 250개 이상의 내장 변환 기능 제공
    - 이상치 필터링, 포맷 변경, 잘못된 값 수정 등
    - 변환을 통해 데이터 이상 감지, 형식 변환, 유효하지 않은 값 수정 등의 작업 자동화 가능
- DataBrew는 시각적 인터페이스와 사전 정의된 변환을 통해 이 과정을 간소화하고, 데이터 과학자가 코딩 대신 모델링에 집중할 수 있도록 한다
- https://aws.amazon.com/glue/features/databrew/
        
