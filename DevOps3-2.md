# linux
## 1. /etc 디렉토리
**/etc 디렉토리란?** 리눅스 시스템 전체 설정 파일이 모여있는 디렉토리   
**왜 중요한가?** 서버에서 문제 나면 대부분 여기 건드림 (서버의 뇌)
### 1-1. /etc 디렉토리 핵심 파일
<details>
    <summary>사용자 & 계정 관련</summary>

사용자 만들고 권한 줄 때 쓰는 핵심  
**/etc/passwd** - 사용자 기본 정보  
**/etc/shadow** - 비밀번호(암호화)  
**/etc/group** - 그룹 정보  
**/etc/gshadow** - 그룹 비밀번호    
**/etc/ckel/** - 새 사용자 생성 시 기본 파일
</details>
<details>
    <summary>네트워크 관련</summary>

인터넷 연결 문제 해결   
**/etc/hosts** - 도메인 ↔ IP 직접 연결  
**/etc/resolv.conf** - DNS 서버 설정    
**/etc/hostname** - 컴퓨터 이름     
**/etc/network/interfaces** - 네트워크 설정     
**/etc/netplan/** - 최신 우분투 네트워크 설정   
**/ etc/services** - 포트 번호 정의
</details>
<details>
    <summary>시스템 & 부팅 관련</summary>

서버 시작/자동화/성능       
**/etc/fstab** - 디스크 자동 연결   
**/etc/rc.local** - 부팅 시 실행할 명령     
**/etc/sysctl.conf** - 커널 설정    
**/etc/crontab** - 자동 실행 작업   
**/etc/localtime** - 시간 설정
</details>
<details>
    <summary>보안 & 권한 관련</summary>

접근제어 + 보안 핵심    
**/etc/sudoers** - 관리자 권한 설정     
**/etc/login.defs** - 로그인 정책   
**/etc/securetty** - root 로그인 허용 터미널    
**/etc/hosts.allow** - 접근 허용    
**/etc/hosts.deny** - 접근 차단
</details>
<details>
    <summary>서비스 & 데몬 관련</summary>

서버 프로그램 설정  
**/etc/systemd/** - 서비스 관리     
**/etc/init.d/** - 구버전 서비스 스크립트   
**/etc/apache2/** - 웹 서버 설정    
**/etc/mysql/** - DB 설정   
**/etc/ssh/sshd_confing** - SHH 접속 설정
</details>
<details>
    <summary>기타</summary>

