### Amazon Personalize 

- 완전 관리형 추천 엔진 서비스 (Fully-managed recommender engine)
- Amazon.com이 실제 사용하는 기술 기반
	- 고도로 정교한 개인화 추천 기능을 고객 애플리케이션에 제공한다
	  
- Personalize의 가장 큰 특징은 **실시간 적응성**
	- 규칙 기반 또는 단순 이력 기반 추천 솔루션과 달리, Personalize는 사용자가 웹사이트나 애플리케이션에서 상호 작용하는 방식에 따라 추천을 동적으로 조정하고 변경되는 고객 행동에 즉각적으로 적응한다. 
	- 사용자는 복잡한 인프라 관리나 ML 모델 개발 없이 **API 접근**을 통해 이 기능을 활용할 수 있다.
	  
- Personalize 모델을 훈련하고 실시간으로 업데이트하기 위해 다양한 유형의 데이터를 서비스에 제공해야 한다
	- **필수 데이터 유형:** 구매 이력, 평점, 장바구니 추가, 인상(impressions) 같은 사용자 상호 작용 데이터는 물론, 카탈로그 정보(catalog) 및 사용자 인구 통계(user demographics) 등이 포함
	- **데이터 입력 방법:** 
		- Amazon S3를 통한 배치 업로드 방식
		- API 통합을 통한 실시간 스트리밍 방식으로 서비스에 피드(feed)
	- **스키마 정의:** Avro 형식의 명시적 스키마 필요 (데이터의 구조를 명시하기 위해)

- JavaScript SDK 또는 API 사용
    
- **주요 API:**
    - `GetRecommendations` → 추천 상품/콘텐츠/유사 아이템
	    - 특정 사용자 ID를 제공했을 때, 해당 사용자에게 가장 관련성 높은 제품, 콘텐츠 등의 목록을 반환
	    - 특정 항목 ID를 기반으로 이와 **유사한 항목(Similar items)** 목록을 반환하는 데도 활용될 수 있으며, 이는 카탈로그 내 항목 발견 가능성을 향상
    - `GetPersonalizedRanking` → 제공된 항목 리스트를 개인화 순위로 재정렬 (편집/큐레이션 가능)
	    - 사용자가 이미 제공한 항목 목록(예: 검색 결과, 프로모션 목록, 큐레이션 목록)을 입력으로 받아, 해당 목록을 특정 사용자에게 예측된 관심도에 따라 **순위를 재조정(re-rank)**하여 반환
        
- 콘솔 및 CLI에서도 사용 가능
    

### Amazon Personalize Features (기능)

- 실시간(Real-time) 추천과 배치(Batch) 추천 요청을 모두 지원
- **콜드 스타트 문제 해결:** 데이터가 충분하지 않아 추천 정확도가 낮은 **신규 사용자(New users) 및 신규 항목(New items)에 대한 추천** 을 생성하는 기능을 제공함으로써, 추천 시스템의 고질적인 콜드 스타트 문제를 해결
- 문맥 기반 추천 
	- 사용자가 상호 작용하는 시점의 환경적 맥락(Contextual metadata)을 활용
	- 여기에는 장치 유형(Device type)이나 시간(time)과 같은 데이터가 포함되며, 이는 추천의 정확도와 관련성을 높이는 데 사용
- 비정형 텍스트 입력 가능
	- 항목의 설명이나 제목과 같은 **비정형 텍스트 입력(Unstructured text input)** 을 지원
	- 텍스트 메타데이터에서 추가적인 의미 정보를 추출해 추천의 질을 향상
- 마케팅 캠페인을 위한 지능적 사용자 세분화(Intelligent segmentation)
	- 사용자의 상호 작용을 기반으로 생성

- 비즈니스 규칙 통합 및 콘텐츠 제어
	- 순수한 ML 예측을 넘어, 고객의 비즈니스 목표와 운영 요구사항을 추천 결과에 통합할 수 있는 강력한 제어 기능을 제공
	  
	- 비즈니스 규칙 및 필터 설정 가능: 
	    - 최근 구매한 항목 제외
	    - 프리미엄 콘텐츠 강조하여 노출도를 높이는 방식
	    - 정 카테고리의 결과물이 전체 추천 결과 중 일정 비율을 차지하도록 보장하는 필터
        
	- 프로모션 기능:
	    - 추천 결과에 홍보 콘텐츠 삽입
	    - “지금 인기 급상승(Trending Now)” 등 동적 반영
        
	- 개인화된 순위 및 추천 리스트 구성 가능
    

### Amazon Personalize Terminology (용어)

- **Datasets**: Personalize에 업로드하는 데이터
	- 데이터 준비
	- 사용자(Users), 아이템(Items), 상호작용(Interactions)
    
