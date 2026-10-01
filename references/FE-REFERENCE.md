# Frontend 기능 분석 Reference

> 이 문서는 Frontend 분석 결과의 출력 형식 Template이다.
>
> 최종 FE 분석 문서는 이 Reference의
> Section 순서, 정보 블록, Tree 표현 방식,
> 상세 수준을 가능한 한 동일하게 유지한다.
>
> 이 문서에 포함된 모든 값은 SAMPLE이다.
> 실제 분석 결과는 현재 SCREEN 문서와
> 실제 Frontend Source를 기준으로 작성한다.


---

# 1. 기능 정보

```text
┌─ 기능 정보 ─────────────────────────────────────
│
│ 화면명       : SAMPLE
│ Action ID    : ACT-XXX
│ 기능명       : SAMPLE 기능
│ UI 유형      : Button
│ Event        : Click
│ Handler      : sampleHandler()
│ 최초 Source  : SamplePage.vue:100-150
│
└─────────────────────────────────────────────────
```


---

# 2. 기능 요약

```text
┌─ 기능 요약 ─────────────────────────────────────
│
│ 사용자가 SAMPLE Action을 실행한다.
│
│ 입력값을 검증하고 필요한 업무 조건을 확인한 후
│ Backend Request Parameter를 생성한다.
│
│ Backend API를 호출하고 Response를
│ Frontend State에 반영한 뒤 화면을 갱신한다.
│
└─────────────────────────────────────────────────
```

실제 문서에서는 현재 Action의 전체 기능을
2~4문장 정도로 요약한다.


---

# 3. 한눈에 보는 기능 흐름

```text
┌──────────────────────────────────────────────┐
│ ACT-XXX SAMPLE 기능                          │
│ sampleHandler()                              │
└─────────────────────┬────────────────────────┘
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
                         추가 호출 조건
                                 │
                      ┌──────────┴──────────┐
                    false                 true
                      │                     │
                      │                     ▼
                      │           ┌──────────────────┐
                      │           │ Backend API 2    │
                      │           │ GET /api/status  │
                      │           └─────────┬────────┘
                      │                     ↓
                      │                 데이터 병합
                      │                     ↓
                      └──────────────→ 화면 갱신


ERROR
  │
  └──→ Message
       ├─ State 초기화
       └─ 처리 종료
```


## 3.1 핵심 정보

```text
┌─ 핵심 정보 ─────────────────────────────────────
│
│ Handler
│   sampleHandler()
│   Source : SamplePage.vue:100-150
│
│ Validation
│   총 12개
│   Source : SamplePage.vue:160-220
│
│ Parameter
│   총 24개
│   Source : sampleParameter.ts:30-100
│
│ Backend API
│   총 2개
│   Source : sampleService.ts
│
│ 주요 Response
│   items
│   totalCount
│
│ 주요 State
│   sampleList
│
│ 화면 반영
│   Grid 갱신
│
│ Error
│   Message
│   State 초기화
│   처리 종료
│
└─────────────────────────────────────────────────
```

### 한눈에 보기 작성 규칙

Validation, Parameter, 조건 / 분기가 많은 경우
모든 항목을 위 Flow에 펼치지 않는다.

반드시 다음 방식으로 요약한다.

```text
Validation 총 N개
├─ 그룹 A N개
├─ 그룹 B N개
├─ 그룹 C N개
└─ 기타 N개
```

Parameter도 동일하게 표현한다.

```text
Parameter 총 N개
├─ 화면 입력      N개
├─ State          N개
├─ 자동 생성      N개
└─ 변환 / 가공    N개
```

상단 Flow는 가능한 경우
2~3단계 깊이를 유지한다.

단, 실행 결과를 크게 변경하는
핵심 분기는 표시한다.

상세 내용은 아래 상세 분석에서
누락 없이 제공한다.


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

전체 실행 Tree에서는
실제 호출 관계와 중요한 중간 Function을
임의로 생략하지 않는다.


---

# 5. 입력 / 출력 요약

