## 취약점 종류

1. **RCE (Remote Code Execution)**: 원격으로 대상 시스템에 코드나 스크립트를 실행 가능하게 하는 제어하는 취약점  
2. **RCE (Remote Command Execution)**: 원격으로 대상 시스템의 셸이나 인터프리터(Interpreter)에 원하는 시스템 명령어를 실행 가능하게 하는 취약점  
3. **인증 우회(Authentication Bypass)**: 사용자 인증을 우회해 인증 없이 시스템을 사용할 수 있게 되는 취약점  
4. **정보 노출(Data/Information Disclosure)**: 시스템 내/외의 정보(파일, 데이터 등)이 노출되는 취약점  

---

## 취약점 종류와 상황

5. **Path Traversal**: 어플리케이션/시스템에 배정된 정해진 디렉토리 및 파일시스템 스코프를 벗어나 다른 디렉토리/스코프의 파일 및 데이터를 읽어오게 되는 취약점  
6. **DOS (Denial of Service)**: 시스템 자원을 고갈 시키거나 중단시켜 가용성을 방해하는 것을 가능하게 하는 취약점  

---

## 상황

- **인증된 상황(Authenticated)**: 취약점 악용에 사용자 인증이 필수적일 때  
  - 예) Authenticated 원격 코드 실행 → 웹앱에 로그인 후 호스트에 원격 코드 실행  

- **미인증 상황(Unauthenticated)**: 취약점 악용에 사용자 인증이 필요하지 않을 때  
  - 예) Unauthenticated 원격 코드 실행 → 포트와 네트워크 서비스만 열려 있을 때, 사용자 인증 및 계정 정보(유저 이름/비밀번호) 필요 없이 원격 코드 실행이 가능할 때 (log4j, ms17-010 등)  

---

## 실무에서는

- **DOS/DDOS**: 고객사의 사전 요청이 없는 한 금지  
  - 가용성 영향  

- **인증 필요 여부 파악**  
  - 인증된 상황이 필요한 취약점인 경우, 계정 정보 확보 필요  
  - 미인증 상황 RCE == 대박  

- **RCE가 정답은 아님**  
  - 힙(Heap) double free를 바탕으로 after-free를 이용해 ASLR을 우회하고 페이로드를 3단계로 쪼개 메모리 영역에 할당한 뒤 RCE 실행하는 방법보다
    정보 노출 → SSH 공개키 확보 → SSH로 로그인하는 것이 효율적일 수 있음.

- **어떤 네트워크 서비스가 있는가? 어떤 취약점이 있는가? 어떤 취약점들을 조합해 내가 원하는 목표를 달성 할 수 있을 것인가?를 고민**

---

# FTP (File Transfer Protocol) - 21
- **FTP**: 파일 전송 프로토콜. 비 암호화 프로토콜 (예: VSFTPD, ftp 등)
- **서비스 버전 확인**
  - FTP는 오래된 네트워크 서비스로, 알려진 취약점(known exploit)이 있을 가능성이 높음
- **익명 로그인(Anonymous Login) 확인**
  - 계정 정보 없이 FTP 사용 가능
- **파일/디렉토리 읽기/쓰기 권한 확인**
  - 파일 업로드, 다운로드
  - 악성코드 업로드, 중요 정보 다운로드/탈취
- **기본 계정 이름 + 비밀번호 확인**
  - 오래된 네트워크 서비스는 기본 계정 이름과 비밀번호가 존재할 수 있음

---

## FTP 공격 시나리오 예시

### 1. 정보 수집 및 취약점 확인

- **nmap을 통한 FTP 서비스 및 버전 확인**
    ```
    nmap -sV -p 21 -Pn -sC [타겟IP]
    ```
    ```
	nnmap -p 21 -sV --script="*ftp*" -Pn -n --open [타겟IP]
    ```
    - 결과 예시:  
      21/tcp open  ftp     vsftpd 2.3.4
	  ftp-anon: Anonymous FTP login allowed (FTP code 230)

- **알려진 취약점 검색**
    ```
    searchsploit vsftpd 2.3.4
    ```
    - 관련 익스플로잇 존재 여부 확인

