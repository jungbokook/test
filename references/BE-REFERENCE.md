# BE ANALYSIS REFERENCE

> 이 문서는 Backend 분석 결과의 표현 형식, 문서 구조,
> 가독성 및 상세 수준을 정의하기 위한 Reference이다.
>
> 이 문서에 포함된 Class, Method, Path, SQL, Table,
> RFC Function, URL, Parameter, Response 및 Sample 값은
> 실제 분석 Evidence가 아니다.
>
> 최종 Backend 분석 문서는 반드시 실제 Source,
> MyBatis SQL, Oracle Metadata 및 확인된 실행 흐름을
> 기준으로 작성한다.

---

# 1. 문서 목적

Backend 분석 문서는 선택한 API 하나의 실제 Backend 실행 흐름을
개발자가 Source를 따라갈 수 있는 수준으로 설명한다.

단순한 Layer 목록이 아니라 실제 실행 순서를 표현한다.

예:

```text
Controller
 ↓
Service
 ↓
Validation
 ↓
DB #1
 ↓
조건
 ├─ YES
 │   ↓
 │  RFC #1
 │   ↓
 │  RFC 결과 처리
 │
 └─ NO
     ↓
    Local 처리
 ↓
DB #2
 ↓
REST #1
 ↓
REST 결과 처리
 ↓
DB #3
 ↓
Response 생성
```

DB / RFC / REST / 기타 외부 연동은
종류별로 재배치하지 않는다.

실제 Business Logic에서 호출되는 위치를 유지한다.

---

# 2. 기본 문서 구조

최종 Backend 문서는 기본적으로 다음 순서로 작성한다.

```text
1. 기능 정보
2. 기능 요약
3. 한눈에 보는 실행 흐름
4. 핵심 정보
5. 전체 실행 Tree
6. Business Logic 상세
7. DB / SQL 상세
8. Oracle Metadata
9. 외부 연동 상세
10. Exception / Transaction
11. Response 생성
12. Source Evidence
13. 미확인 항목
14. 분석 경계
```

실제 기능에 존재하지 않는 항목을
억지로 생성하지 않는다.

---

# 3. 기능 정보

```text
┌─ 기능 정보 ──────────────────────────────────
│ 화면
│   equipment-search
│
│ API ID
│   API-001
│
│ 기능명
│   설비 목록 조회
│
│ HTTP Method
│   POST
│
│ Backend URL
│   /api/equipment/search
│
│ Backend Project
│   gipms-api-equipment
└──────────────────────────────────────────────
```

기능 정보 Block 하나만 복사해도
어떤 API를 분석한 것인지 알 수 있어야 한다.

---

# 4. 기능 요약

긴 Source 분석 내용을 그대로 반복하지 않고
해당 Backend 기능이 무엇을 수행하는지 설명한다.

```text
┌─ 기능 요약 ──────────────────────────────────
│ 설비 검색 요청을 받아 입력값을 검증한 후
│ 설비 기본정보를 조회한다.
│
│ 조회 결과와 요청 조건에 따라 SAP RFC를 호출하고,
│ RFC 결과를 Backend 데이터 구조로 변환한다.
│
│ 이후 상태 정보를 갱신하고 최종 조회 결과를
│ Response 객체로 생성하여 Controller에 반환한다.
└──────────────────────────────────────────────
```

실제 Source에서 확인되지 않은 업무 목적을
추측해서 추가하지 않는다.

---

# 5. 한눈에 보는 실행 흐름

복잡한 Backend 로직을 먼저 짧게 이해할 수 있도록
주요 실행 흐름을 요약한다.

```text
┌─ 한눈에 보는 실행 흐름 ─────────────────────
│ Controller
│   ↓
│ Service
│   ↓
│ Validation
│   ↓
│ DB #1 - 설비 조회
│   ↓
│ 조건 판단
│   ├─ YES → RFC #1
│   └─ NO  → RFC 미호출
│   ↓
│ RFC 결과 가공
│   ↓
│ DB #2 - 상태 갱신
│   ↓
│ Response 생성
└──────────────────────────────────────────────
```

이 Block은 전체 흐름의 요약이다.

상세 호출 관계는 전체 실행 Tree에서 표현한다.

---

# 6. 핵심 정보

```text
┌─ 핵심 정보 ──────────────────────────────────
│ API
│   POST /api/equipment/search
│
│ Controller
│   EquipmentController.search()
│   Source:
│   gipms-api-equipment/src/main/java/.../EquipmentController.java
│
│ Main Service
│   EquipmentServiceImpl.searchEquipment()
│   Source:
│   gipms-api-equipment/src/main/java/.../EquipmentServiceImpl.java
│
│ DB 호출
│   총 3개
│
│   DB #1 SELECT
│   DB #2 UPDATE
│   DB #3 INSERT
│
│ 외부 연동
│   총 3개
│
│   RFC #1
│   RFC #2
│   REST #1
│
│ 주요 Oracle Object
│   TB_EQUIPMENT
│   TB_PLANT
│
│ 최종 Response
│   EquipmentSearchResponse
└──────────────────────────────────────────────
```

호출 수가 많은 경우에도
임의의 일부만 선택하지 않는다.

