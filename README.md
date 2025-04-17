# SearcHRight AI Technical Assignment

## 목표

> **_회사, 재직기간, 직무(타이틀) 만 인재 데이터를 기반으로 LLM 을 활용하여 어떤 경험을 했는지, 어떤 역량을 가지고 있는지 추론하기_**

1. 인재 데이터 example 3 건 제공합니다.
2. 그 인재 데이터들의 회사 데이터 (뉴스, 재직자 수 변동, 자본금 변동, 투자 라운드 정보) 제공합니다.
   - 만약 더 필요하다면, 외부 데이터를 추가하거나 합치는 것은 상관 없습니다.
   - 원본 데이터를 변형해도 무방합니다.
   - **하지만 추가 또는 변동한 데이터에 대한 "객관적인 사실" 정보가 필요합니다. (출처 등)**
3. LLM 중 OpenAI 의 token 제공합니다.
   - 혹시 모를 탈취, 이슈, 보안 등의 이유로 토큰 사용 비용은 $20 으로 제한되어 있습니다.
   - 만약 더 필요하다면 아래에 안내된 이메일 또는 안내 받은 유선 번호로 연락 부탁드립니다.
4. 허용된 API (restAPI 기준) 는 아래와 같습니다.
   - `/v1/models`
   - `/v1/audio`
   - `/v1/chat/completions`
   - `/v1/embeddings`
   - `/v1/images`
   - `/v1/moderations`

## 세부 사항

### 1) 기본 input 과 output flow

```mermaid
flowchart LR
    A[인재데이터 - ***json***] --> B[API 요청]
    B --> C[***LLM 기반 경험 판단 시스템***]
    C --> D[경험값 출력]
    D --> J[초기 스타트업 경험<br />빅테크 경험<br />글로벌 런칭 경험<br />기술 리더십 경험<br />B2C/B2B 도메인 경험<br />물류 도메인 경험 ...]
```

#### *"LLM 기반 경험 판단 시스템" 내부 로직*

```mermaid
flowchart LR
    A[인재 데이터 전처리] --> B

    subgraph 벡터 검색 프로세스
    B[벡터 검색 쿼리 생성] --> C[벡터DB 검색 및 문서 추출]
    end

    C --> D[LLM 컨텍스트 구성]
    D --> E[경험 태그 추론]
    E --> F[결과 후처리]
```

### 2) 제한 사항

1. `Flask` / `Django` / `FastAPI` 셋 중 하나를 사용해 주세요.
2. "목표" 를 이루기 위해 어떠한 형태의 비즈니스 로직이어도 상관 없습니다.
3. 핵심 비즈니스 로직에 대한 테스트 코드는 존재해야 합니다. **최소한의 "단위테스트" 를 작성해주세요.**
4. 만약 `MCP` 형태로 구성하거나 또는 `LangChain` 등의 `LLM` 전용 프레임워크를 사용하신다면 README 에 `flow chart` 를 꼭 추가해주세요.
   - 또는 step by step 으로 글로만 작성해도 무방합니다.
5. 어떤 base 모델을 활용하던 상관 없습니다. **하지만 제공되는 토큰은 OpenAI (gpt 계열 모델) 만 제공됩니다.**
6. `docker compose` 에 `pgvector` RDBMS 로 세팅되어 있습니다.
   - Postgres 에서 vector의 similarity search를 하기 위한 extension 이 추가된 (Vector 관련 기능) DBMS 입니다.
   - https://github.com/pgvector/pgvector 참조
7. `docker compose` 를 적극적으로 사용해 주세요.
   - `nginx` 도 세팅되어 있으며 8000 port 로 리버스프록싱 세팅이 기본으로 설정되어 있습니다.
   - API 구현시 이 부분을 적극적으로 사용해 주세요.

### 3) 성공 테스트셋

1. input `talent_ex1.json`

   - output
     - **`상위권대학교`** (연세대학교)
     - **`성장기스타트업 경험`** (토스 16년도 부터 19년까지 투자 규모 2배 이상 성장, 조직 규모 2배 이상 성장)
     - **`리더쉽`** (타이틀, 챕터 리드, 테크 리드 기반으로 추론)
     - **`대용량데이터처리경험`** (네이버 AI가 21년 부터 하이퍼크로바까지 NLP 관련 처리 많이함)

2. input `talent_ex2.json`

   - output
     - **`상위권대학교`** (서울대학교)
     - **`대규모 회사 경험`** (KT 전략기획실, KT 미디어 사업부 성장전략 팀장 경험)
     - **`리더쉽`** (타이틀, CFO, 성장전략 팀장 경험 기반으로 추론)
     - **`IPO`** (밀리의 서재 CFO 재직 당시, 밀리의 서재가 IPO를 함)
     - **`M&A 경험`** (지니뮤직에서 밀리의 서재 인수 리드)

3. input `talent_ex3.json`

   - output
     - **`상위권대학교`** (연세대학교)
     - **`대규모 회사 경험`** (삼성전자, SKT)
     - **`M&A 경험`** (요기요 재직 중 사모펀드에 매각)
     - **`리더쉽`** (타이틀, CPO 경험 다수, 창업)
     - **`신규 투자 유치 경험`** (C level 재직기간 중 회사 신규 투자유치, Kasa Korea, LBox)

