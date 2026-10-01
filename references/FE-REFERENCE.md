# Frontend 기능 분석 Reference

> 이 문서는 Frontend 분석 결과의 **출력 형식 Template**이다.
>
> 최종 FE 분석 문서는 이 문서의
> Section 순서, Tree 표현 방식, 표 배치 방식,
> 정보의 상세 수준을 가능한 한 동일하게 유지한다.
>
> 이 문서에 포함된 값은 SAMPLE이며 Evidence가 아니다.
> 실제 결과 작성 시 반드시 현재 분석 대상의
> SCREEN 문서와 실제 Frontend Source를 기준으로 교체한다.

---

# 1. 기능 정보

| 항목 | 내용 |
|---|---|
| 화면명 | SAMPLE |
| Action ID | `ACT-XXX` |
| 기능명 | SAMPLE 기능 |
| UI 유형 | Button |
| Event | Click |
| Handler | `sampleHandler()` |
| 최초 Source | `SamplePage.vue:100-150` |

---

# 2. 기능 요약

사용자가 실행한 Action을 기준으로
입력값을 검증하고 필요한 조건을 판단한 후
Backend Request Parameter를 생성한다.

이후 Backend API를 호출하고,
Response를 Frontend State에 반영하여
최종적으로 화면을 갱신한다.

> 실제 문서에서는 현재 Action의 전체 기능을
> 2~4문장 정도로 요약한다.

---

# 3. 한눈에 보는 기능 흐름

```text
┌──────────────────────────────────────┐
│ ACT-XXX SAMPLE 기능                  │
│ sampleHandler()                      │
└───────────────────┬──────────────────┘
                    ↓
          ┌──────────────────┐
          │ Validation 12개  │
          ├──────────────────┤
          │ 기본조건     4개 │
          │ 기간조건     3개 │
          │ 업무조건     3개 │
          │ 기타         2개 │
          └─────────┬────────┘
                    │
           ┌────────┴────────┐
           │                 │
         실패               성공
           │                 │
           ▼                 ▼
    Message + STOP   ┌──────────────────┐
                     │ Parameter 24개   │
                     ├──────────────────┤
                     │ 화면입력    11개 │
                     │ State        5개 │
                     │ 자동생성     3개 │
                     │ 변환/가공    5개 │
                     └─────────┬────────┘
                               ↓
                     ┌──────────────────┐
                     │ Backend API 1    │
                     │ POST /api/sample │
                     └─────────┬────────┘
                               ↓
                           Response
                               ↓
                             State
                               ↓
                           화면 갱신
                               │
                               │ 조건부
                               ▼
                     ┌──────────────────┐
                     │ Backend API 2    │
                     │ GET /api/status  │
                     └─────────┬────────┘
                               ↓
                           데이터 병합
                               ↓
                           화면 갱신


ERROR
  │
  └──→ Message
       ├─ State 초기화
       └─ 처리 종료
```

## 핵심 정보

| 구분 | 내용 | Source |
|---|---|---|
| Handler | `sampleHandler()` | `SamplePage.vue:100-150` |
| Validation | 총 12개 | `SamplePage.vue:160-220` |
| Parameter | 총 24개 | `sampleParameter.ts:30-100` |
| Backend API | 총 2개 | `sampleService.ts` |
| 주요 Response | `items`, `totalCount` | API Response |
| 주요 State | `sampleList` | Store / Component |
| 화면 반영 | Grid 갱신 | Component |
| Error | Message + State 초기화 | `catch` |

> 한눈에 보는 기능 흐름에서는
> 모든 Validation / Parameter / 조건을 펼치지 않는다.
>
> 항목이 많은 경우 반드시
> **총 개수 + 업무 기준 그룹 + 그룹별 개수**
> 형태로 표현한다.
>
> 상세 항목은 아래 상세 분석에서 모두 제공한다.

---

# 4. 전체 실행 Tree

