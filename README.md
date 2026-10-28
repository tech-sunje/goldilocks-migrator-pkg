# GOLDILOCKS Migrator 배포 패키지

Altibase / Oracle / PostgreSQL의 SQL을 **GOLDILOCKS용 SQL 초안으로 분석·변환**하는 도구입니다.

**현재 버전: 0.1.43 · 실행 환경: Java 8 이상(JRE)**

이 저장소는 실행 패키지와 설명서를 공개 배포하는 용도입니다. 프로그램 개발 소스와 비공개 소스 저장소의 Git 이력은 포함하지 않습니다. 패키지의 `source/` 폴더에는 변환을 시험할 수 있는 **SQL 입력 예제**가 들어 있습니다.

## 다운로드

GitHub 로그인 없이 아래 링크로 다운로드할 수 있습니다.

| 파일 | 용도 |
| --- | --- |
| [sqlmigrate-0.1.43-portable.zip](https://github.com/tech-sunje/goldilocks-migrator-pkg/raw/refs/heads/main/sqlmigrate-0.1.43-portable.zip) | Windows 및 공통 배포 ZIP |
| [sqlmigrate-0.1.43-linux.tar.gz](https://github.com/tech-sunje/goldilocks-migrator-pkg/raw/refs/heads/main/sqlmigrate-0.1.43-linux.tar.gz) | Linux 배포 압축파일 |
| [ZIP SHA-256](sqlmigrate-0.1.43-portable.zip.sha256) | ZIP 무결성 확인 |
| [TAR.GZ SHA-256](sqlmigrate-0.1.43-linux.tar.gz.sha256) | TAR.GZ 무결성 확인 |
| [SHA256SUMS](SHA256SUMS) | 두 압축파일의 SHA-256 목록 |

압축파일을 **해제한 후** 실행하십시오. 실행에 별도 JDK, Node.js 또는 Python은 필요하지 않습니다.

## 0.1.43 변경 사항

- REM/REMARK·PROMPT·SQL*Plus 전용 SET을 행 주석으로 자동 처리합니다.
- GROUPING SETS·IDENTITY·암시적 커서 FOR 및 내장 기본값의 인식을 보강했습니다.
- 이름 없는 UNIQUE는 호환으로 분류하고, 동적 SQL 내용 확인과 실행 불가 판정을 구분합니다.
- 숫자 문자열 연결과 날짜/숫자 연산의 오진을 수정했습니다. AUTONOMOUS_TRANSACTION은 검증 대상에서 구문 오류가 확인돼 수동 검토로 유지합니다.
- 입력변환에 전체 건수·자동변환(호환) 비율을 표시하고 실제 문제 행의 원문/변환 강조를 보정했습니다.

배포 기준 소스 커밋과 빌드 정보는 [BUILD-INFO.txt](BUILD-INFO.txt)에 기록합니다. 기존 프로그램을 종료하고 새 패키지로 실행하십시오.

## 1. 실행

### Windows

1. `java -version`으로 Java 8 이상이 설치되어 있는지 확인합니다.
2. ZIP을 다운로드하고 원하는 폴더에 압축을 풉니다.
3. 압축을 푼 폴더에서 **`run-web.cmd`**를 더블클릭합니다.
4. 브라우저에서 GOLDILOCKS Migrator 화면을 확인합니다.

브라우저가 자동으로 열리지 않으면 실행 창에 표시되는 `Browser URL:` 주소를 복사해 브라우저에서 여십시오.

명령 프롬프트에서도 실행할 수 있습니다.

```bat
java -jar sqlmigrate.jar web
```

`java`를 찾을 수 없다면 Java 실행 경로를 `PATH`에 추가하거나 `JAVA_HOME`을 설정하십시오. `run-web.cmd`는 `JAVA_HOME`이 설정되어 있으면 해당 Java를 사용합니다.

### Linux

```sh
tar -xzf sqlmigrate-0.1.43-linux.tar.gz
cd sqlmigrate-0.1.43
sh run.sh web
```

공통 Java 명령도 사용할 수 있습니다.

```sh
java -jar sqlmigrate.jar web
```

브라우저를 자동으로 열지 않고 실행하려면 다음과 같이 실행합니다.

```sh
java -jar sqlmigrate.jar web --no-browser
```

웹 화면은 프로그램을 실행한 PC의 로컬 주소에서 동작합니다. 브라우저 탭만 닫으면 Java 프로그램은 계속 실행됩니다. 종료할 때는 웹 화면의 **프로그램 종료** 버튼 또는 실행 터미널의 `Ctrl+C`를 사용하십시오.

## 2. SQL 직접 입력변환

파일을 만들지 않고 SQL을 붙여 넣어 시험할 수 있습니다.

1. 메인 화면에서 **프로그램 종료 왼쪽의 `입력변환`** 버튼을 누릅니다.
2. 새 창 상단에서 **Altibase / Oracle / PostgreSQL**과 원본 버전을 선택합니다.
3. 왼쪽 **원본 SQL** 편집기에 SQL을 입력하거나 붙여 넣습니다.
4. 가운데 **변환 →** 버튼 또는 `Ctrl+Enter`를 누릅니다.
5. 오른쪽 **변환 SQL 초안**과 아래 판정 내역을 확인합니다.

원본과 변환 SQL은 모두 편집할 수 있으며 각각 복사·SQL 저장 버튼을 제공합니다. 창 크기에 따라 편집기와 표 크기가 조정되고, 좁은 화면에서는 두 편집기가 위아래로 배치됩니다.

### 설정과 규칙

- 새 창은 처음 열 때 메인 화면에서 편집한 INI 설정과 변환 규칙을 가져옵니다.
- 새 창의 소스 DB·버전 선택에 맞춰 해당 벤더의 분석 프로필과 변환 규칙을 적용합니다.
- 메인 설정을 다시 가져오려면 **메인 설정 가져오기**를 누릅니다.
- 입력변환에는 입력 파일·출력 폴더를 지정할 필요가 없습니다.
- 입력변환 결과는 메인 화면의 파일 변환 결과를 덮어쓰지 않습니다.

### 판정과 하이라이트

| 분류 | 의미 |
| --- | --- |
| 자동 변환 | 설정된 규칙으로 SQL 초안을 변환한 항목 |
| 추가 확인 | 반환 타입·비교 의미 등 확인이 필요한 항목 |
| 수동 검토 | 대상 미지원 기능이나 별도 수정이 필요한 항목 |
| 호환·원문 유지 | 규칙 범위에서 원문을 유지한 항목 |

표의 행을 선택하면 해당 SQL 위치를 강조합니다. 원본이나 설정을 바꿨다면 **다시 변환**하여 판정과 하이라이트를 갱신하십시오. 변환 SQL을 직접 수정한 경우 기존 판정은 재분석되지 않으며 원문 위치만 강조합니다. 입력변환은 한 번에 200,000자 이내의 SQL을 처리합니다.

## 3. SQL 파일·폴더 변환

1. 필요하면 기존 INI를 선택하고 **불러오기**를 누릅니다. INI 없이 화면에서 설정할 수도 있습니다.
2. 원본 SQL **파일 또는 폴더**를 지정합니다. 폴더를 선택하면 하위 폴더의 `.sql` 파일도 분석합니다.
3. **결과 저장 폴더**를 지정합니다.
4. **원본 DB와 버전**을 확인합니다.
5. **추가 설정 · INI 전체 항목 · 개별 변환 규칙**에서 규칙을 확인하거나 편집합니다.
6. **분석·변환 실행**을 누릅니다.
7. HTML 보고서에서 객체별 원본·변환 SQL, 판정과 하이라이트를 확인합니다.

보고서는 화면에서 보거나 새 탭으로 열 수 있고, HTML·Excel·전체 결과 ZIP을 저장할 수 있습니다. 원본 SQL 파일은 변경하지 않습니다. 기존 결과 폴더를 교체하려면 화면에서 교체 허용을 선택해야 합니다.

일반적인 결과 폴더에는 다음 항목이 생성됩니다.

```text
converted/          변환 SQL 초안
review/             객체별 검토 내역
reports/report.html HTML 보고서
reports/report.xlsx Excel 보고서(선택 시)
changes.json        분석·변환 내역
```

보고서에서 SQL을 직접 수정한 경우 상단 **변환 SQL 전체 저장**으로 수정본을 다운로드하십시오. 화면에서 편집한 내용은 브라우저의 현재 화면에 보관되며 새로고침 후 유지되지 않습니다.

## 4. 포함된 데모

각 DBMS의 입력 SQL을 다른 폴더로 분리했습니다.

| 설정 파일 | 원본 DB | 입력 폴더 | 기본 출력 폴더 |
| --- | --- | --- | --- |
| `migrate.ini` / `alti-demo.ini` | Altibase 7.3 | `source/alti` | `target/alti` |
| `ora-demo.ini` | Oracle 19c | `source/ora` | `target/ora` |
| `pg-demo.ini` / `postgresql-demo.ini` | PostgreSQL 18 | `source/pg` | `target/pg` |

웹에서 INI를 불러오거나 다음 명령으로 실행합니다.

```sh
java -jar sqlmigrate.jar --config ora-demo.ini
java -jar sqlmigrate.jar --config pg-demo.ini
```

테이블 자료형 예제, 300행 이상의 프로시저·함수·패키지 예제와 수동 검토용 벤더 전용 객체가 포함됩니다. 예제 SQL에 실제 DB에서 생성 가능한 기능과 수동 검토용 기능이 함께 있으므로 판정 내역을 확인하십시오. 데모의 기본 설정은 Altibase 입력만 대상으로 합니다.

PostgreSQL SERIAL 계열 예제는 각 정수 범위에 맞는 IDENTITY 변환을 확인할 수 있도록 타입별 테이블로 나누었습니다.

## 5. 터미널 모드

브라우저 없이 파일 변환을 실행합니다.

```sh
java -jar sqlmigrate.jar --config migrate.ini
```

기존 TUI 검토 화면을 사용하려면 다음과 같이 실행합니다.

```sh
java -jar sqlmigrate.jar review --config migrate.ini
```

Linux에서는 `sh run.sh review --config migrate.ini`도 사용할 수 있습니다. 입력변환 새 창과 웹의 화면 배치 기능은 웹 전용이며 SQL 분석·변환 규칙은 공통 엔진을 사용합니다.

## 6. 무결성 확인

Linux에서는 내려받은 파일과 체크섬 파일을 같은 폴더에 두고 실행합니다.

```sh
sha256sum -c sqlmigrate-0.1.43-portable.zip.sha256
sha256sum -c sqlmigrate-0.1.43-linux.tar.gz.sha256
```

Windows PowerShell에서는 다음 값과 체크섬 파일의 값을 비교합니다.

```powershell
Get-FileHash .\sqlmigrate-0.1.43-portable.zip -Algorithm SHA256
```

## 7. 사용 범위

- 파일 변환과 입력변환은 정적 분석으로 동작하며 대상 DB에 SQL을 실행하지 않습니다.
- 변환 결과는 검토용 초안입니다. 판정 내역과 SQL을 확인한 후 테스트 DB에서 생성·실행을 검증하십시오.
- 모든 벤더의 모든 SQL 문법을 완전히 지원한다는 의미는 아닙니다. 미지원 기능과 판정 범위는 보고서에 표시합니다.
- 패키지에는 필요한 외부 라이브러리의 라이선스 문서가 포함됩니다.

자세한 설정과 규칙은 압축파일 안의 `docs/user_guide.md`, `docs/browser_quickstart.md`, `docs/demo_catalog.md`를 참조하십시오.

0.1.42: Altibase INSERT의 빈 문자열을 자동 호환으로 분류합니다. BIGINT 정수 문자열 리터럴은 명시적 CAST로 자동 변환하며, 잘못된 정수와 64비트 범위 초과는 검토 대상으로 유지합니다.

0.1.43: Altibase ALTER SESSION SET AUTOCOMMIT = TRUE/FALSE를 gsql SET AUTOCOMMIT ON/OFF로 자동 변환합니다. 주석과 뒤따르는 SQL은 보존합니다.