```text
┌─ 입력 / 출력 요약 ──────────────────────────────
│
│ [입력]
│
│ 화면 입력
│   ├─ 검색조건
│   ├─ 기간
│   └─ 유형
│
│ Source State
│   └─ 검색조건 관련 State
│
│
│ [처리]
│
│ Validation
│   └─ 총 12개
│
│ Request Parameter
│   └─ 총 24개
│
│ Backend API
│   └─ 총 2개
│
│
│ [출력]
│
│ Response
│   ├─ items
│   └─ totalCount
│
│ Frontend State
│   └─ sampleList
│
│ 화면 결과
│   └─ Grid 갱신
│
└─────────────────────────────────────────────────
```

항목이 많으면 이 Section에서는
핵심 입력 / 출력만 요약한다.

전체 항목은 상세 분석에서 제공한다.


---

# 6. 상세 분석


## 6.1 Validation

```text
┌─ Validation 요약 ───────────────────────────────
│
│ 총 12개
│
│ 기본조건 : 4개
│ 기간조건 : 3개
│ 업무조건 : 3개
│ 기타     : 2개
│
│ 실패
│   └─ Message → return → STOP
│
│ 전체 통과
│   └─ Parameter 생성으로 진행
│
└─────────────────────────────────────────────────
```


### Validation 상세

```text
┌─ V-01 : 필수값 A ──────────────────────────────
│ 그룹   : 기본조건
│ 대상   : 필수값 A
│ 조건   : 값 없음
│ 실패   : Message → STOP
│ Source : SamplePage.vue:160-165
└─────────────────────────────────────────────────

┌─ V-02 : 필수값 B ──────────────────────────────
│ 그룹   : 기본조건
│ 대상   : 필수값 B
│ 조건   : 값 없음
│ 실패   : Message → STOP
│ Source : SamplePage.vue:167-172
└─────────────────────────────────────────────────

┌─ V-03 : 상태값 확인 ───────────────────────────
│ 그룹   : 기본조건
│ 대상   : 상태값
│ 조건   : 사용 불가
│ 실패   : Message → STOP
│ Source : SamplePage.vue:174-180
└─────────────────────────────────────────────────

┌─ V-04 : 권한 확인 ─────────────────────────────
│ 그룹   : 기본조건
│ 대상   : 사용자 권한
│ 조건   : 조건 불충족
│ 실패   : Message → STOP
│ Source : SamplePage.vue:182-188
└─────────────────────────────────────────────────

┌─ V-05 : 시작일 필수 ───────────────────────────
│ 그룹   : 기간조건
│ 대상   : 시작일
│ 조건   : 값 없음
│ 실패   : Message → STOP
│ Source : SamplePage.vue:190-194
└─────────────────────────────────────────────────

┌─ V-06 : 종료일 필수 ───────────────────────────
│ 그룹   : 기간조건
│ 대상   : 종료일
│ 조건   : 값 없음
│ 실패   : Message → STOP
│ Source : SamplePage.vue:196-200
└─────────────────────────────────────────────────

┌─ V-07 : 조회기간 확인 ─────────────────────────
│ 그룹   : 기간조건
│ 대상   : 시작일 / 종료일
│ 조건   : 시작일 > 종료일
│ 실패   : Message → STOP
│ Source : SamplePage.vue:202-208
└─────────────────────────────────────────────────

┌─ V-08 : 업무조건 A ────────────────────────────
│ 그룹   : 업무조건
│ 조건   : 조건 불충족
│ 실패   : Message → STOP
│ Source : SamplePage.vue:210-212
└─────────────────────────────────────────────────

┌─ V-09 : 업무조건 B ────────────────────────────
│ 그룹   : 업무조건
│ 조건   : 조건 불충족
│ 실패   : Message → STOP
│ Source : SamplePage.vue:213-215
└─────────────────────────────────────────────────

┌─ V-10 : 업무조건 C ────────────────────────────
│ 그룹   : 업무조건
│ 조건   : 조건 불충족
│ 실패   : Message → STOP
│ Source : SamplePage.vue:216-218
└─────────────────────────────────────────────────

┌─ V-11 : 기타조건 A ────────────────────────────
│ 그룹   : 기타
│ 조건   : 조건 불충족
│ 실패   : Message → STOP
│ Source : SamplePage.vue:219
└─────────────────────────────────────────────────

┌─ V-12 : 기타조건 B ────────────────────────────
│ 그룹   : 기타
│ 조건   : 조건 불충족
│ 실패   : Message → STOP
│ Source : SamplePage.vue:220
└─────────────────────────────────────────────────
```


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