```text
ACT-XXX SAMPLE 기능
│
└─ Click
   │
   └─ sampleHandler()
      [SamplePage.vue:100-150]
      │
      ├─ 화면 / State 값 조회
      │
      ├─ validateSample()
      │  [SamplePage.vue:160-220]
      │  │
      │  ├─ 기본조건 검증
      │  ├─ 기간조건 검증
      │  ├─ 업무조건 검증
      │  │
      │  ├─ FAIL
      │  │  ├─ Message 표시
      │  │  ├─ return
      │  │  └─ STOP
      │  │
      │  └─ PASS
      │
      ├─ createSampleParameter()
      │  [sampleParameter.ts:30-100]
      │  │
      │  ├─ 화면 입력값 조회
      │  ├─ State 값 조회
      │  ├─ 자동 생성값 설정
      │  ├─ 값 변환 / 가공
      │  └─ Request Parameter 생성
      │
      ├─ sampleStore.search(params)
      │  [sampleStore.ts:80-115]
      │  │
      │  └─ sampleService.search(params)
      │     [sampleService.ts:30-55]
      │     │
      │     └─ POST /api/sample
      │        │
      │        ├─ SUCCESS
      │        │  └─ Response
      │        │     ├─ items
      │        │     │  └─ sampleList
      │        │     └─ totalCount
      │        │
      │        └─ ERROR
      │           └─ Exception 처리
      │
      ├─ 추가 API 호출 조건
      │  │
      │  ├─ false
      │  │  └─ 추가 호출 없음
      │  │
      │  └─ true
      │     └─ sampleService.getStatus()
      │        [sampleService.ts:60-75]
      │        │
      │        └─ GET /api/status
      │           └─ Response
      │              └─ 기존 데이터와 병합
      │
      └─ State 변경
         └─ 화면 갱신
```

> 전체 실행 Tree는 한눈에 보는 기능 흐름과 다르게
> 실제 호출 관계와 중요한 중간 Function을 생략하지 않는다.

---

# 5. 입력 / 출력 요약

| 구분 | 내용 |
|---|---|
| 화면 입력 | 검색조건, 기간, 유형 등 |
| Source State | 검색조건 관련 State |
| Validation | 총 12개 |
| Request Parameter | 총 24개 |
| Backend API | 총 2개 |
| 주요 Response | `items`, `totalCount` |
| Frontend State | `sampleList` |
| 최종 화면 결과 | Grid 갱신 |

> 실제 항목이 많은 경우 이 Section에서는
> 핵심 입력 / 출력만 요약한다.
>
> 전체 항목은 상세 분석에서 제공한다.

---

# 6. 상세 분석

## 6.1 Validation

**총 12개**

### Validation 그룹

| 그룹 | 개수 | 설명 |
|---|---:|---|
| 기본조건 | 4 | 기본 입력 및 상태 확인 |
| 기간조건 | 3 | 시작일 / 종료일 / 기간 관계 |
| 업무조건 | 3 | 기능 수행을 위한 업무 조건 |
| 기타 | 2 | 추가 조건 |
| **합계** | **12** | |

### Validation 상세

| ID | 그룹 | 검증 대상 | 조건 | 실패 처리 | Source |
|---|---|---|---|---|---|
| V-01 | 기본 | 필수값 A | 값 없음 | Message + STOP | `SamplePage.vue:160-165` |
| V-02 | 기본 | 필수값 B | 값 없음 | Message + STOP | `SamplePage.vue:167-172` |
| V-03 | 기본 | 상태값 | 사용 불가 | Message + STOP | `SamplePage.vue:174-180` |
| V-04 | 기본 | 권한 | 조건 불충족 | Message + STOP | `SamplePage.vue:182-188` |
| V-05 | 기간 | 시작일 | 값 없음 | Message + STOP | `SamplePage.vue:190-194` |
| V-06 | 기간 | 종료일 | 값 없음 | Message + STOP | `SamplePage.vue:196-200` |
| V-07 | 기간 | 시작/종료일 | 시작일 > 종료일 | Message + STOP | `SamplePage.vue:202-208` |
| V-08 | 업무 | 업무조건 A | 조건 불충족 | Message + STOP | `SamplePage.vue:210-212` |
| V-09 | 업무 | 업무조건 B | 조건 불충족 | Message + STOP | `SamplePage.vue:213-215` |
| V-10 | 업무 | 업무조건 C | 조건 불충족 | Message + STOP | `SamplePage.vue:216-218` |
| V-11 | 기타 | 기타조건 A | 조건 불충족 | Message + STOP | `SamplePage.vue:219` |
| V-12 | 기타 | 기타조건 B | 조건 불충족 | Message + STOP | `SamplePage.vue:220` |

