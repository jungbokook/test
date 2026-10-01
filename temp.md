### 6. 상세 실행 흐름

전체 실행 Tree에서 확인된 처리 단계를
실제 실행 순서에 따라 상세하게 설명한다.

기능 성격에 따라 하위 Section을 구성한다.

예:

- 6.1 Validation
- 6.2 Parameter 생성
- 6.3 조건 / 분기
- 6.4 Backend API 호출
- 6.5 Response 처리 및 화면 갱신
- 6.6 예외 처리

하위 Section은 위 목록을 기계적으로 생성하지 않는다.

현재 Action에서 실제로 존재하는
처리 단계와 실행 순서를 기준으로 구성한다.

각 처리 단계에는 가능한 경우 다음을 함께 표현한다.

- 처리 목적
- 입력 값 / Source State
- 조건 / 분기
- 데이터 변환
- Function / Method 호출
- Backend API 호출
- 처리 결과
- State 변경
- 화면 반영
- 예외 처리
- Source Path
- Function / Method
- Line Range

동일한 처리 내용을 여러 Section에
반복하여 작성하지 않는다.

실행 흐름이 끊어지지 않도록
관련 정보를 하나의 처리 단계 안에서 함께 설명한다.