- **Recipes (레시피)** : AWS가 제공하는 사전 구성된 ML 알고리즘 (알고리즘 선택)
    - `USER_PERSONALIZATION`: 사용자별 개인화 추천
    - `PERSONALIZED_RANKING`: 제공된 항목을 개인화 순위로 정렬
    - `RELATED_ITEMS`: 유사 아이템 추천
    - `USER_SEGMENTATION`: 사용자 그룹화
        
- **Solutions (솔루션)** : 레시피와 파라미터를 결합하여 모델을 훈련하고 목표에 최적화하는 과정
    - 모델 학습 수행
    - 관련성(Relevance)과 추가 목표(가격, 길이 등 수치형 변수)를 함께 최적화
    - 하이퍼파라미터 자동 최적화(HPO) 지원
        
- **Campaigns (캠페인)**: 배포된 솔루션 버전으로, 실시간 추천을 생성하기 위한 전용 트랜잭션 용량(capacity)을 프로비저닝 (모델 배포 및 추론)
    - 학습된 솔루션 버전 배포
    - 실시간 추천을 위한 용량 자동 프로비저닝

- **HPO**: Hyperparameter Optimization (하이퍼파라미터 최적화)
	- 모델 성능 향상을 위해 최적의 파라미터 조합을 자동 탐색하는 과정
        

### Amazon Personalize Hyperparameters (하이퍼파라미터)

- **User-Personalization / Personalized-Ranking 관련 주요 파라미터**  
  해당 레시피들은 순환 신경망(RNN) 기반 기술을 활용하며, 다음과 같은 주요 파라미터를 튜닝할 수 있다.

    - `hidden_dimension`: 
	    - 잠재 요인 차원 수 (모델의 복잡성을 결정하는 은닉층의 크기) (HPO 적용)
    - `bptt (Back-Propagation Through Time)`: 
	    - RNN 훈련 시 역전파가 과거로 전파되는 시간 단계를 제어하는 파라미터
    - `recency_mask`: 
	    - 최근 이벤트에 가중치 부여 여부, 이는 보통 `True` 또는 `False`의 범주형 값
    - `min/max_user_history_length_percentile`: 
	    - 사용자 상호 작용 이력의 최소 및 최대 길이를 백분위로 설정하여, 자동화된 봇(robots)과 같은 비정상적인 사용자를 필터링하는 데 사용
    - `exploration_weight`: 
	    - 0에서 1 사이의 가중치를 가지며, 추천 결과의 **관련성(Relevance)** 과 **탐색(Exploration)** 사이의 균형을 제어
	    - 일반적으로 덜 추천될 수 있는 신규 항목이나 상호 작용이 적은 항목을 포함하여 카탈로그의 발견 가능성을 높이는 데 사용
    - `exploration_item_age_cut_off`: 얼마나 과거 데이터까지 포함할지 결정
        
- **Similar-items 관련** (유사 항목 추천 레시피)
    - `item_id_hidden_dim`, `item_metadata_hidden_dim`: 아이템 메타데이터 학습 차원 수 (HPO 가능)
        

### Maintaining Relevance (추천 정확도 유지)

추천 모델이 최신 사용자 선호도와 카탈로그 변화를 반영하도록 유지하는 것은 성능을 유지하는 데 중요

- 최신 데이터 유지: 주기적인 증분 데이터 가져오기(Incremetal import)
    
- `PutEvents` API로 실시간 사용자 행동 반영
	- 사용자 행동이 발생할 때마다, 실시간으로 상호 작용 데이터를 피드인(feed in)함으로써 모델이 즉각적인 사용자 행동 변화에 대응
    
- 모델 재훈련 **(Retrain the model)** 필요: 모델을 재훈련하여 새로운 솔루션 버전(new solution version)을 생성
    - 기본 2시간마다 자동 업데이트
    - **전체 재훈련 권장 주기:** 매주 한 번은 전체 학습(`trainingMode=FULL`) 권장
        

### Amazon Personalize Security (보안)

- **데이터 격리:** 계정 간 데이터 공유 없음
- **저장 중 암호화 (Encryption at Rest):**
	- AWS KMS(Key Management Service) 키(AWS 관리형 또는 고객 제공형)를 사용
	- 데이터는 고객 리전 내에서 SSE-S3(Server-Side Encryption with Amazon S3 managed keys) 방식으로 암호화 지원
- **전송 중 암호화 (Encryption in Transit):** 전송 중 데이터는 TLS 1.2로 암호화
- IAM 기반 접근 제어
	- S3 버킷 정책을 통해 Personalize 접근 권한 부여 필요