### Validation 실행 흐름

```text
V-01
 ↓ PASS
V-02
 ↓ PASS
V-03
 ↓
...
 ↓
V-12
 ↓ PASS
Parameter 생성
```

하나라도 실패:

```text
Validation FAIL
        ↓
Message
        ↓
return
        ↓
API 호출 안 함
```

---

## 6.2 조건 / 분기

현재 기능의 실행 결과에 영향을 주는
주요 조건 / 분기를 표현한다.

```text
검색 유형
│
├─ NORMAL
│  └─ 기본 검색조건 사용
│
└─ DETAIL
   └─ 상세 검색조건 추가
```

조건이 많은 경우:

```text
Business 조건: 총 18개

├─ 검색유형 관련     2개
├─ 권한 관련         2개
├─ Parameter 관련    5개
├─ API 호출 관련     3개
└─ 기타              6개
```

상세 조건은 표로 작성한다.

| 조건 | TRUE | FALSE | 영향 |
|---|---|---|---|
| `searchType === "DETAIL"` | 상세조건 추가 | 기본조건 사용 | Parameter |
| `includeStatus === true` | 상태 API 호출 | 호출 안 함 | API |

---

## 6.3 Parameter / 데이터 처리

**총 24개**

### Parameter 구성

```text
Parameter 24개
├─ 화면 입력      11개
├─ State           5개
├─ 자동 생성       3개
└─ 변환 / 가공     5개
```

### Parameter 상세

| Parameter | Source | 원본 값 | 변환 / 가공 | 최종 전달 | 전송 조건 |
|---|---|---|---|---|---|
| `keyword` | 화면 입력 | `" ABC "` | `trim()` | `"ABC"` | 값 존재 |
| `searchType` | State | `"ALL"` | `ALL → null` | 미전송 | 값 존재 시 |
| `plantCode` | State | `"1000"` | 없음 | `"1000"` | 항상 |
| `startDate` | 화면 | Date | 날짜 형식 변환 | `yyyyMMdd` | 값 존재 |
| `endDate` | 화면 | Date | 날짜 형식 변환 | `yyyyMMdd` | 값 존재 |
| ... | ... | ... | ... | ... | ... |

> 실제 분석에서는 확인된 Parameter를 모두 기록한다.
> Parameter가 많다는 이유로 상세 목록을 생략하지 않는다.

### 주요 값 변환

```text
화면
" ABC "
   ↓
trim()
   ↓
"ABC"
   ↓
request.keyword
```

---

## 6.4 State 처리

| State | 변경 전 | 변경 조건 | 변경 후 | 사용 위치 |
|---|---|---|---|---|
| `loading` | `false` | API 호출 시작 | `true` | Loading |
| `sampleList` | 기존값 | API 성공 | `response.items` | Grid |
| `loading` | `true` | API 완료 | `false` | Loading |

현재 Action과 관계없는 State는
나열하지 않는다.

---

## 6.5 Backend API 호출

### API 호출 1 — SAMPLE 조회

| 항목 | 내용 |
|---|---|
| 호출 Function | `sampleService.search()` |
| HTTP Method | `POST` |
| URL | `/api/sample` |
| 호출 조건 | Validation 전체 통과 |
| Request | Parameter 24개 |
| Source | `sampleService.ts:30-55` |