---

### 2. 익명 로그인(Anonymous Login) 시도

- **ftp 클라이언트로 접속**
    ```
    ftp [타겟IP]
    ```
    - 로그인 시 아이디: `anonymous`, 비밀번호: 아무 값이나 입력

---

### 3. 파일/디렉토리 다운로드/업로드

- **디렉토리 내 파일 목록 확인**
    ```
    ftp> ls
    ```
    - 예시 결과:  
      `/upload/` 디렉터리 존재 확인
	  `secret.txt` 파일 존재 확인

- **파일 다운로드**
    ```
    ftp> get secret.txt
    ```
    - 파일이 칼리 리눅스(로컬)에 다운로드됨

- **다운로드한 파일 내용 확인**
    ```
    cat secret.txt
    ```

- **로컬 파일을 FTP 서버에 업로드**
    ```
    ftp> put [로컬경로]/[파일명]
    ```
    - 예시:  
      ```
      ftp> put /home/kali/malicious.php
      ```

---

### 중요!!! 4. 추가 확인 및 활용

- 업로드한 파일이 웹 서버에서 실행 가능한지 확인  
  (예를 들어 위의 웹쉘 업로드 후 http://[타겟IP]/uploads/malicious.php 접근)
 
- 현재 익명 로그인이 되었으며 FTP에 업로드한 파일을 웹 서버에서 실행 가능함 >  웹 쉘 실행이 가능

---




## FTP 취약점 진단 시 체크리스트

- **익명 로그인이 가능한가?**
- **어떤 파일을 다운로드 받을 수 있는가?**
- **어떤 파일을 업로드할 수 있는가?**
- **다운로드/업로드 가능한 파일 경로는 어디인가?**
- **업로드한 파일을 다른 네트워크 서비스(예: 웹 서버)에서 실행할 수 있는가?**

---

# SSH (Secure Shell) - 22

- **SSH**: 원격 호스트에 암호화된 방식으로 접근해 명령 실행 및 파일 전송에 사용되는 네트워크 프로토콜  
  - 예) OpenSSH

- 보안 중점, 1999년도 배포, 오픈소스 운영으로 인해 취약점이 많지 않음

- 사용자 인증 방법 (공개키 로그인, 비밀번호 로그인)

- **취약점 예시** 
  - OpenSSH 7.7 미만 버전:  
    유효한 사용자 이름(계정명)을 추측할 수 있는 정보 노출 취약점 존재  
    - 이로 인해 시스템에 존재하는 계정명을 알아내고, 이후 무차별 대입(브루트포스) 공격에 활용 가능  

  - 브루트포스 공격 방어 미흡 혹은 부재  
    - SSH는 기본적으로 로그인 시도 횟수 제한이 약하거나 없는 경우가 많아, 여러 계정/비밀번호 조합을 시도하는 공격에 노출될 수 있음  

  - OpenSSH 6.6 미만 + SFTP 활성화 시 원격 코드 실행 취약점 존재

---

## SSH 공격 시나리오 예시

1. **nmap으로 SSH 서비스 및 버전 확인**
    ```
    nmap -sV -sC -Pn -p 2222 [타겟IP]
    ```
    - 예시 결과:  
      ```
      2222/tcp open  ssh  OpenSSH 7.2p2 Ubuntu
      ```
    - OpenSSH 7.2p2는 7.7 미만으로, 사용자 이름 정보 수집 취약점에 포함됨

2. **사용자 이름 리스트 준비**  
   - Google에서 `username_seclist github` 검색 후, 사용자들이 자주 사용하는 계정명 리스트를 다운로드  
   - 칼리 리눅스(로컬)에 `username.txt` 파일로 저장

3. **Metasploit을 이용한 사용자 이름 정보 수집 (ssh_enumuser 모듈)**
    ```
    msfconsole
    ```
    ```
    search ssh_enumuser
    use auxiliary/scanner/ssh/ssh_enumuser
    options
    set rhosts [타겟IP]
    set rport 2222
    set user_file /home/kali/username.txt
    run
    ```
    - 결과로 유효한 계정을 알 수 있음

