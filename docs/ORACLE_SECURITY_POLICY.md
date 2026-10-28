# Oracle 감사 정책과 PROFILE 변환 규칙

규칙 버전 0.1.44 / 검토 기준 Oracle19 → GOLDILOCKS26c.1. 기존 토큰/위치 기반 Analyzer와 보고서를 재사용한다. 새 DB 조회나 실서버 DDL은 수행하지 않는다.

## 서로 다른 정책

| 구문 | 목적 | 이번 범위 |
|---|---|---|
| Oracle CREATE AUDIT POLICY / GOLDILOCKS CREATE AUDIT POLICY | SQL action·권한·롤 사용 감사 기록 | 공통 범위 처리 |
| Oracle/GOLDILOCKS CREATE PROFILE | 비밀번호 기간·실패 횟수·잠금·재사용 | 공통 범위 처리 |
| PostgreSQL CREATE POLICY | USING/WITH CHECK로 행 접근 제한 | 기존 처리 유지. 감사 정책으로 변환하지 않음 |
| Oracle DBMS_RLS.ADD_POLICY | VPD 행 접근 제한 | 이번 변경 대상 아님 |
| Altibase7.3 AUDIT | 이름 있는 정책 객체 대신 감사 조건 설정, ACCESS/SESSION 단위 | 기존 처리 유지 |

대안인 POLICY라는 단어의 일괄 치환은 목적을 바꾸므로 사용하지 않는다. 지원 확인된 옵션을 그대로 보존하고 수치 표현만 필요한 경우 정확하게 바꾼다. 미확인 보안 조건은 제거하지 않는다.

## PROFILE

- CREATE/ALTER PROFILE 이름 LIMIT을 인식하고 이름·주석·옵션 순서를 유지한다.
- FAILED_LOGIN_ATTEMPTS / PASSWORD_REUSE_MAX: 양의 정수, DEFAULT, UNLIMITED. 현 정적 구현은 signed32bit 범위 내 횟수를 자동 처리하며 그 밖의 수치는 검토한다.
- PASSWORD_LOCK_TIME / PASSWORD_LIFE_TIME / PASSWORD_GRACE_TIME / PASSWORD_REUSE_TIME: 정수 일수 또는 정수/정수 분수, DEFAULT, UNLIMITED. Oracle19 공통 범위인 1초~24855일을 확인한다. GOLDILOCKS는 최대100000일이지만 원본 범위를 넓히지 않는다.
- 소수 0.5일은 1/2일로 정확히 변환한다. BigDecimal/BigInteger를 사용하며 부동소수점 반올림을 하지 않는다. 임의 함수/산술식은 실행·평가하지 않는다. 상수 스케일/표현 범위를 초과하면 계산 불가로 표시한다.
- PASSWORD_VERIFY_FUNCTION NULL/DEFAULT는 보존한다. 내장 이름이라도 Oracle의 함수 본문 이관과 대상 내장 정책 선택은 다르므로 검토한다. GOLDILOCKS ORA12C_VERIFY_FUNCTION의 goldilocks 금지어 검사처럼 차이가 명시돼 있다. 사용자 정의·스키마 한정 함수는 자동 대체하지 않는다.
- CPU_PER_SESSION, SESSIONS_PER_USER, IDLE_TIME 등 자원 제한, INACTIVE_ACCOUNT_TIME, PASSWORD_ROLLOVER_TIME, CONTAINER는 MANUAL로 해당 토큰을 표시한다. 보안 조건을 버리거나 제한을 느슨하게 바꾸지 않는다.
- 중복/값 누락/0/음수/0분모/범위 밖/알 수 없는 후행 절은 검토한다. 객체 내 미해결 항목이 있으면 소수 일수 변환까지 포함해 원문 전체를 유지한다.
- 생략/DEFAULT 항목은 **대상 DEFAULT profile**에서 상속한다. 원본과 대상 DEFAULT의 실제 값이 같다는 뜻은 아니다. 원본 정책 값의 추출·계정 연결은 별도 확인한다. 기존 CREATE/ALTER USER의 PROFILE 절을 재사용한다.

```sql
CREATE PROFILE app_profile LIMIT
  FAILED_LOGIN_ATTEMPTS 5
  PASSWORD_LIFE_TIME 90
  PASSWORD_GRACE_TIME 7
  PASSWORD_LOCK_TIME 1/24
  PASSWORD_REUSE_TIME 365
  PASSWORD_REUSE_MAX 10
  PASSWORD_VERIFY_FUNCTION NULL;
ALTER USER app_user PROFILE app_profile;
```

위 예제는 변환 범위 설명이며 실행을 권장하는 조직 보안 정책이 아니다.

## AUDIT POLICY

