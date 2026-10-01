---
name: be-call-tracing
description: Backend API의 Controller부터 Service, 내부 호출, MyBatis/SQL, Oracle, RFC 및 기타 외부 연동까지 실제 Source Evidence와 실행 순서를 기준으로 추적하는 규칙을 정의한다.
---

# Backend Call Tracing

## 1. 목적

이 Rule은 선택한 Backend API의 실제 호출 관계를
Source Evidence를 기준으로 끝까지 연결하기 위한 규칙이다.

분석의 핵심은 단순한 Layer 나열이 아니다.

다음을 실제 실행 순서와 호출 관계로 연결한다.

```text
Controller
    ↓
Service
    ↓
ServiceImpl
    ↓
Internal Method
    ↓
Other Service
    ↓
Common Module
    ↓
Mapper
    ↓
MyBatis XML
    ↓
SQL
    ↓
Oracle

그리고 실행 흐름 중간의

RFC
REST API
SOAP
다른 Backend API
Message
File
기타 외부 연동
```

실제 Source에 존재하는 중간 단계를 임의로 생략하지 않는다.


## 2. 기본 추적 원칙

호출 관계는 실제 Source를 기준으로 확인한다.

기본 원칙:

```text
Caller
   ↓
실제 호출문
   ↓
Callee
   ↓
Callee 실제 Source
   ↓
다음 호출
```

Class명이나 Method명만으로
호출 관계를 추측하지 않는다.

Code Index MCP의:

```text
Symbol
Reference
Caller
Callee
Call Relationship
```

등은 Source 탐색에 사용한다.

최종 호출 관계는 실제 Source를 확인하여 확정한다.


## 3. 기존 API 분석 결과 활용

Backend 분석을 시작할 때
기존 API Contract 문서를 우선 시작점으로 사용한다.

예:

```text
docs/analysis/{화면명}/api/API-001-*.md
```

API 문서에서 다음 정보를 확보한다.

```text
HTTP Method
Backend URL
Backend Project
Controller
Controller Method
Source Path
Request DTO
Response DTO
```

이미 확인된 내용을 찾기 위해
전체 프로젝트를 처음부터 다시 탐색하지 않는다.

단, Backend Business Logic 분석을 시작하기 전에
실제 Controller Source에서 시작점을 다시 확인한다.


## 4. Controller 추적

Controller에서 다음을 확인한다.

```text
Controller Class
Controller Method
Request
Validation
호출하는 Service / Method
Return 처리
Exception 관련 처리
```

Controller에서 여러 Service 또는 Method를 호출한다면
실제 실행 순서대로 모두 기록한다.

예:

```text
Controller
 │
 ├─ validationService.validate()
 │
 ├─ equipmentService.search()
 │
 └─ responseFactory.create()
```

하나만 대표로 선택하지 않는다.


## 5. Interface → 구현체 추적

Service가 Interface 형태로 선언되어 있다면
실제 구현체를 확인한다.

예:

```text
EquipmentService
        ↓
EquipmentServiceImpl
```

다음과 같은 근거를 확인한다.

```text
implements
Spring Bean
Injection Type
Qualifier
Bean Name
실제 Reference
실제 호출 관계
```

구현체가 여러 개라면
이름만 보고 하나를 임의로 선택하지 않는다.

예:

```text
EquipmentService
 ├─ EquipmentServiceImpl
 └─ ExternalEquipmentServiceImpl
```

실제 호출 구현체를 Source Evidence로 확정할 수 없다면:

```text
실제 구현체: 확인되지 않음
```

으로 기록한다.


## 6. Service Method 추적

Service / ServiceImpl Method에 진입하면
Method Body를 실제 실행 순서대로 분석한다.

예:

```text
searchEquipment()
 │
 ├─ 1. Request 값 추출
 ├─ 2. validateCondition()
 ├─ 3. mapper.selectPlant()
 ├─ 4. 조건 판단
 ├─ 5. callSap()
 ├─ 6. convertResult()
 ├─ 7. mapper.updateStatus()
 └─ 8. return response
```

다음을:

```text
Validation
DB 처리
RFC 처리
Response 처리
```

처럼 종류별로 재배열하지 않는다.

