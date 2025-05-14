
## 1. 사용자 정보 및 계정 탐색

- LFI 취약 파라미터에 `/etc/passwd` 입력
	- `etc/passwd` 파일을 LFI로 조회  
	- 리눅스 시스템의 사용자 계정 목록 확인 가능
	- `/bin/bash`를 사용하는 실제 사용자 계정(예: groot) 파악

## 2. SSH 개인키 탈취 및 접속
**타겟 계정(groot)의 SSH 개인키 다운로드**

- LFI 취약 파라미터에 `/home/groot/.ssh/id_rsa` 입력
	- groot 계정의 SSH 개인키가 노출되는 경우, 전체 내용을 복사
	- `vim groot-privatekey.pem` 편집기로 파일 생성 후 내용 붙여넣기
	- `chmod 600 groot-privatekey.pem`
- `ssh -i groot-privatekey.pem groot@<타겟IP>` 탈취한 계정으로 SSH 접속


## 3. 추가 정보 수집

### 3.1 DB 정보
- LFI 취약 파라미터에 `/etc/mysql/my.cnf` 입력
	- DB 계정, 비밀번호 등 민감 정보 확인

### 3.2 Samba 경로 확인 및 Reverse Shell 업로드


- LFI 취약 파라미터에 `/etc/samba/smb.conf` 입력
	- 공유 경로(path) 확인
	- 공유 경로 확인하여 FTP+Reverse 연계 공격과 유사하게 put 명령어를 사용하여 Reverse 쉘 업로드 후, 웹에서 공유 경로로 접근하여 nc 연결 가능

---
