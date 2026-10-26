# GOLDILOCKS Migrator 다운로드

실행 패키지 배포 전용 저장소입니다. 애플리케이션 소스 코드는 포함하지 않습니다.

현재 버전: **0.1.34** · 실행 환경: **Java 8 이상(JRE)**

## 다운로드

- [Windows / 공통 ZIP](sqlmigrate-0.1.34-portable.zip?raw=true)
- [Linux TAR.GZ](sqlmigrate-0.1.34-linux.tar.gz?raw=true)
- [ZIP SHA-256](sqlmigrate-0.1.34-portable.zip.sha256)
- [TAR.GZ SHA-256](sqlmigrate-0.1.34-linux.tar.gz.sha256)

## 실행

Windows에서는 ZIP 압축을 풀고 `run-web.cmd`를 실행합니다.
Linux에서는 압축을 푼 디렉터리에서 `sh run.sh web`을 실행합니다.
공통 명령은 `java -jar sqlmigrate.jar web`입니다.

웹의 **입력변환** 버튼으로 직접 SQL을 입력하고 Altibase / Oracle / PostgreSQL을 선택해 변환할 수 있습니다. 파일·폴더 변환 및 기존 터미널 검토 모드도 제공합니다. 자세한 설정은 패키지의 README와 docs 문서를 참조하십시오.

변환 결과는 검토용 초안이며, 입력변환 및 파일 변환 과정에서 대상 DB에 SQL을 실행하지 않습니다. 패키지에는 변환 테스트용 SQL 예제와 필요한 라이선스 문서가 포함됩니다.
