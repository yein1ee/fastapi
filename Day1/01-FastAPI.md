# FastAPI를 활용한 백엔드 개발 - 1일차
- 날짜: 2025-11-24
- 장소: Next Academy 201호
- FastAPI 전반적인 이해+기본 API 구축

## Fast API 에 대하여
FastAPI는 **“현대적인 타입 힌트+비동기+자동 문서화”** 삼박자를 노리고 나온 파이썬 API 프레임워크라고 보면 됩니다.

### 1. FastAPI가 나오게 된 배경

#### 1-1. 시대 흐름: “API-First + 비동기 + 타입 힌트”
2010년대 중반 이후 백엔드는 점점:

- SPA/모바일 중심 → **REST/JSON API** |t 기반 비동기 I/O** 중요
- 규모 커짐 → **정적 타입 / 스키마 기반 개발**로 안정성 요구
- API 문서 자동화 → OpenAPI/Swagger, ReDoc 같은 도구 필수

파이썬 진영에서는 Flask, Django가 강자였지만:
- Flask: 가볍고 자유롭지만, 타입 기반 검증/문서화/비동기 지원은 거의 전부 “확장/서드파티 + 수작업”
- Django+DRF: 강력하지만 “풀스택 + ORM + 템플릿 + admin” 등 거대한 생태계 위에서 돌아가고, 비동기 지원은 늦고, 타이핑/스키마 기반 문서화는 덜 직관적

이 틈에 “타입 힌트를 적극 활용해서, 빠르고, 자동으로 문서화되는 API 프레임워크”를 만들겠다고 나온 게 FastAPI입니다.

#### 1-2. 탄생과 설계 철학
- 개발자: Sebastián Ramírez
- 첫 릴리즈: 2018-12-05
- 핵심 목표:
  - **고성능(High-performance):** Starlette(ASGI) 위에 얹어서 비동기 처리 극대화
  - **타입 기반 개발:** Python type hints + Pydantic으로 데이터 검증·직렬화 자동화
  - **OpenAPI 기반 자동 문서화:** /docs(Swagger UI), /redoc(ReDoc) 자동 생성
  - **개발 생산성:** “코드 한 번 쓰면 → 검증 + 문서 + IDE 자동완성까지 한 번에”

FastAPI 공식 문서에서도 자기소개를 이렇게 합니다:
> “modern, fast (high-performance) web framework for building APIs with Python based on standard Python type hints”

즉, “타입 힌트에 모든 걸 걸어본 API 프레임워크”라고 봐도 됩니다.

### 2. Python 타입 힌트 기반의 데이터 검증 & 직렬화
요청/응답 스키마를 **Pydantic 모델**로 정의하면:
- 자동으로 **입력 검증** (필수/옵션, 정수 범위, 문자열 길이 등)
- **응답 모델 직렬화** (DB 모델 → Pydantic → JSON)
- 스키마가 그대로 **OpenAPI** 스펙으로 반영

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    price: float
    is_offer: bool | None = None

@app.post("/items/")
async def create_item(item: Item) -> Item:
    return item
```
이렇게 타입만 잘 적어두면:
- 이상한 타입 들어오면 자동 422 오류 + 오류 내용 JSON
- Swagger UI / ReDoc에 **필드 설명, 타입, 예제**까지 자동 문서화

### 3. 자동 생성 API 문서 (Swagger UI / ReDoc)
- `/docs` → Swagger UI (Try it out 버튼으로 바로 테스트 가능)
- `/redoc` → ReDoc 기반 문서
- OpenAPI 스펙을 자동 생성하고, 메타데이터(title, description, version 등)도 설정 가능

이 덕분에:
- 프론트/모바일 팀이 **스펙 문서 없이도 /docs 보고 바로 개발** 가능
- 내부 PoC나 사내 API 서버 만들 때 문서 작업 부담이 크게 줄어듦

### 4. ASGI & 비동기 I/O (Starlette + Uvicorn)
- 내부적으로 Starlette(ASGI 프레임워크) + **Uvicorn**(ASGI 서버)를 사용
- `async def` 엔드포인트를 자연스럽게 지원 →
  - 대규모 동시 요청 처리
  - 외부 API 콜/DB I/O 등에서 **컨텍스트 스위칭으로 효율적**
  - 벤치마크 기준으로 Flask/Django보다 **수 배 높은 RPS**를 보여주는 사례 다수

(예: FastAPI 15,000–20,000 req/s vs Flask 2,000–3,000 req/s라는 비교도 있음)

### 5. 의존성 주입(Dependency Injection) 시스템
- 엔드포인트 함수 파라미터에 `Depends`를 사용하는 DI 시스템:

공통 로직(인증, DB 세션, 설정 로딩 등)을 깔끔하게 재사용
테스트 시 mock 주입이 편함
NestJS의 DI, Spring의 DI 느낌을 파이썬스럽게 가져온 것이라고 FastAPI 문서에서도 설명

### 6. “API에 올인”된 설계
- 템플릿, 세션, 인증 등 웹 풀스택 기능은 최소한으로 제공

대신:

- API 중심
- JSON/웹소켓
- OAuth2/OpenID Connect 같은 인증 흐름도 문서 수준에서 잘 정리

그래서 **ML 서빙, 백엔드 API, 마이크로서비스, BFF** 용도로 특히 많이 쓰입니다.

## 타입 힌트에 대하여
> FastAPI가 타입 힌트를 **“핵심 설계 철학”으로 삼은 이유**는 단순히 코드 보기 좋으라고가 아니라, **API 개발의 모든 과정을 자동화·안정화·가속화**하기 위한 전략입니다.

### 1. 타입 힌트는 “데이터 스키마”를 표현하는 가장 깔끔한 방법

API는 기본적으로 **데이터를 규격화해서 주고받는 시스템**입니다.

예를 들어 `/users` 엔드포인트에서 JSON을 받는다고 해봅시다:

```json
{
  "name": "Danny",
  "age": 30
}
```
이 데이터가:
- name → 문자열
- age → 정수

라는 규칙을 **어디선가 명확하게 표현해야 합니다.**

기존 Flask/Django 시절에는:

- JSON Schema 따로 작성
- Swagger 문서 따로 작성
- 유효성 검사 로직 따로 작성

→ 코드·문서·검증이 전부 따로 놀았습니다.

FastAPI는 **Python typing + Pydantic**을 조합해 이 스키마를 딱 한 번 작성하면 아래 작업을 모두 자동화합니다.

### 2. “단 한 번 타입을 적으면 → 검증 + 직렬화 + 문서화 + 자동완성” 4가지가 자동
예:

```python
class User(BaseModel):
    name: str
    age: int