실제 Method 실행 순서를 보존한다.


## 7. 내부 Method 추적

현재 Method에서 같은 Class의 다른 Method를 호출하면
현재 Business Logic에 영향을 주는 경우 추적한다.

예:

```text
save()
 ↓
validate()
 ↓
createSapRequest()
 ↓
callSap()
 ↓
convertSapResult()
```

다음에 영향을 주는 내부 Method는 생략하지 않는다.

```text
Validation
조건
Parameter 생성
데이터 변환
상태 변경
DB
외부 연동
Exception
Response
```

단순 getter/setter나 현재 기능의 의미에 영향을 주지 않는
기계적인 호출까지 무조건 확장하지 않는다.


## 8. 다른 Service 추적

현재 Service에서 다른 Service를 호출하면
현재 API 실행 흐름에 직접 참여하는 경우 계속 추적한다.

예:

```text
EquipmentService.save()
        ↓
PlantService.getPlant()
        ↓
CommonCodeService.getCode()
        ↓
SapService.call()
```

Service가 바뀌었다는 이유로 추적을 중단하지 않는다.

현재 API 흐름으로 다시 돌아오는 지점도 표현한다.

```text
EquipmentService.save()
        │
        ├─ PlantService.getPlant()
        │      ↓
        │   결과 반환
        │
        └─ 반환값으로 다음 처리
```

## 9. 호출 후 원래 흐름으로 복귀

하위 Method / Service / DB / 외부 연동을 분석한 뒤에는
반드시 Caller의 다음 실행 흐름으로 복귀한다.

예:

```text
EquipmentService.save()
 │
 ├─ Step 1. validate()
 │
 ├─ Step 2. SapService.call()
 │      │
 │      ├─ RFC Request 생성
 │      ├─ RFC 호출
 │      ├─ RFC Response 변환
 │      └─ return sapResult
 │
 ├─ Step 3. sapResult 확인
 │
 ├─ Step 4. Mapper UPDATE
 │
 └─ Step 5. Response 생성
```

RFC 분석이 끝났다는 이유로
전체 Backend 분석을 종료하지 않는다.


## 10. 다단계 호출 보존

다음과 같은 호출 관계가 실제 Source에 있다면:

```text
Service A
 ↓
Service B
 ↓
Common Service
 ↓
Interface Service
 ↓
Adapter
 ↓
Client
 ↓
External System
```

다음처럼 축약하지 않는다.

```text
Service A
 ↓
External System
```

Business Logic 이해에 필요한 중간 호출 계층은 보존한다.

각 단계에서:

```text
입력
변환
조건
출력
```

이 존재하면 함께 분석한다.


## 11. Mapper Interface 추적

Mapper Method가 호출되면
실제 Mapper Interface를 확인한다.

확인 항목:

```text
Mapper Interface
Mapper Method
Parameter
Return Type
Source Path
Line Range
```

예:

```text
EquipmentMapper.selectEquipment(request)
```

Method 이름만 보고 SQL을 추측하지 않는다.


## 12. Mapper → MyBatis XML 연결

Mapper Interface에서
실제 MyBatis XML Statement를 연결한다.

기본 연결 기준:

```text
Mapper Interface Full Name
        ↕
MyBatis namespace

Mapper Method
        ↕
Statement id
```

예:

```text
Interface
com.example.EquipmentMapper

Method
selectEquipment

        ↕

XML namespace
com.example.EquipmentMapper

Statement
<select id="selectEquipment">
```

이 연결을 실제 Source에서 확인한다.


## 13. 동일 Statement 이름 처리

서로 다른 Mapper XML에
동일한 Statement ID가 존재할 수 있다.

예:

```text
MapperA.xml
<select id="search">

MapperB.xml
<select id="search">
```

`search`라는 이름만으로 Statement를 선택하지 않는다.

반드시:

```text
namespace + statement id
```

조합으로 식별한다.


## 14. MyBatis Annotation SQL

SQL이 XML이 아니라 Mapper Annotation에 존재할 수도 있다.

예:

```text
@Select
@Insert
@Update
@Delete
```

이 경우 XML을 억지로 찾지 않는다.

실제 Annotation SQL을 분석한다.