- CREATE는 PRIVILEGES/ACTIONS/ROLES 중 하나 이상, ALTER는 단일 ADD 또는 DROP action 집합만 공통 인식한다. Oracle 원본의 PRIVILEGES→ACTIONS→ROLES 순서, 같은 절의 중복과 비정상 후행 문법은 검토한다.
- PRIVILEGES는 CREATE/ALTER/DROP/SELECT/INSERT/UPDATE/DELETE/LOCK ANY TABLE의 확인된 공통 집합에 한정한다. Oracle ANY PROCEDURE는 PACKAGE도 포함하지만 대상은 별도 권한이므로 같은 이름만으로 호환 처리하지 않는다. ROLES는 롤 이름 목록을 유지한다. 이름 존재·실제 권한 부여 여부는 자동 생성하지 않는다.
- 객체 ACTIONS: ALTER/COMMENT/DELETE/GRANT/INDEX/INSERT/LOCK/RENAME/SELECT/UPDATE/EXECUTE ON [schema.]object. 실제 객체 유형에 적용 가능한 action과 객체 존재는 대상 실행 시 확인한다. DIRECTORY/MINING MODEL은 이 구조와 다르므로 검토한다.
- 시스템 ACTIONS는 두 매뉴얼에서 확인한 DROP TABLE/TRUNCATE TABLE을 우선 처리한다. 그 외 전체 동적 action 카탈로그를 추정하지 않는다. ALL/ALL ON은 DB별 action 집합 차이로 검토하며 실제 필요한 항목 목록을 결정한다.
- AUDIT POLICY: BY/EXCEPT 사용자 목록, WHENEVER [NOT] SUCCESSFUL. NOAUDIT POLICY: 선택적 BY 사용자 목록. EXCEPT/WHENEVER를 NOAUDIT에 자동 붙이지 않는다.
- WHEN/EVALUATE, ONLY TOPLEVEL, CONTAINER, COMPONENT, CONDITION, BY USERS WITH GRANTED ROLES, ALTER의 ADD와 DROP 동시 지정은 검토한다. 해당 옵션은 삭제하지 않는다.
- 감사 생성은 활성화가 아니다. AUDIT POLICY 구문이 따로 필요하며 GOLDILOCKS에서 변경은 새 세션부터 적용된다. AUDIT SYSTEM ON DATABASE 권한과 감사 로그 운영 설정은 별도 준비한다.
- 감사 대상에 쓰인 DROP TABLE/TRUNCATE TABLE 및 ALTER AUDIT POLICY ... DROP을 실제 테이블 삭제로 오인하지 않도록 한정된 정의 파서가 먼저 처리한다. 실제 최상위 DROP AUDIT POLICY / DROP PROFILE은 기존 차단을 유지한다.
- Oracle 통합 감사가 없는 12c 미만 프로필은 검토로 남긴다. 전 버전의 audit action·모든 옵션 지원을 보장하지 않는다.

```sql
CREATE AUDIT POLICY audit_orders
  ACTIONS SELECT ON app.orders, UPDATE ON app.orders;
AUDIT POLICY audit_orders BY app_user WHENEVER NOT SUCCESSFUL;
```

## 검증과 구현 위치

`OracleSecurityPolicy`는 토큰 위치를 보존하는 DdlCursor로 검증하고, `Analyzer`에서 Oracle에만 호출한다. 정확한 소수 일수 표현 변경은 `rule.oracle_profile_interval`로 설정할 수 있다. 기본 AUTO_CONVERT, PRESERVE/REVIEW는 원문 유지·검토 표시다. 기존 파서와 일반 SQL 안전 규칙을 교체하지 않는다.

`test/java/OracleSecurityPolicyTest.java`: 공통 구문, 분수·소수 일수, 값/옵션/검증 함수 차이, 하이라이트, 다중 객체, 인용·주석, 원본 범위, 실제DROP 차단을 검증한다. 전체 `sh test/check.sh`로 기존 변환·보고서·브라우저 서버 회귀를 확인한다. 실제 대상 DB의 CREATE/ALTER/감사 기록·로그인 정책 적용은 **미검증**이다.

## 공식 매뉴얼 근거

- [GOLDILOCKS CREATE PROFILE / CREATE AUDIT POLICY](https://github.com/sunjesoft/goldilocks-manuals/blob/main/docs/goldilocks/26c.1/ko/part-03-sql-manual/19-sql-references-c-g.md)
- [GOLDILOCKS ALTER PROFILE / ALTER AUDIT POLICY / AUDIT POLICY](https://github.com/sunjesoft/goldilocks-manuals/blob/main/docs/goldilocks/26c.1/ko/part-03-sql-manual/18-sql-references-a-b.md)
- [GOLDILOCKS NOAUDIT POLICY](https://github.com/sunjesoft/goldilocks-manuals/blob/main/docs/goldilocks/26c.1/ko/part-03-sql-manual/20-sql-references-h-z.md)
- [Oracle19 CREATE PROFILE](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/CREATE-PROFILE.html)
- [Oracle19 ALTER PROFILE](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/ALTER-PROFILE.html)
- [Oracle19 CREATE AUDIT POLICY](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/CREATE-AUDIT-POLICY-Unified-Auditing.html)
- [Oracle19 ALTER AUDIT POLICY](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/ALTER-AUDIT-POLICY-Unified-Auditing.html)
- [Oracle19 AUDIT](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/AUDIT-Unified-Auditing.html)
- [Oracle19 NOAUDIT](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/NOAUDIT-Unified-Auditing.html)
- [PostgreSQL18 CREATE POLICY](https://www.postgresql.org/docs/18/sql-createpolicy.html)
- [Altibase7.3 SQL Reference](https://github.com/ALTIBASE/Documents/blob/master/Manuals/Altibase_7.3/kor/SQL%20Reference.md#audit)