### 4) 목표 데이터 셋

- `talent_ex4.json` 을 API body (`payload`) 값으로 요청했을때 아래 output (`response`) 로 돌아와야 합니다.

  - output
    - **`상위권대학교`** (서울대학교)
    - **`대규모 회사 경험`** (삼성전자, 네이버)
    - **`성장기스타트업 경험`** (토스 재직 시 조직 4.5배 확장)
    - **`리더쉽`** (CTO, Director, 팀장 포지션)
    - **`대용량데이터처리경험`** (네이버 하이퍼클로바 개발)
    - **`M&A 경험`** (요기요 재직 중 사모펀드에 매각)
    - **`신규 투자 유치 경험`** (토스 시리즈 F, 엘박스 시리즈 B 투자 유치)

### 5) 평가 척도

- 차례로 가중치의 크기 입니다.

1. 문제 해결 접근 방식

   - 문제 이해 및 해결 방법의 창의성
   - LLM 활용의 효율성 및 적절성
   - 제약 조건(토큰 비용 등) 내에서의 최적화 노력
   - 요구사항 외 추가 기능 구현 및 개선점 제안

2. 목표 데이터셋 성공 여부

   - 기대하는 output 값이 정확하게 추출되는지 여부
   - 인재 데이터의 변화에 따른 추론 결과의 일관성
   - 추론 결과의 정확성과 근거의 타당성

3. LLM application 구조

   - 코드의 모듈화 및 구조화 수준
   - 적절한 디자인 패턴 적용 여부
   - 확장성 및 유지보수 용이성
   - 효율적인 벡터 검색 구현 방식

4. API 설계 및 구현

   - RESTful API 설계 원칙 준수 여부
   - 응답 시간 및 성능 최적화
   - 에러 핸들링 및 예외 처리 방식
   - API 문서화 및 사용 편의성

5. 테스트 코드 품질

   - 단위 테스트의 커버리지 및 완성도
   - 테스트 케이스의 다양성 및 엣지 케이스 고려 여부
   - 테스트 자동화 및 실행 용이성
   - 테스트 코드의 가독성 및 유지보수성

6. 코드 품질 및 완성도
   - 코드 가독성 및 일관된 코딩 스타일
   - 변수명, 함수명 등의 명명 규칙 준수
   - 주석 및 문서화 수준
   - 리소스 관리 및 최적화 수준

## 기본 세팅 정보

- `python` 3.13 이상 버전 사용 (`pyenv`)
- `poetry` 사용 (다른 가상환경, 패키지 매니저로 변경해도 무방하나, 라이브러리 버전에 유의해야 함)
  - `requirements.txt` 는 제공되지 않습니다. `poetry` 를 사용하지 않는다면, `pyproject.toml` 를 참조해주세요.
  - 자유롭게 라이브러리를 추가 및 설치하셔도 좋습니다.

### 1) 사전 데이터 세팅하기

> 가상환경과 docker compose 를 실행한 상태에서 아래 커멘드, 사전 세팅을 F/U 해주셔야 정상 작동 합니다.

1. `example_datas` 경로로 이동한 상태에서 아래 스크립트를 차례로 실행해 주세요.
2. `python ./setup_company_data.py`
3. `python ./setup_company_news_data.py`

### 2) 구현 QnA

1. 사전 데이터에 특정 회사가 없어요!

   - 의도적으로 제외 시킨 회사 정보입니다.
   - 외부 데이터를 전혀 활용하지 않거나, Base Model (LLM) 의 추론에 기대해도 괜찮고 직접 특정 외부 데이터를 직접 넣어서 일부분만 활용해도 괜찮습니다.

2. 사전 데이터에 특정 회사 뉴스데이터가 없어요!

   - 1번에 대한 답변과 같습니다.

3. "성공 테스트셋" 에서 기대되는 결과 그 이상의 값을 output 으로 만들어도 되나요?

   - 네, 테스트셋의 output 은 최소한으로 꼭 판단되어야 하는 tag 에 대한 명시입니다.

4. 요청할때마다 output 이 달라져도 되나요?

   - 네, 하지만 위 2번에서 말한것과 같이 최소한으로 꼭 판단되어야 하는 tag는 동일해야 합니다.

5. 요청의 응답이 json 이어야 하나요?

   - 네!

6. API 의 구현에 대한 제한은 없나요?

   - 없습니다. 다만 RestFul 한지, 낮은 결합도 높은 응집도에 대한 고민이라던지, 테스트 하기 용이한다던지에 대한 평가 척도는 존재합니다.

7. DBMS 의 테이블 구조 변경 / 추가 / 삭제에 대한 제한은 없나요?

   - 초기 데이터, 테이블 세팅만 지켜진다면 제한 사항이 없습니다. 자유롭게 바꿔주세요.

8. Vector DBMS 는 다른 것을 사용해도 되나요?

   - 네, 무방합니다. 하지만 평가를 위해 오픈소스 기반이어야 하며 실행하기 위한 기본 조건, 제한 조건을 README 에 기록해 주세요.

9. docker compose 의 내용을 바꿔도 되나요?

   - 위 8번에 대한 답변과 동일합니다.

10. 사용하는 데이터에 대한 스키마 정의가 되어 있나요?
    - 없습니다. 데이터 해석도 해당 과제에 포함되어 있습니다.