---

# 7. 전체 실행 Tree

Backend 문서에서 가장 중요한 영역이다.

실제 Source 실행 순서를 기준으로 작성한다.

```text
Controller
│
├─ EquipmentController.search()
│
└─ EquipmentService.searchEquipment()
    ↓
    EquipmentServiceImpl.searchEquipment()
    │
    ├─ [1] Request 값 확인
    │
    ├─ [2] validateRequest()
    │      │
    │      ├─ plantCode 확인
    │      └─ searchType 확인
    │
    ├─ [3] DB #1 - 설비 기본정보 조회
    │      │
    │      ├─ EquipmentMapper.selectEquipment()
    │      ├─ namespace
    │      │    com.example.EquipmentMapper
    │      ├─ Statement ID
    │      │    selectEquipment
    │      └─ SELECT ...
    │
    ├─ [4] 조회 결과 확인
    │      │
    │      ├─ 결과 있음
    │      │    ↓
    │      │   다음 처리
    │      │
    │      └─ 결과 없음
    │           ↓
    │          Empty Response
    │
    ├─ [5] SAP 호출 조건 판단
    │      │
    │      ├─ sapUseYn == "Y"
    │      │    ↓
    │      │   SapService.callEquipment()
    │      │    ↓
    │      │   SapAdapter.call()
    │      │    ↓
    │      │   RFC #1
    │      │
    │      └─ 그 외
    │           ↓
    │          RFC 호출 없음
    │
    ├─ [6] RFC #1 Response 처리
    │
    ├─ [7] DB #2 - 상태 갱신
    │
    ├─ [8] REST #1
    │
    ├─ [9] REST Response 처리
    │
    ├─ [10] RFC #2
    │
    ├─ [11] DB #3
    │
    ├─ [12] Response 생성
    │
    └─ return
```

호출 순서를 보기 좋게 만들기 위해
DB와 외부 연동을 별도 단계로 이동시키지 않는다.

---

# 8. Business Logic 상세

Business Logic은 실행 순서에 따라
독립적인 Block으로 작성한다.

```text
┌─ Business Logic #1 : 요청값 검증 ────────────
│ 호출 위치
│   EquipmentServiceImpl.searchEquipment()
│
│ 처리 내용
│   검색 요청값을 확인한다.
│
│ 입력
│   plantCode
│   searchType
│
│ 조건
│   plantCode가 비어 있음
│     → Validation Error
│
│   searchType이 허용값이 아님
│     → Validation Error
│
│ 정상 처리
│   DB #1 실행
│
│ Source
│   gipms-api-equipment/src/main/java/.../
│   EquipmentServiceImpl.java
│
│ Method
│   searchEquipment()
│
│ Line Range
│   55-72
└──────────────────────────────────────────────
```

각 Block은 다른 Block을 보지 않아도
무슨 처리를 하는지 이해할 수 있어야 한다.

---

# 9. 조건 / 분기 상세

중요 조건은 독립 Block으로 표현할 수 있다.

```text
┌─ 조건 #1 : SAP 호출 여부 ───────────────────
│ 판단 위치
│   EquipmentServiceImpl.searchEquipment()
│
│ 조건
│   sapUseYn == "Y"
│
│ YES
│   RFC #1 호출
│
│ NO
│   RFC 호출 없이 다음 Business Logic 수행
│
│ 후속 처리
│   RFC 호출 여부와 관계없이 이후 상태 처리 진행
│
│ Source
│   gipms-api-equipment/src/main/java/.../
│   EquipmentServiceImpl.java
└──────────────────────────────────────────────
```

Source에 없는 YES / NO 결과를 만들지 않는다.

---

# 10. DB 상세 기본 원칙

각 DB 호출은 별도의 독립 Block으로 작성한다.

```text
DB #1
DB #2
DB #3
...
```

같은 Mapper가 반복 호출되어도
실행 위치가 다르면 별도 호출로 표현한다.

---

# 11. DB 상세 Block

```text
┌─ DB #1 : 설비 기본정보 조회 ────────────────
│ 호출 위치
│   EquipmentServiceImpl.searchEquipment()
│
│ 호출 목적
│   검색 조건에 해당하는 설비 기본정보 조회
│
│ Mapper
│   EquipmentMapper.selectEquipment()
│
│ Mapper Source
│   gipms-api-equipment/src/main/java/.../
│   EquipmentMapper.java
│
│ MyBatis XML
│   gipms-api-equipment/src/main/resources/.../
│   EquipmentMapper.xml
│
│ Namespace
│   com.example.EquipmentMapper
│
│ Statement ID
│   selectEquipment
│
│ SQL Type
│   SELECT
│
│ 입력 Parameter
│   plantCode
│   equipmentName
│   useYn
│
│ 주요 Oracle Object
│   TB_EQUIPMENT
│   TB_PLANT
│
│ Result
│   EquipmentVO List
│
│ 호출 후 처리
│   조회 결과 존재 여부를 판단한 후
│   SAP RFC 호출 여부를 결정한다.
└──────────────────────────────────────────────
```

DB Block 하나만 복사해도
호출 위치, Mapper, SQL 역할,
입력과 후속 처리를 이해할 수 있어야 한다.