```text
Validation PASS
       ↓
Parameter 생성
       ↓
sampleStore.search()
       ↓
sampleService.search()
       ↓
POST /api/sample
```

### API 호출 2 — 상태 조회

| 항목 | 내용 |
|---|---|
| 호출 Function | `sampleService.getStatus()` |
| HTTP Method | `GET` |
| URL | `/api/status` |
| 호출 조건 | `includeStatus === true` |
| Source | `sampleService.ts:60-75` |

```text
includeStatus
│
├─ false
│  └─ 호출 안 함
│
└─ true
   └─ GET /api/status
```

> 하나의 Action에서 API가 여러 개 발견되면
> 모두 기록한다.

---

## 6.6 Response 처리 / 화면 반영

### 조회 API

```text
POST /api/sample
        ↓
Response
        ↓
response.items
        ↓
sampleList
        ↓
Grid Data
        ↓
화면 갱신
```

### 추가 API가 존재하는 경우

```text
기본 Response
      +
상태 Response
      ↓
데이터 병합
      ↓
sampleList
      ↓
Grid 갱신
```

---

## 6.7 예외 처리

```text
Backend API
│
├─ SUCCESS
│  └─ Response 처리
│     └─ State 변경
│        └─ 화면 갱신
│
└─ ERROR
   └─ catch
      ├─ 오류 Message
      ├─ State 초기화
      └─ 처리 종료
```

공통 Error Handler가 존재하면
현재 Action에서 실제로 연결되는 범위까지만 분석한다.

---

# 7. Source Evidence

| 처리 단계 | Function / Method | Source Path | Line Range |
|---|---|---|---|
| Action 시작 | `sampleHandler()` | `src/views/sample/SamplePage.vue` | 100-150 |
| Validation | `validateSample()` | `src/views/sample/SamplePage.vue` | 160-220 |
| Parameter 생성 | `createSampleParameter()` | `src/utils/sampleParameter.ts` | 30-100 |
| Store 처리 | `search()` | `src/stores/sampleStore.ts` | 80-115 |
| 조회 API | `search()` | `src/api/sampleService.ts` | 30-55 |
| 상태 API | `getStatus()` | `src/api/sampleService.ts` | 60-75 |

Source Evidence는 가능한 경우:

```text
Source Path
+
Function / Method
+
Line Range
```

를 함께 제공한다.

Line Range를 확인할 수 없으면:

`확인되지 않음`

으로 기록한다.

Line Number를 추측하지 않는다.

---

# 8. Backend API 후보

다음 API 규격 분석 단계에서
사용자가 선택할 수 있도록 발견된 API를 정리한다.

| 후보 | 기능 | Method | URL | 호출 조건 |
|---|---|---|---|---|
| 1 | SAMPLE 조회 | `POST` | `/api/sample` | Validation 성공 |
| 2 | 상태 조회 | `GET` | `/api/status` | `includeStatus=true` |

Frontend 분석 단계에서는
최종 `API-001`, `API-002` ID를 부여하지 않는다.

API ID는 API 규격 분석 단계에서 부여한다.

---

# 9. 미확인 항목

| 항목 | 상태 | 사유 |
|---|---|---|
| SAMPLE 항목 | 확인되지 않음 | Source에서 확인할 수 없음 |

미확인 정보가 없다면:

`미확인 항목 없음`

으로 기록한다.

확인되지 않은 내용을
추측하여 채우지 않는다.

---

# 10. 분석 경계

```text
이번 Frontend 분석 범위

Action
  ↓
Handler
  ↓
Validation
  ↓
조건 / 분기
  ↓
Parameter / 데이터 처리
  ↓
Frontend Business Logic
  ↓
Backend API 호출
  ↓
Response
  ↓
State
  ↓
화면 반영

══════════════ STOP ══════════════

Backend Controller 내부
Service / ServiceImpl
Mapper
MyBatis Mapper XML
SQL
Oracle
SAP 내부 처리

→ Backend 영역은 현재 FE 분석에서 분석하지 않음
```

