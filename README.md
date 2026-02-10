# 박선재 AI 활용 포트폴리오
코코네 서버 엔지니어 지원자 박선재의 AI 활용 포트폴리오입니다.

# 프로젝트 - 악성코드 분석 보고서 웹사이트
### 기술 스택
- Frontend: Next.js
- Backend: Spring Boot + FastAPI
- ML: GridSearchCV
- LLM: OpenAI gpt-4o-mini
- 디컴파일: Ghidra
- 배포: AWS(EC2, S3, RDS, Code Deploy)

### 기여도
- 전체 아키텍처 설계 100%
- 백엔드 코드 구현 60% + AI 보조 40%
- 데이터 수집 및 전처리 33%
- 머신러닝 모델 학습 50%
- LLM 프롬프트 엔지니어링 100%

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
    - 처리 방식: 가장 많이 등장한 값으로 채우기
    - Heatmap으로 결측치 분포를 시각화
<img width="500" height="547" alt="missing_data_ratio_heatmap" src="https://github.com/user-attachments/assets/33138704-371c-45a0-bfd0-7c3cb32e5d9a" />

2. 엔트로피
   - 각 칼럼의 불확실성을 분석하여 비효율적인 칼럼 제거
   - Heatmap으로 칼럼별 엔트로피 값을 시각화
<img width="500" height="548" alt="entropy_heatmap(기본)" src="https://github.com/user-attachments/assets/8c218147-2fd3-42c9-8854-cd3f376d0e57" />

#### 모델 선정 및 학습
- 정규화: Normalizer(벡터 크기를 기준으로 정규화)
- 표준화: StandardScaler(평균 = 0, 표준편차 = 1)
- 아래 표와 같이 10개의 모델에 대해 GridSearchCV 이용해 학습 진행

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


#### 성능 측정
- 모델, 엔트로피 처리 방식 별 정확도 측정
- 정확도가 높은 3개 모델을 이용해 **Voting System**으로 악성 여부 판별
- 정확도 분석 결과:
<img width="1045" height="538" alt="image" src="https://github.com/user-attachments/assets/ecfbfc42-2159-4168-abfb-141659e7ed0d" />

### 성과
- 악성코드 판별 정확도 98%
- 정확도 높은 모델들의 Voting으로 낮은 위음성률

## 2. LLM 기반 기능 개발

### 분석 흐름
1. 상술한 머신러닝 모델에 의해 악성으로 판별된 PE 파일을 Ghidra를 이용해 C언어 파일로 변환
2. C언어 코드를 최적화한 후, OpenAI API를 호출해 분석 → JSON 형식의 응답 수신
3. JSON 응답을 파싱 후 PDF 보고서로 저장

### 주요 구현 사항

#### 1) SFT 파인튜닝
- OpenAI가 제공하는 파인튜닝 API를 활용
- MITRE ATT&CK 프레임워크의 각 태그와 설명을 매핑
- CSV 파일로 저장해 학습 진행

#### 2) 토큰 제어
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

