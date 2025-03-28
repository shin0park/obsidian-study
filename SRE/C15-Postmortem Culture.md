## 포스트모텀 문화: 실패로부터 배우기

#### 포스트모텀의 필요성
- SRE는 대규모, 복잡한, 분산된 시스템을 다루며, 서비스에 새로운 기능을 추가하고 변경 속도가 빠르다
- 이러한 환경에서는 사고와 중단이 불가피하다.
- 사고 발생 시 원인을 수정하고 서비스를 정상화하지만, 특정 프로세스가 없다면 동일한 사고가 반복될 가능성이 크다.
- 따라서 포스트모텀은 사고를 예방하고 재발방지를 하는 필수적인 도구로 자리잡았다.

#### 포스트 모텀
- 장애의 발생기록과 그 영향
- 장애를 완화하거나 해결하기 위해 수행한 작업
- 장애의 근본 원인
- 향후 재발 방지를 위한 후속조치
들이 기록된 문서

#### 구글의 포스트모텀 철학
목적
- 장애에 대한 내용 문서화
- 근본원인에 대해 이해
- 재발 가능성 줄이는 예방 조치 마련

작성 기준
- 사용자 다운타임 경험했거나 또는 신뢰성이 목표치 이하로 떨어진 경우
- 데이터 손실 발생한 경우
- 비상 대기 엔지니어 개입이 발생한 경우 (롤백, 트래픽 재설정 등)
- 장애해결 시간이 일정 기준 초과
- 모니터링 장애시(장애를 사람이 직접 발견하는 경우)
-> 포스트 모텀을 수행하는 조건을 미리 선정하여 이해하게끔 하는 것이 중요

**blameless 문화**- 서로를 비난하지 않는 포스트 모텀
- 사고 원인을 분석할 때 개인이나 팀의 잘못을 지적하지 않고, 시스템적 문제에 초점을 맞춤
- 실수가 치명적인 결과로 이어지기 쉬운 의료, 항공 산업에서 먼저 이 문화가 생겨남
- "실수"를 시스템 강화의 기회로 보고, 올바른 판단을 내릴 수 있는 절차로 고친다 
- 개발자에 있어 포스트모텀은 취약점을 고칠 수 있는 기회인 동시에 회사 전체를 더 견고하게 만드는 기회

### 협업과 지식의 공유
- 포스트모템 작성은 협업과 지식 공유를 핵심으로 한다.
- 구글은 직접 정의한 템플릿을 이용해 작성한다. 아래의 핵심 기능을 제공하는 것이 중요
	- **실시간 협업:** 데이터와 아이디어를 실시간으로 신속하게 수집가능. 모스트모텀 누서 생성 초기에 유용
	- 열린 댓글/주석 시스템: 집단 지성을 쉽게 활용하며, 그 적용 범위를 넓힘
	- **이메일 알림:** 문서를 함께 편집 중이 사람들에게 메일 알림 및 다른 사람들의 추가의견을 기대할 때 그 사람을 문서 편집자로 초대가능
- 모든 포스트모텀 문서는 반드시 리뷰를 거쳐야한다.
- 작성된 초안은 팀 내부에 먼저 공유하고 시니어 엔지니어의 검토를 거친다. 주요 검토항목은 아래와 같다
	- 나중을 대비해 장애에 대한 핵심데이터를 수집하고 있는가
	- 장애의 영향이 완벽하게 처리되었는가.
	- 장애 근본 원인이 충분히 사려 깊게 분석되었는가?
	- 후속 조치 계획이 적절하며, 버그 수정 작업들의 우선순위가 적절하게 조정되었는가
	- 관련 의사 결정자들에게 이 결과를 공유했는가?
- 검토 완료 후, 더 넓은 엔지니어링 팀 또는 내부 메일링 리스트로 공유


### 포스트모텀 문화 도입하기
- 이달의 포스트 모텀: 흥미롭고 잘 작성된 포스트모텀을 뉴스레터로 공유
- 구글플러스 포스트모텀 그룹: 내부 및 외부 포스트모텀 사례 공유
- 포스츠모첨 리딩 클럽: 흥미롭고 영향력 있는 포스트모텀을 골라 토론
- wheel of misfortune 불행의 바퀴: 과거의 사고를 시뮬레이션하며 하학습
- 우수한 포스트모텀 작성자와 사고 대응자 보상을 지급 - Google의 TGIF(주간 전체 회의)에서 사고 대응 사례를 발표하고 칭찬

### 결론 및 지속적인 개선
- 구글은 포스트모템 문화에 지속적으로 투자하며, 이를 통해 사고를 줄이고 사용자 경험을 개선
- 향후 머신러닝 도입하여 취약점 예측하고 실시간 장애조치를 더 쉽게 하며, 장애 중복 사고를 줄이는 것이 목표