---

# 12. SQL 상세

SQL이 중요하거나 복잡하면
DB Block과 별도로 SQL 상세 Block을 작성한다.

```text
┌─ DB #1 SQL ──────────────────────────────────
│ SQL Type
│   SELECT
│
│ FROM
│   TB_EQUIPMENT E
│
│ JOIN
│   TB_PLANT P
│
│ Join Condition
│   E.PLANT_CODE = P.PLANT_CODE
│
│ 기본 조건
│   E.USE_YN = 'Y'
│
│ Dynamic 조건
│
│   plantCode 존재
│     → E.PLANT_CODE = #{plantCode}
│
│   equipmentName 존재
│     → EQUIPMENT_NAME LIKE ...
│
│   equipmentIds 존재
│     → foreach를 사용하여 IN 조건 생성
│
│ 정렬
│   실제 SQL 기준으로 기록
│
│ Result Mapping
│   실제 resultType / resultMap 기준으로 기록
└──────────────────────────────────────────────
```

SQL 전체를 무조건 복사하는 것이 목적이 아니다.

개발자가 실제 처리 내용을 이해하는 데 필요한
SQL 구조와 조건을 설명한다.

단, 중요한 계산식이나 업무 조건은 생략하지 않는다.

---

# 13. Dynamic SQL 상세

```text
┌─ DB #1 Dynamic SQL ──────────────────────────
│ plantCode
│   값 있음
│     → PLANT_CODE 조건 추가
│
│   값 없음
│     → PLANT_CODE 조건 추가 안 함
│
│ equipmentName
│   값 있음
│     → EQUIPMENT_NAME 검색 조건 추가
│
│ equipmentIds
│   Collection 존재
│     → foreach
│     → 각 equipmentId를 Parameter Mapping
│     → IN (...) 생성
│
│ useYn
│   값 있음
│     → 전달된 값 사용
│
│   값 없음
│     → 기본 조건 적용
└──────────────────────────────────────────────
```

실제 MyBatis XML의 분기 구조를 보존한다.

---

# 14. SQL Parameter Mapping

```text
┌─ DB #1 Parameter Mapping ────────────────────
│ plantCode
│
│ API Request
│   plantCode
│     ↓
│ Service
│   request.getPlantCode()
│     ↓
│ Mapper
│   plantCode
│     ↓
│ MyBatis
│   #{plantCode}
│     ↓
│ Oracle
│   TB_EQUIPMENT.PLANT_CODE
│
│ equipmentName
│
│ API Request
│   equipmentName
│     ↓
│ Mapper
│   equipmentName
│     ↓
│ MyBatis
│   #{equipmentName}
│     ↓
│ Oracle
│   TB_EQUIPMENT.EQUIPMENT_NAME
└──────────────────────────────────────────────
```

Mapping을 Source에서 확인할 수 없는 단계는
억지로 연결하지 않는다.

---

# 15. SQL Result Mapping

```text
┌─ DB #1 Result Mapping ───────────────────────
│ Oracle
│   EQUIPMENT_ID
│     ↓
│ MyBatis
│   equipmentId
│     ↓
│ VO
│   EquipmentVO.equipmentId
│
│ Oracle
│   EQUIPMENT_NAME
│     ↓
│ MyBatis
│   equipmentName
│     ↓
│ VO
│   EquipmentVO.equipmentName
└──────────────────────────────────────────────
```

필드가 매우 많은 경우
대량 항목 표시 규칙을 적용한다.

---

# 16. Oracle Metadata

Oracle Metadata는 Column 단위 비교가 중요하므로
가독성을 위해 이 영역에 한해 Markdown Table 사용을 허용한다.

Oracle Object 자체의 기본 정보는
독립 Block으로 표현한다.

```text
┌─ Oracle Object #1 ───────────────────────────
│ Object
│   TB_EQUIPMENT
│
│ Object Type
│   TABLE
│
│ 사용 위치
│   DB #1 - 설비 기본정보 조회
│
│ Metadata Source
│   Oracle MCP
└──────────────────────────────────────────────
```

### 사용 Column

| Column | Data Type | Nullable | PK | 사용 위치 / 용도 |
|---|---|---|---|---|
| EQUIPMENT_ID | VARCHAR2(...) | NO | YES | 조회 / Result Mapping |
| PLANT_CODE | VARCHAR2(...) | NO | NO | 검색 조건 / JOIN |
| EQUIPMENT_NAME | VARCHAR2(...) | YES | NO | 조회 / 검색 조건 |
| USE_YN | VARCHAR2(...) | YES | NO | 검색 조건 |

Oracle Metadata는 실제 Oracle MCP에서 확인된 정보만 기록한다.

현재 API의 SQL에서 실제 사용되는 Column을 우선 표시한다.

다음과 같이 SQL에서 사용되는 Column의 역할을
가능한 경우 함께 기록한다.

```text
SELECT
WHERE
JOIN
INSERT
UPDATE
DELETE
MERGE
GROUP BY
ORDER BY
Result Mapping
```

Column이 많더라도 현재 API의 SQL에서 실제 사용되는
중요 Column을 임의로 누락하지 않는다.

