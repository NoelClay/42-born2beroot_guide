# Born2beroot 동료 평가 대비 가이드

> 이 문서는 Born2beroot 동료 평가(P2P Evaluation)를 통과하기 위한 개념 튜토리얼 및 실습 훈련지입니다.
> [Born2beroot 42 가이드 (한국어)](README_KR.md)와 함께 사용하세요.

---

## 목차

**이론편**
1. [가상 머신이란?](#1-가상-머신이란)
2. [Debian vs Rocky Linux](#2-debian-vs-rocky-linux)
3. [패키지 관리: aptitude vs apt](#3-패키지-관리-aptitude-vs-apt)
4. [AppArmor vs SELinux](#4-apparmor-vs-selinux)
5. [파티션과 LVM](#5-파티션과-lvm)
6. [sudo란?](#6-sudo란)
7. [UFW란?](#7-ufw란)
8. [SSH란?](#8-ssh란)
9. [cron이란?](#9-cron이란)
10. [강력한 비밀번호 정책의 의미](#10-강력한-비밀번호-정책의-의미)

**실습편**
11. [평가 시작 전 체크리스트](#11-평가-시작-전-체크리스트)
12. [서명 검증](#12-서명-검증)
13. [서비스 상태 확인](#13-서비스-상태-확인)
14. [사용자와 그룹 관리](#14-사용자와-그룹-관리)
15. [비밀번호 정책 검증](#15-비밀번호-정책-검증)
16. [호스트명 변경](#16-호스트명-변경)
17. [파티션 확인](#17-파티션-확인)
18. [sudo 설정 검증](#18-sudo-설정-검증)
19. [UFW 방화벽 조작](#19-ufw-방화벽-조작)
20. [SSH 접속 검증](#20-ssh-접속-검증)
21. [모니터링 스크립트 설명](#21-모니터링-스크립트-설명)
22. [cron 주기 변경](#22-cron-주기-변경)

---

# 이론편

평가자가 던지는 이론 질문에 핵심 개념을 정확히 답변할 수 있어야 합니다. 단순 암기가 아니라 **왜 그런 구조인지** 이해하는 것이 중요합니다.

---

## 1. 가상 머신이란?

### 정의

**가상 머신(Virtual Machine, VM)** 은 물리적 하드웨어 위에 소프트웨어적으로 구현된 컴퓨터입니다. 하이퍼바이저(Hypervisor)라는 소프트웨어 계층이 물리 자원(CPU, 메모리, 디스크, 네트워크)을 가상화하여, 하나의 물리 머신 위에서 여러 독립적인 운영체제를 동시에 실행할 수 있게 해줍니다.

### 동작 원리

```
┌─────────────────────────────────────────┐
│          VM 1        VM 2        VM 3   │  ← 게스트 OS
├─────────────────────────────────────────┤
│              하이퍼바이저                  │  ← 자원 분배/격리
├─────────────────────────────────────────┤
│           호스트 OS (Type 2의 경우)        │
├─────────────────────────────────────────┤
│              물리 하드웨어                 │  ← CPU, RAM, Disk
└─────────────────────────────────────────┘
```

- **Type 1 (Bare-metal):** 하드웨어 위에서 직접 실행. 성능이 우수하며 서버 환경에서 사용. (예: VMware ESXi, Proxmox)
- **Type 2 (Hosted):** 기존 OS 위에서 애플리케이션으로 실행. 개인 PC에서 사용. (예: VirtualBox, VMware Workstation)

### 가상 머신의 목적

| 용도 | 설명 |
|------|------|
| **격리(Isolation)** | 각 VM은 독립된 환경이므로, 한 VM의 장애가 다른 VM에 영향을 주지 않음 |
| **서버 통합** | 하나의 물리 서버에서 여러 서비스를 격리하여 운영 |
| **테스트 환경** | 운영체제나 소프트웨어를 안전하게 실험 가능 |
| **보안** | 악성코드 분석, 취약점 테스트를 격리된 환경에서 수행 |
| **이식성** | VM 이미지를 그대로 다른 호스트로 이동 가능 |

### 평가 시 예상 질문과 답변 요점

> **Q: 가상 머신은 어떻게 동작하나요?**

하이퍼바이저가 물리적 하드웨어 자원을 추상화하여 각 가상 머신에 독립적인 가상 CPU, 메모리, 디스크, 네트워크 인터페이스를 할당합니다. 각 VM은 자신만의 운영체제와 커널을 갖고 있으며, 서로 완전히 격리된 상태에서 동작합니다. VirtualBox는 Type 2 하이퍼바이저로, 호스트 OS 위에서 실행됩니다.

---

## 2. Debian vs Rocky Linux

### 비교표

| 항목 | Debian | Rocky Linux |
|------|--------|-------------|
| **계열** | Debian 계열 | RHEL(Red Hat Enterprise Linux) 계열 |
| **패키지 형식** | `.deb` | `.rpm` |
| **패키지 관리자** | apt / aptitude | dnf (구 yum) |
| **보안 모듈** | AppArmor | SELinux |
| **방화벽** | UFW | firewalld |
| **릴리스 주기** | 약 2년마다 안정 버전 출시 | RHEL 소스 기반 리빌드 |
| **커뮤니티** | 매우 크고 오래된 커뮤니티 | CentOS 후속, 기업 환경에 적합 |
| **난이도** | 초보자 친화적 | 상대적으로 복잡 |

### 왜 Debian을 선택했는가? (예시 답변)

Debian은 안정성이 검증된 배포판으로, 초보자에게 친화적인 설치 과정과 풍부한 커뮤니티 문서를 제공합니다. Ubuntu의 기반이 되는 배포판이기도 하여, 여기서 배운 지식이 이후 Ubuntu 환경에서도 직접 활용됩니다.

---

## 3. 패키지 관리: aptitude vs apt

### apt (Advanced Package Tool)

**apt**는 Debian 계열에서 패키지를 설치, 업데이트, 제거하는 커맨드라인 도구입니다.

```bash
apt install <패키지명>    # 패키지 설치
apt remove <패키지명>     # 패키지 제거
apt update               # 패키지 목록 갱신
apt upgrade              # 설치된 패키지 업그레이드
```

### aptitude

**aptitude**는 apt의 프론트엔드(front-end)로, 더 높은 수준의 패키지 관리 기능을 제공합니다.

| 차이점 | apt | aptitude |
|--------|-----|----------|
| 인터페이스 | CLI 전용 | CLI + ncurses TUI |
| 의존성 충돌 | 오류 표시 후 중단 | 대안 해결책을 제안 |
| 미사용 패키지 | 수동 정리 필요 | 자동 제거 제안 |
| 복잡도 | 단순 | 더 지능적인 의존성 해석 |

**핵심:** aptitude는 의존성 충돌 시 여러 해결 방안을 제시하며, apt는 단순히 충돌을 보고합니다.

---

## 4. AppArmor vs SELinux

### AppArmor (Debian 기본)

**AppArmor(Application Armor)** 는 리눅스 보안 모듈(LSM)로, **프로그램별 프로파일**을 기반으로 접근을 제어합니다.

- **경로 기반(Path-based):** 파일 경로를 기준으로 접근 권한을 설정
- **프로파일 모드:**
  - `enforce` — 정책을 강제 적용하고 위반 시 차단
  - `complain` — 위반을 로그에 기록하지만 차단하지 않음

```bash
sudo aa-status    # AppArmor 상태 확인
```

### SELinux (Rocky Linux 기본)

**SELinux(Security-Enhanced Linux)** 는 NSA가 개발한 보안 모듈로, **레이블 기반(Label-based)** 의 MAC(Mandatory Access Control)을 구현합니다.

- 모든 파일, 프로세스, 포트에 보안 레이블(Context)을 부여
- 정책이 허용하지 않는 모든 접근을 기본적으로 차단

### 비교

| 항목 | AppArmor | SELinux |
|------|----------|---------|
| 접근 제어 방식 | 경로 기반 | 레이블 기반 |
| 설정 난이도 | 상대적으로 쉬움 | 상대적으로 복잡 |
| 보안 수준 | 높음 | 매우 높음 |
| 기본 채택 | Debian, Ubuntu | RHEL, CentOS, Rocky |

---

## 5. 파티션과 LVM

### 파티션(Partition)이란?

**파티션**은 하나의 물리 디스크를 논리적으로 나눈 독립적인 영역입니다. 각 파티션은 독립된 파일 시스템을 가질 수 있으며, 운영체제는 각 파티션을 별도의 디스크처럼 취급합니다.

```
물리 디스크 (예: /dev/sda)
├── /dev/sda1  →  /boot     (부트 파티션)
├── /dev/sda2  →  [암호화된 LVM]
│   ├── LV root    →  /
│   ├── LV home    →  /home
│   ├── LV var     →  /var
│   ├── LV tmp     →  /tmp
│   ├── LV srv     →  /srv
│   ├── LV var-log →  /var/log
│   └── LV swap    →  [swap]
```

### LVM (Logical Volume Manager)

**LVM**은 물리 디스크 위에 추상화 계층을 추가하여, 파티션 크기를 유연하게 관리할 수 있게 해주는 기술입니다.

```
물리 볼륨(PV)  →  볼륨 그룹(VG)  →  논리 볼륨(LV)
Physical Volume   Volume Group      Logical Volume
```

| 계층 | 역할 |
|------|------|
| **PV (Physical Volume)** | 실제 디스크 또는 파티션을 LVM이 관리하는 단위로 초기화 |
| **VG (Volume Group)** | 여러 PV를 하나의 저장소 풀로 묶음 |
| **LV (Logical Volume)** | VG에서 필요한 만큼 잘라내어 사용하는 가상 파티션 |

### LVM의 장점

- **유연한 크기 조절:** 시스템 가동 중에도 LV 크기를 확장/축소 가능
- **스냅샷:** 특정 시점의 데이터 상태를 저장
- **여러 디스크 통합:** 여러 물리 디스크를 하나의 VG로 묶어 대용량 볼륨 구성 가능

### 암호화 (LUKS)

Born2beroot에서는 **암호화된 LVM**을 사용합니다. LUKS(Linux Unified Key Setup)로 파티션을 암호화하면, 디스크에 물리적으로 접근하더라도 비밀번호 없이는 데이터를 읽을 수 없습니다.

---

## 6. sudo란?

### 정의

**sudo (Superuser Do)** 는 일반 사용자가 root 또는 다른 사용자의 권한으로 명령어를 실행할 수 있게 해주는 프로그램입니다.

### 왜 root로 직접 로그인하지 않는가?

| 문제 | 설명 |
|------|------|
| **감사 추적 불가** | root로 직접 로그인하면 누가 어떤 작업을 했는지 추적이 어려움 |
| **실수의 파급력** | root 상태에서의 오타 하나가 시스템 전체를 파괴할 수 있음 |
| **최소 권한 원칙** | 필요한 순간에만, 필요한 명령에만 권한을 상승시키는 것이 보안의 기본 |

### Born2beroot의 sudo 설정

```
Defaults  secure_path="..."     # sudo로 실행 가능한 경로 제한
Defaults  requiretty            # 터미널(TTY) 필수
Defaults  badpass_message="..."  # 비밀번호 오류 시 커스텀 메시지
Defaults  logfile="/var/log/sudo/sudo.log"  # 로그 파일
Defaults  log_input             # 입력 로깅
Defaults  log_output            # 출력 로깅
Defaults  iolog_dir=/var/log/sudo  # I/O 로그 디렉터리
Defaults  passwd_tries=3        # 비밀번호 시도 3회 제한
```

---

## 7. UFW란?

### 정의

**UFW (Uncomplicated Firewall)** 는 iptables를 쉽게 사용할 수 있도록 만든 인터페이스입니다.

### 방화벽의 역할

방화벽은 네트워크 트래픽의 **문지기** 역할을 합니다. 사전에 정의된 규칙에 따라 들어오는(incoming) 트래픽과 나가는(outgoing) 트래픽을 허용하거나 차단합니다.

```
인터넷  ──→  [방화벽: 포트 4242만 허용]  ──→  서버
              ↓ 차단
         포트 80, 443, 22 등
```

### UFW vs firewalld

| 항목 | UFW | firewalld |
|------|-----|-----------|
| 사용 배포판 | Debian 계열 | RHEL/Rocky 계열 |
| 백엔드 | iptables / nftables | iptables / nftables |
| 설정 방식 | 단순한 CLI 명령어 | zone 기반 설정 |
| 난이도 | 매우 쉬움 | 상대적으로 복잡 |

---

## 8. SSH란?

### 정의

**SSH (Secure Shell)** 는 안전하지 않은 네트워크를 통해 원격 머신에 **암호화된 채널**로 접속하는 프로토콜입니다.

### 동작 원리 (간략)

1. 클라이언트가 서버에 접속 요청
2. 서버가 공개키를 전송하여 신원 확인
3. 대칭 키 교환으로 암호화된 세션 수립
4. 사용자 인증 (비밀번호 또는 키 기반)
5. 암호화된 터널을 통해 명령어 송수신

### 왜 SSH를 사용하는가?

- **암호화:** 모든 통신이 암호화되어 도청(sniffing) 방지
- **인증:** 비밀번호 또는 공개키 기반 인증으로 접근 제어
- **포트 포워딩:** 안전한 터널을 통해 다른 서비스에 접근 가능

### Born2beroot의 SSH 설정

- 포트: **4242** (기본 22 대신)
- root 접속: **차단** (`PermitRootLogin no`)

포트를 변경하는 이유: 기본 포트 22는 자동화된 공격(브루트포스)의 주요 대상이므로, 비표준 포트를 사용하면 불필요한 공격 시도를 줄일 수 있습니다.

---

## 9. cron이란?

### 정의

**cron**은 유닉스 계열 OS에서 **시간 기반으로 작업을 자동 실행**하는 데몬(daemon)입니다.

### crontab 문법

```
┌───────────── 분 (0-59)
│ ┌───────────── 시 (0-23)
│ │ ┌───────────── 일 (1-31)
│ │ │ ┌───────────── 월 (1-12)
│ │ │ │ ┌───────────── 요일 (0-7, 0과 7은 일요일)
│ │ │ │ │
* * * * *  명령어
```

### 예시

| 표현식 | 의미 |
|--------|------|
| `*/10 * * * *` | 매 10분마다 |
| `*/1 * * * *` | 매 1분마다 |
| `0 3 * * *` | 매일 오전 3시 |
| `0 0 * * 0` | 매주 일요일 자정 |

### 스크립트를 수정하지 않고 중단하는 방법

평가 시 이 질문이 나올 수 있습니다:

```bash
sudo crontab -u root -e    # crontab 편집 → 해당 줄 주석 처리 또는 삭제
# 또는
sudo systemctl stop cron   # cron 데몬 자체를 중지
```

핵심: **스크립트 파일 자체를 수정하지 않고** crontab에서 스케줄을 제거하거나 cron 서비스를 중지하면 됩니다.

---

## 10. 강력한 비밀번호 정책의 의미

### 왜 필요한가?

약한 비밀번호는 시스템 보안의 가장 흔한 취약점입니다. 브루트포스 공격, 사전 공격 등으로 단순한 비밀번호는 쉽게 탈취됩니다.

### Born2beroot의 비밀번호 정책

| 규칙 | 설정 | 목적 |
|------|------|------|
| 최소 길이 10자 | `minlen=10` | 브루트포스 공격 난이도 증가 |
| 대문자 포함 | `ucredit=-1` | 문자 집합 확대 |
| 소문자 포함 | `lcredit=-1` | 문자 집합 확대 |
| 숫자 포함 | `dcredit=-1` | 문자 집합 확대 |
| 연속 동일 문자 3자 제한 | `maxrepeat=3` | 패턴 기반 공격 방지 |
| 사용자명 포함 금지 | `reject_username` | 추측 공격 방지 |
| 이전 비밀번호와 7자 이상 차이 | `difok=7` | 유사 비밀번호 재사용 방지 |
| 30일마다 만료 | `PASS_MAX_DAYS 30` | 장기 사용으로 인한 유출 위험 감소 |
| 변경 간 최소 2일 | `PASS_MIN_DAYS 2` | 정책 우회를 위한 빠른 변경 방지 |

### 장단점

**장점:** 보안 수준 대폭 향상, 자동화 공격에 대한 저항력 증가

**단점:** 사용자가 비밀번호를 기억하기 어려워져 별도 기록 가능성, 잦은 변경으로 인한 불편

---

# 실습편

아래는 실제 평가에서 시연해야 하는 항목들입니다. **각 명령어를 직접 실행해 보고 출력 결과를 이해**하세요.

---

## 11. 평가 시작 전 체크리스트

평가가 시작되기 전 반드시 확인할 사항:

- [ ] Git 저장소에 `signature.txt`와 `README.md`만 존재하는지 확인
- [ ] VM에 **스냅샷이 없는 상태**인지 확인 (v5.2 요구사항)
- [ ] `signature.txt`의 서명이 현재 VM의 `.vdi` 파일 서명과 일치하는지 확인
- [ ] VM 시작 시 **그래픽 환경 없이** 부팅되는지 확인
- [ ] 부팅 시 **암호화 비밀번호**가 요구되는지 확인

---

## 12. 서명 검증

평가자가 가장 먼저 확인하는 항목입니다.

**호스트 터미널에서:**

```bash
# VM이 저장된 디렉터리로 이동
cd ~/VirtualBox\ VMs/<VM이름>/

# .vdi 파일의 SHA-1 해시 생성
sha1sum <파일명>.vdi
# macOS의 경우: shasum <파일명>.vdi
# UTM의 경우: shasum <VM이름>.utm/Images/disk-0.qcow2
```

출력된 해시값이 `signature.txt`의 내용과 **정확히 일치**해야 합니다.

---

## 13. 서비스 상태 확인

### UFW 상태 확인

```bash
sudo ufw status
```

`Status: active`가 표시되어야 하며, 포트 4242 규칙이 보여야 합니다.

### SSH 상태 확인

```bash
sudo systemctl status ssh
```

`active (running)` 상태이고, 포트 4242에서 리스닝 중이어야 합니다.

### AppArmor 상태 확인 (Debian)

```bash
sudo aa-status
```

프로파일이 로드되어 있고 enforce 모드인 것을 확인합니다.

### 운영체제 확인

```bash
# 아키텍처 및 커널 정보
uname -a

# 배포판 정보
cat /etc/os-release
```

---

## 14. 사용자와 그룹 관리

### 현재 사용자의 그룹 확인

```bash
# 사용자가 sudo와 user42 그룹에 속해 있는지 확인
getent group sudo
getent group user42

# 또는 현재 사용자의 모든 그룹 확인
groups <your_login>
```

### 평가자를 위한 새 사용자 생성

```bash
# 새 사용자 생성 (비밀번호 정책이 적용되는지 확인)
sudo adduser <evaluator_username>

# 새 그룹 생성
sudo groupadd evaluating

# 사용자를 그룹에 추가
sudo usermod -aG evaluating <evaluator_username>

# 그룹 소속 확인
getent group evaluating
```

비밀번호 설정 시 정책(최소 10자, 대소문자+숫자, 연속 문자 제한 등)이 적용되는지 반드시 확인합니다.

---

## 15. 비밀번호 정책 검증

### 비밀번호 만료 규칙 확인

```bash
sudo chage -l <your_login>
```

다음 값들이 표시되어야 합니다:

```
Maximum number of days between password change : 30
Minimum number of days between password change : 2
Number of days of warning before password expires : 7
```

### 비밀번호 복잡도 정책 확인

```bash
# 설정 파일 확인
cat /etc/pam.d/common-password | grep pam_pwquality
```

`minlen=10 difok=7 maxrepeat=3 dcredit=-1 ucredit=-1 lcredit=-1 reject_username enforce_for_root`가 포함되어 있어야 합니다.

---

## 16. 호스트명 변경

### 현재 호스트명 확인

```bash
hostnamectl
```

`<your_login>42` 형식이어야 합니다.

### 호스트명 변경 (평가자 로그인으로)

```bash
# 호스트명 변경
sudo hostnamectl set-hostname <evaluator_login>42

# /etc/hosts 파일에서도 변경
sudo nano /etc/hosts
# 127.0.1.1 줄의 호스트명을 새 호스트명으로 변경

# 재부팅
sudo reboot
```

재부팅 후 `hostnamectl`로 변경이 적용되었는지 확인합니다.

### 원래 호스트명 복원

동일한 절차로 원래 호스트명(`<your_login>42`)으로 되돌립니다.

---

## 17. 파티션 확인

```bash
lsblk
```

서브젝트의 예시와 비교하여 암호화된 LVM 파티션 구조가 올바른지 확인합니다. 최소 2개 이상의 암호화된 파티션이 LVM 위에 존재해야 합니다.

보너스 파트를 수행한 경우 `srv`, `var--log` 등 추가 논리 볼륨이 보여야 합니다.

---

## 18. sudo 설정 검증

### sudo 설치 확인

```bash
dpkg -l | grep sudo
```

### sudo 그룹에 사용자 추가

```bash
sudo usermod -aG sudo <evaluator_username>
```

### sudo 로그 확인

```bash
# 로그 디렉터리 존재 확인
ls /var/log/sudo/

# 로그 파일 내용 확인
cat /var/log/sudo/sudo.log
```

### 로그 갱신 테스트

```bash
# sudo로 아무 명령 실행
sudo ls /root

# 로그에 기록되었는지 확인
cat /var/log/sudo/sudo.log
```

새로운 로그 항목이 추가되어 있어야 합니다.

---

## 19. UFW 방화벽 조작

### 현재 규칙 확인

```bash
sudo ufw status numbered
```

포트 4242에 대한 규칙이 존재해야 합니다.

### 포트 8080 규칙 추가

```bash
sudo ufw allow 8080
sudo ufw status numbered    # 추가 확인
```

### 추가한 규칙 삭제

```bash
sudo ufw delete <규칙번호>
# 규칙번호는 status numbered 출력에서 확인
# IPv4와 IPv6 두 개의 규칙이 추가되므로, 두 번 삭제해야 함

sudo ufw status numbered    # 삭제 확인
```

---

## 20. SSH 접속 검증

### SSH 설치 확인

```bash
dpkg -l | grep ssh
```

### 포트 4242로 접속 (호스트 터미널에서)

```bash
ssh <your_login>@127.0.0.1 -p 4242
# 포트 포워딩 설정에 따라 포트 번호가 다를 수 있음
# 예: ssh <your_login>@localhost -p 2222
```

### root로 SSH 접속 불가 확인

```bash
ssh root@127.0.0.1 -p 4242
# "Permission denied"가 표시되어야 함
```

---

## 21. 모니터링 스크립트 설명

평가자가 스크립트의 동작 원리를 물어볼 것입니다. 각 항목별로 사용되는 명령어와 의미를 이해하세요.

### 스크립트 확인

```bash
cat /etc/cron.d/monitoring.sh
# 또는 본인이 저장한 경로
```

### 항목별 핵심 명령어

| 출력 항목 | 핵심 명령어 | 설명 |
|-----------|------------|------|
| Architecture | `uname -a` | 커널 이름, 호스트명, 커널 버전, 아키텍처 등 시스템 전체 정보 |
| Physical CPU | `lscpu \| grep Socket` | 물리 CPU 소켓 수 |
| vCPU | `nproc` | 사용 가능한 프로세서(논리 코어) 수 |
| Memory Usage | `free -m` | 메모리 사용량 (MiB 단위) |
| Disk Usage | `df -h` | 디스크 사용량 |
| CPU load | `mpstat` | CPU 사용률 (`sysstat` 패키지) |
| Last boot | `who -b` | 마지막 부팅 시간 |
| LVM use | `/etc/fstab`에서 `/dev/mapper` 확인 | LVM 매핑 장치 존재 여부 |
| TCP Connections | `ss -t state established` | 수립된 TCP 연결 수 |
| User log | `w` | 현재 로그인한 사용자 수 |
| Network | `ip address` | IPv4 주소 및 MAC 주소 |
| Sudo | `/var/log/sudo/seq` | sudo 실행 횟수 (36진수 시퀀스) |

---

## 22. cron 주기 변경

평가 마지막에 모니터링 스크립트가 **매 1분**마다 실행되도록 변경해야 합니다.

```bash
# root의 crontab 편집
sudo crontab -u root -e
```

기존 줄:

```
*/10 * * * * bash /etc/cron.d/monitoring.sh | wall
```

다음으로 변경:

```
*/1 * * * * bash /etc/cron.d/monitoring.sh | wall
```

저장 후 약 1분 기다리면 메시지가 모든 터미널에 표시됩니다.

확인이 끝나면 원래대로 `*/10`으로 복원하거나, 서버를 재시작합니다.

---

> 이 가이드의 모든 명령어를 **직접 실행해 보고**, 출력 결과를 이해한 상태에서 평가에 임하세요. 평가자는 단순히 명령어를 아는지가 아니라, **왜 그렇게 동작하는지** 설명할 수 있는지를 봅니다.