문서에는 SQL Source가:

```text
MyBatis XML
```

인지:

```text
Mapper Annotation
```

인지 구분해서 기록한다.


## 15. MyBatis Dynamic SQL

실제 Statement의 Dynamic SQL 구조를 보존한다.

예:

```text
SELECT ...
FROM ...
WHERE 1 = 1

<if test="plantCode != null">
    AND PLANT_CODE = #{plantCode}
</if>

<choose>
    <when test="useYn != null">
        AND USE_YN = #{useYn}
    </when>
    <otherwise>
        AND USE_YN = 'Y'
    </otherwise>
</choose>
```

분석 결과에서는:

```text
기본 SQL
 ↓
plantCode 존재?
 ├─ YES → PLANT_CODE 조건 추가
 └─ NO  → 조건 추가 안 함
 ↓
useYn 존재?
 ├─ YES → 전달된 useYn 사용
 └─ NO  → USE_YN = 'Y'
```

처럼 실제 분기를 설명한다.

Dynamic SQL을 하나의 고정 SQL처럼 왜곡하지 않는다.


## 16. include / SQL Fragment 추적

MyBatis XML에서:

```text
<include refid="...">
```

가 사용되면 실제 SQL Fragment를 확인한다.

예:

```text
<select id="search">
    <include refid="baseColumns"/>
</select>
```

다음처럼 끝내지 않는다.

```text
baseColumns 사용
```

현재 SQL을 이해하는 데 필요한 실제 Fragment 내용을 확인한다.

다만 현재 Statement와 관련 없는 다른 Fragment까지
전체 분석하지 않는다.


## 17. foreach 추적

`<foreach>`가 존재하면 반복 구조를 보존한다.

예:

```text
<foreach collection="equipmentIds"
         item="id"
         separator=",">
    #{id}
</foreach>
```

분석 결과:

```text
equipmentIds 반복
        ↓
각 id를 SQL Parameter로 Mapping
        ↓
IN (...) 조건 구성
```

실제 반복 Collection과 Item을 기록한다.


## 18. SQL Parameter Mapping

Service / Mapper에서 전달된 값이
SQL의 어느 Parameter로 사용되는지 추적한다.

예:

```text
Service
request.getPlantCode()
        ↓
Mapper Parameter
plantCode
        ↓
MyBatis
#{plantCode}
        ↓
SQL
PLANT_CODE = #{plantCode}
```

Parameter 이름만 나열하지 않고
가능하면 전달 흐름을 연결한다.


## 19. SQL Result Mapping

SQL 결과가 Backend 객체로 어떻게 Mapping되는지 확인한다.

확인 대상:

```text
resultType
resultMap
column
property
Nested Mapping
Collection
Association
```

예:

```text
EQUIPMENT_ID
        ↓
equipmentId

EQUIPMENT_NAME
        ↓
equipmentName
```

SQL 결과와 이후 Business Logic의 연결에 필요한 범위까지 확인한다.


## 20. Oracle Metadata 교차 확인

SQL에서 실제 Oracle Object가 사용되면
필요한 경우 Oracle MCP로 Metadata를 확인한다.

예:

```text
SQL
TB_EQUIPMENT.PLANT_CODE
        ↓
Oracle Metadata
TB_EQUIPMENT
 └─ PLANT_CODE VARCHAR2(...)
```

확인 가능한 항목:

```text
Table / View
Column
Data Type
Nullable
Primary Key
Object 존재 여부
```

Oracle MCP 결과가 Source SQL과 다르면
차이를 숨기지 않고 기록한다.

DB Metadata 확인만 수행하며
DML / DDL은 실행하지 않는다.


## 21. DB 호출 후 원래 흐름 복귀

Mapper / SQL 분석 후
반드시 해당 Mapper를 호출한 Business Logic으로 돌아간다.

예:

```text
Service
 │
 ├─ DB #1
 │    ├─ Mapper
 │    ├─ MyBatis
 │    ├─ SQL
 │    └─ Result
 │
 ├─ DB #1 Result 조건 확인
 │
 ├─ RFC #1
 │
 └─ DB #2
```

SQL을 확인했다고 Backend 분석을 종료하지 않는다.