Oracle MCP에서 확인할 수 없는 값은 추측하지 않고:

```text
확인되지 않음
```

으로 기록한다.

SQL Source와 Oracle Metadata가 서로 다른 경우
차이를 숨기지 않고 별도로 명시한다.

예:

```text
Metadata 확인 결과

SQL Source
  TB_EQUIPMENT.PLANT_CD

Oracle Metadata
  PLANT_CODE

판정
  Source와 Oracle Metadata 불일치

추가 확인 필요
```

Table / View가 여러 개이면
각 Object별로 Metadata를 분리한다.

예:

```text
Oracle Object #1
  TB_EQUIPMENT

Oracle Object #2
  TB_PLANT

Oracle Object #3
  VW_EQUIPMENT_STATUS
```

각 Object 아래에 해당 Object의 Column Metadata Table을 작성한다.

---

# 17. 외부 연동 기본 원칙

외부 연동은 실제 실행 위치에 따라 번호를 부여한다.

예:

```text
RFC #1
REST #1
RFC #2
SOAP #1
```

종류별로 다시 정렬하지 않는다.

전체 실행 Tree에서는 실제 순서를 유지한다.

---

# 18. RFC 상세 Block

```text
┌─ RFC #1 : 설비정보 조회 ────────────────────
│ 호출 위치
│   EquipmentServiceImpl.searchEquipment()
│
│ 호출 경로
│   EquipmentServiceImpl.searchEquipment()
│     ↓
│   SapService.getEquipment()
│     ↓
│   SapAdapter.call()
│     ↓
│   RFC Client
│
│ RFC Function
│   Z_PM_EQUIPMENT_SEARCH
│
│ 호출 조건
│   sapUseYn == "Y"
│
│ Request 생성
│   SapEquipmentRequest 생성
│
│ Input Mapping
│   request.plantCode
│     ↓
│   SapEquipmentRequest.plant
│     ↓
│   I_WERKS
│
│ Response
│   RFC Result
│
│ Response Mapping
│   E_RESULT
│     ↓
│   SapEquipmentResponse.result
│     ↓
│   Service 결과 판단
│
│ Error 처리
│   실제 Source 기준으로 기록
│
│ 호출 후 처리
│   RFC 결과를 변환한 후 DB #2 실행
└──────────────────────────────────────────────
```

RFC 상세 Block만 Excel에 복사해도
호출 목적과 전후 관계를 이해할 수 있어야 한다.

---

# 19. RFC Parameter Mapping

Parameter가 많거나 Mapping이 중요한 경우
별도 Block을 사용한다.

```text
┌─ RFC #1 Parameter Mapping ───────────────────
│ plantCode
│
│ Backend
│   request.plantCode
│     ↓
│ SAP Request
│   plant
│     ↓
│ RFC
│   I_WERKS
│
│ equipmentId
│
│ Backend
│   equipmentId
│     ↓
│ SAP Request
│   equipment
│     ↓
│ RFC
│   I_EQUNR
└──────────────────────────────────────────────
```

Response Mapping도 같은 원칙으로 표현한다.

```text
RFC Response
 ↓
Backend Mapping
 ↓
Business Logic 사용
```

---

# 20. REST / HTTP 상세 Block

```text
┌─ REST #1 : 외부 상태 조회 ──────────────────
│ 호출 위치
│   EquipmentServiceImpl.searchEquipment()
│
│ 호출 경로
│   EquipmentServiceImpl.searchEquipment()
│     ↓
│   InterfaceService.getStatus()
│     ↓
│   ExternalStatusClient.getStatus()
│
│ HTTP Method
│   POST
│
│ URL
│   실제 Source에서 확인된 URL
│
│ Header
│   실제 Source에서 확인된 Header
│
│ Query / Path
│   실제 Source 기준
│
│ Request Body
│   실제 Request 객체 기준
│
│ Response
│   실제 Response 객체 기준
│
│ Status 처리
│   실제 Source 기준
│
│ Error 처리
│   실제 Source 기준
│
│ Timeout
│   실제 Source에서 확인되는 경우 기록
│
│ 호출 후 처리
│   Response 상태를 확인한 후 다음 Business Logic 수행
└──────────────────────────────────────────────
```

URL이 Configuration과 Runtime 값의 조합으로 만들어지는 경우
확인 가능한 구성 과정을 그대로 표현한다.

확인되지 않은 Base URL을 임의로 완성하지 않는다.

---

# 21. 기타 외부 연동

SOAP, Message, File 등의 연동도
동일한 원칙으로 독립 Block을 생성한다.

예:

```text
┌─ SOAP #1 ────────────────────────────────────
│ 호출 위치
│   ...
│
│ 호출 경로
│   ...
│
│ Request
│   ...
│
│ Response
│   ...
│
│ Error 처리
│   ...
│
│ 호출 후 처리
│   ...
└──────────────────────────────────────────────
```

Message 예:

```text
┌─ Message #1 ─────────────────────────────────
│ 호출 위치
│   ...
│
│ Destination
│   실제 Source에서 확인된 Topic / Queue
│
│ Payload
│   ...
│
│ Header
│   ...
│
│ 호출 조건
│   ...
│
│ 호출 후 처리
│   ...
└──────────────────────────────────────────────
```

