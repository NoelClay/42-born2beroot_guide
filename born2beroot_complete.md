# Born2beroot 완전 정복 가이드
> "왜 하는가"부터 "어떻게 하는가"까지 — 검색 없이 이 문서 하나로 끝내기

---

## 목차

1. [배경 지식: 이 프로젝트가 뭔가?](#1-배경-지식)
2. [가상 머신 생성](#2-가상-머신-생성)
3. [운영체제 설치 — 부팅부터 파티셔닝까지](#3-운영체제-설치)
4. [SSH 설정](#4-ssh-설정)
5. [방화벽 (UFW)](#5-방화벽-ufw)
6. [sudo와 그룹 관리](#6-sudo와-그룹-관리)
7. [비밀번호 정책](#7-비밀번호-정책)
8. [모니터링 스크립트](#8-모니터링-스크립트)
9. [WordPress 설치 (보너스)](#9-wordpress-설치-보너스)
10. [추가 서비스 (보너스)](#10-추가-서비스-보너스)
11. [서명과 스냅샷 — 제출 전 마무리](#11-서명과-스냅샷)

---

## 1. 배경 지식

### 이 프로젝트가 도대체 뭔가?

Born2beroot는 **실제 서버를 운영하는 방법**을 배우는 프로젝트다.
현실에서 회사들은 서버를 통해 웹사이트, 데이터베이스, 파일 등을 운영한다.
이 프로젝트는 그 서버를 **가상 머신(VM)** 으로 만들어서 직접 설정해보는 것이다.

### 가상 머신(VM)이란?

컴퓨터 안에 컴퓨터를 만드는 것.
네 맥북 안에서 리눅스 컴퓨터가 돌아가는 것. 서로 완전히 독립적이다.
VM이 박살나도 맥북에는 아무 영향 없다. 그래서 실습용으로 딱이다.

### 하이퍼바이저란?

VM을 만들고 관리하는 소프트웨어.
- **Type 1 (베어메탈)**: 하드웨어 위에 직접 설치. 서버실에서 씀 (VMware ESXi, Proxmox)
- **Type 2 (호스트형)**: 기존 OS 위에서 돌아감. 개인 PC에서 씀 (VirtualBox, VMware Workstation)

이 프로젝트는 **VirtualBox** (Type 2) 사용.

### 왜 Debian인가?

Born2beroot는 Rocky Linux 또는 Debian 중 선택.
- **Debian**: 역사 오래됨, 안정적, Ubuntu의 기반, 커뮤니티 자료 많음 → **초보자에게 추천**
- **Rocky Linux**: CentOS 후속, 기업 서버 환경에 가까움 → 더 어려움

---

## 2. 가상 머신 생성

### 준비물

- VirtualBox 설치 (virtualbox.org에서 다운)
- Debian ISO 파일 (debian.org/distrib/netinst → **amd64** 선택)

### VirtualBox에서 VM 만들기

**New 클릭** 후 아래처럼 설정:

| 항목 | 값 | 이유 |
|------|-----|------|
| Name | 아무거나 (예: born2beroot) | 식별용 이름 |
| ISO Image | 다운받은 .iso 파일 선택 | 설치 미디어 |
| ✅ Skip Unattended Installation | 반드시 체크! | 안 체크하면 VirtualBox가 자동으로 설치해버림. 우리가 직접 해야 함 |
| RAM | 4096 MB (4GB) | 최소 요구사항. 적으면 설치 중 멈춤 |
| CPU | 2코어 | 성능 |
| Disk | 20GB | 파티션 나눌 공간 필요 |
| ✅ Pre-allocate Full Size | 체크 | 디스크 공간을 미리 확보. 성능 안정적 |

> **42 서울 학생**: 홈 디렉터리 용량 제한 있음. sgoinfre 또는 외장 드라이브에 저장할 것.

---

## 3. 운영체제 설치

### 화면 1: 부팅 메뉴

```
Install                    ← 이거 선택
Graphical install
Advanced options
```

**→ Install 선택 → 엔터**

왜 Graphical install 아님?: 텍스트 모드가 서버 환경에 적합함. 그래픽 없이 운영하는 게 서버의 기본.

---

### 화면 2~6: 기본 설정

| 화면 | 선택 | 이유 |
|------|------|------|
| 언어 | English | 오류 메시지 검색이 영어로 더 잘 됨 |
| 국가 | other → Asia → Korea, Republic of | 시간대 설정에 영향 |
| 로케일 | en_US.UTF-8 | 영어 환경, UTF-8은 한글 등 다국어 지원 |
| 키보드 | American English | 표준 키보드 배열 |

---

### 화면 7: 호스트명 (Hostname)

```
입력값: 본인42아이디42  (예: namykim42)
```

**왜**: 호스트명은 네트워크에서 이 컴퓨터를 식별하는 이름.
터미널에서 `namykim42:~$` 처럼 표시됨.
Born2beroot 규정: **42 아이디 + 42** 형식 필수.

---

### 화면 8: 도메인 이름

```
그냥 엔터 (비워두기)
```

**왜**: 도메인은 인터넷에서 접속할 때 쓰는 이름 (예: google.com).
우리 VM은 로컬 환경이라 필요 없음.

---

### 화면 9~10: root 비밀번호

```
기억할 수 있는 것으로 설정
확인 입력
```

**root가 뭔가**: 리눅스의 최고 관리자 계정. Windows의 Administrator와 같음.
모든 파일, 모든 명령어에 접근 가능. 이 비밀번호는 절대 잊으면 안 됨.

> 나중에 비밀번호 정책 설정 후 변경해야 하므로, 일단 기억 가능한 걸로 설정.

---

### 화면 11~14: 일반 사용자 계정 생성

| 화면 | 입력값 | 이유 |
|------|--------|------|
| 전체 이름 | 42 아이디 (예: namykim) | 표시 이름 |
| 계정 이름 | 42 아이디 그대로 | 로그인 ID |
| 비밀번호 | 설정 | 일반 계정 비밀번호 |

**왜 root 말고 별도 계정이 필요한가**: 
평소에 root로 작업하면 실수 한 번에 시스템 전체가 날아갈 수 있음.
일반 계정으로 작업하다가 필요할 때만 `su -`로 root 전환하는 게 안전한 운영 방식.

---

### 화면 15: 시간대

```
Seoul 선택
```

---

### 화면 16: 파티셔닝 방법 ⚠️ 핵심

```
Guided - use entire disk
Guided - use entire disk and set up LVM
Guided - use entire disk and set up encrypted LVM    ← 이거
Manual
```

**→ Guided - use entire disk and set up encrypted LVM 선택**

**파티션이 뭔가**: 
디스크를 여러 구역으로 나누는 것. 방 하나를 칸막이로 나누는 것과 같음.
각 파티션은 독립적으로 동작함. `/home`이 가득 차도 `/`(루트)에는 영향 없음.

**LVM(Logical Volume Manager)이 뭔가**: 
파티션을 유연하게 관리하는 시스템. 나중에 크기 조절 가능.
일반 파티션은 한번 만들면 크기 바꾸기 어려움.

**Encrypted(암호화)가 뭔가**:
디스크 전체를 암호화함. VM 파일을 통째로 복사해가도 비밀번호 없이는 데이터 못 읽음.
물리적 보안이 안 되는 환경(카페, 공유 컴퓨터)에서 필수적인 보안.

---

### 화면 17: 디스크 선택

```
SCSI3 (0,0,0) (sda) - 21.5 GB ATA VBOX HARDDISK    ← 이거
```

**→ sda 선택 → 엔터**

sda = 첫 번째 디스크. VirtualBox에서 만든 20GB 가상 디스크.

---

### 화면 18: 파티션 구성

```
All files in one partition
Separate /home partition
Separate /home, /var and /tmp partitions    ← 이거
```

**→ Separate /home, /var and /tmp partitions 선택**

**왜 분리하는가**:

| 파티션 | 역할 | 분리하는 이유 |
|--------|------|---------------|
| `/` (루트) | OS 핵심 파일 | 다른 파티션 문제가 OS에 영향 안 미치게 |
| `/home` | 사용자 파일 | 사용자 데이터가 OS 공간 잠식 방지 |
| `/var` | 로그, 캐시 | 로그 폭주해도 OS 안 죽음 |
| `/tmp` | 임시 파일 | 임시 파일이 시스템 공간 잡아먹기 방지 |
| `/srv` | 서비스 데이터 (보너스) | 웹서버 데이터 분리 |
| `/var/log` | 로그 전용 (보너스) | 로그만 따로 관리 |

---

### 화면 19: 디스크 초기화 확인

```
Write the changes to disks and configure LVM?
<Yes>    <No>
```

**→ Yes → 엔터**

기존 데이터 삭제하고 파티션 테이블 새로 씀. 어차피 새 VM이라 지울 것 없음.

---

### 화면 20~21: 암호화 비밀번호

```
암호화 비밀번호 입력
확인 입력
```

**세 비밀번호 정리**:

| 비밀번호 | 언제 사용 | 용도 |
|---------|---------|------|
| 암호화 비밀번호 | VM 켤 때마다 제일 먼저 | 디스크 암호 해제. 이게 없으면 OS 자체가 안 켜짐 |
| root 비밀번호 | `su -` 입력할 때 | 최고 관리자 접속 |
| 사용자 비밀번호 | 로그인할 때 | 일반 계정 접속 |

> **절대 잊으면 안 됨**: 암호화 비밀번호를 잊으면 VM 통째로 날아감. 복구 불가.

---

### 화면 22: 볼륨 그룹 크기 ⚠️ 핵심

```
Amount of volume group to use for guided partitioning:
[ 20.5 GB ]    ← 이걸 줄여야 함
```

**왜 줄이는가**:
Guided 파티셔닝이 공간을 다 써버리면 나중에 `srv`, `var-log` 추가할 공간이 없음.
지금 약간 남겨둬야 나중에 보너스 볼륨 만들 수 있음.

**→ 숫자 지우고 `18GB` 입력 → Continue → 엔터**

---

### 화면 23: 파티션 목록

이런 화면이 뜸:
```
Guided partitioning
Configure software RAID
Configure the Logical Volume Manager    ← 이거 선택
Configure encrypted volumes
...
LVM VG debian-vg, LV home
LVM VG debian-vg, LV root
...
Finish partitioning and write changes to disk
```

**→ Configure the Logical Volume Manager 선택 → 엔터**

---

### 화면 24~27: LVM에서 보너스 볼륨 추가

**화면 24: LVM 메뉴**
```
Display configuration details
Create volume group
Create logical volume    ← 이거
Delete logical volume
Extend volume group
Finish
```

**→ Create logical volume → 엔터**

**화면 25: 볼륨 그룹 선택**
```
debian-vg (또는 본인아이디-vg)    ← 이거
```
→ 엔터

**화면 26: 이름 입력**
```
srv
```
→ Continue → 엔터

**화면 27: 크기 입력**
```
1GB
```
→ Continue → 엔터

**다시 Create logical volume 선택해서 var-log도 만들기:**
- 이름: `var--log` **(하이픈 두 개! 중요)**
- 크기: `1GB`

> **왜 하이픈 두 개?**: LVM에서 하이픈(`-`)은 특수 구분자. 실제 이름에 하이픈을 넣으려면 두 개(`--`)로 써야 함.

**→ Finish → 엔터**

---

### 화면 23 다시: 파티션 목록에서 srv, var-log 설정

목록에 `srv`와 `var--log`가 새로 생겼을 것.

**srv 설정:**
1. `LVM VG ..., LV srv` 선택 → 엔터
2. `#1` 선택 → 엔터
3. `Use as: Ext4 journaling file system` 설정
4. `Mount point: /srv` 설정
5. `Done setting up the partition` → 엔터

**var-log 설정:**
1. `LVM VG ..., LV var--log` 선택 → 엔터
2. `#1` 선택 → 엔터
3. `Use as: Ext4 journaling file system` 설정
4. `Mount point: Enter manually` 선택 → `/var/log` 직접 입력
5. `Done setting up the partition` → 엔터

**Ext4가 뭔가**: 리눅스 표준 파일시스템. 안정적이고 빠름. 리눅스에서 기본으로 씀.

---

### 화면: 파티셔닝 완료

```
Finish partitioning and write changes to disk    ← 선택
```
→ 최종 확인 `Yes` → 엔터

---

### 화면: Scan extra installation media?

```
<Yes>    <No>    ← 이거
```
**→ No → 엔터**

추가 설치 미디어 없음. Yes 누르면 헛수고.

---

### 화면: Mirror country

```
Korea, Republic of    ← 선택
```
→ 엔터

**미러가 뭔가**: 패키지(프로그램) 다운로드 서버. 한국 서버가 제일 빠름.

---

### 화면: Mirror 서버

```
deb.debian.org    ← 선택
```
→ 엔터

공식 기본 서버. 안정적.

---

### 화면: HTTP proxy

```
빈칸 그대로 → 엔터
```

프록시 없음. 뭔가 입력하면 인터넷 연결 안 됨.

---

### 화면: Popularity contest

```
<Yes>    <No>    ← 이거
```
**→ No → 엔터**

사용 패키지 통계 수집 설문. 불필요.

---

### 화면: Software selection ⚠️ 절대 중요

**스페이스바로 체크/해제. 반드시 이 상태로:**

```
[ ] Debian desktop environment    ← 반드시 해제
[ ] ... GNOME                     ← 반드시 해제
[ ] ... Xfce                      ← 반드시 해제
[ ] ... GNOME Flashback           ← 반드시 해제
[ ] ... KDE Plasma                ← 반드시 해제
[ ] ... Cinnamon                  ← 반드시 해제
[ ] ... MATE                      ← 반드시 해제
[ ] ... LXDE                      ← 반드시 해제
[ ] ... LXQt                      ← 반드시 해제
[ ] web server                    ← 반드시 해제
[*] SSH server                    ← 반드시 체크
[*] standard system utilities     ← 반드시 체크
[ ] Choose a Debian Blend         ← 반드시 해제
```

**→ Continue → 엔터**

**왜 Desktop Environment 설치하면 안 되는가**:
Born2beroot v5.2 규정: X.org, Wayland 등 그래픽 서버 설치 금지. 설치 시 **0점**.
서버는 화면이 없어도 됨. SSH로 원격 접속해서 터미널로만 관리하는 게 실제 서버 운영 방식.

**SSH server를 왜 체크하는가**:
설치 직후부터 SSH 원격 접속이 가능하게 하기 위함.
나중에 맥 터미널에서 VM으로 SSH 접속해서 편하게 작업하기 위함.

---

### 화면: GRUB 설치

```
Install the GRUB boot loader to your primary drive?
<Yes>    <No>
```
**→ Yes → 엔터**

**GRUB이 뭔가**: 컴퓨터 켤 때 OS를 불러오는 부트로더.
GRUB이 없으면 Debian 자체가 안 켜짐. 반드시 설치.

---

### 화면: GRUB 설치 위치

```
Enter device manually
/dev/sda (ATA VBOX HARDDISK)    ← 이거
```

**→ /dev/sda → 엔터**

첫 번째 디스크. 여기에 설치해야 부팅됨.

---

### 화면: 설치 완료

```
Installation complete
<Continue>
```
**→ Continue → 엔터**

VM 재부팅됨. 설치 완료.

---

## 4. SSH 설정

### SSH가 뭔가?

SSH(Secure Shell)는 **원격으로 다른 컴퓨터에 접속하는 프로토콜**.
지금 만들고 있는 건 서버. 서버는 원격에서 터미널로 관리하는 게 기본.
VirtualBox 창에서 직접 타이핑하는 것보다 맥 터미널에서 SSH 접속해서 작업하는 게 훨씬 편함 (복사/붙여넣기 가능, 화면 크기 제한 없음).

### 지금 상태 확인

재부팅 후:
1. 암호화 비밀번호 입력
2. root로 로그인

```bash
systemctl status ssh
```

`active (running)` 이 보이면 SSH 서비스 실행 중. 포트 22에서 동작 중일 것.

---

### SSH 설정 파일 수정

```bash
vi /etc/ssh/sshd_config
```

**vi 사용법 (이것만 알면 됨):**

| 키 | 동작 |
|----|------|
| `i` | 편집 모드 시작 (글자 입력 가능해짐) |
| `ESC` | 편집 모드 종료 |
| `/검색어` + 엔터 | 해당 단어로 이동 |
| `x` | 커서 위치 글자 한 개 삭제 (편집 모드 아닐 때) |
| `Ctrl+H` | 편집 모드에서 backspace 역할 |
| `:wq` + 엔터 | 저장 후 나가기 |
| `:q!` + 엔터 | 저장 안 하고 강제 나가기 |

**수정할 내용:**

**① Port 변경**
`/Port 22` 입력 → 엔터로 해당 줄 이동

변경 전:
```
#Port 22
```
변경 후:
```
Port 4242
```
(`#` 삭제, `22`를 `4242`로 변경)

**왜 4242인가**:
포트 22는 전 세계 SSH 기본 포트. 해커들이 자동으로 22번 포트를 공격함 (무차별 대입).
4242로 바꾸면 자동 공격 대부분 회피 가능.
42 school이 4242를 지정한 건 보안 습관 교육 목적 + 학교 숫자 42 갖다붙인 것.

**왜 `#`을 제거하는가**:
`#`으로 시작하는 줄은 주석(comment). 주석은 컴퓨터가 무시함.
`#Port 22`는 "Port 22는 기본값이야" 라는 설명이지 실제 설정이 아님.
`Port 4242`로 써야 실제로 적용됨.

**② PermitRootLogin 변경**
`/PermitRootLogin` 입력 → 엔터

변경 전:
```
#PermitRootLogin prohibit-password
```
변경 후:
```
PermitRootLogin no
```

**왜 root SSH 로그인을 막는가**:
root는 모든 권한을 가진 계정. 해커가 root로 SSH 접속 성공하면 서버 완전 장악.
`no`로 설정하면 SSH로 root 직접 접속 불가.
반드시 일반 계정으로 먼저 로그인 후 `su -`로 root 전환해야 함.
이 과정이 보안 레이어를 하나 추가함.

**저장:**
```
ESC → :wq → 엔터
```

---

### SSH 재시작

```bash
systemctl restart ssh
```

**왜 재시작하는가**: 설정 파일을 수정했다고 자동으로 적용되지 않음.
서비스를 재시작해야 새 설정을 읽어들임.

확인:
```bash
systemctl status ssh
```
`4242` 보이면 성공.

---

### VirtualBox 포트 포워딩

**왜 포트 포워딩이 필요한가**:
VM은 네트워크 상 독립된 컴퓨터. 맥에서 VM으로 접속하려면 연결 다리가 필요.
포트 포워딩 = "맥의 A포트로 오는 요청을 VM의 B포트로 전달해줘"

VirtualBox에서:
VM 선택 → Settings → Network → Advanced → Port Forwarding

`+` 버튼으로 규칙 추가:

| Name | Protocol | Host IP | Host Port | Guest IP | Guest Port |
|------|----------|---------|-----------|----------|------------|
| SSH | TCP | (비워두기) | 4242 | (비워두기) | 4242 |

→ OK

---

### SSH 접속 테스트

맥 터미널 열고:
```bash
ssh 본인아이디@localhost -p 4242
```

접속되면 성공. 이제 VirtualBox 창 안 보고 맥 터미널에서 작업 가능.

---

## 5. 방화벽 (UFW)

### 방화벽이 뭔가?

서버로 들어오고 나가는 **네트워크 트래픽을 제어하는 문지기**.
"포트 4242로 오는 SSH 연결만 허용, 나머지는 전부 차단" 같은 규칙을 설정.

현재 SSH로 접속된 상태에서 root 전환:
```bash
su -
```

### UFW 설치 및 설정

```bash
apt install ufw
```
**`apt`가 뭔가**: Advanced Package Tool. 패키지(프로그램) 설치 도구. `apt install 프로그램명` = 프로그램 설치.

```bash
ufw default deny incoming
```
**의미**: 기본적으로 모든 수신 트래픽 차단. "일단 다 막고 필요한 것만 열자."

```bash
ufw default allow outgoing
```
**의미**: 기본적으로 모든 송신 트래픽 허용. 서버에서 인터넷으로 나가는 건 허용.

```bash
ufw allow 4242
```
**의미**: 포트 4242(SSH)로 오는 수신은 허용. 이게 없으면 방화벽 켜는 순간 SSH 연결 끊김.

```bash
ufw enable
```
**의미**: 방화벽 활성화. 이제 규칙이 실제로 적용됨.

확인:
```bash
ufw status
```
`4242 ALLOW` 보이면 성공.

> **Born2beroot v5.2**: 방화벽은 VM 시작 시 반드시 활성 상태여야 함.

---

## 6. sudo와 그룹 관리

### sudo가 뭔가?

일반 사용자가 **root 권한으로 명령어를 실행**할 수 있게 해주는 도구.

`sudo 명령어` = "이 명령어를 root 권한으로 실행해줘"

**왜 root로 항상 로그인 안 하고 sudo를 쓰는가**:
- 매번 root 비밀번호 입력 불필요
- 어떤 명령어가 sudo로 실행됐는지 로그 남길 수 있음
- `sudo` 권한을 특정 사용자에게만 부여 가능 → 세밀한 권한 관리

### sudo 설치

```bash
apt install sudo
```

### sudo 설정

```bash
visudo
```

**왜 일반 `vi`가 아니라 `visudo`인가**:
`/etc/sudoers` 파일은 문법 오류 있으면 sudo 자체가 망가짐.
`visudo`는 저장 전 문법 검사를 함. 오류 있으면 경고 줌.

파일 맨 아래에 아래 내용 추가:

```
Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```
**의미**: sudo로 실행할 수 있는 명령어 경로 제한. 악성 프로그램이 PATH 조작으로 다른 프로그램 실행 못 하게 막음. (이미 있으면 그대로 둠)

```
Defaults requiretty
```
**의미**: sudo 사용 시 반드시 실제 터미널(TTY) 필요. 스크립트나 원격 공격으로 sudo 실행 방지.

```
Defaults badpass_message="WRONG PASSWORD"
```
**의미**: sudo 비밀번호 틀렸을 때 표시할 메시지 커스텀. 평가 때 평가관이 볼 수 있음.

```
Defaults logfile="/var/log/sudo/sudo.log"
```
**의미**: sudo 명령어 실행 기록을 이 파일에 저장. 누가 언제 뭘 실행했는지 추적 가능.

```
Defaults log_input
```
**의미**: sudo로 실행된 명령어의 입력(키보드 타이핑)도 로그에 기록.

```
Defaults log_output
```
**의미**: sudo로 실행된 명령어의 출력 결과도 로그에 기록.

```
Defaults iolog_dir=/var/log/sudo
```
**의미**: 입출력 로그를 저장할 디렉터리.

```
Defaults passwd_tries=3
```
**의미**: sudo 비밀번호 최대 3번 틀릴 수 있음. 4번째부터 차단.

저장 후 로그 디렉터리 생성:
```bash
mkdir -p /var/log/sudo
```

---

### 그룹 설정

**그룹이 뭔가**: 사용자들을 묶어서 권한을 한꺼번에 관리하는 단위.
`sudo` 그룹에 속한 사용자 = sudo 명령어 사용 가능.

```bash
groupadd user42
```
**의미**: `user42`라는 그룹 생성. Born2beroot 요구사항.

```bash
usermod -a -G user42,sudo 본인아이디
```
**의미**:
- `usermod`: 사용자 계정 수정
- `-a`: 기존 그룹에서 빼지 않고 추가 (append)
- `-G`: 보조 그룹 지정
- `user42,sudo`: 두 그룹에 동시 추가

**`-a` 없이 `-G`만 쓰면**: 기존 그룹에서 모두 제거하고 새 그룹에만 추가됨. 위험.

확인:
```bash
cat /etc/group | grep 본인아이디
```
`user42`와 `sudo` 그룹에 이름 보이면 성공.

---

## 7. 비밀번호 정책

### 왜 비밀번호 정책이 필요한가?

기본 리눅스는 비밀번호 제한이 없음. `1234`도 됨.
Born2beroot는 기업 서버 보안 수준을 흉내내는 것.
실제 기업에서는 비밀번호 복잡도 강제, 주기적 변경 강제가 기본.

---

### 1단계: 만료 정책 설정

```bash
vi /etc/login.defs
```

아래 세 줄 찾아서 수정:

```
PASS_MAX_DAYS   30
```
**의미**: 비밀번호 최대 30일. 30일 지나면 강제 변경. 비밀번호가 유출돼도 피해 기간 제한.

```
PASS_MIN_DAYS   2
```
**의미**: 비밀번호 변경 후 최소 2일은 다시 변경 불가. "비밀번호 바꾸고 바로 원래대로 되돌리기" 방지.

```
PASS_WARN_AGE   7
```
**의미**: 만료 7일 전부터 경고 메시지 표시. 기본값이 이미 7이면 그대로 둬도 됨.

---

### 2단계: 기존 사용자에게 적용

`login.defs` 수정은 **새로 만드는 계정에만** 적용됨.
이미 만들어진 계정(root, 본인 계정)은 직접 명령어로 설정해야 함.

```bash
chage -M 30 본인아이디
chage -m 2 본인아이디
chage -M 30 root
chage -m 2 root
```

**`chage`가 뭔가**: change age. 계정 비밀번호 만료 정책 변경 도구.
- `-M 30`: PASS_MAX_DAYS = 30일
- `-m 2`: PASS_MIN_DAYS = 2일

---

### 3단계: 복잡도 정책 설정

```bash
apt install libpam-pwquality
```

**PAM이 뭔가**: Pluggable Authentication Modules. 리눅스 인증 시스템.
비밀번호 규칙, 로그인 제한 등을 모듈 형태로 추가할 수 있음.
`libpam-pwquality`는 비밀번호 품질(복잡도) 검사 모듈.

```bash
vi /etc/pam.d/common-password
```

아래 줄을 찾아:
```
password requisite pam_pwquality.so retry=3
```

이렇게 교체:
```
password        requisite                       pam_pwquality.so retry=3 minlen=10 difok=7 maxrepeat=3 dcredit=-1 ucredit=-1 lcredit=-1 reject_username enforce_for_root
```

**각 옵션 의미**:

| 옵션 | 값 | 의미 |
|------|-----|------|
| `retry=3` | 3 | 비밀번호 설정 시 최대 3번 재시도 허용 |
| `minlen=10` | 10 | 최소 10자 이상 |
| `difok=7` | 7 | 이전 비밀번호와 최소 7자 달라야 함 (root 예외) |
| `maxrepeat=3` | 3 | 같은 문자 최대 3번 연속 (예: `aaa`는 안 됨) |
| `dcredit=-1` | -1 | 숫자 최소 1개 이상 (음수 = 최소 개수) |
| `ucredit=-1` | -1 | 대문자 최소 1개 이상 |
| `lcredit=-1` | -1 | 소문자 최소 1개 이상 |
| `reject_username` | - | 비밀번호에 사용자 이름 포함 금지 |
| `enforce_for_root` | - | root에도 이 규칙 적용 |

**비밀번호 업데이트** (새 정책에 맞게 변경):
```bash
passwd 본인아이디
passwd root
```

---

## 8. 모니터링 스크립트

### 왜 만드는가?

서버 관리자는 서버 상태를 주기적으로 확인해야 함.
이 스크립트는 CPU, RAM, 디스크 등 서버 상태를 10분마다 자동으로 모든 터미널에 출력함.
평가 때 이 스크립트의 **동작 원리를 설명**해야 하므로 코드를 직접 이해해야 함.

---

### 패키지 설치

```bash
apt install bc sysstat
```

- `bc`: 수학 계산 (소수점 연산)
- `sysstat`: `mpstat` 등 CPU 통계 도구 포함

---

### 스크립트 작성

```bash
vi /etc/cron.d/monitoring.sh
```

아래 내용 입력:

```bash
#!/bin/bash

# OS 아키텍처와 커널 버전
ARCH=$(uname -a)

# 물리 CPU 수 (실제 하드웨어 프로세서 개수)
PCPU=$(grep "physical id" /proc/cpuinfo | sort -u | wc -l)

# 가상 CPU 수 (논리적 코어 수, 하이퍼스레딩 포함)
VCPU=$(nproc)

# RAM 정보 (MB 단위)
TOTAL_MEM=$(free -m | awk '/^Mem:/{print $2}')
USED_MEM=$(free -m | awk '/^Mem:/{print $3}')
MEM_PCT=$(free | awk '/^Mem:/{printf("%.2f"), $3/$2*100}')

# 디스크 사용량
TOTAL_DISK=$(df -h --total | grep total | awk '{print $2}')
USED_DISK=$(df -h --total | grep total | awk '{print $3}')
DISK_PCT=$(df --total | grep total | awk '{print $5}')

# CPU 사용률 (100 - idle = 사용 중인 %)
CPU_LOAD=$(mpstat | awk '$12 ~ /[0-9.]+/{printf "%.1f%%", 100-$12}')

# 마지막 재부팅 시간
LAST_BOOT=$(who -b | awk '{print $3, $4}')

# LVM 사용 여부
LVM=$(lsblk | grep -q "lvm" && echo "yes" || echo "no")

# 활성 TCP 연결 수
TCP=$(ss -ta | grep ESTAB | wc -l)

# 현재 로그인한 사용자 수
USERS=$(users | wc -w)

# IP 주소와 MAC 주소
IP=$(hostname -I | awk '{print $1}')
MAC=$(ip link | awk '/ether/{print $2}')

# sudo 명령어 실행 횟수
SUDO=$(journalctl _COMM=sudo | grep COMMAND | wc -l)

# wall로 모든 터미널에 전송
wall "
#Architecture: $ARCH
#Physical CPU: $PCPU
#vCPU: $VCPU
#Memory Usage: $USED_MEM/${TOTAL_MEM}MB ($MEM_PCT%)
#Disk Usage: $USED_DISK/$TOTAL_DISK ($DISK_PCT)
#CPU load: $CPU_LOAD
#Last boot: $LAST_BOOT
#LVM use: $LVM
#TCP Connections: $TCP ESTABLISHED
#User log: $USERS
#Network: IP $IP ($MAC)
#Sudo: $SUDO cmd"
```

실행 권한 부여:
```bash
chmod +x /etc/cron.d/monitoring.sh
```

테스트:
```bash
bash /etc/cron.d/monitoring.sh
```
정보가 출력되면 성공.

---

### Cron Job 설정

**cron이 뭔가**: 정해진 시간에 자동으로 명령어/스크립트를 실행하는 스케줄러.
"매일 새벽 3시에 백업", "매 10분마다 상태 확인" 같은 것 가능.

```bash
crontab -e
```

맨 아래에 추가:
```
*/10 * * * * bash /etc/cron.d/monitoring.sh | wall
```

**cron 문법 설명**:
```
*/10  *  *  *  *   명령어
  │   │  │  │  │
  │   │  │  │  └─ 요일 (0=일요일)
  │   │  │  └──── 월
  │   │  └─────── 일
  │   └────────── 시간
  └────────────── 분 (*/10 = 10분마다)
```

`*/10` = 0, 10, 20, 30, 40, 50분에 실행 = 매 10분마다.

`wall`이 뭔가: write all. 현재 로그인한 모든 사용자 터미널에 메시지 전송.

---

## 9. WordPress 설치 (보너스)

### 무엇이 필요한가?

WordPress를 돌리려면 세 가지가 필요:

| 역할 | 소프트웨어 | 설명 |
|------|-----------|------|
| 웹 서버 | lighttpd | HTTP 요청 받아서 웹페이지 전달 |
| 데이터베이스 | MariaDB | 글, 댓글 등 데이터 저장 |
| PHP | PHP + 모듈들 | WordPress 코드 실행 (WordPress는 PHP로 만들어짐) |

---

### 패키지 설치

```bash
apt install lighttpd
```
**왜 lighttpd?**: Apache, NGINX는 Born2beroot 보너스에서 사용 불가. lighttpd는 경량 웹서버로 허용됨.

```bash
apt install mariadb-server
```
**MariaDB가 뭔가**: MySQL에서 파생된 오픈소스 데이터베이스. 사실상 MySQL과 동일하게 사용.

```bash
apt install php php-pdo php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath php-opcache php-json php-cgi
```
WordPress가 필요로 하는 PHP 본체와 확장 모듈들.

---

### PHP 활성화 및 방화벽 설정

```bash
lighttpd-enable-mod fastcgi fastcgi-php
systemctl restart lighttpd
```
**왜**: lighttpd 기본 상태에서는 PHP 파일을 실행 못 함. FastCGI 모듈을 활성화해야 PHP가 동작.

```bash
ufw allow http
```
**왜**: 현재 방화벽은 4242만 열려있음. 웹(HTTP)은 80번 포트 사용. 이걸 열어야 브라우저에서 접속 가능.

---

### VirtualBox 포트 포워딩 추가

VM Settings → Network → Advanced → Port Forwarding

| Name | Protocol | Host Port | Guest Port |
|------|----------|-----------|------------|
| HTTP | TCP | 8080 | 80 |

---

### 데이터베이스 설정

```bash
mariadb
```

MariaDB 프롬프트에서:

```sql
CREATE DATABASE 본인아이디_db;
```
WordPress가 쓸 데이터베이스 생성.

```sql
CREATE USER 본인아이디@localhost;
```
WordPress가 DB에 접속할 사용자 계정 생성. `localhost`만 접속 허용 = 외부에서 직접 DB 접속 불가 (보안).

```sql
GRANT ALL PRIVILEGES ON 본인아이디_db.* TO 본인아이디@localhost;
```
생성한 사용자에게 해당 DB의 모든 권한 부여.

```sql
FLUSH PRIVILEGES;
```
권한 변경 즉시 적용.

```sql
EXIT;
```

---

### WordPress 설치

```bash
cd /var/www/html/
rm -rf *
wget https://wordpress.org/latest.tar.gz
tar xvf latest.tar.gz
rm latest.tar.gz
mv wordpress/* .
rm -r wordpress
chown -R www-data:www-data /var/www/html/
```

**각 명령어 의미**:
- `cd /var/www/html/`: 웹서버 루트 디렉터리로 이동 (웹서버가 파일 서빙하는 위치)
- `rm -rf *`: 기존 기본 파일 삭제
- `wget`: URL에서 파일 다운로드
- `tar xvf`: 압축 해제
- `mv wordpress/* .`: wordpress 폴더 안의 파일을 현재 위치로 이동
- `chown -R www-data:www-data`: 웹서버 프로세스(`www-data`)가 파일 읽고 쓸 수 있게 소유권 변경

브라우저에서 `http://127.0.0.1:8080` 접속 → WordPress 설치 화면 완료.

---

## 10. 추가 서비스 (보너스)

### 선택 가능한 서비스 (NGINX/Apache2 제외 모두 가능)

| 분류 | 서비스 |
|------|--------|
| 모니터링 | Netdata, Prometheus |
| 데이터 시각화 | Grafana |
| 파일 스토리지 | Nextcloud |
| Git 서버 | GitLab |
| 비밀번호 관리 | Vaultwarden |
| VPN | WireGuard |
| 미디어 서버 | Jellyfin |
| CMS | Ghost |

---

### Netdata 설정 (추천)

**Netdata가 뭔가**: 실시간 서버 모니터링 도구. CPU, RAM, 디스크, 네트워크를 브라우저에서 그래프로 볼 수 있음. 설치가 가장 쉬움.

```bash
apt install netdata
ufw allow 19999
```
포트 19999에서 웹 UI 제공. 방화벽에서 열어줘야 접속 가능.

```bash
systemctl is-enabled netdata
```
부팅 시 자동 시작 확인. `enabled` 보이면 OK.

VirtualBox 포트 포워딩 추가:

| Name | Protocol | Host Port | Guest Port |
|------|----------|-----------|------------|
| Netdata | TCP | 19999 | 19999 |

필요 시 설정 파일에서 IP 수정:
```bash
vi /etc/netdata/netdata.conf
```
`bind to = 127.0.0.1` 줄을 `bind to = 0.0.0.0` 으로 변경.

브라우저에서 `http://127.0.0.1:19999` 접속.

---

## 11. 서명과 스냅샷

### 왜 서명이 필요한가?

제출 시 VM 파일(`.vdi`)의 SHA1 해시값을 `signature.txt`에 저장해서 제출.
평가 때 평가관이 해시를 다시 계산해서 비교함.
해시가 다르면 제출 후 VM을 수정한 것 = 규정 위반.

**SHA1 해시가 뭔가**: 파일 내용을 기반으로 만든 고유한 지문. 파일이 1바이트라도 변경되면 완전히 다른 해시값이 나옴.

---

### 스냅샷 먼저 생성

VM 종료:
```bash
sudo poweroff
```

VirtualBox에서 VM 선택 → **Machine → Take Snapshot**

**스냅샷이 뭔가**: VM의 현재 상태를 저장하는 것. 나중에 문제가 생겨도 이 시점으로 돌아올 수 있음.

> **v5.2 스냅샷 정책**:
> - 제출 시 스냅샷 없는 상태여야 함
> - 평가 시작할 때 스냅샷 생성, 평가 끝나면 삭제
> - VM 재시작하면 서명 변경될 수 있으므로 주의

---

### SHA1 해시 생성

VM 파일이 있는 디렉터리로 이동 (맥 터미널에서):
```bash
cd ~/VirtualBox\ VMs/born2beroot/
sha1sum *.vdi
```

출력된 해시값을 복사해서:
```bash
echo "해시값" > signature.txt
```

---

### Git 제출

```
Git 저장소에 제출할 것:
✅ signature.txt
✅ README.md
❌ VM 파일 자체 (.vdi, .vbox 등) — 절대 금지
```

---

## 최종 체크리스트

제출 전 전부 확인:

- [ ] SSH 포트 4242에서 동작: `systemctl status ssh`
- [ ] root SSH 로그인 불가: `PermitRootLogin no` 확인
- [ ] UFW 활성화, 4242만 열림: `ufw status`
- [ ] sudo 설정 완료 (로그 포함): `cat /var/log/sudo/sudo.log`
- [ ] user42 그룹 존재, 본인 계정 포함: `cat /etc/group | grep user42`
- [ ] 비밀번호 정책 적용: `chage -l 본인아이디`
- [ ] 모니터링 스크립트 10분마다 실행: `crontab -l`
- [ ] WordPress 동작 (보너스): 브라우저에서 접속 확인
- [ ] 추가 서비스 동작 (보너스)
- [ ] signature.txt Git 저장소에 제출
- [ ] README.md Git 저장소에 제출
- [ ] VM 파일 Git에 없음 확인
