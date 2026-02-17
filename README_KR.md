# Born2beroot 42 가이드

![Born2beroot](img/born2beroot.png)

> 이 문서는 [chlimous](https://github.com/chlimous)의 Born2beroot 가이드를 한국어로 번역한 것입니다.
> Subject Version 5.2 기준으로 업데이트되었습니다.

## 소개

Born2beroot은 가상화(Virtualization)와 서버 관리(Server Administration)의 세계를 소개하는 프로젝트입니다.
이 가이드에서는 가상 머신(VM) 생성과 운영체제 설치부터 서비스 구성과 보안 정책 적용까지, 프로젝트 전 과정을 단계별로 안내합니다.

---

## 목차

1. [가상 머신 생성](#가상-머신-생성)
    1. [하이퍼바이저](#하이퍼바이저)
    2. [리눅스 배포판 선택](#리눅스-배포판-선택)
    3. [시작하기](#시작하기)
2. [운영체제 설치](#운영체제-설치)
    1. [부팅](#부팅)
    2. [파티셔닝](#파티셔닝)
3. [SSH 설정](#ssh-설정)
4. [방화벽](#방화벽)
5. [sudo와 그룹](#sudo와-그룹)
    1. [sudo](#sudo)
    2. [그룹](#그룹)
6. [비밀번호 정책](#비밀번호-정책)
7. [모니터링 스크립트](#모니터링-스크립트)
    1. [스크립트 작성](#스크립트-작성)
    2. [Cron Job](#cron-job)
8. [WordPress 설치](#wordpress-설치)
    1. [패키지 설치](#패키지-설치)
    2. [방화벽 설정과 PHP 활성화](#방화벽-설정과-php-활성화)
    3. [포트 포워딩](#포트-포워딩)
    4. [데이터베이스](#데이터베이스)
    5. [통합 구성](#통합-구성)
9. [추가 서비스](#추가-서비스)
    1. [무엇을 선택할까?](#무엇을-선택할까)
    2. [Netdata 설정](#netdata-설정)
10. [서명과 스냅샷](#서명과-스냅샷)

---

## 가상 머신 생성

### 하이퍼바이저

가상 머신을 생성하려면 먼저 **하이퍼바이저(Hypervisor)** 가 필요합니다.

하이퍼바이저는 두 가지 유형으로 나뉩니다:

**Type 1 (Bare-metal, 베어메탈형):**
호스트 하드웨어 위에서 직접 실행되며, 리소스를 최적화하여 배분합니다. VMware ESXi, Microsoft Hyper-V, Proxmox 등이 대표적이며, 주로 서버 환경에서 사용됩니다.

**Type 2 (Hosted, 호스트형):**
기존 운영체제 위에서 추가 소프트웨어 계층(layer)으로 동작하며, 가상 머신의 관리와 접근이 비교적 간편합니다. VirtualBox, VMware Workstation 등이 이에 해당하며, 개인 PC에서 주로 사용됩니다.

이 가이드에서는 프로젝트 요구사항에 따라 오픈소스 Type 2 하이퍼바이저인 **VirtualBox**를 사용합니다.

### 리눅스 배포판 선택

리눅스 배포판은 **Rocky Linux**와 **Debian** 중 하나를 선택할 수 있습니다.

- **Rocky Linux**: 개발이 중단된 CentOS의 후속 프로젝트로, 서버 환경에 적합한 안정적인 플랫폼을 제공합니다.
- **Debian**: 오랜 역사와 검증된 안정성으로 유명하며, Ubuntu의 기반이 되는 배포판입니다.

시스템 관리가 처음이라면 **Debian**이 좋은 선택입니다.

### 시작하기

먼저 최신 Debian ISO 이미지를 [Debian 공식 웹사이트](https://www.debian.org/distrib/netinst)에서 "amd64"를 선택하여 다운로드합니다.

VirtualBox를 열고 "New"를 클릭합니다. 가상 머신의 이름을 지정하고, 다운로드한 ISO 이미지를 선택한 후, 데이터를 저장할 위치를 지정합니다. 42 서울에서 작업 중이라면 home 디렉터리 용량 제한을 고려하여 외부 드라이브나 sgoinfre를 활용하세요. 다음 단계로 넘어가기 전에 반드시 **"Skip Unattended Installation"** 을 체크하세요.

이제 가상 머신에 리소스를 할당합니다. 최소 **4GiB RAM**과 **2 vCPU 코어**를 할당하는 것을 권장합니다.

스토리지는 **10GiB** 면 충분하지만, 여유를 두고 20GiB를 설정해도 좋습니다. **"Pre-allocate Full Size"** 를 체크합니다.

> **v5.2 참고:** 디스크 크기는 각 파티션이 정상적으로 작동하면서도 불필요한 디스크 사용을 피할 수 있도록 적절히 결정해야 합니다.

![Virtualbox](img/vm_creation.gif)

---

## 운영체제 설치

### 부팅

가상 머신이 준비되었으므로, 일반 컴퓨터에 운영체제를 설치하는 것처럼 OS를 설치합니다.

VirtualBox에서 "Start"를 클릭합니다. "Install"을 선택하고 화면의 안내를 따릅니다. Debian의 설치 과정은 Arch나 Gentoo 같은 배포판에 비하면 매우 간단합니다.

![Debian1](img/debian1.gif)

호스트명(hostname)은 프로젝트 지침에 따라 **본인의 로그인 뒤에 42를 붙여** 설정합니다 (예: `wil42`). 이것은 로컬 네트워크에서 해당 장치를 식별하는 이름입니다.

도메인 이름(domain name) 설정은 건너뜁니다. **root 비밀번호**를 설정합니다. 본인의 로그인 이름으로 사용자를 생성하고 비밀번호를 설정합니다.

![Debian2](img/debian2.gif)

### 파티셔닝

디스크를 파티셔닝할 차례입니다. **"Guided - use entire disk and set up encrypted LVM"** 을 선택한 후 **"Separate /home, /var and /tmp partitions"** 을 선택합니다. 이 설정은 보너스 파트를 완수하기 위해 필요합니다.

암호화 비밀번호를 설정합니다. 이 암호화를 통해 디스크에 물리적으로 접근하더라도 비밀번호 없이는 데이터를 읽을 수 없습니다.

![Debian3](img/debian3.gif)

가이드 파티셔닝에서 볼륨 그룹(Volume Group) 크기를 조정하여 추가 논리 볼륨(Logical Volume)을 위한 공간을 확보합니다.

**"Configure the Logical Volume Manager"** 로 이동하여 두 개의 논리 볼륨을 생성합니다: `srv`와 `var-log`.

두 볼륨 모두 **"Ext4 journaling file system"** 으로 설정하고, 각각 `/srv`와 `/var/log`에 마운트합니다.

![Debian4](img/debian4.gif)

추가 설치 미디어 스캔은 건너뜁니다. 기본 Debian 아카이브 미러를 선택하고, 프록시 설정은 비워 둡니다. 패키지 사용 설문조사 참여는 거부합니다. **GRUB 로더는 반드시 설치합니다.**

소프트웨어 선택 화면에서는 **"SSH server"** 와 **"standard system utilities"** 만 선택합니다. 스페이스바로 체크/해제합니다. 설치가 완료되면 가상 머신을 재부팅합니다.

> **v5.2 참고:** X.org, Wayland 또는 기타 동등한 그래픽 서버의 설치는 금지되어 있습니다. 설치 시 0점 처리됩니다.

![Debian5](img/debian5.gif)

---

## SSH 설정

서버에 로그인할 수 있지만, 원격 접근이 필요하다면 SSH를 설정해야 합니다.

root로 로그인하고 SSH 서비스 상태를 확인합니다:

`systemctl status ssh`

SSH 서비스가 기본 포트 22에서 활성화되어 있어야 합니다. Born2beroot에서는 **포트 4242**에서 SSH를 실행해야 합니다. SSH 설정 파일 `/etc/ssh/sshd_config`를 편집합니다:

- `Port 22`를 `Port 4242`로 변경
- `PermitRootLogin`을 `no`로 설정

변경 후 해당 줄의 주석(`#`)을 제거하는 것을 잊지 마세요.

설정을 마쳤으면 SSH를 재시작합니다:

`systemctl restart ssh`

![ssh1](img/ssh1.gif)

다음으로 VirtualBox에서 **포트 포워딩**을 설정합니다:

가상 머신 설정 → Network → Advanced → Port Forwarding으로 이동합니다.

"SSH"라는 이름의 새 규칙을 생성합니다. 호스트의 사용 가능한 포트를 게스트 포트 4242로 리다이렉트하도록 설정합니다. (예: 호스트 포트 2222 → 게스트 포트 4242)

포트 포워딩이 설정되면 VirtualBox는 호스트 머신의 포트 2222로 들어오는 요청을 게스트 머신의 포트 4242로 전달합니다. 이를 통해 호스트에서 가상 머신으로 SSH 접속이 가능해집니다.

![ssh2](img/ssh2.gif)

설정이 완료되면 가상 머신에서 로그아웃합니다:

`logout`

호스트 터미널에서 다음 명령어를 입력합니다:

`ssh <your_login>@localhost -p 2222` — `localhost`의 포트 `2222`로 SSH 접속

![ssh3](img/ssh3.gif)

---

## 방화벽

SSH에 접속한 상태에서 root 사용자로 전환합니다: `su -`

서버 보안을 강화하기 위해 **UFW(Uncomplicated Firewall)** 를 설정합니다. UFW는 방화벽 규칙을 관리하기 위한 간편하고 효과적인 도구입니다.

`apt install ufw` — UFW를 설치합니다.

`ufw default deny incoming` — 모든 수신 요청을 차단합니다.

`ufw default allow outgoing` — 모든 송신 요청을 허용합니다.

`ufw allow 4242` — 포트 4242로의 수신 트래픽을 허용합니다. SSH 접속을 유지하기 위해 반드시 필요합니다.

`ufw enable` — UFW를 활성화합니다.

> **v5.2 참고:** 방화벽은 가상 머신 시작 시 반드시 활성 상태여야 합니다.

![ufw](img/ufw.gif)

---

## sudo와 그룹

### sudo

**sudo** 프로그램은 일반 사용자가 root(또는 다른 사용자)의 권한으로 명령어를 실행할 수 있게 해주는 핵심 도구입니다.

설치: `apt install sudo`

sudo 설정을 조정합니다. `visudo` 명령어로 설정 파일을 엽니다.

다음 줄들을 추가합니다:

`Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"` — sudo로 실행할 수 있는 명령어의 경로를 제한합니다. 이 설정은 이미 존재할 수 있으며, 그대로 두어도 됩니다.

`Defaults requiretty` — sudo 사용 시 TTY(터미널)를 요구합니다.

`Defaults badpass_message="WRONG PASSWORD"` — sudo에서 잘못된 비밀번호 입력 시 표시할 사용자 지정 메시지를 설정합니다.

`Defaults logfile="/var/log/sudo/sudo.log"` — sudo 입력 로그 파일의 경로를 지정합니다.

`Defaults log_input` — 입력을 로깅합니다.

`Defaults log_output` — 출력을 로깅합니다.

`Defaults iolog_dir=/var/log/sudo` — 추가 입출력 로그를 저장할 디렉터리를 설정합니다.

`Defaults passwd_tries=3` — sudo 비밀번호 시도 횟수를 3회로 제한합니다.

![sudo1](img/sudo1.gif)

### 그룹

Born2beroot 요구사항에 따라 사용자를 `user42`와 `sudo` 그룹에 추가해야 합니다.

`groupadd user42` — `user42` 그룹을 생성합니다.

`usermod -a -G user42,sudo <your_login>` — 사용자를 `user42`와 `sudo` 그룹에 추가합니다.

변경 사항을 확인하려면: `cat /etc/group`

이제 사용자가 sudo 그룹에 속해 있으므로, `sudo` 명령어를 통해 관리자 권한으로 작업을 수행할 수 있습니다.

![sudo2](img/sudo2.gif)

---

## 비밀번호 정책

`/etc/login.defs` 설정 파일에서 다음 항목을 수정합니다:

`PASS_MAX_DAYS 30` — 비밀번호가 30일마다 만료되도록 설정합니다.

`PASS_MIN_DAYS 2` — 비밀번호 변경 사이의 최소 일수를 2일로 설정합니다.

`PASS_WARN_AGE 7` — 비밀번호 만료 7일 전에 경고 메시지를 표시합니다. 기본값이 이미 7이면 그대로 둡니다.

위 변경사항을 기존 사용자에게도 적용합니다:

`chage -M 30 <your_login>` — `PASS_MAX_DAYS`를 30일로 설정합니다.

`chage -m 2 <your_login>` — `PASS_MIN_DAYS`를 2일로 설정합니다.

**사용자 계정과 root 계정 모두에 적용하세요.**

![password_policy](img/pwd_policy1.gif)

비밀번호 정책을 더 강화하기 위해 **pwquality** 모듈을 사용합니다:

`apt install libpam-pwquality`

`/etc/pam.d/common-password` 설정 파일을 열어 비밀번호 규칙을 설정합니다.

다음 줄을:

`password requisite pam_pwquality.so retry=3`

아래와 같이 변경합니다:

```
password        requisite                       pam_pwquality.so retry=3 minlen=10 difok=7 maxrepeat=3 dcredit=-1 ucredit=-1 lcredit=-1 reject_username enforce_for_root
```

각 옵션 설명:

| 옵션 | 설명 |
|------|------|
| `retry` | 비밀번호 입력 최대 재시도 횟수 |
| `minlen` | 새 비밀번호의 최소 길이 |
| `difok` | 이전 비밀번호에 없어야 하는 새 문자의 최소 개수. `enforce_for_root`가 있더라도 root는 이전 비밀번호를 요구하지 않으므로 이 규칙이 적용되지 않음 |
| `maxrepeat` | 동일한 문자의 최대 연속 허용 횟수 |
| `dcredit` | 최소 숫자 포함 개수 (음수 값) |
| `ucredit` | 최소 대문자 포함 개수 (음수 값) |
| `lcredit` | 최소 소문자 포함 개수 (음수 값) |
| `reject_username` | 비밀번호에 사용자 이름 포함 금지 |
| `enforce_for_root` | root에도 규칙 적용 |

새 규칙을 적용한 후, 사용자 계정과 root 계정의 비밀번호를 업데이트합니다:

`passwd <your_login>` — 사용자 비밀번호를 변경합니다.

![pwquality](img/pwquality.gif)

---

## 모니터링 스크립트

### 스크립트 작성

[monitoring.sh 스크립트](monitoring.sh)를 참고하되, 직접 작성해 보세요.

스크립트 실행에 필요한 패키지를 설치합니다:

`apt install bc sysstat`

스크립트가 표시해야 하는 정보는 다음과 같습니다:

| 항목 | 설명 |
|------|------|
| Architecture | OS 아키텍처 및 커널 버전 |
| Physical CPU | 물리 프로세서 수 |
| vCPU | 가상 프로세서 수 |
| Memory Usage | 현재 RAM 사용량 및 사용률(%) |
| Disk Usage | 현재 디스크 사용량 및 사용률(%) |
| CPU load | 현재 CPU 사용률(%) |
| Last boot | 마지막 재부팅 일시 |
| LVM use | LVM 활성화 여부 |
| TCP Connections | 활성 TCP 연결 수 |
| User log | 서버 사용 중인 사용자 수 |
| Network | 서버 IPv4 주소 및 MAC 주소 |
| Sudo | sudo로 실행된 명령 수 |

스크립트 출력 예시:

```
Broadcast message from root@wil (tty1) (Sun Apr 25 15:45:00 2021):
#Architecture: Linux wil 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64 GNU/Linux
#Physical CPU: 1
#vCPU: 1
#Memory Usage: 74/987MB (7.50%)
#Disk Usage: 1009/2Gb (49%)
#CPU load: 6.7%
#Last boot: 2021-04-25 14:45
#LVM use: yes
#TCP Connections: 1 ESTABLISHED
#User log: 1
#Network: IP 10.0.2.15 (08:00:27:51:9b:a5)
#Sudo: 42 cmd
```

> **참고:** 평가(peer review) 시 이 스크립트의 동작 원리를 설명해야 하며, 스크립트를 수정하지 않고 중단하는 방법도 알아야 합니다. `cron`을 살펴보세요.

### Cron Job

**cron** 프로그램은 설정된 일정에 따라 백그라운드에서 스크립트를 자동 실행합니다.

root 권한으로 crontab 파일을 편집합니다:

`crontab -e`

다음 줄을 추가합니다:

`*/10 * * * * bash /etc/cron.d/monitoring.sh | wall`

이 설정은 10분마다 `monitoring.sh` 스크립트를 실행하고, 출력을 `wall` 명령어로 모든 터미널에 전송합니다.

![cron](img/cron.gif)

---

## WordPress 설치

필수 파트를 마쳤으니 보너스 파트로 넘어갑니다.

### 패키지 설치

WordPress를 구동하려면 **웹 서버**, **데이터베이스 관리 시스템**, **PHP**가 필요합니다.

`apt install lighttpd` — 경량 웹 서버 lighttpd를 설치합니다.

`apt install mariadb-server` — MySQL의 포크(fork)인 MariaDB를 설치합니다.

`apt install php php-pdo php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath php-opcache php-json php-cgi` — PHP 본체 및 추가 패키지를 설치합니다.

![Wordpress1](img/wordpress1.gif)

### 방화벽 설정과 PHP 활성화

웹 서버에서 PHP를 활성화합니다:

`lighttpd-enable-mod fastcgi fastcgi-php`

웹 서버를 재시작하여 변경 사항을 적용합니다:

`systemctl restart lighttpd`

현재 방화벽은 포트 4242만 허용하고 있습니다. HTTP 요청을 위해 포트 80을 열어야 합니다:

`ufw allow http`

![Wordpress2](img/wordpress2.gif)

### 포트 포워딩

VirtualBox에서 다시 포트 포워딩을 설정합니다.

호스트의 사용 가능한 포트를 게스트 포트 80으로 리다이렉트하는 규칙을 추가합니다.

설정 후 웹 서버에 [127.0.0.1:호스트포트](http://127.0.0.1:1672)로 접속할 수 있습니다. (호스트 포트를 실제 설정한 포트로 변경하세요)

![Wordpress3](img/wordpress3.gif)

### 데이터베이스

MariaDB 프롬프트에 접속합니다: `mariadb`

다음 SQL 명령어들을 실행합니다:

`CREATE DATABASE <your_login>_db;` — 데이터베이스를 생성합니다.

`CREATE USER <your_login>@localhost;` — 사용자를 생성하고, `localhost`에서만 접속하도록 제한합니다.

`GRANT ALL PRIVILEGES ON <your_login>_db.* TO <your_login>@localhost;` — 해당 사용자에게 데이터베이스의 모든 테이블에 대한 전체 권한을 부여합니다.

`FLUSH PRIVILEGES;` — 권한 변경을 즉시 적용합니다.

![MariaDB](img/mariadb.gif)

### 통합 구성

마지막으로 WordPress를 설치합니다.

웹 서버 루트 `/var/www/html/`로 이동하고, 기존 파일을 삭제합니다: `rm *`

`wget https://wordpress.org/latest.tar.gz` — 최신 WordPress를 다운로드합니다.

`tar xvf *.tar.gz && rm *.tar.gz` — 압축을 해제하고 아카이브를 삭제합니다.

`mv wordpress/* . && rm -r wordpress` — WordPress 파일을 루트 디렉터리로 이동하고 빈 디렉터리를 삭제합니다.

`chown -R www-data:www-data /var/www/html/` — `/var/www/html/` 디렉터리의 소유권을 `www-data`로 변경합니다. 웹 서버가 정상 동작하려면 이 설정이 필요합니다.

![Wordpress4](img/wordpress4.gif)

브라우저에서 웹 서버 주소로 접속하면 WordPress 설치 화면이 표시됩니다. 언어를 선택하고, 데이터베이스 이름과 사용자 정보를 입력하면 설치가 완료됩니다.

![Wordpress5](img/wordpress5.gif)

---

## 추가 서비스

### 무엇을 선택할까?

서버에서 호스팅할 수 있는 서비스는 무궁무진합니다. 다음은 몇 가지 제안입니다:

| 분류 | 서비스 |
|------|--------|
| 생성형 AI | Stable Diffusion |
| 모니터링 | Prometheus |
| 데이터 시각화 | Grafana |
| 금융 | Ghostfolio |
| ERP/CRM | Odoo |
| BI(비즈니스 인텔리전스) | Metabase |
| 협업 | CryptPad |
| 블록체인 노드 | Bitcoin Core |
| 비밀번호 관리 | Vaultwarden |
| 동영상 다운로더 | MeTube |
| 미디어 서버 | Jellyfin |
| 게임 서버 | Minecraft |
| 파일 스토리지 | Nextcloud |
| 동영상 스트리밍 | Peertube |
| 전자책 관리 | Calibre |
| 음악 관리 | Lidarr |
| 사진 관리 | PhotoPrism |
| 번역 | LibreTranslate |
| 분석 | Matomo |
| 속도 테스트 | LibreSpeed |
| 컨테이너 | Docker |
| 호스팅 패널 | Cloudron |
| 자동화 | n8n |
| 토렌트 | qBittorrent |
| IDE | VS Code Server |
| 소셜 플랫폼 | Revolt |
| 채팅 | Rocket.Chat |
| 붙여넣기 | Hastebin |
| 다이어그램 | draw.io |
| VPN | WireGuard |
| 화상회의 | Jitsi |
| 노트 | Standard Notes |
| Git 서버 | GitLab |
| CMS | Ghost |
| 포럼 | Discourse |
| RSS | FreshRSS |
| 대시보드 | Organizr |

> **주의:** NGINX / Apache2는 선택할 수 없습니다!

### Netdata 설정

이 가이드에서는 Prometheus/Grafana와 유사한 오픈소스 모니터링 서비스인 **Netdata**를 설치합니다.

`apt install netdata`

Netdata는 포트 19999에서 실행되므로 방화벽에서 이 포트를 열어야 합니다:

`ufw allow 19999`

부팅 시 자동 시작이 이미 활성화되어 있는지 확인합니다:

`systemctl is-enabled netdata`

VirtualBox에서 호스트의 미사용 포트를 게스트 포트 19999로 포트 포워딩합니다.

필요하다면 `/etc/netdata/netdata.conf` 파일에서 IP 주소를 `0.0.0.0`으로 변경합니다.

설정이 완료되면 브라우저에서 `127.0.0.1:호스트포트`로 접속할 수 있습니다.

![Netdata](img/netdata.gif)

---

## 서명과 스냅샷

마지막으로, `.vdi` 파일의 해시를 생성해야 합니다. 이를 통해 프로젝트 제출과 이후 평가 사이에 가상 머신이 변경되지 않았음을 보장합니다.

게스트 머신을 종료하고 VirtualBox에서 **스냅샷**을 생성합니다. 스냅샷을 통해 시스템의 현재 상태를 언제든 복원할 수 있습니다.

> **v5.2 스냅샷 정책:**
> - 각 평가는 **스냅샷이 없는 상태**에서 시작해야 합니다.
> - 평가 시작 시 전용 스냅샷을 생성하고, 평가 종료 후 삭제합니다.
> - 제출 전에 스냅샷 기능을 테스트해 보는 것을 권장합니다.
> - 가상 머신을 다시 시작하면 서명이 변경될 수 있으므로, 가상 디스크 파일을 복제하거나 각 평가용 스냅샷을 활용하세요.

가상 머신이 저장된 디렉터리로 이동합니다.

`sha1sum debian.vdi` — `debian.vdi` 파일의 SHA-1 해시를 생성합니다.

가상 디스크 크기에 따라 시간이 오래 걸릴 수 있습니다.

> **v5.2 제출물:** `signature.txt`와 함께 **README.md**도 Git 저장소의 루트에 반드시 제출해야 합니다. 가상 머신 자체를 Git 저장소에 포함하는 것은 **절대 금지**됩니다.

> **v5.2 평가 중 수정 요청:** 평가 과정에서 프로젝트의 간단한 수정이 요청될 수 있습니다. 함수나 스크립트의 소규모 업데이트, 표시 변경, 데이터 구조 조정 등이 해당하며, 수 분 내에 완료할 수 있어야 합니다. 이는 프로젝트에 대한 실질적 이해도를 검증하기 위한 것입니다.

![Signature](img/signature.gif)