하나라도 FAIL

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

```text
┌─ 조건 / 분기 요약 ─────────────────────────────
│
│ Business 조건 : 총 18개
│
│ 검색유형 관련    : 2개
│ 권한 관련        : 2개
│ Parameter 관련   : 5개
│ API 호출 관련    : 3개
│ 기타             : 6개
│
└─────────────────────────────────────────────────
```


### 주요 실행 분기

```text
검색 유형
│
├─ NORMAL
│  └─ 기본 검색조건 사용
│
└─ DETAIL
   └─ 상세 검색조건 추가


includeStatus
│
├─ false
│  └─ 상태 API 호출 안 함
│
└─ true
   └─ 상태 API 호출
```


### 조건 상세

```text
┌─ C-01 : 검색 유형 ─────────────────────────────
│ 조건식 : searchType === "DETAIL"
│
│ TRUE
│   └─ 상세 검색조건 추가
│
│ FALSE
│   └─ 기본 검색조건 사용
│
│ 영향
│   └─ Request Parameter
│
│ Source
│   └─ SamplePage.vue:230-245
└─────────────────────────────────────────────────

┌─ C-02 : 상태정보 추가 조회 ────────────────────
│ 조건식 : includeStatus === true
│
│ TRUE
│   └─ Status API 호출
│
│ FALSE
│   └─ 추가 API 호출 없음
│
│ 영향
│   └─ Backend API 호출
│
│ Source
│   └─ SamplePage.vue:250-265
└─────────────────────────────────────────────────
```


---

## 6.3 Parameter / 데이터 처리

```text
┌─ Parameter 요약 ────────────────────────────────
│
│ 총 24개
│
│ 화면 입력      : 11개
│ State          : 5개
│ 자동 생성      : 3개
│ 변환 / 가공    : 5개
│
└─────────────────────────────────────────────────
```


### Parameter 상세

```text
┌─ P-01 : keyword ────────────────────────────────
│ Source    : 화면 입력
│ 원본 값   : " ABC "
│ 변환      : trim()
│ 최종 값   : "ABC"
│ 전송 조건 : 값 존재
└─────────────────────────────────────────────────

┌─ P-02 : searchType ─────────────────────────────
│ Source    : State
│ 원본 값   : "ALL"
│ 변환      : ALL → null
│ 최종 값   : 미전송
│ 전송 조건 : 값 존재 시
└─────────────────────────────────────────────────

┌─ P-03 : plantCode ──────────────────────────────
│ Source    : State
│ 원본 값   : "1000"
│ 변환      : 없음
│ 최종 값   : "1000"
│ 전송 조건 : 항상
└─────────────────────────────────────────────────

┌─ P-04 : startDate ──────────────────────────────
│ Source    : 화면 입력
│ 원본 값   : Date
│ 변환      : yyyyMMdd
│ 최종 값   : 변환된 날짜
│ 전송 조건 : 값 존재
└─────────────────────────────────────────────────

┌─ P-05 : endDate ────────────────────────────────
│ Source    : 화면 입력
│ 원본 값   : Date
│ 변환      : yyyyMMdd
│ 최종 값   : 변환된 날짜
│ 전송 조건 : 값 존재
└─────────────────────────────────────────────────

...

확인된 Parameter를 동일한 형식으로
마지막 Parameter까지 모두 기록한다.
```


### 주요 데이터 변환

```text
화면 입력
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

```text
┌─ State : loading ───────────────────────────────
│
│ 변경 전   : false
│ 변경 조건 : API 호출 시작
│ 변경 후   : true
│ 사용 위치 : Loading UI
│
│ API 완료
│   ↓
│ true → false
│
└─────────────────────────────────────────────────