## 22. RFC 호출 지점 탐색

RFC가 직접 Service에 존재하지 않을 수 있다.

예:

```text
EquipmentService
 ↓
SapService
 ↓
SapAdapter
 ↓
RfcClient
 ↓
RFC Function
```

현재 Project Root 아래 실제 Source가 존재하면
호출 지점까지 계속 추적한다.

단순히 Class 이름에 `Sap`, `Rfc`가 있다는 이유만으로
RFC라고 확정하지 않는다.

실제 호출 Source를 확인한다.


## 23. RFC Function 확인

실제 Source에서 확인되는 경우
RFC Function을 기록한다.

예:

```text
Z_PM_EQUIPMENT_SEARCH
```

확인할 항목:

```text
RFC Function
호출 Method
Request 생성
Import Parameter
Table Parameter
기타 Input
Response
Export Parameter
Table Result
Return / Error
```

실제 Source에서 확인되지 않는 항목은 추측하지 않는다.


## 24. RFC Parameter Mapping

Backend 값이 RFC Parameter로 어떻게 변환되는지 추적한다.

예:

```text
request.plantCode
        ↓
SapRequest.plant
        ↓
RFC Parameter
I_WERKS
```

Response도 동일하게 연결한다.

```text
RFC
E_RESULT
        ↓
SapResponse.result
        ↓
Service 조건 판단
```

가능하면 단순 Field 목록보다
실제 Mapping 흐름을 기록한다.


## 25. RFC 반복 호출

반복문에서 RFC가 호출되면
반복 구조를 유지한다.

예:

```text
for each equipment
        │
        ├─ RFC Request 생성
        ├─ RFC #1 호출
        ├─ Response 확인
        └─ 다음 equipment
```

반복 횟수를 Source만으로 확정할 수 없다면
특정 숫자를 임의로 만들지 않는다.


## 26. REST / HTTP 호출 추적

REST 또는 HTTP 외부 호출이 발견되면
실제 Client 호출까지 추적한다.

예:

```text
Service
 ↓
InterfaceService
 ↓
ExternalClient
 ↓
WebClient / RestTemplate / 기타 Client
 ↓
External Endpoint
```

실제 Source에서 확인 가능한 경우:

```text
Method
URL
Header
Path
Query
Body
Request Mapping
Response Type
Response Mapping
Status 처리
Error 처리
Timeout
```

을 분석한다.


## 27. URL 구성 추적

외부 URL이 여러 값으로 조합되는 경우
실제 구성 과정을 확인한다.

예:

```text
baseUrl
+
"/equipment/"
+
equipmentId
```

최종 URL을 Source Evidence 없이 임의로 생성하지 않는다.

Configuration 값이 현재 Project Source에서 확인되지 않으면:

```text
Base URL: 현재 Project Source에서 확인되지 않음
```

으로 기록한다.


## 28. SOAP / WebService 추적

SOAP 또는 WebService 호출이 발견되면
현재 Project Source에서 확인 가능한 범위를 추적한다.

예:

```text
Service
 ↓
WebService Adapter
 ↓
Request 생성
 ↓
WebService Client 호출
 ↓
Response
 ↓
Response Mapping
```

WSDL이나 외부 Library 내부까지
자동으로 분석 범위를 확장하지 않는다.


## 29. Message / Queue 연동

Message 또는 Queue 연동이 발견되면
실제 Source에서 확인 가능한 경우 다음을 분석한다.

```text
Producer / Publisher
Destination
Topic / Queue
Message 생성
Payload
Header
호출 조건
전송 후 처리
Error 처리
```

실제 Message 발행은 수행하지 않는다.


## 30. File 연동

File 기반 외부 연동이 발견되면
실제 Source에서 확인 가능한 경우 다음을 분석한다.

```text
File 생성 / 읽기
Format
Field Mapping
경로 구성
호출 조건
외부 전달 지점
결과 처리
Error 처리
```

실제 업무 File을 생성하거나 전송하지 않는다.


## 31. 외부 연동 번호 부여

현재 API 실행 흐름에서 외부 연동이 여러 개면
실제 호출 순서를 기준으로 식별한다.

예:

