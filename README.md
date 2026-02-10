# 박선재 AI 활용 포트폴리오
코코네 서버 엔지니어 지원자 박선재의 AI 활용 포트폴리오입니다.

# 프로젝트 1 - 악성코드 분석 보고서 웹사이트
### 기술 스택
- Frontend: Next.js
- Backend: Spring Boot + FastAPI
- ML: XGBoost
- LLM: OpenAI gpt-4o-mini
- 배포: AWS(EC2, S3, RDS, Code Deploy)

### 기여도
- 전체 아키텍처 설계 100%
- 백엔드 코드 구현 60% + AI 보조 40%
- 데이터 수집 및 전처리 33%
- 머신러닝 모델 학습 50%
- LLM 프롬프트 설계 100%

### Github 링크
- [https://github.com/Jae-Ju-Do/backend-fastapi](https://github.com/Jae-Ju-Do/backend-fastapi)

## 1. AI를 활용한 자동화
### 상황
- 악성 프로그램(Malware)을 분석하는 과정은 시간과 노력의 소요가 매우 큼
  - **정적 분석**: 파일의 헤더, 리소스 등을 조사
  - **동적 분석**: 가상 환경에서 실행 후 행위를 모니터링
  - 결과를 종합하여 MITRE ATT&CK 프레임워크에 매
- 이 과정 중 **정적 분석**을 자동화 하고자 함

### AI 활용 방법
- XGBoost 기반 머신러닝 모델로 악성 여부를 우선 검사
- Ghidra를 이용해 C언어로 디컴파일 → OpenAI API를 호출해 디컴파일된 코드 분석

### ML 학습 과정
#### 데이터 수집
- Malware Bazaar에서 악성 PE 파일 다운로드 (총 42만개)
- 헤더 파일 정보 추출 → 칼럼으로 활용
- Virus Total의 Category 정보를 라벨 정보로 활용

#### 데이터 전처리
1. 결측치
    - 누락된 데이터를 식별하고 적절히 보완하여 분석 정확도 향상
    - Heatmap으로 결측치 분포를 시각화
<img width="5453" height="5967" alt="missing_data_ratio_heatmap" src="https://github.com/user-attachments/assets/33138704-371c-45a0-bfd0-7c3cb32e5d9a" />

2. 엔트로피
   - 각 칼럼의 불확실성을 분석하여 비효율적인 칼럼 제거
   - Heatmap으로 칼럼별 엔트로피 값을 시각화
<img width="5440" height="5967" alt="entropy_heatmap(기본)" src="https://github.com/user-attachments/assets/8c218147-2fd3-42c9-8854-cd3f376d0e57" />

#### 모델 선정 및 학습
- 정규화: Normalizer(벡터 크기를 기준으로 정규화)
- 표준화: StandardScaler(평균 = 0, 표준편차 = 1)
- 아래 표와 같이 10개의 모델 선정

| 모델 이름 | 파라미터 값 |
| --- | --- |
| **RandomForest** | `n_estimators: [100, 200]`<br>`max_depth: [None, 10, 20`<br>`min_samples_splict : [2, 5]` |
| **DecisionTree** | `max_depth: [None, 10, 20]`<br>`min_samples_split: [2, 5]` |
| **XGBoost** | `n_estimators: [200, 250, 300]`<br>`learning_rate: [0.15, 0.2, 0.25]`<br>`max_depth: [6, 7, 9]`<br>`subsample: [0.9, 1.0]`<br>`gamma: [0, 0.1, 0.2]`<br>`tree_method: ['hist']` |
| **GradientBoosting** | `n_estimators: [100, 200]`<br>`learning_rate: [0.05, 0.1]`<br>`max_depth: [3, 5]` |
| **LogisticRegression** | `C: [0.1, 1, 10]`<br>`solver: ['liblinear']` |
| **LGBM** | `n_estiimators: [100, 200]`<br>`learning_rate: [0.05, 0.1]`<br>`max_depth: [-1, 10, 20]` |
| **CatBoost** | `iterations: [100, 200]`<br>`learning_rate: [0.05, 0.1]`<br>`depth: [6, 8]` |
| **ExtraTrees** | `n_estimtors: [100, 200]`<br>`max_depth: [None, 10, 20]` |
| **KNN** | `n_neighbors: [3, 5, 7]`<br>`weights: ['uniform', 'distance']` |
| **GaussianNB** | `기본 값만 사용` |
| **Bagging** | `n_estimaotrs: [10, 50]`<br>`max_samples: [0.5, 1.0]` |


### 성능 측정
#### 머신러닝 모델
- 분석 정확도: 96%

#### 보고서 정확성
- Atomic Red Team의 d

### 결과 및 의의
- 처리 시간: 2시간 → 10분 (92% 단축)
- 분류 정확도: 95%

## 2. LLM 기반 기능 개발


## 3. 사용한 프롬프트 템플릿
### 프롬프트 템플릿

#### 초안
"이 코드를 리팩토링해줘"

#### 개선 후
"""
다음 Python 코드를 리팩토링해주세요:
- 목표: 가독성 향상 및 성능 최적화
- 제약사항: Python 3.9+ 문법 사용, 타입 힌트 포함
- 스타일: PEP 8 준수

[코드]
"""

## 개선 효과
- 응답 정확도 70% → 95%
- 원하는 형식의 답변 획득률 상승

# 프로젝트 2 - 수의과 마취 기록 보조 앱
## 1. AI를 활용한 프로토타입 제작 사례

## 2. AI로 생성한 초기 산출물 vs 본인이 수정한 최종 산출물