File 연동 예:

```text
┌─ File #1 ────────────────────────────────────
│ 호출 위치
│   ...
│
│ File 처리
│   생성 / 읽기 / 변환
│
│ Format
│   ...
│
│ Field Mapping
│   ...
│
│ 외부 전달 지점
│   ...
│
│ Error 처리
│   ...
└──────────────────────────────────────────────
```

실제 기능에 존재하지 않는 연동 Block은 생성하지 않는다.

---

# 22. 반복 처리

반복문 안에서 DB 또는 외부 연동이 발생하면
반복 구조를 명확히 표시한다.

```text
┌─ 반복 처리 #1 ───────────────────────────────
│ 반복 대상
│   equipmentList
│
│ 각 equipment 처리
│
│   equipment
│     ↓
│   DB #2
│     ↓
│   조건 확인
│     ↓
│   RFC #2
│     ↓
│   결과 저장
│
│ 반복 횟수
│   Runtime 값에 따라 결정
└──────────────────────────────────────────────
```

Source에서 실제 반복 횟수를 알 수 없다면
숫자를 임의로 생성하지 않는다.

---

# 23. Exception 처리

```text
┌─ Exception #1 : RFC 호출 실패 ──────────────
│ 발생 위치
│   SapAdapter.call()
│
│ 발생 조건
│   실제 Source에서 확인되는 조건
│
│ 처리
│   catch
│     ↓
│   Error Mapping
│     ↓
│   BusinessException
│
│ 후속 흐름
│   현재 처리 중단
│
│ Source
│   gipms-api-common/src/main/java/.../
│   SapAdapter.java
└──────────────────────────────────────────────
```

Framework 일반 동작만으로
Exception 흐름을 추측하지 않는다.

Exception이 여러 개이면:

```text
Exception #1
Exception #2
Exception #3
```

처럼 각각 구분한다.

---

# 24. Transaction

```text
┌─ Transaction ────────────────────────────────
│ 적용 위치
│   EquipmentServiceImpl.searchEquipment()
│
│ 설정
│   @Transactional
│
│ 실제 실행 순서
│   DB #1 SELECT
│     ↓
│   DB #2 UPDATE
│     ↓
│   RFC #1
│     ↓
│   DB #3 INSERT
│
│ Rollback
│   실제 Source / Transaction 설정에서
│   확인되는 범위만 기록
└──────────────────────────────────────────────
```

외부 연동 실패 시 Rollback 여부를
추측하지 않는다.

Transaction이 확인되지 않으면:

```text
Transaction
  명시적인 Transaction 설정 확인되지 않음
```

으로 표현할 수 있다.

---

# 25. Response 생성

```text
┌─ Response 생성 ──────────────────────────────
│ 입력
│   DB 조회 결과
│   RFC 결과
│   REST 결과
│
│ 처리
│   결과 데이터 변환
│     ↓
│   Response DTO 생성
│     ↓
│   필드 Mapping
│
│ Response
│   EquipmentSearchResponse
│
│ Controller Return
│   ResponseEntity<EquipmentSearchResponse>
└──────────────────────────────────────────────
```

Backend 분석은 SQL이나 외부 연동에서 끝내지 않고
최종 Response까지 연결한다.

---

# 26. Response Mapping

필요한 경우 상세 Mapping을 독립 Block으로 작성한다.

```text
┌─ Response Mapping ───────────────────────────
│ equipmentId
│
│ DB
│   EQUIPMENT_ID
│     ↓
│ VO
│   equipmentId
│     ↓
│ Response
│   equipmentId
│
│ status
│
│ RFC
│   E_STATUS
│     ↓
│ Service
│   status
│     ↓
│ Response
│   status
└──────────────────────────────────────────────
```

필드가 많으면 전체 개수와 그룹을 먼저 표시하고
상세 Mapping을 이어서 작성한다.

---

# 27. Source Evidence

```text
┌─ Source Evidence ────────────────────────────
│ Controller
│   Path:
│   gipms-api-equipment/src/main/java/.../
│   EquipmentController.java
│
│   Method:
│   search()
│
│   Line:
│   40-65
│
│ Service
│   Path:
│   gipms-api-equipment/src/main/java/.../
│   EquipmentServiceImpl.java
│
│   Method:
│   searchEquipment()
│
│   Line:
│   55-140
│
│ Mapper
│   Path:
│   gipms-api-equipment/src/main/java/.../
│   EquipmentMapper.java
│
│   Method:
│   selectEquipment()
│
│ MyBatis
│   Path:
│   gipms-api-equipment/src/main/resources/.../
│   EquipmentMapper.xml
│
│   Namespace:
│   com.example.EquipmentMapper
│
│   Statement:
│   selectEquipment
│
│ RFC
│   Path:
│   gipms-api-common/src/main/java/.../
│   SapAdapter.java
│
│   Method:
│   call()
└──────────────────────────────────────────────
```

Source Path는 Project Root 기준 상대경로를 사용한다.

Line Range를 확인할 수 없다면:

```text
Line Range:
확인되지 않음
```