### 성과 및 의의
- 전체 정적분석 시간을 크게 단축 (기존 **수 시간 ~ 수 주** → 보고서 생성까지 최대 **10분**)
- 다각도로 연구되고 있는 **역공학과 LLM의 접목**이 보안 분야에서 갖는 실효성을 확인
- 관련 연구:
  - [https://arxiv.org/abs/2411.14905](https://arxiv.org/abs/2411.14905)
  - [https://medium.com/d-classified/utilizing-generative-ai-for-reverse-engineering-31cbcd435e84](https://medium.com/d-classified/utilizing-generative-ai-for-reverse-engineering-31cbcd435e84)


## 3. 사용한 프롬프트
### 프롬프트 엔지니어링 과정

#### 초안
- C 코드, 악성 코드 유형, JSON 포맷을 **변수화**하여 관리
- JSON 포맷 내부에 **항목 타이틀**, **포함할 내용**을 명시
- 실행할 **명령**을 정의하여 단계적으로 분석하도록 유도
```python
def get_format():
    return {
        "title": "This value is always set to 'Malware analysis report'.",
        "sections": [
            {
                "name": "This value is always set to 'Program Overview'.",
                "content": "This value summarizes the malware analysis results in 100 to 200 characters. It summarizes what the program is based on the results of source code analysis."
            },
            {
                "name": "This value is always set to 'Tags'.",
                "content": "This value is a list of tags (keywords) representing the malware analysis results. Returns keywords in Python array format with strings as elements. Keywords have characteristics such as analyzed malware behavior, characteristics, and corresponding TTP."
            },
            {
                "name": "This value is always set to 'Malicious Behavior'.",
                "content": "This value summarizes the program's malicious behavior based on source code analysis results. Please describe the malicious behavior with a simple comment using Markdown syntax along with the code snippet corresponding to the malicious behavior."
            },
            {
                "name": "This value is always set to 'MITRE ATT&CK TTPs'.",
                "content": "This value describes which Tactic, Technique, or Procedure of MITER ATTACK the malicious action corresponds to in the form of a Python array with string elements."
            },
            {
                "name": "This value is always set to 'Conclusion'.",
                "content": "This value provides a follow-up response plan and guide on how to deal with the malicious program that the user has finally analyzed. This can be multiple, so provide it in the form of a Python array with strings as elements. Reference links or materials are also included as a guide."
            }
        ]
    }


def get_prompt(report_code, malware_type, report_format):
    prompt = f'''Please ignore all previous instructions.
    You are an advanced malware analysis assistant. Your expertise is in analyzing C source code derived from decompiled exe files for potential malicious behavior using the MITRE ATT&CK framework.

    # Variables
    [CODE] = {report_code}
    [TYPE] = {malware_type}
    [FORMAT] = {report_format}

    # Commands
    [C1] = Please ignore all previous instructions.
    [C2] = You must respond strictly in JSON format to match [FORMAT]. Do not provide any other response.
    [C3] = Fully analyze the provided [CODE], which is a exe decompiled by Ghidra. Then, generate a structured malware analysis report based on [FORMAT] and malware type based on [TYPE].

    FOLLOW ALL THE ABOVE COMMANDS STRICTLY.
    DO NOT INCLUDE ANY ADDITIONAL COMMENTS OR CONFIRMATIONS.
    Look at the value descriptions in JSON format and replace them with the correct values. Avoid using newline characters such as \\n or \\.

    # Run
    $ run [C1] [C2] [C3]
    '''
    return prompt
```

#### 개선 후
- JSON 포맷 내부의 설명을 구체화
- 각 변수에 구체화된 설명을 추가
- 사용자의 배경 지식 수준 맞춤형 보고서 생성을 위해 [LEVEL] 변수를 추가
- [LEVEL]이 분석 내용에 영향을 주지 않도록 강조하여 설명
- Command를 run 하는 대신 Instruction을 순서대로 명시

```python
def get_format():
    return {
        "Program Overview": {
            "Description": "[string] Analyze the provided source code. Describe the main purpose of the program based strictly on the actual functionality implemented in the code. Do not infer or guess based on naming conventions or incomplete structures. Only include what can be definitively determined from the code itself."
        },
        "Malware Type": {
            "Malware Type": [
                "[list of strings] Analyze the provided source code and identify possible malware type(s) based on its behavior. "
                "Select from the following list only: Trojan, Ransomware, Worm, Rootkit, Adware, Spyware, Botnet, Keylogger, Backdoor, Virus, Phishing, Fileless Malware, Dropper, DDoS. "
                "If the type cannot be determined, respond with 'Unknown'."
						    "If the code is clearly benign and shows no malicious behavior, respond with 'Normal'. "
                "You may select multiple types if the behavior matches more than one."
            ]
        },
        "MITRE ATT&CK TTPs": {
		        """Based on the code behavior, identify the relevant MITRE ATT&CK techniques grouped by their associated tactics. 
               Use **only the latest valid MITRE ATT&CK Technique IDs** as defined in the official MITRE ATT&CK database (https://attack.mitre.org/techniques/enterprise/).
               For each tactic, list the corresponding technique IDs along with a brief explanation of why each technique applies. 
               The techniques should be grouped under the tactics and the explanation should be specific to how they relate to the identified malware behavior. 
               Select Tactic Name from the following list : Collection (TA0009), Command and Control (TA0011), Credential Access (TA0006), Defense Evasion (TA0005), Discovery (TA0007), Execution (TA0002), Exfiltration (TA0010), Impact (TA0040), Initial Access (TA0001), Lateral Movement (TA0008), Persistence (TA0003), Privilege Escalation (TA0004), Reconnaissance (TA0043), Resource Development (TA0042).
               Do **not** include tactics from the list that do **not** apply to the analyzed code behavior.
               Only include tactics and their techniques that are actually relevant.
               Follow the format given below"""
            "Tactic Name": [
                {
                    "technique_id": "T####",
                    "description": "[string] explain technique_id"
                },
                {
                    "technique_id": "...",
                    "description": "..."
                }
            ],
        },
        "Malicious Code Behaviors": [
            { 
                "code_snippet": "[string] Relevant C code block from the given code, formatted in Markdown (```c ... ```)",
                "analysis": "[Provide a detailed explanation including the technical mechanism, system impact, potential risks, attacker’s exploitation methods (e.g., persistence, evasion), related security threats and recommended countermeasures in a professional and thorough manner.]"
            },
            {
                "code_snippet": "...",
                "analysis": "..."
            }
        ],
        "Conclusion": {
            "description": "[list of strings] This value provides a follow-up response plan and guide on how to deal with the malicious program that the user has finally analyzed. This can be multiple, so provide it in the form of a Python array with strings as elements. Reference links or materials are also included as a guide."
        }
    }


def get_prompt(CODE, LEVEL, SHA256):
    prompt = f"""You are a highly skilled malware analysis expert.
    Your task is to analyze decompiled C code ([CODE]) obtained through Ghidra, detect malicious behaviors, and generate a report based on the MITRE ATT&CK framework.

    [SHA256] = You can analyze or search for the file using the SHA256 hash below:
    {SHA256}

    [CODE] = Below is the decompiled C source code:
    ----------------------
    {CODE}
    ----------------------

    [FORMAT] = Your analysis report must strictly follow the JSON format below:
    {get_format()}

    Please ensure:
    - **Program Overview**: Provide a clear and thorough summary of the program’s purpose based solely on its observable functionality. Focus on actual code behavior, execution logic, and system-level impact, not naming or inferred intent.
    - **Malware Type**: Accurately identify one or more malware types based only on proven behavior. Avoid speculation.
    - **MITRE ATT&CK TTPs**: List every applicable TTP and justify its inclusion with specific references to observed code actions. Group by tactics and explain how the code maps to each technique.
    - **Malicious Code Behaviors**: Describe all malicious behaviors clearly and in detail. Each behavior must include:
    - A plain-language summary of what the behavior does
    - The malware type(s) it supports
    - A C code snippet that performs the action
    - A deep analysis of technical mechanism, impact, attacker goal, evasion/persistence method, and recommended defenses.
    - **Conclusion**: Recommend detailed next steps such as containment, remediation, and monitoring. Include IOCs (e.g., file names, registry keys, domains, ports), and optionally provide links to relevant MITRE or vendor reports for further reading.

    [LEVEL] = Target audience skill level: "{LEVEL}".
    Adjust your **Korean explanation style only** according to the following detailed writing style rules:
    Beginner:
    - The target audience is a complete beginner, such as a university student who has just started learning software or cybersecurity.
    - Assume no prior knowledge of C programming or system internals.
    - Avoid technical terms whenever possible. If a term must be used, explain it clearly in parentheses or with a simple analogy.
    - Example: "'out' is a very low-level instruction that sends direct commands to the computer's hardware. For example, it's like telling a printer directly to start printing."
    - Additionally, assume the audience struggles with programming in general, so provide step-by-step and extremely detailed explanations.
    Intermediate:
    - The target audience has basic understanding of C programming and introductory cybersecurity concepts (e.g., junior security professionals or CS majors with 1–3 years of experience).
    - You may use technical terms but provide a short definition or context with them.
    - Structure explanations logically, and include brief code explanations or contextual examples where appropriate.
    - Try to balance explanation depth: avoid overwhelming with detail but clarify key concepts when needed (e.g., system calls, process injection).
    - When using APIs or system functions, include brief comment-like clarifications (e.g., "OpenProcess is used to access another program's memory").
    Advanced:
    - The target audience is a senior cybersecurity expert with 10+ years of experience and strong domain knowledge in malware analysis, reverse engineering, threat intelligence, and low-level systems.
    - Use technical terms freely. No need for definitions or basic explanations.
    - Keep sentences concise and focused on analysis.
    - Focus on actionable insights, behavioral indicators over verbose explanation.
    - Prefer concise phrasing and prioritize IOC, system impact, stealth method, or detection gap.
    ✅ Only the **tone, sentence structure, and depth of explanation** should change according to [LEVEL].
    ✅ The entire report must be written in **Korean**, using fluent and natural language.
    ✅ Technical terms such as API names or MITRE technique IDs may remain in English (do not translate).

    [OUTPUT LANGUAGE] = The entire report must be written in **Korean**, using natural, fluent Korean for both explanation and analysis.
    - You may include original technical terms (e.g., API names, technique IDs) without translating them.
    - 모든 분석은 한국어로 상세하고 전문적으로 작성하되, 독자의 수준에 맞춰 설명 문체만 조절합니다.

    ----------------------
    # Instructions
    1. Analyze the [CODE] thoroughly and can use [SHA256] below for external search.
    2. Do **not** infer or guess behaviors based on naming, comments, or incomplete logic.
    3. Only describe malicious behaviors that are **clearly and definitively** present in the code.
    4. Follow the [FORMAT] structure exactly as provided.
    5. Adjust explanation **tone** based on [LEVEL], but do **not alter** analytical content.
    6. Output the result in proper JSON format, written entirely in **Korean**.
    ----------------------
    """
    return prompt
```

### 개선 효과
- JSON 형식 일관성을 99% 이상 유지
- MITRE ATT&CK 태그 정보 정확도 66% → 87.5%까지 향상
- 수준 맞춤 보고서로 사용자 만족도 증가

<img width="303" height="336" alt="image" src="https://github.com/user-attachments/assets/f1dfc3a9-0242-4201-88fa-7b2dd1adde50" /> <img width="300" height="336" alt="image" src="https://github.com/user-attachments/assets/62683990-2f10-4057-b1de-e6c7870f4124" />