---

# Reference 적용 규칙

이 문서는 단순한 설명 문서가 아니라
**Frontend 분석 결과의 출력 형식 Template**이다.

최종 FE 문서는 가능한 한 이 Reference와
동일한 형태로 작성한다.

반드시 유지할 항목:

1. Section 순서
2. Section 제목
3. `한눈에 보는 기능 흐름` 위치
4. ASCII Flow 표현 방식
5. `핵심 정보` 표
6. `전체 실행 Tree`
7. Validation 그룹 요약 + 상세 표
8. 조건 / 분기 표현
9. Parameter 그룹 요약 + 상세 표
10. State 표
11. API별 상세 표현
12. Response → State → 화면 흐름
13. 예외 처리 Tree
14. Source Evidence 표
15. Backend API 후보 표
16. 미확인 항목
17. 분석 경계

현재 기능에 특정 항목이 존재하지 않는 경우에는
없는 내용을 임의로 생성하지 않는다.

그 경우 해당 Section에:

`해당 처리 없음`

으로 표시할 수 있다.

---

# 대량 항목 표시 규칙

Validation, Parameter, 조건 / 분기가 많아도
`3. 한눈에 보는 기능 흐름`을 과도하게 확장하지 않는다.

상단에서는:

```text
Validation 총 N개
├─ 그룹 A N개
├─ 그룹 B N개
└─ 그룹 C N개
```

형태로 표현한다.

Parameter도 동일하다.

```text
Parameter 총 N개
├─ 화면 입력 N개
├─ State N개
├─ 자동 생성 N개
└─ 변환 / 가공 N개
```

상단 Flow는 가능한 경우
2~3단계 깊이를 유지한다.

그러나 아래 `6. 상세 분석`에서는
확인된 전체 항목을 누락하지 않는다.

즉:

```text
상단 = 한눈에 보기
하단 = 전체 상세
```

원칙을 유지한다.

---

# SAMPLE 데이터 사용 금지

이 Reference에 사용된 다음 값은
오직 출력 형식을 보여주기 위한 SAMPLE이다.

- `ACT-XXX`
- `sampleHandler()`
- `SamplePage.vue`
- `validateSample()`
- `sampleService`
- `/api/sample`
- `/api/status`
- Validation 12개
- Parameter 24개
- 기타 모든 SAMPLE 값

실제 분석 결과에 위 값을 복사하지 않는다.

실제 문서에서는 반드시
현재 SCREEN 문서와
현재 Frontend Source에서 확인된 값으로 교체한다.

Code Index MCP는 Source 탐색 및 호출 관계 확인에 사용한다.

필요한 경우 Chrome DevTools MCP를
Runtime Request / Response 확인에 사용한다.

Reference 자체는 Evidence로 사용하지 않는다.

---

# 최종 출력 원칙

최종 FE 분석 결과는 다음 순서로 읽을 수 있어야 한다.

```text
1~2
기능이 무엇인지 확인
        ↓
3
한눈에 전체 흐름 확인
        ↓
4
실제 실행 순서 확인
        ↓
5~6
입력 / Validation / Parameter /
State / API / Response 상세 확인
        ↓
7
실제 Source 위치 확인
        ↓
8
다음 API 분석 대상 선택
```

개발자가 빠르게 확인하려는 경우에는
`1 → 2 → 3`만 읽어도
기능의 전체 흐름을 이해할 수 있어야 한다.

개발자가 수정 또는 장애 분석을 해야 하는 경우에는
`4 → 6 → 7`을 통해
실제 Source까지 추적할 수 있어야 한다.

분석 상세도를 낮추지 않는다.

**상단은 단순하게,
하단은 상세하게 작성한다.**