사용자 환경 설정    
**/etc/profile** - 로그인 시 환경 설정  
**/etc/bash.bashrc** - bash 환경 설정   
**/etc/environment** - 환경 변수 설정   
**/etc/opt/** - 추가 프로그램 설정
</details>

### 1-2. DNS 동작 흐름
**DNS란?** 도메인을 IP주소로 바꿔주는 시스템
<details>
    <summary>1단계: 브라우저 캐시 확인</summary>

- 브라우저 내부 캐시 확인
    - 접속 한 적 있음 → 바로 IP 사용 (종료)
    - 접속 한 적 없음 → 다음 단계
</details>
<details>
    <summary>2단계: OS 캐시 확인</summary>

- 운영체제 내부 DNS 캐시 확인
    - 있으면 → 사용
    - 없으면 → 다음 단계
- DNS 조회 결과를 메모리에 저장 (TTL 지나면 삭제)
</details>
<details>
    <summary>3단계: /etc/hosts 확인</summary>

- 수동으로 등록된 도메인 확인
    - 있으면 → IP 사용
    - 없으면 → 다음 단계
- DNS보다 우선순위 높음
</details>
<details>
    <summary>4단계: DNS Resolver (재귀 DNS 서버)</summary>

- 사용자 대신 IP를 찾아주는 서버
    - ISP DNS(KT, SK, LG)
    - 공용 DNS
</details>
<details>
    <summary>5-1단계: Root DNS 서버</summary>

- DNS 계층 구조 최상위 서버
    - TLD 서버 위치 알려줌
- 전세계 13개 논리 서버
- 가장 먼저 접촉
</details>
<details>
    <summary>5-2단계: TLD DNS서버</summary>

- 해당 도메인의 네임서버 위치 반환
- Top Level Domain
</details>
<details>
    <summary>5-3단계: Authoritative DNS 서버</summary>

- 도메인의 실제 IP 제공
- 최종 결과 제공
- 도메인 소유자가 관리
</details>

### 1-3. hosts
**hosts란?** 도메인과 IP를 직접 연결해주는 로컬 파일    
**위치?** Linux: /etc/hosts     
Windows: C:\windows\System32\drivers\etc\hosts
<details>
    <summary>특징</summary>

- DNS보다 우선순위 높음
- 직접 수정해야 함
- 네트워크 없이도 작동
</details>
<details>
    <summary>장점</summary>

- 서버 안 거치고 바로 연결되어서 빠름
- 가짜 서버 연결 가능
- 특정 사이트 차단 가능
</details>
<details>
    <summary>단점</summary>

- 수동 관리
- 확장성 없음
- 대규모 환경이여서 유지보수 어려움
</details>

### 1-4. DNS(Domain Name System)
**DNS란?** 도메인을 IP로 변환해주는 분산 시스템
<details>
    <summary>특징</summary>

- 자동 처리
- 계층 구조
- 전세계 사용
</details>
<details>
    <summary>장점</summary>

- 사용자가 신경 안 써도 됨 (자동화)
- 인터넷 전체 사용 가능 (확장성 좋음)
- 중앙 관리 가능
</details>
<details>
    <summary>단점</summary>

- 여러 서버를 거쳐서 느릴 수 있음
- 서버가 죽으면 접속 불가
- 보안 문제 발생 가능성 있음
</details>

### 1-5. hosts와 DNS 차이점
| **구분** | **hosts** | **DNS** |
| --- | --- | --- |
| **설정 방식** | 수동 | 자동 |
| **위치** | 내 컴퓨터(로컬 파일) | 외부 서버 |
| **관리 주제** | 사용자 | DNS 서버 관리자 |
| **적용 범위** | 내 컴퓨터만 | 전세계 |
| **우선순위** | DNS보다 먼저 | hosts 다음 |
| **속도** | 매우 빠름(로컬) | 상대적으로 느림 |
| **유지보수** | 어려움(직접 수정) | 쉬움(중앙 관리) |
| **확장성** | 없음 | 매우 좋음 |
| **네트워크 필요** | 필요 없음 | 필요 있음 |
| **사용 목적** | 테스트, 차단 | 실제 서비스 |

## 2. 네트워크 설정 파일 분석
### 2-1. UID/GID
**UID란?** 사용자 고유 번호     
**GID란?** 그룹 고유 번호
<details>
    <summary>UID 특징</summary>

- 모든 사용자마다 고유한 UID 존재
- 파일, 프로세스, 권한 판단할 때 전부 UID 기준으로 판단
</details>
<details>
    <summary>GID 특징</summary>

- 한 사용자는 여러 그룹에 속할 수 있음
- 기본 그룹 + 추가 그룹
</details>

**UID 종류는?**
| **UID 범위** | **의미** |
| --- | --- |
| 0 | root (관리자) |
| 1~999 | 시스템 계정 (서비스용) |
| 1000 이상 | 일반 사용자 |

<details>
    <summary>UID/GID가 쓰이는 곳</summary>

- 파일 권한 관리

- 권한 판단 방식
- 프로세스 실행
</details>
<details>
    <summary>UID/GID가 중요한 이유</summary>

- 권한 관리
    - 파일 접근 제어
    - 보안 핵심

- Docker에서 핵심
    - 컨테이너에서 UID가 같으면 같은 사용자처럼 동작
- 서버 운영
    - 서비스 계정 분리 필요
</details>

### 2-2. UID와 GID 차이점
| **구분** | **UID (User ID)** | **GID (Group ID)** |
| --- | --- | --- |
| **의미** | 사용자 식별 번호 | 그룹 식별 번호 |
| **목적** | “누구” 접근하는지 판단 | “어떤 그룹 소속인지” 판단 |
| **대상** | 개별 사용자 | 사용자 집합 |
| **고유성** | 사용자마다 하나 | 그룹마다 하나 |
| **소속 관계** | 한 사용자=1 UID | 한 사용자=여러 GID 가능 |
| **사용 위치** | 파일 소유자 | 파일 그룹 |
| **권한 적용** | 사용자 권한 | 그룹 권한 |
| **확인 명령어** | id, /etc/passwd | id, /etc/group |
| **특이사항** | UID 0=root (최고 권한) | 그룹으로 권한 공유 가능 |

### 2-3. root 계정
**root 계정이란?** 리눅스에서 모든 권한을 가진 슈퍼유저
<details>
    <summary>특징</summary>

- 모든 파일 접근 가능
- 권한 무시
- 시스템 설정 변경 가능
- 모든 프로세스 제어 가능
- 소유권 변경 가능
- 시스템 파괴 가능
</details>
<details>
    <summary>로그인 방식</summary>

- 직접 로그인: login: root
- sudo 사용: sudo command / sudo apt install nginx
- root 전환: su -
</details>
<details>
    <summary>root 사용이 위험한 이유</summary>

- 실수 = 시스템 터짐
    - 파일 삭제 혹은 설정 잘못 건드림 → 서버 다운
- 해킹 당하면 끝
    - root 계정 탈취 = 시스템 전체 장악
- 로그 추적 어려움
    - 누가 했는지 구분 안됨
</details>

### 2-4. root과 일반 사용자 차이점
| **구분** | **root** | **일반 사용자** |
| --- | --- | --- |
| **UID** | 0 | 1000 이상 |
| **권한** | 무제한 | 제한됨 |
| **파일 접근** | 전부 가능 | 권한 필요 |
| **시스템 변경** | 가능 | 제한 |
| **위험성** | 매우 높음 | 낮음 |