4. **발견한 사용자 이름을 바탕으로 브루트포스 공격 시도 (예: hydra)**
    ```
    hydra -L valid_users.txt -P passwords.txt ssh://[타겟IP] -s 2222
    ```

---

### SSH 취약점 진단 시 체크리스트

**존재하지 않는 사용자로 브루트 포스 취약점이 존재하는지 확인**
- 실무에서는 최소한으로 진행, 브루트포스 공격으로 인하여 계정 잠김 이슈 가능성 존재하므로 아래 명령어 사용
	- `hydra -L thisuserdoesnotexit -P password.txt ssh://<IP>:<Port> -V -f` 
	- 존재하지 않는 유저 이름을 사용해 계정 잠금 방지
	- 비밀번호 사전 파일의 사용되는 비밀번호는 30개 내외
	- 엄격한 내부망/사내망 시 고객사 허락 필요
	- 중간에 IP가 차단되어 공격이 불가하다면, 브루트 포스 취약점 X


---

### 참고 및 대응 방안

- SSH 서버를 최신 버전으로 업데이트  
- 비밀번호 인증 비활성화 및 공개키 인증 활성화 권장  
- fail2ban, iptables 등으로 로그인 시도 제한 설정  
- 불필요한 계정 비활성화 및 강력한 비밀번호 정책 적용  
- SFTP 설정 시 보안 취약점 점검  

---

# HTTP (Hypertext Transfer Protocol)

- HTTP/S: 웹에서 정보를 교환하기 위한 애플리케이션 프로토콜. 웹 서버 및 웹 어플리케이션과 통신하기 위해 사용.
- 웹 보안 - 매우 중요, 매우 큰 도메인. BHPT에서는 기초만.
- 웹 서버 버전 + 웹 어플리케이션 + 프레임워크 버전 확인
    - 공개 익스플로잇(Known Exploit) 기반 모의해킹 기초
- OWASP Top 10 및 흔한 웹 취약점 확인
- 디렉토리 브루트포싱
- 버츄얼 호스트 (Virtual Host) 이름 확인

- **Source & Sink**
  - 유저는 어느 "장소"를 통해 입력값을 제공하는가? (Source)
    - GET/POST 파라미터, API 요청, 등
  - 유저의 입력값은 어디에서 사용되는가? (Sink)
    - 유저 이름 & 비밀번호 -> 데이터베이스에서 계정 정보 확인
    - 제품 ID -> 데이터베이스에서 제품 Lookup
    - 페이지 이름 -> 백엔드에서 파일을 읽어와 "포함" 시키기

- **공격자 관점에서 고려할 점**
	- 나는 어떤 종류/형식의 입력값을 제어할 수 있는가?
	- Sink에서 내 입력값이 어떻게 사용되는가?
	- 이를 악용할 페이로드는 어떻게 만들 수 있는가?

- **웹 서버, 어플리케이션, 기술 스택 확인  **
  - 도구 예시: Burp Suite, Wappalyzer(크롬 확장 도구, 기술이나 버전 확인 가능)

- **robots.txt** 파일 확인  
  - 브라우저 또는 Burp Suite로 접근해 숨겨진 경로나 민감 정보 확인

- OWASP Top 10 기반 취약점 점검

- 흔한 웹 공격 기법  
  - 기본 계정 이름 + 비밀번호 조합 점검  
  - Local/Remote File Inclusion (LFI/RFI)  
  - 커맨드 인젝션 (Command Injection)  
  - 파일 업로드 우회  
  - SQL 인젝션 (SQL Injection)  
  - CSRF (Client-Side Request Forgery)  
  - SSRF (Server-Side Request Forgery)

---

## LFI 취약점 (Local File Inclusion)

- **정의**: 웹 애플리케이션이 외부 입력값을 통해 서버 내 로컬 파일을 포함(include)할 때 발생하는 취약점
- **위험성**: 공격자는 `/etc/passwd` 등 민감 파일을 읽거나, 로그 파일을 통한 원격 코드 실행 시도 가능
- **주요 발생 환경**: PHP, JSP, ASP 등에서 `include()`, `require()` 등 함수에 사용자 입력이 검증 없이 사용될 경우 발생
- **활용**: LFI 취약점이 발견된다면, 웹 쉘 업로드 후, URL에 직접 접근하지 않고도 실행이 가능함

