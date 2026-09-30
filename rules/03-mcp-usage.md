---
description: 분석 단계별 MCP의 책임과 사용 경계를 정의한다.
---

# MCP Usage Rule

## Purpose

각 MCP는 지정된 역할에 맞게 사용한다.

MCP를 사용할 수 있다는 이유만으로
현재 분석 범위를 확장하지 않는다.

현재 분석 단계와 직접 관련된 MCP만 사용한다.


## Chrome DevTools MCP

Chrome DevTools MCP는
Frontend Runtime Evidence 확인에 사용한다.

주요 대상:

- 실제 페이지
- DOM
- UI Element
- Runtime Event
- Network
- Request
- Response
- Console
- Popup 동작

Frontend Source Code의 전체 관계를
Chrome DevTools만으로 판단하지 않는다.


## Code Index MCP

Code Index MCP는
관련 Source Code를 발견하고
Source 관계를 좁히는 데 사용한다.

주요 대상:

- Symbol (Class / Method / Function 등)
- Source Path
- Reference
- Caller
- Callee
- Call Relationship

Code Index MCP에서 관계가 발견되었다는 이유만으로
Business Logic을 확정하지 않는다.

필요한 경우 발견된 실제 Source Code를 확인한다.


## Oracle MCP

Oracle MCP는
Oracle Database 구조와 Metadata 검증에 사용한다.

주요 대상:

- Schema
- Table
- View
- Column
- Data Type
- Nullable
- Primary Key
- Foreign Key

분석 과정에서는 필요한 Read / Metadata 조회만 수행한다.

다음 데이터 변경 작업은 수행하지 않는다.

- INSERT
- UPDATE
- DELETE
- MERGE
- CREATE
- ALTER
- DROP
- TRUNCATE


## Stage Boundary

MCP 사용은 현재 분석 단계의 범위를 넘을 수 없다.

예:

Screen Action Discovery 중
Chrome DevTools에서 Network Request를 발견하더라도
API 상세 분석으로 자동 진행하지 않는다.

Code Index에서 Backend Symbol을 발견하더라도
현재 단계가 Frontend 분석이라면
Backend Business Logic으로 자동 진행하지 않는다.

Oracle MCP 연결이 가능하더라도
현재 단계가 DB 분석 단계가 아니라면
Oracle 분석을 자동 실행하지 않는다.


## Tool Selection

분석 목적에 맞는 MCP를 우선 사용한다.

Runtime 사실 확인:

`Chrome DevTools MCP`

Source 위치 및 관계 탐색:

`Code Index MCP`

Database Metadata 확인:

`Oracle MCP`

하나의 MCP 결과만으로 확인할 수 없는 경우
현재 분석 범위 안에서 다른 Evidence와 교차 확인할 수 있다.


## No Unnecessary MCP Calls

현재 작업에 필요하지 않은 MCP를
단순 확인 목적으로 호출하지 않는다.

예:

Action 목록을 만드는 중이라는 이유만으로
Oracle MCP를 호출하지 않는다.

DB Metadata를 확인하는 중이라는 이유만으로
Chrome DevTools를 호출하지 않는다.

MCP 호출 자체가 분석 목적이 되어서는 안 된다.


## Verification

Rule 구축 단계에서는
MCP 사용 규칙을 설명하는 테스트 요청에 대해
마지막에 다음 문구를 출력한다.

`RULE_CHECK: MCP_USAGE_RULE_APPLIED`