```
이 한 줄로 얻는 이익이 어마무시합니다.

#### ① 요청 데이터 검증 자동
`age: int`라고 적어두면:

- 문자열 들어오면? → ❌ 자동 422 검증 오류
- 누락되면? → ❌ 필수값 누락
- 타입 맞으면? → ✔ 통과

검증 로직을 **한 줄도 안 써도 되는 것**이 핵심입니다.

#### ② 응답 JSON 직렬화 자동
Python 객체 → JSON 변환할 때도 타입을 보고 필드를 자동 변환합니다.

#### ③ Swagger / ReDoc 문서 자동 생성
타입 힌트가 바로 **OpenAPI 스키마로 변환**됩니다.

`/docs` → 자동 API 문서

프론트/모바일 팀은 문서 없이도 개발할 수 있습니다.

**API 개발 속도가 미친 듯이 빨라지는 이유가 여기에 있습니다.**

### 3. 파이썬 타입 힌트는 사실상 “런타임 검증 언어”가 됨
모든 타입 정보를 Pydantic이 받아서:
- strict 타입 검증
- 기본값 처리
- 필드 설명, 예제까지 자동 스키마화

예:
```python
class Item(BaseModel):
    title: str = Field(..., description="이름")
    price: float = Field(gt=0, example=9.99)
```
이를 기반으로 FastAPI가 API 문서를 자동 생성합니다.

즉 타입 힌트 = **데이터 스키마 + 문서의 근간**입니다.

### 4. 비동기(ASGI) + 타입 힌트 → 매우 안정적인 함수 시그니처
FastAPI는 내부적으로 함수를 분석해 자동으로 종속성을 주입합니다.

```python
async def get_user(user_id: int, db: Session = Depends(get_db)):
    ...
```

여기서:
- `user_id: int` → Path/Query에서 int 검증 후 주입
- `db: Session` → DI 주입
- 타입 힌트로 모든 기능이 결정됨

타입 힌트 정보를 바탕으로 FastAPI가 **함수 시그니처만 보고 라우터·파싱·검증을 모두 자동 구성**합니다.

즉, 타입 힌트를 써야 FastAPI의 DI/검증/문서 자동화 기능이 제대로 작동합니다.

### 5. FastAPI 성능 최적화에도 기여
FastAPI는 내부적으로 타입 정보를 이용해:
- 파싱/검증 단계에서 컴파일된 Pydantic 모델 사용
- 러스트 기반 pydantic-core로 초고속 JSON 처리
- 런타임 병목을 최소화

타입 힌트가 없으면 이런 최적화를 할 수 없기 때문에 FastAPI는 타입 힌트를 **필수적인 기반 기술**로 사용합니다.

### 6. “타입 기반 개발 = 버그를 줄이고 팀 생산성을 폭발적으로 올림”
API 개발에서 버그의 70% 이상은 “데이터 타입/스키마 불일치”입니다.
- 프론트에서 string 보냄 → 백엔드 int 기대
- 필드 명 누락
- 값의 범위 벗어남
- 문서의 스키마가 실제 코드와 불일치

타입 힌트를 사용하면:
- **문서 자동화** → API 스펙 mismatch 감소
- **검증 자동화** → 오류 조기 차단
- **자동완성/정적 분석** → 개발 속도 증가
- **테스트 작성 감소** → 검증 로직 안 만들어도 됨

결국 **팀 전체 생산성이 폭증**하는 효과가 있습니다.