**LFI 취약점 탐지 도구 예시 - LFImap**

- GitHub 저장소: https://github.com/hansmach1ne/LFImap
- 설치 및 실행 방법:
`git clone https://github.com/hansmach1ne/LFImap`
`cd LFImap`
`pip3 install -r requirements.txt`
`python3 lfimap.py -U "<대상URL>" -C "<쿠키값>" -a`

---

### LFI + 이미지 파일을 이용한 웹쉘 업로드 우회 기법

1. **xxd 도구 설치**
- `sudo apt install xxd`
2. **정상 이미지 파일의 바이트 확인**
- `xxd cat.png`
- 이 명령어로 이미지 파일의 헤더(시그니처) 확인 가능
3. **이미지 헤더와 웹쉘 결합**
- `(head -c 8 cat.png; cat webshell.php) > webshell-cat.php`
- 정상 이미지의 첫 8바이트(이미지 헤더)를 웹쉘 앞에 추가
- 이렇게 하면 파일 시그니처 검사에서 이미지로 인식됨
4. **생성된 파일 검증**
- `xxd webshell-cat.php`
- 결합된 파일이 이미지 헤더로 시작하는지 확인
5. **파일 업로드 시 확장자 변경**
- Burp Suite를 이용해 `webshell-cat.php` 파일을 업로드할 때 확장자를 `.png`로 변경
- 서버에는 `webshell-cat.png`로 업로드됨

여기서 
**중요한 점은, 일반적으로 파일 업로드 시 이미지 확장자는 웹 서버에서 실행되지 않는다.그러나 LFI 취약점 연계 공격 시 이미지 파일도 실행이 가능함**

---

## 파일 업로드 자동화 도구 활용 (fuxploider)

- **fuxploider**: 다양한 확장자/우회 기법을 자동으로 테스트하여 파일 업로드 취약점을 진단하는 도구
  - **설치 및 사용법**
    ```
	git clone https://github.com/almandin/fuxploider.git
	cd ./fuxploider
	pip3 install -r requirements.txt
	python3 fuxploider.py --url <대상 URL/업로드 경로> --cookies "<쿠키>" --not-regx "<실패 판단 문자열>" -d "<필요 파라미터명=값>"
    ```
- 결과에서 `<INFO>`를 통해 어떤 확장자나 우회 기법으로 파일 업로드가 가능한지 알려줌
- "Y" 입력 시, 실제 업로드 진행
- 업로드 성공/실패(True/False) 결과와 업로드 파일명을 알려줌
- 업로드한 파일명은 반드시 기록하여 고객사에 추후 제거 요청 필요

---


# 디렉토리 브루트포싱
- **디렉토리 브루트포싱 목적**
  - 숨겨진 웹앱/관리자 페이지/민감 파일 등 추가 엔드포인트 발견
  - 예시:  
    - http://<ip>/phpmyadmin  
    - http://<ip>/cgi/users.sh
- **디렉토리/파일 브루트포싱 명령어 예시**
  - 사전 파일의 경로는 `/usr/share/dirbuster/wordlists` 위치
  - 디렉토리 브루트포싱:
    ```
	apt install gobuster
    gobuster dir -u <URL> -w <사전 경로/파일명>
    gobuster dir -u <URL> -w <사전 경로/파일명> -s "<HTTP 응답 코드>"
    gobuster dir -u <URL> -w <사전 경로/파일명> --exclude-length <응답 크기>
    ```
  - 파일 이름 브루트포싱:
    ```
    gobuster dir -u <URL> -w <사전 경로/파일명> -x <확장자>
    ```
  - **(-t) 스레드 조심**: 컨테이너, EC2, K8s 등 DOS 공격에 약한 호스트는 스레드 수 조절 필요

- **기타 도구**
  - wfuzz, fuff, dirb 
  

---
 
