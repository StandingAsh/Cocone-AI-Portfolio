# 박선재 AI 활용 포트폴리오
코코네 서버 엔지니어 지원자 박선재의 AI 활용 포트폴리오입니다.

# 1. AI를 활용해 해결한 문제 또는 자동화 사례
## 문제 상황
- 매주 100개 이상의 고객 문의를 수동으로 분류하는 데 평균 2시간 소요

## AI 활용 방법
- Claude API를 활용한 자동 분류 시스템 구축
- 프롬프트 엔지니어링으로 카테고리 분류 정확도 향상

## 개선 결과
- 처리 시간: 2시간 → 10분 (92% 단축)
- 분류 정확도: 95%


# 2. 실제 사용한 프롬프트 또는 프롬프트 템플릿
## 프롬프트 템플릿

### 초안
"이 코드를 리팩토링해줘"

### 개선 후
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

# 3. AI로 생성한 초기 산출물 vs 본인이 수정한 최종 산출물
## AI 생성 초안
[AI가 생성한 코드 스크린샷 또는 코드 블록]

## 문제점
1. 에러 핸들링 부재
2. 메모리 누수 가능성
3. 비효율적인 알고리즘 사용

## 최종 수정본
[수정 후 코드]

## 주요 수정 사항
- try-except 구문 추가
- 컨텍스트 매니저 활용
- O(n²) → O(n log n) 최적화

# 4. AI를 개발 워크플로에 통합한 사례
## 통합 사례: GitHub Actions + AI 코드 리뷰

### 구조
1. PR 생성 시 자동으로 Claude API 호출
2. 변경사항 분석 및 리뷰 코멘트 생성
3. GitHub API를 통해 자동 코멘트 등록

### 효과
- 코드 리뷰 시간 50% 단축
- 일관된 코드 품질 유지
- 팀원들의 리뷰 부담 경감

[GitHub Actions Workflow 파일 링크]

# 5. AI의 한계를 겪고 해결한 경험
## 문제 상황
Claude가 존재하지 않는 라이브러리 함수를 제안

## 해결 과정
1. AI 응답에 대한 검증 프로세스 구축
2. 공식 문서 자동 참조 시스템 추가
3. 테스트 코드 자동 생성 및 실행

## 결과
- 환각 문제 발생률 80% 감소
- 신뢰도 높은 코드 생성

# 6. AI를 활용한 프로토타입 제작 사례
## 프로토타입: AI 기반 일정 관리 앱

### 개발 기간
48시간 (2일)

### AI 활용 범위
- UI 컴포넌트 초안: Claude 생성 (80%) + 수동 조정 (20%)
- API 엔드포인트 설계: Claude 제안 → 검토 및 수정
- 테스트 코드: GitHub Copilot 활용 (60%)

### 본인 기여
- 아키텍처 설계: 100%
- 비즈니스 로직 구현: 70%
- 통합 및 배포: 100%

[데모 링크] | [GitHub Repository]

# 7. LLM 기반 기능 개발 경험
## 프로젝트: 악성코드 분석 보고서 웹사이트

### 기술 스택
- LLM: OpenAI GPT-4o-mini API
- Backend: Springboot + FastAPI
- Framework: LangChain

### 프로젝트 기여도
- 프롬프트 엔지니어링 (60%)
- 전체 파이프라인 설계 (100%)
- 백엔드 구현 (100%)

### 주요 구현 사항

#### 1. 워크 플로우
1. 사용자가 악성 PE 파일을 업로드
2. Ghidra를 통해 디컴파일 → C 언어 파일(.c) 생성
3. c언어 코드 최적화 후 LLM 분석 요청
4. LLM 답변 파싱 → PDF 보고서 반환


#### 2. 토큰 제어
- 입력 토큰 제한: 최대 128,000 토큰
- 응답 길이 제어: max_tokens=4000
- 컨텍스트 윈도우 관리: 정규식 이용 C 코드 최적화 → 입력 토큰 수 50% 감소

```python
def code_optimizer(c_code):
    functions = extract_function(c_code)
    merge_code = merge_function(functions)
    return merge_code

def extract_function(code):
    pattern = re.compile(r'/\* Function: (\w+) \*/(.*?)\n(?=/\* Function: |\Z)', re.DOTALL)
    functions = pattern.findall(code)

    cleaned_functions = []
    for func_name, func_body in functions:
        cleaned_functions.append((func_name, clean_function_body(func_body)))

    return cleaned_functions

def merge_function(function):
    merge_code = ""
    for func_name, func_body in function:
        merge_code = merge_code + func_body + "\n\n"

    return merge_code
```


#### 4. 비용 통제
- 월 예산: $200
- 캐싱 전략으로 API 호출 30% 감소
- 실제 비용: $150/월 (25% 절감)

### 성과
- 문서 검색 시간: 5분 → 30초
- 사용자 만족도: 4.5/5.0
- ROI: 업무 시간 절감으로 월 $2,000 상당 효과