- **모니터링 및 로깅:** 모든 API 호출 및 서비스 활동은 모니터링 및 감사를 위해 기록
	- CloudWatch / CloudTrail을 통한 모니터링 및 로깅 지원
    

### Amazon Personalize Pricing (요금)

- 데이터 수집(ingestion): GB당 과금
- 학습(training): 시간당 과금
	- 훈련 시간은 컴퓨팅 용량(4v CPUs, 8 GiB 메모리 기준) 사용 시간으로 측정
- 추론(inference): TPS(초당 처리량)-시간 단위 과금
- 배치 추천(batch recommendations): 실시간 추천과 달리, 배치 추천은 요청된 사용자 수 또는 항목 수에 따라 요금 부과
    

---

### Other AWS ML Services (기타 ML 서비스)

**Amazon Textract**

- 문서 OCR + 양식(Form), 표(Table), 필드(Field) 인식 지원
	- 스캔된 문서에서 텍스트, 필기, 레이아웃 요소 및 데이터를 자동으로 추출
	- 단순한 광학 문자 인식(OCR) 기능을 넘어, **양식(forms), 필드(fields), 테이블(tables)**에서 데이터를 식별, 이해 및 추출하는 것이 특징
    

**AWS DeepRacer**

- 강화학습 기반 1/18 스케일 RC 자율주행차
    

**Amazon Lookout 시리즈**

- 센서 데이터 및 비전을 사용하여 산업 장비 및 비즈니스 메트릭의 이상 징후를 자동으로 감지

- **Lookout for Equipment / Metrics / Vision**
    - Equipment: 센서 데이터 이상 감지, 설비 이상 예측
    - Metrics: S3, RDS, Redshift, SaaS 앱 데이터 모니터링하여 이상 감지
    - Vision: 컴퓨터 비전을 사용하여 반도체, 회로 기판 결함 탐지
        

**Amazon Monitron**

- 산업 장비 모니터링 및 **예측 유지보수(predictive maintenance)** 를 위한 End-to-End 시스템
- 산업 기계의 비정상적인 동작을 감지하여 계획되지 않은 다운타임을 줄이는 데 기여
    

**TorchServe & AWS Neuron**

- **TorchServe**: PyTorch용 모델 서빙 프레임워크
	- PyTorch 모델 서빙을 위한 프레임워크로, Facebook(Meta)의 PyTorch 오픈 소스 프로젝트의 일부
		- 따라서 AWS가 붙지 않음
	- 모델 크기나 분포에 관계없이 PyTorch 모델을 다양한 AWS 인스턴스(CPU, GPU, Neuron, Graviton)에 고성능으로 배포하는 데 사용된다
- **AWS Neuron**: Inferentia 칩 전용 SDK (EC2 Inf1 인스턴스용)
	- SageMaker 또는 딥러닝 AMI, 컨테이너 환경 등과 통합됨
    

**AWS DeepComposer**

- AI 기반 음악 작곡용 키보드 (교육용)
- 사용자가 입력한 멜로디를 AI가 전체 곡으로 작곡해 주는 기능을 제공
    

**Amazon Fraud Detector**

- 온라인 결제 사기, 가짜 계정 생성 등 잠재적인 사기 온라인 활동을 식별하는 완전 관리형 서비스
- 고객은 자신의 과거 사기 데이터를 업로드하여 선택한 템플릿에서 **맞춤형 모델**을 구축 가능
- 이 모델은 신규 계정 등록이나 온라인 결제와 같은 시나리오에서 실시간으로 위험을 평가하는 API를 노출
    

**Amazon CodeGuru**

- 자동 코드 리뷰 및 성능/보안 문제 탐지
- Java, Python 지원
    

**Contact Lens for Amazon Connect**

- 고객 지원 콜 센터를 위한 대화 분석 서비스
- 감정 분석, 발화(utterance) 패턴 분석, 주제 탐지, 통화 품질 측정
    

**Amazon Kendra**

- 자연어 기반 엔터프라이즈 검색
- 파일 시스템, SharePoint, 인트라넷, S3 등 다양한 데이터 소스의 데이터를 하나의 검색 가능한 저장소로 결합한다
- 문서 신선도, 조회수 가중치 조정(Relevance tuning)하여 튜닝 가능
- 피드백(thumbs up/down) 기반 학습
    

**Amazon Augmented AI (A2I)**

- ML 예측 결과를 사람 검토(Human-in-the-loop)
	- ML 예측의 신뢰도가 낮거나 정확성이 중요할 때, **인간 검토(Human review)** 워크플로우를 쉽게 구축할 수 있도록 지원
- Amazon Textract, Rekognition, SageMaker와 통합
- Amazon Mechanical Turk 또는 외부 인력 활용 (제3자 공급업체 vendors)