# SMB (Server Message Block) - 445
- **SMB**: FTP와 유사하게 파일 및 프린터 공유 프로토콜로, 명령 수행과 원격 접속에도 사용 가능  
- **Null Session**: 계정 이름과 비밀번호 없이 접속 후 파일 업로드/다운로드 가능
- **파일 읽기/쓰기 권한 확인**: 공유된 파일에 대해 읽기 및 쓰기 권한이 있는지 점검  
- **접근 가능한 쉐어(Share) 및 파일 확인**: 네트워크 상 공유된 폴더 및 파일 목록 확인  
- **SMB 관련 주요 취약점 점검**  
  - MS08-067 (Windows Server 서비스 원격 코드 실행 취약점)  
  - MS17-010 (EternalBlue 취약점, WannaCry 랜섬웨어 악용)  
  - SMBGhost (CVE-2020-0796, SMBv3 원격 코드 실행 취약점)  
  - MS15-011 등 기타 SMB 취약점  
- **네트워크 기반 공격 관련 설정 점검**  
  - SMB Signing 활성화 여부  
  - SMBv1 사용 여부 (구버전 SMB 프로토콜로 보안 취약)  
## SMB 취약점 진단 실무 명령어 및 도구
- **1.SMB 서비스 및 버전 확인 (nmap)**
  - `nmap -Pn -p 445 -sV -sC -n --open <대상IP>`
- **2.공개 취약점 검색**
  - `searchsploit samba <버전> -w`
  - `-w` 옵션 사용하여, URL 접속하여 어떤 공격인지 상세 확인 가능 
- **3. Null Session 및 공유 폴더 정보 수집 (enum4linux)**
  - `enum4linux -a <대상IP>`	
  - "Session Check" 부분에서 id, pw 없이 세션 연결 성공 시 Null Session 가능  
  - "Share Enumeration" 부분에서 공유 폴더 및 권한 확인 가능  
  - 권한에 따라 읽기/쓰기 가능 여부 판단 → 공격 경로 확보에 중요
- **4. 최신 실무용 SMB 진단 도구 (NetExec)**
  - ```	
	sudo pip3 install pipx
	pipx install https://github.com/Pennyw0rth/NetExec.git
	pipx ensurepath
	새 터미널 연 뒤,
	nxc smb <대상IP>
	nxc smb <대상IP> -u '' -p '' --shares
	```
  - Null Session 및 공유 폴더 권한 상세 확인 가능  
  - 자동화된 SMB 취약점 진단 및 권한 분석 지원
- **5. 칼리 리눅스 기본 SMB 진단 도구 (smbmap)**
  - `smbmap -H <대상IP>`	
	- 공유 폴더 및 권한 확인  
	- 읽기/쓰기 가능 여부 빠르게 파악 가능
	
- **6. SMB 클라이언트 접속 및 파일 조작**
  - `smbclient \\\\<대상IP>\\samba-share -U ""`	
	- 익명 접속 시 `-U ""` 사용
	- `ls`, `get`, `put` 명령어로 파일 목록 확인, 다운로드, 업로드 가능
- **7. 상세 취약점 진단**
  - `nmap -p 445 -sV --script="smb-vuln-*" -Pn -n --open <대상IP>`	
	- namp의 기본 제공하는 smb 관련 모든 스크립트로 진단 수행
## ⚠ 실무 팁 및 주의사항
- SMB는 윈도우 환경에서 매우 중요한 네트워크 프로토콜로, 취약점 진단 및 대응이 반드시 필요합니다.  

---

# NFS (Network File System)
- **NFS**: 네트워크 상에서 호스트의 파일 및 디렉토리를 공유하는 프로토콜/네트워크 서비스
- 주로 *Nix 서버들에서 많이 사용
- 111 - RPCbind/Portmapper: NFS와 관련된 포트, 프로세스 등의 정보 확인
- 2049 - NFS: 데이터 전송을 위한 실제 NFS 서버
- **접근 제어**: 어떤 디렉토리들이 노출 중이며, 어떤 호스트가 접근 가능한가 확인
- **파일 권한**: 파일 및 디렉토리들의 읽기/쓰기 권한 확인
- **No_Root_Squash**: 클라이언트들의 유저 권한을 무시하는 no_root_squash 설정이 비활성화 되어 있는지 확인
## NFS 취약점 진단 및 실무 활용
- **1. 서비스 및 취약점 스캔**
  - `nmap -p 111,2049 -Pn -n --open -sV --script="nfs-*" <대상 IP>`
	- NFS 서비스가 열려있는지, 어떤 공유 디렉토리가 노출되어 있는지, 익명 접근 가능 여부 등 자동 스크립트로 확인
	- 결과의 `/public-nfs` 등 공유 디렉토리 이름과 접근 가능한 IP 대역 정보 확인 가능
	