```text
RFC #1
REST #1
RFC #2
SOAP #1
REST #2
```

같은 RFC Function을 두 번 호출하더라도
실행 위치가 다르면 각각의 호출로 표현한다.

예:

```text
RFC #1 - Z_PM_CHECK
...
RFC #2 - Z_PM_CHECK
```

두 호출을 자동으로 하나로 합치지 않는다.


## 32. DB 호출 번호 부여

DB 호출 역시 실제 실행 순서를 기준으로 구분한다.

예:

```text
DB #1 - SELECT
DB #2 - UPDATE
DB #3 - SELECT
DB #4 - INSERT
```

같은 Mapper Statement가 여러 위치에서 호출되면
실행 위치가 다른 호출은 Execution Tree에서 구분한다.


## 33. 조건부 호출 표현

DB / RFC / REST / 기타 호출이
특정 조건에서만 발생한다면 조건을 보존한다.

예:

```text
sapUseYn == "Y" ?
 │
 ├─ YES
 │    ↓
 │   RFC #1
 │
 └─ NO
      ↓
     RFC 호출 안 함
```

조건을 제거하고 RFC가 항상 실행되는 것처럼 표현하지 않는다.


## 34. Exception 흐름 추적

호출 중 Exception이 발생할 수 있고
현재 Source에서 처리 코드가 확인되면 흐름을 추적한다.

예:

```text
RFC 호출
 ↓
Exception
 ↓
catch
 ↓
Error Code 생성
 ↓
BusinessException
```

또는:

```text
REST 호출
 ↓
Status != 200
 ↓
Error Response 변환
 ↓
다음 처리 중단
```

실제 Source에서 확인되지 않는 Error 흐름을
일반적인 Framework 동작으로 추가하지 않는다.


## 35. Transaction과 호출 순서

Transaction이 존재하면
DB 변경과 외부 연동의 실제 순서를 보존한다.

예:

```text
@Transactional
        │
        ├─ DB #1 UPDATE
        ├─ RFC #1
        ├─ DB #2 INSERT
        └─ return
```

이 경우 문서에서도 동일한 순서를 유지한다.

RFC 실패 시 DB가 반드시 Rollback된다고
Source Evidence 없이 단정하지 않는다.

Transaction 설정과 Exception 처리에서
확인되는 범위만 기록한다.


## 36. 재귀 / 순환 호출 방지

호출 관계가 순환되는 경우
무한히 반복 추적하지 않는다.

예:

```text
ServiceA.methodA()
 ↓
ServiceB.methodB()
 ↓
ServiceA.methodA()
```

이미 현재 Call Path에서 분석한 동일 Method로 다시 진입하면:

```text
순환 호출 감지
→ ServiceA.methodA() 재진입
→ 추가 확장 중단
```

으로 표시한다.

단, 호출 자체가 존재한다는 사실은 Execution Tree에 남긴다.


## 37. 공통 Method 재사용

같은 공통 Method가 여러 실행 위치에서 호출될 수 있다.

공통 Method의 내부 구현을 매번 동일하게 장문으로 반복할 필요는 없지만
각 호출 위치는 Execution Tree에서 반드시 유지한다.

예:

```text
Step 2
CommonService.getCode()
 ↓
공통 로직 #1

...

Step 8
CommonService.getCode()
 ↓
공통 로직 #1 재호출
```

단, Excel에 복사되는 개별 상세 Block이
독립적으로 이해되어야 하는 경우
필요한 Context는 해당 Block에 포함한다.


## 38. Source Evidence 규칙

각 주요 호출에는 가능한 경우 다음 Evidence를 기록한다.

```text
Source Path
Class
Method
Line Range
```

Mapper:

```text
Mapper Interface
Mapper Method
Source Path
Line Range
```

MyBatis:

```text
Mapper XML
Namespace
Statement ID
Line Range
```

외부 연동:

```text
Source Path
Class
Method
Client / Adapter
RFC Function 또는 Endpoint
Line Range
```

Line Range를 확인할 수 없는 경우:

```text
Line Range: 확인되지 않음
```

으로 기록한다.

Line 번호를 추측하지 않는다.


## 39. Source Path 규칙

모든 Source Path는
Code Index Project Root 기준 상대경로로 기록한다.

