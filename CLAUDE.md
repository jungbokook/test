# Code Analysis Project Guide

## 1. Purpose

이 프로젝트는 Frontend URL을 시작점으로
실제 Runtime 동작과 Source Code를 추적하여
개발 참조용 기능 분석 문서를 생성한다.

전체 시스템을 한 번에 분석하지 않는다.
항상 사용자가 선택한 범위만 단계적으로 분석한다.


## 2. Analysis Flow

분석은 다음 순서로 진행한다.

1. Screen Action Discovery
2. Selected Action → API Discovery
3. Selected API → API Specification
4. Frontend Business Analysis
5. Backend Business Analysis
6. MyBatis / Oracle Analysis
7. Final Business Flow

각 단계가 완료되면 반드시 STOP 한다.

사용자가 다음 분석 대상을 명시적으로 선택하기 전까지
다음 단계로 자동 진행하지 않는다.


## 3. Progressive Analysis

예:

Frontend URL
→ Action Discovery
→ ACT-001 / ACT-002 / ACT-003
→ STOP

사용자가 ACT-001 선택

ACT-001
→ API Discovery
→ API-001 / API-002
→ STOP

사용자가 API-001 선택

API-001
→ API Specification
→ STOP

이후 단계도 동일한 원칙을 적용한다.


## 4. MCP Responsibilities

### Chrome DevTools MCP

Frontend Runtime 분석에 사용한다.

- 실제 화면
- DOM
- UI Action
- Network
- Request / Response
- Console
- Runtime 동작


### Code Index MCP

Source Code 관계 탐색에 사용한다.

- Symbol
- Source Path
- Reference
- Caller
- Callee
- Call Relationship

MCP 탐색 결과만으로 Business Logic을 확정하지 않는다.
관련 Source를 직접 확인한다.


### Oracle MCP

Database 검증에 사용한다.

- Schema
- Table / View
- Column
- Data Type
- PK / FK
- Database Metadata

분석 과정에서 데이터 변경 작업은 수행하지 않는다.


## 5. Evidence

분석 결과는 확인 가능한 Evidence를 기반으로 작성한다.

우선순위:

1. Runtime Evidence
2. Source Code
3. Database Metadata
4. MCP Discovery Result

확인되지 않은 내용을 추측하여 사실처럼 작성하지 않는다.


## 6. Scope Control

현재 선택된 분석 대상과
직접 관련된 범위만 탐색한다.

전체 Repository를 불필요하게 탐색하거나
현재 단계보다 이후 영역을 미리 분석하지 않는다.


## 7. STOP Policy

각 단계의 결과를 생성하면 STOP 한다.

Action Discovery
→ Action Inventory
→ STOP

Action API Discovery
→ API Inventory
→ STOP

API Specification
→ API Specification
→ STOP

다음 분석 대상은 사용자가 선택한다.


## 8. Core Principle

Discover
→ Narrow Scope
→ Verify
→ Document
→ STOP
→ User Selects Next Target