┌─ State : sampleList ────────────────────────────
│
│ 변경 전   : 기존 목록
│ 변경 조건 : 검색 API 성공
│ 변경 값   : response.items
│ 사용 위치 : Grid
│
└─────────────────────────────────────────────────
```

현재 Action과 관계없는 State는
나열하지 않는다.


---

## 6.5 Backend API 호출


### API 호출 1 — SAMPLE 조회

```text
┌─ API 1 : SAMPLE 조회 ──────────────────────────
│
│ Function : sampleService.search()
│ Method   : POST
│ URL      : /api/sample
│ 조건     : Validation 전체 통과
│ Request  : Parameter 총 24개
│ Source   : sampleService.ts:30-55
│
│ 실행 흐름
│
│ Validation PASS
│       ↓
│ Parameter 생성
│       ↓
│ sampleStore.search()
│       ↓
│ sampleService.search()
│       ↓
│ POST /api/sample
│
└─────────────────────────────────────────────────
```


### API 호출 2 — 상태 조회

```text
┌─ API 2 : 상태 조회 ────────────────────────────
│
│ Function : sampleService.getStatus()
│ Method   : GET
│ URL      : /api/status
│ 조건     : includeStatus === true
│ Source   : sampleService.ts:60-75
│
│ 실행 흐름
│
│ includeStatus
│ │
│ ├─ false
│ │  └─ 호출 안 함
│ │
│ └─ true
│    └─ GET /api/status
│
└─────────────────────────────────────────────────
```

하나의 Action에서 Backend API가 여러 개 발견되면
실제 발견된 API를 모두 기록한다.

Frontend 분석 단계에서는
Backend API 내부 구현으로 진입하지 않는다.


---

## 6.6 Response 처리 / 화면 반영

```text
┌─ Response 처리 ─────────────────────────────────
│
│ POST /api/sample
│       ↓
│ Response
│       ↓
│ response.items
│       ↓
│ sampleList
│       ↓
│ Grid Data
│       ↓
│ 화면 갱신
│
└─────────────────────────────────────────────────
```

추가 API가 존재하는 경우:

```text
┌─ Response 병합 ─────────────────────────────────
│
│ 기본 API Response
│        +
│ 상태 API Response
│        ↓
│ 데이터 병합
│        ↓
│ sampleList
│        ↓
│ Grid 갱신
│
└─────────────────────────────────────────────────
```

중간 데이터 변환이 존재하면
생략하지 않는다.


---

## 6.7 예외 처리

```text
┌─ 예외 처리 ─────────────────────────────────────
│
│ Backend API
│ │
│ ├─ SUCCESS
│ │  └─ Response 처리
│ │     └─ State 변경
│ │        └─ 화면 갱신
│ │
│ └─ ERROR
│    └─ catch
│       ├─ 오류 Message
│       ├─ State 초기화
│       └─ 처리 종료
│
└─────────────────────────────────────────────────
```

공통 Error Handler가 존재하면
현재 Action에서 실제로 연결되는 범위까지만 기록한다.


---

# 7. Source Evidence

```text
┌─ E-01 : Action 시작 ────────────────────────────
│ Function : sampleHandler()
│ Source   : src/views/sample/SamplePage.vue
│ Line     : 100-150
└─────────────────────────────────────────────────

┌─ E-02 : Validation ─────────────────────────────
│ Function : validateSample()
│ Source   : src/views/sample/SamplePage.vue
│ Line     : 160-220
└─────────────────────────────────────────────────

┌─ E-03 : Parameter 생성 ─────────────────────────
│ Function : createSampleParameter()
│ Source   : src/utils/sampleParameter.ts
│ Line     : 30-100
└─────────────────────────────────────────────────

┌─ E-04 : Store 처리 ─────────────────────────────
│ Function : search()
│ Source   : src/stores/sampleStore.ts
│ Line     : 80-115
└─────────────────────────────────────────────────

┌─ E-05 : 조회 API ───────────────────────────────
│ Function : search()
│ Source   : src/api/sampleService.ts
│ Line     : 30-55
└─────────────────────────────────────────────────

┌─ E-06 : 상태 API ───────────────────────────────
│ Function : getStatus()
│ Source   : src/api/sampleService.ts
│ Line     : 60-75
└─────────────────────────────────────────────────
```

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

```text
┌─ Backend API 후보 1 ────────────────────────────
│
│ 기능      : SAMPLE 조회
│ Method    : POST
│ URL       : /api/sample
│ 호출 조건 : Validation 성공
│
└─────────────────────────────────────────────────