허용 예:

```text
gipms-api-equipment/src/main/java/...
gipms-api-common/src/main/java/...
```

사용하지 않는 형식:

```text
C:\work\...
D:\project\...
```

사용자에게 절대경로를 요구하지 않는다.


## 40. JAR / 외부 Dependency 경계

다음 영역은 자동 추적하지 않는다.

```text
*.jar
JAR 내부 Class
Decompiled Class
Maven Repository
.m2/
Gradle Cache
.gradle/
Project Root 외부 Source
```

현재 Source가 외부 Library를 호출하면
Project Source에서 확인 가능한 호출 경계까지만 분석한다.

예:

```text
Current Project Source
 ↓
ExternalClient.execute(request)
 ↓

════════ External Dependency Boundary ════════

JAR 내부 구현
분석하지 않음
```

단, `gipms-api-common` 등
Project Root 아래 실제 Source가 존재하는 내부 공통 프로젝트는
계속 추적할 수 있다.


## 41. Source 불일치 처리

Code Index 검색 결과와 실제 Source가 다르거나
호출 관계를 확정할 수 없다면 추측하지 않는다.

예:

```text
호출 대상 구현체: 확인되지 않음
```

또는:

```text
Code Index에서 후보 2개 확인

후보 A
...

후보 B
...

실제 Runtime 구현체:
확인되지 않음
```

하나를 임의로 선택해서 Execution Tree를 만들지 않는다.


## 42. 전체 실행 흐름 검증

분석 완료 전 전체 Call Path를 다시 확인한다.

예:

```text
Controller
 ↓
Service
 ↓
Internal Method
 ↓
DB #1
 ↓
Service 복귀
 ↓
조건
 ↓
Common Service
 ↓
RFC #1
 ↓
Common Service 복귀
 ↓
원 Service 복귀
 ↓
DB #2
 ↓
REST #1
 ↓
원 Service 복귀
 ↓
Response
```

중간 호출에서 분석이 끊기지 않았는지 확인한다.


## 43. 호출 관계 축약 금지

다음과 같은 실제 흐름:

```text
Service A
 ↓
Service B
 ↓
Common Service
 ↓
Adapter
 ↓
RFC Client
 ↓
RFC
```

을:

```text
Service A → RFC
```

로 축약하지 않는다.

또한:

```text
Service
 ↓
Mapper
 ↓
XML
 ↓
SQL
```

을:

```text
Service → DB
```

로만 표현하지 않는다.

전체 Execution Tree에서는
개발자가 실제 Source를 따라갈 수 있는 수준의
호출 관계를 유지한다.


## 44. 분석 결과의 기준

호출 관계의 신뢰도 우선순위는 다음과 같다.

```text
실제 Source
        ↓
실제 Mapper / MyBatis XML / SQL
        ↓
Oracle Metadata
        ↓
Runtime Evidence
        ↓
Code Index 탐색 결과
```

Code Index는 탐색 도구이지
Business Logic 자체의 Evidence를 대신하지 않는다.

Runtime Evidence가 존재하더라도
실행되지 않은 Source Branch가 존재할 수 있으므로
Runtime 결과만으로 전체 Business Logic을 제한하지 않는다.


## 45. 안전 규칙

분석 과정에서 실제 업무 처리를 실행하지 않는다.

금지:

```text
Oracle DML / DDL 실행
실제 RFC 업무 Function 실행
외부 REST 변경 요청 실행
실제 SOAP 업무 요청 실행
Message Publish
업무 File 전송
저장 / 수정 / 삭제 API 실행
```

Backend Source와 Metadata를
Read-Only 방식으로 분석한다.


## 46. STOP 조건

현재 선택한 API에서 실제 호출 관계로 연결되는
Backend Business Logic 추적이 완료되면 STOP 한다.

자동으로 다음 범위로 확장하지 않는다.

```text
다른 API
다른 화면
관련 없는 Service
관련 없는 Mapper
관련 없는 SQL
관련 없는 Table
프로젝트 전체 RFC
프로젝트 전체 외부 API
프로젝트 전체 Business Logic
```

현재 API Call Path에 실제로 연결되는 Source만 분석한다.