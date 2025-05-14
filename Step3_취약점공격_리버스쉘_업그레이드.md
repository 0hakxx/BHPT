# 🛠️ 리버스 쉘 업그레이드

## 일반적인 리버스 쉘 == Dumb(멍청한) 쉘
- 탭 완성, 단축키, Interactive CLI, job control 사용 불가능
- 명령 입력/출력 "만" 가능

**이유 : 단축키 / Job Control은 리버스 쉘을 받은 호스트에서 실행됨**
- 단축키 예시: `Ctrl + C`
- 복사 단축키가 아닌 프로세스 상태 변경 (멈춤, 실행, 백그라운드 등) 되어 리버스 쉘이 종료

** 사용 제약 예시**
- 일반적인 리버스 쉘(Dumb) 상태에서는:
  - `Ctrl + C` 불가능
  - `mysql` 실행 불가능

## 📌 Dumb Shell → Fully Interactive TTY Shell 업그레이드 필요

- 사실상 SSH 혹은 컴퓨터 앞에 앉아있는 것과 동일한 환경 제공
- SSH나 다른 원격 접속 서비스가 불가능한 경우에 사용
- 모든 단축키, job control, interactive CLI 프로그램 실행 가능
- 단, 

🔄 Full TTY 업그레이드 절차
1. **(리버스 쉘)**  
   ```bash
   python3 -c 'import pty; pty.spawn("/bin/bash")'
   Ctrl + Z  # 셸 백그라운드로 전환하여 로컬(공격자) 터미널 프롬포트로 돌아온다.
   ```

2. **(칼리)**  
   `stty raw -echo; fg`
   위 명령어 입력 후 reset 입력 시  다시 bindShell 세션(희생자) 터미널로 돌아온다.

3. **(리버스 쉘)**  
   ```bash
   export SHELL=bash
   export TERM=xterm-256color
   ```

이 방식은 대상 시스템에 Python이 설치되어 있어야만 사용 가능합니다.  Python이 없는 경우 아래 대안을 사용

💡 대안 (Python이 없을 때) 
✅ 전체 절차 (script 명령 사용)
 리버스 쉘로 접속한 후 아래 명령을 실행:
1. **(리버스 쉘)**  
```bash
   /usr/bin/script -qc /bin/bash /dev/null
   Ctrl + Z  # 셸 백그라운드로 전환하여 로컬(공격자) 터미널 프롬포트로 돌아온다.
```

2. **(칼리)**  
`stty raw -echo; fg`
위 명령어 입력 후 엔터를 치면 다시 bindShell 세션(희생자) 터미널로 돌아온다.


3. **(리버스 쉘)** 🔎 필요 시 사용, 테스트 시 해당 절차 없어도 인터랙티브 쉘 가능 확인
```
reset
export SHELL=bash
export TERM=xterm-256color
```