┌─ Backend API 후보 2 ────────────────────────────
│
│ 기능      : 상태 조회
│ Method    : GET
│ URL       : /api/status
│ 호출 조건 : includeStatus === true
│
└─────────────────────────────────────────────────
```

Frontend 분석 단계에서는
`API-001`, `API-002` 등의 최종 API ID를 부여하지 않는다.

API ID는 API 규격 분석 단계에서 부여한다.


---

# 9. 미확인 항목

미확인 항목이 존재하는 경우:

```text
┌─ 미확인 항목 1 ─────────────────────────────────
│
│ 항목 : SAMPLE 항목
│ 상태 : 확인되지 않음
│ 사유 : 관련 Source에서 확인할 수 없음
│
└─────────────────────────────────────────────────
```

미확인 항목이 없다면:

```text
┌─ 미확인 항목 ───────────────────────────────────
│
│ 미확인 항목 없음
│
└─────────────────────────────────────────────────
```

확인되지 않은 내용을
추측하여 채우지 않는다.


---

# 10. 분석 경계

```text
┌─ Frontend 분석 범위 ────────────────────────────
│
│ Action
│   ↓
│ Handler
│   ↓
│ Validation
│   ↓
│ 조건 / 분기
│   ↓
│ Parameter / 데이터 처리
│   ↓
│ Frontend Business Logic
│   ↓
│ Backend API 호출
│   ↓
│ Response
│   ↓
│ State
│   ↓
│ 화면 반영
│
├─────────────────────────────────────────────────
│                  STOP
├─────────────────────────────────────────────────
│
│ Backend Controller 내부
│ Service / ServiceImpl
│ Mapper
│ MyBatis Mapper XML
│ SQL
│ Oracle
│ SAP 내부 처리
│
│ → 현재 FE 분석에서 분석하지 않음
│
└─────────────────────────────────────────────────
```


---

# 11. Excel 단일 셀 복사 규칙

이 Reference는 분석 결과의 일부 Section 또는
정보 블록을 사용자가 선택하여
Excel의 **셀 하나에 통째로 복사**할 수 있도록 작성한다.

따라서 최종 FE 문서에서는
Markdown Table을 사용하지 않는다.

다음 형식은 사용하지 않는다.

```text
| 항목 | 내용 | Source |
|------|------|--------|
| ...  | ...  | ...    |
```

대신 다음 형식을 사용한다.

```text
┌─ 제목 ──────────────────────────────────────────
│ 항목 A : 값
│ 항목 B : 값
│ 항목 C : 값
└─────────────────────────────────────────────────
```

목록이 여러 개인 경우:

```text
┌─ ITEM-01 ───────────────────────────────────────
│ ...
└─────────────────────────────────────────────────

┌─ ITEM-02 ───────────────────────────────────────
│ ...
└─────────────────────────────────────────────────
```

형태로 각각 구분한다.

Tree가 적합한 정보는
Tree 형태를 유지한다.

```text
Handler
   ↓
Validation
   ↓
Parameter
   ↓
API
   ↓