으로 기록한다.

Line Range를 추측하지 않는다.

---

# 28. 미확인 항목

```text
┌─ 미확인 항목 ────────────────────────────────
│ REST #1 Base URL
│   현재 Project Source에서 확인되지 않음
│
│ RFC #2 특정 Return Field 의미
│   현재 Source에서 확인되지 않음
│
│ External Client 내부 구현
│   외부 Dependency 경계로 인해 분석하지 않음
└──────────────────────────────────────────────
```

확인되지 않은 값을 추측해서 채우지 않는다.

미확인 항목이 없으면:

```text
미확인 항목
  없음
```

으로 표현할 수 있다.

---

# 29. 분석 경계

```text
┌─ 분석 경계 ──────────────────────────────────
│ 분석 대상
│   API-001
│
│ 포함
│   Controller
│   Service / ServiceImpl
│   Internal Method
│   Other Service / Common Service
│   Mapper
│   MyBatis
│   SQL
│   Oracle Metadata
│   RFC
│   REST / HTTP
│   기타 실제 외부 연동
│   Exception
│   Transaction
│   Response
│
│ 제외
│   다른 API
│   다른 화면
│   관련 없는 Service
│   관련 없는 Mapper / SQL
│   관련 없는 Oracle Object
│   Project 전체 RFC
│   Project 전체 외부 연동
│   JAR 내부
│   Decompiled Source
│   Project Root 외부 Source
└──────────────────────────────────────────────
```

---

# 30. Excel 단일 셀 복사 규칙

Backend 문서는 사용자가 필요한 영역을
Excel 한 셀에 직접 복사하는 것을 고려하여 작성한다.

따라서 Oracle Metadata를 제외한 일반 영역에서는
기본적으로 Markdown Table을 사용하지 않는다.

다음 형식을 우선한다.

```text
┌─ 제목 ─────────────────────────────
│ 항목
│   값
│
│ 항목
│   값
└────────────────────────────────────
```

또는:

```text
조건
 ↓
처리
 ↓
결과
```

각 Block은 독립적으로 복사 가능해야 한다.

Oracle Metadata는 Column 비교 가독성을 위해
Markdown Table 사용을 허용한다.

---

# 31. Excel Block 독립성

각 Block은 다른 Block을 함께 복사하지 않아도
내용을 이해할 수 있어야 한다.

피해야 할 표현:

```text
위와 동일
앞에서 설명
이전 SQL 참고
상기 RFC 참고
동일 Mapper 사용
```

대신 필요한 핵심 Context를
현재 Block 안에 다시 포함한다.

단, 의미 없는 장문 반복은 피한다.

---

# 32. Markdown Table 사용 제한

Backend 최종 문서는 Excel 단일 셀 복사와
ASCII 실행 흐름의 가독성을 위해
기본적으로 Markdown Table을 사용하지 않는다.

다만 다음 영역은 가독성을 위해 예외적으로
Markdown Table을 사용할 수 있다.

```text
Oracle Metadata
```

Oracle Metadata는 Column별:

```text
Column
Data Type
Nullable
PK
사용 위치 / 용도
```

를 비교해야 하므로 Table 형식을 우선한다.

예:

| Column | Data Type | Nullable | PK | 사용 위치 / 용도 |
|---|---|---|---|---|
| EQUIPMENT_ID | VARCHAR2(...) | NO | YES | Result Mapping |
| PLANT_CODE | VARCHAR2(...) | NO | NO | WHERE / JOIN |
| USE_YN | VARCHAR2(...) | YES | NO | WHERE |

그 외 영역:

```text
기능 정보
기능 요약
실행 Tree
Business Logic
조건 / 분기
DB 상세
SQL 상세
Dynamic SQL
Parameter Mapping
Result Mapping
RFC
REST
기타 외부 연동
Exception
Transaction
Response
Source Evidence
미확인 항목
분석 경계
```

은 기본적으로 독립 Text Block + ASCII Tree 형식을 사용한다.

Oracle Metadata Table 역시
실제 Oracle MCP에서 확인된 Metadata만 사용하며
Reference의 Sample 값을 실제 결과에 복사하지 않는다.

---

# 33. 대량 항목 표시 규칙

Parameter, Column, Mapping 등이 많으면
임의의 일부만 표시하지 않는다.

먼저 전체 규모를 설명한다.

예:

```text
Request Parameter
총 27개

그룹
- 검색 조건: 8개
- 설비 정보: 11개
- 상태 정보: 5개
- 제어 정보: 3개
```

그 후 필요한 상세 내용을 아래 Block에서 계속 작성한다.

`대표 5개만 표시`처럼
근거 없이 일부를 생략하지 않는다.

Oracle Column의 경우
현재 API에서 실제 사용하는 Column을
Metadata Table에 누락 없이 표시하는 것을 우선한다.

---

# 34. 다중 DB 호출 표시

DB 호출이 많아도 하나의 DB Block으로 합치지 않는다.

예:

```text
DB 호출 총 5개

DB #1
  SELECT

DB #2
  SELECT

DB #3
  UPDATE

DB #4
  INSERT

DB #5
  SELECT
```

상세 내용은 각각:

```text
DB #1 상세
DB #2 상세
DB #3 상세
DB #4 상세
DB #5 상세
```

로 작성한다.

동일 Mapper / Statement가 반복 호출되어도
실제 실행 위치가 다르면 호출 자체는 구분한다.

---

# 35. 다중 외부 연동 표시

외부 연동 역시 합치지 않는다.

예:

```text
외부 연동 총 4개

RFC #1
RFC #2
REST #1
RFC #3
```

전체 실행 Tree에서는 반드시
실제 호출 순서를 유지한다.

예:

```text
DB #1
 ↓
RFC #1
 ↓
DB #2
 ↓
RFC #2
 ↓
REST #1
 ↓
DB #3
 ↓
RFC #3
```

같은 RFC Function이나 Endpoint가 반복 호출되어도
실행 위치가 다르면 각각 표현한다.

---

# 36. 실제 실행 순서 우선

다음처럼 Layer 기준으로 문서를 해석하지 않는다.

잘못된 예:

```text
Controller
 ↓
Service
 ↓
Mapper
 ↓
SQL
 ↓
RFC
 ↓
Response
```

실제 Source가 다음이라면:

```text
Service
 ↓
DB #1
 ↓
조건
 ↓
RFC #1
 ↓
RFC 결과 처리
 ↓
DB #2
 ↓
REST #1
 ↓
REST 결과 처리
 ↓
RFC #2
 ↓
DB #3
 ↓
Response
```

그 순서를 그대로 문서에 반영한다.

---

# 37. 호출 복귀 표현

하위 호출 이후 원래 Business Logic으로 돌아오는 구조를
가능하면 실행 Tree에서 표현한다.

```text
Service A
 │
 ├─ Service B
 │    │
 │    ├─ DB #1
 │    └─ return result
 │
 ├─ Service B 결과 처리
 │
 ├─ RFC #1
 │    │
 │    └─ return RFC result
 │
 ├─ RFC 결과 처리
 │
 └─ Response
```

더 깊은 구조도 동일한 원칙을 적용한다.

```text
Service A
 │
 ├─ Service B
 │    │
 │    ├─ Common Service
 │    │    │
 │    │    ├─ Adapter
 │    │    │    ↓
 │    │    │   RFC #1
 │    │    │
 │    │    └─ RFC 결과 변환
 │    │
 │    └─ Service B 결과 처리
 │
 ├─ Service A 복귀
 │
 ├─ DB #2
 │
 └─ Response
```

호출된 하위 기능만 설명하고
원래 Caller의 후속 처리를 빠뜨리지 않는다.

---

# 38. Source와 Runtime 구분

Source에서 확인한 사실과
Runtime에서 관찰한 값을 혼동하지 않는다.

예:

```text
Source
  RFC 호출 조건:
  sapUseYn == "Y"

Runtime 관찰
  이번 실행에서 sapUseYn = "Y"
  → RFC 호출 발생
```

Runtime에서 한 번 관찰된 값이
전체 Business Rule인 것처럼 표현하지 않는다.

Runtime에서 실행되지 않은 Source Branch도
현재 API의 실제 Business Logic에 포함될 수 있다.

---

# 39. Oracle Metadata와 SQL 구분

SQL Source와 Oracle Metadata는 역할이 다르다.

```text
MyBatis SQL
  실제 Query
  JOIN
  WHERE
  Dynamic SQL
  Parameter
  Result Mapping

Oracle MCP
  Object 존재 여부
  Column
  Data Type
  Nullable
  Primary Key
```

Oracle Metadata만 보고
Business Logic을 추측하지 않는다.

반대로 SQL Source에 Column이 존재한다고 해서
Oracle Metadata를 확인한 것처럼 표현하지 않는다.

---

# 40. Reference 적용 규칙

이 Reference는 다음을 결정한다.

```text
문서 구조
표현 형식
상세 수준
ASCII Tree 형식
Excel 복사 구조
Block 독립성
Oracle Metadata Table 형식
```

이 Reference는 다음을 결정하지 않는다.

```text
실제 Class
실제 Method
실제 Source Path
실제 SQL
실제 Table
실제 Column
실제 RFC Function
실제 Endpoint
실제 Parameter
실제 Response
```

실제 값은 분석 대상 Source와
확인된 Metadata에서 가져온다.

---

# 41. SAMPLE 데이터 사용 금지

이 Reference에 있는 모든 이름과 값은 Sample이다.

예:

```text
EquipmentController
EquipmentServiceImpl
EquipmentMapper
TB_EQUIPMENT
TB_PLANT
Z_PM_EQUIPMENT_SEARCH
/api/equipment/search
plantCode
```

최종 문서에 Sample 값을
Evidence처럼 복사하지 않는다.

실제 Source 또는 Metadata에서 확인되지 않은 경우:

```text
확인되지 않음
```

으로 기록한다.

---

# 42. Source Evidence 원칙

최종 문서의 중요한 판단은
가능한 경우 Source Evidence와 연결한다.

예:

```text
Business Logic
 ↓
Source Path
 ↓
Class / Method
 ↓
Line Range
```

DB:

```text
Service
 ↓
Mapper
 ↓
namespace + Statement ID
 ↓
MyBatis XML
 ↓
SQL
 ↓
Oracle Metadata
```

외부 연동:

```text
Service
 ↓
Common Service
 ↓
Adapter / Client
 ↓
RFC Function / Endpoint
```

Code Index 검색 결과만으로
Business Logic을 확정하지 않는다.

---

# 43. 확인되지 않음 처리

다음과 같은 경우 추측하지 않는다.

```text
실제 구현체를 확정할 수 없음
Configuration 실제 값 확인 불가
외부 Dependency 내부 구현
Runtime에서만 보이고 Source Mapping 불가
Oracle Metadata 확인 불가
Line Range 확인 불가
RFC Field 의미 확인 불가
```

표현:

```text
확인되지 않음
```

필요하면 이유를 함께 기록한다.

예:

```text
Base URL
  확인되지 않음

사유
  현재 Project Source에서 실제 Configuration 값을 확인할 수 없음
```

---

# 44. JAR / 외부 Dependency 경계

다음은 Backend 분석 범위로 자동 확장하지 않는다.

```text
*.jar
JAR 내부 Class
Decompiled Class
.m2/
.gradle/
Maven Repository
Gradle Cache
Project Root 외부 Source
```

예:

```text
Current Project Source
 ↓
ExternalClient.execute(request)
 ↓

════════ External Dependency Boundary ════════

외부 Library 내부 구현
분석하지 않음
```

단:

```text
gipms-api-common
gipms-api-interface
```

등이 실제 Project Root 아래 Source Project로 존재하고
현재 API Call Path와 연결된다면 분석할 수 있다.

---

# 45. 안전 원칙

Backend 문서 생성을 위한 분석은
Read-Only 방식으로 수행한다.

실행하지 않는다.

```text
Oracle INSERT
Oracle UPDATE
Oracle DELETE
Oracle MERGE
DDL
실제 RFC 업무 Function
상태 변경 REST 요청
SOAP 업무 요청
Message Publish
업무 File 전송
실제 저장 API
실제 수정 API
실제 삭제 API
```

실제 업무 데이터를 변경하지 않는다.

---

# 46. 최종 검증

최종 Backend 문서를 작성하기 전에
다음 항목을 확인한다.

```text
Controller 확인
 ↓
실제 Service / 구현체 확인
 ↓
Business Logic 실제 실행 순서 확인
 ↓
내부 Method 추적
 ↓
다른 Service / 공통 Service 추적
 ↓
하위 호출 후 Caller 복귀 확인
 ↓
Mapper 연결 확인
 ↓
namespace + Statement ID 확인
 ↓
MyBatis XML / Annotation SQL 확인
 ↓
Dynamic SQL 확인
 ↓
Parameter Mapping 확인
 ↓
Result Mapping 확인
 ↓
Oracle Metadata 확인
 ↓
RFC 확인
 ↓
REST / 기타 외부 연동 확인
 ↓
외부 연동 후 Caller 복귀 확인
 ↓
조건 / 분기 확인
 ↓
반복 구조 확인
 ↓
Exception 확인
 ↓
Transaction 확인
 ↓
Response 생성 확인
 ↓
Source Evidence 확인
```

현재 API에 존재하지 않는 항목은
억지로 만들어내지 않는다.

---

# 47. 최종 출력 원칙

최종 Backend 문서는 다음 조건을 만족해야 한다.

```text
실제 Source 기반

실제 실행 순서 기반

Controller부터 Response까지 연결

중간 Method / Service 호출 보존

하위 호출 후 Caller 복귀

DB 호출 위치 보존

Mapper → MyBatis → SQL 연결

namespace + Statement ID 확인

Dynamic SQL 보존

SQL Parameter Mapping

SQL Result Mapping

Oracle Metadata 교차 확인

Oracle Metadata는 Table 형식 허용

RFC / REST / 기타 외부 연동 위치 보존

외부 연동 다중 호출 보존

외부 연동 후 Caller 복귀

조건 / 분기 / 반복 보존

Exception / Transaction 확인

Source Evidence 포함

확인되지 않은 정보 추측 금지

Project Root 상대경로 사용

JAR / 외부 Dependency 경계 준수

Excel 단일 셀 복사 가능

Oracle Metadata 이외 Markdown Table 기본 사용 금지

각 상세 Block 독립적으로 이해 가능
```

문서를 보기 좋게 만드는 것보다
실제 Backend Business Logic을 정확하게 보존하는 것을 우선한다.

동시에 개발자가 필요한 부분을
Excel 한 셀에 복사했을 때도
해당 Block만으로 내용을 이해할 수 있도록 작성한다.

---

# 48. STOP

현재 선택한 API의 Backend 분석 문서가 완성되면 STOP 한다.

자동으로 다음 API를 분석하지 않는다.

자동으로 다른 화면을 분석하지 않는다.

자동으로 관련 없는 다음 범위까지 확장하지 않는다.

```text
다른 Service
다른 Mapper
다른 SQL
다른 Oracle Object
다른 RFC
다른 외부 API
다른 Business Logic
```

현재 API의 실제 Call Path에 연결되는 범위까지만 작성한다.