- **2. 조금 더 자세하게 알아보려면 직접 마운트하여 직접 NFS에 접근하여 확인**
	- `showmount -e <대상 IP>`
     - NFS 서버에서 외부에 공개(export)된 디렉토리 목록과, 각 디렉토리에 접근 가능한 IP 대역을 확인할 수 있음
	 **마운트 디렉토리 생성**
	 - `sudo mkdir /mnt/testo`
	 **NFS 공유 마운트**
	 - `sudo mount <대상IP>:/public-nfs /mnt/testo`
	 **디렉토리 접근 및 파일 목록 확인**
		 ```
		cd /mnt/testo
		ls -al
		```
    - 실제 NFS 서버의 파일 및 디렉토리 목록이 보이면, 접근이 허용된 것임
    - 파일 생성/삭제/수정 등도 시도하여 쓰기 권한까지 확인
**No_Root_Squash**:  
    - no_root_squash 옵션이 설정되어 있으면, 클라이언트에서 root 권한으로 접근 시 실제 서버에서도 root 권한으로 동작  
    - 이 설정이 비활성화되어 있지 않으면, 공격자가 root 권한으로 시스템 파일을 조작하거나 백도어 설치 등 심각한 보안 위협이 발생할 수 있음
	- 확인하는 방법은
	  `touch root_test.txt`
	  `ls -al`
	  직접 파일 생성하여 생성된 파일의 소유자가 root로 표시되면 no_root_squash가 활성화됨
# MySQL (My-Structured Query Language)

- **MySQL**: 오픈소스 관계형 데이터베이스 관리 시스템
- **버전 정보 및 호스트 이름 확인**
- **접근 제어 및 기본 계정 정보 확인**  
  (예: root:"", root:root, root:password)
- **접속 가능 여부 확인 → 데이터베이스 데이터 탈취 가능성 점검**
  - 유저 계정 정보 확인
  - 비밀번호 해시화 여부 확인
  - 취약한 해시 알고리즘 사용 여부 확인
  - 해시 크래킹(Hash Cracking) 개념 증명용
- **탈취한 계정 정보를 다른 네트워크 서비스에 사용 가능 여부 확인**  
  (크레덴셜 스터핑 Credential Stuffing)


## MySQL 취약점 진단 명령어 및 절차
**1. Nmap을 이용한 MySQL 서비스 및 정보 수집**
- `nmap -p <대상 포트> -sV -sC --script="mysql*" -Pn -n --open <대상IP>`
 - 결과의 `mysql-enum`, `mysql-info` 부분에 계정 정보, 버전 등 다양한 정보 확인 가능
 - 예시:  
   - `root:<empty>` → root 계정의 비밀번호 Null로 로그인 가능함을 의미
   
**2. 직접 MySQL 접속 및 데이터베이스 정보 확인**
 `mysql -u root -p -h <대상IP> -P <대상 포트>`
 - 접속 후 중요한 데이터베이스 및 테이블 내용 탈취 가능
 
**3. 탈취된 비밀번호 해시화 및 취약점 점검**
- 칼리 리눅스에서 `hash-identifier` 
- 탈취한 비밀번호 해시를 입력하면 어떤 암호화 알고리즘인지 식별 가능

**4. 취약점 판단 기준**
- 비밀번호가 평문으로 저장되어 있으면 명백한 취약점
- 암호화되어 있어도 취약한 해시 알고리즘(MD5, SHA1 등)을 사용하면 취약
- 강력한 해시 알고리즘(SHA256 이상)과 솔트(salt) 적용 여부도 중요

---