Response
```

즉:

- 정보 요약 → Text Block
- 실행 관계 → Tree
- Validation → Validation별 Text Block
- Parameter → Parameter별 Text Block
- 조건 / 분기 → Tree + Text Block
- State → State별 Text Block
- API → API별 Text Block
- Response → Tree
- Error → Tree
- Source Evidence → Evidence별 Text Block

형식을 사용한다.


---

# 12. Excel 복사 시 독립성 규칙

각 정보 블록은
다른 Section을 함께 복사하지 않아도
내용을 이해할 수 있도록 작성한다.

예:

```text
┌─ P-02 : searchType ─────────────────────────────
│ Source    : State
│ 원본 값   : "ALL"
│ 변환      : ALL → null
│ 최종 값   : 미전송
│ 전송 조건 : 값 존재 시
└─────────────────────────────────────────────────
```

이 블록 하나만 Excel 셀에 붙여 넣어도
무슨 Parameter인지 이해할 수 있어야 한다.

따라서 다음과 같은 표현은 피한다.

```text
위와 동일
앞의 조건 참조
상기 내용 참고
동일 처리
```

필요한 핵심 정보는
각 블록 안에 직접 기록한다.

단, 지나친 중복으로 문서가 불필요하게 커지는 경우에는
현재 항목을 이해하는 데 필요한 최소 정보까지만 반복한다.


---

# 13. Reference 적용 규칙

이 문서는 단순 참고 자료가 아니라
Frontend 분석 결과의 **출력 형식 Template**이다.

최종 FE 문서는 가능한 한
이 Reference와 동일한 형태로 작성한다.

반드시 유지한다.

1. Section 순서
2. Section 제목
3. 한눈에 보는 기능 흐름
4. ASCII Flow / Tree 표현
5. Markdown Table 미사용
6. Excel 단일 셀 복사가 가능한 Text Block
7. Validation 요약 + Validation별 상세 Block
8. 조건 / 분기 Tree
9. Parameter 요약 + Parameter별 상세 Block
10. State별 Block
11. API별 Block
12. Response 흐름
13. 예외 처리 Tree
14. Source Evidence별 Block
15. Backend API 후보별 Block
16. 미확인 항목
17. 분석 경계

현재 기능에 특정 항목이 존재하지 않으면
없는 내용을 임의로 생성하지 않는다.

필요한 경우:

```text
해당 처리 없음
```

으로 표시한다.


---

# 14. 대량 항목 표시 규칙

Validation, Parameter, 조건 / 분기가 많더라도
`3. 한눈에 보는 기능 흐름`을
지나치게 크게 만들지 않는다.

상단에서는:

```text
Validation 총 N개
├─ 그룹 A N개
├─ 그룹 B N개
└─ 그룹 C N개
```

Parameter는:

```text
Parameter 총 N개
├─ 화면 입력      N개
├─ State          N개
├─ 자동 생성      N개
└─ 변환 / 가공    N개
```

형태로 요약한다.

하지만 상세 Section에서는
확인된 전체 항목을 제공한다.

예:

```text
상단
Validation 총 30개
        ↓
그룹별 요약

하단
V-01
V-02
V-03
...
V-30
전체 상세
```

가독성을 위해
실제 분석 내용을 삭제하지 않는다.


---

# 15. SAMPLE 데이터 사용 금지

이 Reference에 사용된 다음 값은
출력 형식을 보여주기 위한 SAMPLE이다.

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

실제 분석 결과에 SAMPLE 값을 복사하지 않는다.

현재 분석 결과는 반드시:

- 현재 SCREEN 문서
- 현재 Frontend Source
- Code Index MCP 탐색 결과
- 필요한 경우 Chrome DevTools MCP Runtime 정보

를 기준으로 작성한다.

Reference 자체는 Evidence가 아니다.


---

# 16. Source / Runtime 구분

Source에서 확인한 일반 처리 규칙과
Runtime에서 실제 관찰한 값을 구분한다.

예:

```text
┌─ Source 확인 ───────────────────────────────────
│
│ 조건
│   searchType === "ALL"
│
│ 처리
│   request.searchType = null
│
└─────────────────────────────────────────────────
```

Runtime에서 확인한 경우:

```text
┌─ Runtime 확인 ──────────────────────────────────
│
│ 관찰된 Source 값
│   searchType = "ALL"
│
│ 실제 Request
│   searchType 미전송
│
└─────────────────────────────────────────────────
```

Runtime에서 한 번 관찰한 실제 값을
전체 Business Rule로 일반화하지 않는다.


---

# 17. 최종 출력 원칙

최종 FE 분석 결과는 다음 순서로 읽을 수 있어야 한다.

```text
1. 기능 정보
      ↓
2. 기능 요약
      ↓
3. 한눈에 보는 기능 흐름
      ↓
4. 전체 실행 Tree
      ↓
5. 입력 / 출력
      ↓
6. 상세 분석
      ↓
7. Source Evidence
      ↓
8. Backend API 후보
```

빠르게 기능을 파악하려는 경우:

```text
1
↓
2
↓
3

여기까지만 읽어도
기능 전체 흐름 파악 가능
```

상세 개발 / 수정 / 장애 분석이 필요한 경우:

```text
4
↓
6
↓
7

실제 실행 흐름과
Source까지 추적 가능
```

최종 원칙:

```text
상단
→ 단순하고 한눈에

하단
→ 상세하고 누락 없이

표
→ 사용하지 않음

정보
→ Excel 셀 하나에 복사 가능한 Block

흐름
→ Tree / ASCII Diagram

Evidence
→ 실제 Source 기준
```