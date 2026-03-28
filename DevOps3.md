# Networking
## 1. IPv4와 IPv6
**IPv4와 IPv6의 공통점은?** 인터넷에서 장치를 식별하는 주소 체계    
(데이터 패킷 전달 / 라우팅 가능)

### 1-1. IPv4
**IPv4(Internet Protocol version 4)란?** 인터넷에서 사용하는 4번째 통신 규칙
<details>
  <summary>특징</summary>

  - 32비트 주소
    - 8비트씩 4개
    - 각 숫자 범위: 0~255 
  - 점(.)으로 구분된 10진수
</details>
<details>
  <summary>장점</summary>

  - 구조가 단순해서 이해가 쉽고 오래 사용됨
  - 대부분 시스템이 지원
  - 호환성 높음
</details>
<details>
  <summary>단점</summary>

  - 주소 개수 부족
    - 약 43억 개
    - 이미 거의 다 사용됨 -> NAT 같은 기술 등장 
  - NAT 필요
    - 구조 복잡 + 성능 저하 가능
  - 보안 약함
</details>

### 1-2. IPv6
**IPv6(Internet Protocol version 6)란?** 인터넷에서 사용하는 6번째 통신 규칙

**왜 IPv6이 중요할까?** IOT시대에 IPv4로는 감당이 불가하기도 하고 AWS, GCP 같은 클라우드에서 IPv6이 점점 기본화되고 있기 때문에
<details>
  <summary>특징</summary>

  - 주소 개수 부족
    - 약 43억 개
    - 이미 거의 다 사용됨 -> NAT 같은 기술 등장 
  - NAT 필요
    - 구조 복잡 + 성능 저하 가능
  - 보안 약함
</details>
<details>
    <summary>장점</summary>

- 주소가 거의 무한개
    - 기기마다 공인 IP
- NAT 불필요
    - 구조가 단순해짐
    - 직접 통신 가능
- 보안 강화
- 효율적인 라우팅
    - 헤더 구조 개선 -> 처리 속도 향상
</details>
<details>
    <summary>단점</summary>

- 복잡함
    - 16진수 + 길어서 사람이 보기 어려움
- 완전한 전환 안됨
    - 아직 IPv4랑 같이 사용 중 -> 관리 복잡
- 일부 장비는 IPv6를 지원하지 않음
    - 오래된 장비는 IPv6 미지원
</details>

### 1-3. IPv4와 IPv6 차이점
| 구분 | IPv4 | IPv6 |
| ----- | ----- |-----|
|주소 길이|32비트|128비트|
|주소 개수|약 43억|거의 무한|
|표기 방식|10진수+점(.)|16진수+클론(:)|
|NAT 필요성|필요(주소 부족)|필요 없음|
|속도|보통|이론적으로 더 효율적|
|보안|옵션|기본적으로 강화됨|

## 2. ARP/RARP
**ARP/RARP이란?** IP주소 ↔ MAC 주소 변환 프로토콜   
(OSI 2계층 + 3계층(IP) 사이에서 작동 / 브로드캐스트 기반 요청 사용)

### 2-1. ARP(Address Resolution Protocol)
IP -> MAC 주소 찾기
<details>
    <summary>특징</summary>

- 브로드캐스트 기반 동작
    - 거의 모든 네트워크에서 사용
    - 같은 LAN에 있는 모든 장비가 요청을 받음
- ARP Cache에 저장해서 재사용
    - 한 번 찾은 MAC 주소는 메모리에 저장
    - 빠른 통신과 일정 시간이 지나면 자동 삭제됨 (속도 향상 및 TTL 존재)
</details>
<details>
    <summary>장점</summary>

- 빠르고 효율적인 주소 변환
    - 캐싱 덕분에 대부분 즉시 MAC 조회 가능
- 구조 단순
    - 구현 쉽고 거의 모든 OS/장비 지원
- 네트워크 필수 프로토콜
    - 실제 데이터는 MAC 주소 기반으로 이동함
</details>
<details>
    <summary>단점</summary>

- 보안 취약(ARP 스푸핑 공격 가능)
- 브로드캐스트 -> 트래픽 증가
    - 장비 많으면 네트워크 부담 증가
- 같은 네트워크에서만 사용 가능
- 수동 인증 없음
    - 누구나 거짓 응답 가능 -> 보안 취약
</details>

### 2-2. RARP(Reverse ARP)
MAC -> IP 주소 찾기
<details>
    <summary>특징</summary>

- 서버 기반 구조
    - RARP 서버가 필요
- 현재는 거의 사용 안함
    - 기능 제한
    - 확장성 부족
    - DHCP 등장
</details>
<details>
    <summary>장점</summary>

- IP 없는 장치도 네트워크 시작 가능
- 자동 IP 할당 가능 (초기 형태)
    - 부팅 시 IP 자동으로 받아옴
- 단순 구조
    - 구현 쉬움
</details>
<details>
    <summary>단점</summary>

- 반드시 서버가 필요
- 확장성 부족
    - 네트워크가 커지면 관리 어려움
- 기능 제한
    - 게이트웨이
    - DNS
    - 서브넷
</details>

### 2-3. ARP랑 RARP 차이점

|항목|ARP|RARP|
|----|----|----|
|역할|IP -> MAC|MAC -> IP|
필요성|필수|거의 폐기
동작 방식|브로드캐스트 + 응답|브로드캐스트 + 서버 응답
캐싱|있음(속도 향상)|없음
보안|취약(스푸핑)|상대적으로 덜 중요
현재 사용|계속 사용|DHCP로 대체

### 2-4. 캐싱(Cache)
**캐싱이란?** 자주 사용하는 데이터를 임시로 저장해두고 빠르게 다시 쓰는 것

( 한 번 가져온 걸 저장해두고 재사용 → 속도 빨라짐 / 서버 부담 줄어듦)

## 3. Handshake
**Handshake란?** 네트워크에서 통신을 시작하기 전에 서로 상태를 확인하고 연결을 설정하는 과정

**왜 필요한가?** 확인 → 연결 → 통신 이라는 명확한 절차가 필요하기 때문에
<details>
    <summary>특징</summary>

- 연결 지향 통신에서 사용
    - TCP 같은 프로토콜
- 신뢰성 보장
    - 서로 상태 확인 후 시작
- 초기 설정 과정
    - 통신 전에 반드시 거침
</details>
<details>
    <summary>장점</summary>

- 안정적인 통신
    - 준비 안 된 상태에서는 데이터를 안 보냄
- 오류 줄임
    - 연결 상태 확인
</details>
<details>
    <summary>단점</summary>

- 시간 소모
    - 바로 통신 못 하고 단계 거침
- 공격 당하기 가능
</details>

### 3-1. 3-Way Handshake
**3-Way Handshake란?** TCP에서 클라이언트와 서버가 연결을 설정하는 과정     
**왜 필요한가?** 신뢰성 있고 순서를 보장해줌 + 오류 체크
<details>
    <summary>전체 흐름</summary>

**1단계: SYN** - 클라이언트 → 서버 (연결 요청)

**2단계: SYN + ACK** - 서버 → 클라이언트 (서버가 요청 수락)

**3단계: ACK** - 클라이언트 → 서버 (클라이언트가 최종 확인)
</details>
<details>
    <summary>장점</summary>

- 신뢰성 보장
    - 서로 준비 상태 확인
- 데이터 순서 보장
    - Sequence Number 사용
- 오류 방지
    - 패킷 누락 확인 가능
</details>

**SYN flood란?** TCP 3-Way Handshake를 악용해서 서버를 마비시키는 공격

### 3-2. 4-Way Handshake
**4-Way Handshake란?** TCP에서 데이터를 다 주고받은 후 연결을 종료하는 4단계 과정   
**왜 4단계나 필요한가?** TCP가 양방향 통신이여서 2번이 아니라 4번이 필요함
<details>
    <summary>전체 흐름</summary>

1단계: FIN - 클라이언트 → 서버  
2단계: ACK - 서버 → 클라이언트  
3단계: FIN - 서버 → 클라이언트  
4단계: ACK - 클라이언트  → 서버
</details>
<details>
    <summary>장점</summary>

- 안정적인 종료
    - 데이터 손실 없이 종료
- 양방향 종료 보장
    - 한쪽만 끊는 문제 방지
- 남은 데이터 처리 가능
    - 서버가 남은 데이터를 다 보내고 종료
</details>
<details>
    <summary>단점</summary>

- 과정이 긺
    - 4번 통신 필요 → 느림
- 공격 가능
    - FIN 기반 공격 가능 (DoS)
- TIME_WAIT로 자원 사용
    - 연결 많으면 포트 부족 문제 발생
</details>

### 3-3. 3-Way와 4-Way 차이점

구분|3-Way Handshake|4-Way Handshake
|----|----|----
목적|연결 생성|연결 종료
단계|3단계|4단계
특징|빠르게 연결|안전하게 종료
핵심|SYN|FIN

## 4. 캐스트(Cast)
**캐스트란?** 네트워크에서 데이터를 누구에서 보낼지 정하는 방식

### 4-1. 브로드캐스트(Broadcast)
**브로드캐스트란?** 네트워크 전체에게 보내는 방식
<details>
    <summary>특징</summary>

- 같은 네트워크(LAN) 전체에 전송
- 모든 장비가 메시지 받음
</details>
<details>
    <summary>장점</summary>

- 대상 몰라도 됨
- 초기 통신에 유리 (IP찾기 등)
</details>
<details>
    <summary>단점</summary>

- 네트워크 낭비
    - 필요 없는 장비도 다 받음
- 트래픽 증가
    - 장비 많으면 부담 커짐
- 라우터 못 넣음
    - 같은 네트워크에서만 사용
</details>

### 4-2. 유니캐스트(Unicast)
**유니캐스트란?** 한 명에게만 보내는 1:1 방식
<details>
    <summary>특징</summary>

- 송신자 1-> 수신자 1
- 가장 일반적인 통신 방식
- 정확한 목적지 필요
</details>
<details>
    <summary>장점</summary>

- 필요한 대상에게만 보냄 → 효율적임
- 특정 대상만 받음 → 보안성 좋음
</details>
<details>
    <summary>단점</summary>

- 다수에게  보내는 게 목적 → 비효율
</details>

### 4-3. 애니캐스트 (Anycast)

**애니캐스트란?** 여러 서버 중 가장 가까운 하나에게 보내는 방식
<details>
    <summary>특징</summary>

- 여러 서버가 같은 IP사용
- 네트워크가 가장 가까운 서버 선택
</details>
<details>
    <summary>장점</summary>

- 속도 빠름
- 트래픽 여러 서버로 나뉨 → 부하 분산
- 한 서버가 죽어도 다른 서버를 연결 → 장애 대응
</details>
<details>
    <summary>단점</summary>

- 구현 복잡 (라우팅 설정 필요)
- 디버깅 어려움 (어떤 서버 연결됐는지 알기 어려움)
</details>

### 4-4. 각 캐스트들의 차이점

| **구분** | **유니캐스트** | **브로드캐스트** | **애니캐스트** |
| --- | --- | --- | --- |
| **전송 대상** | 1명 | 같은 네트워크 전체 | 여러 중 1명 |
| **통신 방식** | 1:1 | 1:N | 1:1 |
| **효율성** | 높음 | 낮음 | 매우 높음 |
| **속도** | 보통 | 느림 | 빠름 |
| **네트워크 범위** | 어디든 가능 | LAN만 가능 | 전세계 가능 |
| **대표 사용** | 웹 / API | ARP / DHCP | DNS / CDN |

## 5. TCP/UDP

### 5-1. TCP(Transmission Control Protocol)
**TCP란?** 신뢰성 있는 통신을 위한 프로토콜
<details>
    <summary>특징</summary>

- 연결 지향 (통신 전에 3-way handshake 수행)
- 신뢰성 보장
    - 데이터 누락 시 재전송
    - 순서 보장
- 흐름 제어 및 혼잡 제어
    - 상대 속도 맞춰서 전송
    - 네트워크 상태 고려
</details>
<details>
    <summary>장점</summary>

- 데이터 손실 없음 (안정적인 통신)
- 순서 보장 (영상, 파일 전송에 중요)
- 오류 복구 가능
</details>
<details>
    <summary>단점</summary>

- 속도 느림 (handshake + 확인 과정 때문 )
- 오버헤드 큼
    - 제어 정보 많음
- 실시간 통신에 부적합
    - 지연 발생
</details>

### 5-2. UDP (User Datagram Protocol)
**UDP란?** 빠른 전송을 위한 프로토콜
<details>
    <summary>특징</summary>

- 비연결형 (handshake가 없어서 바로 전송 가능)
- 신뢰성 없음 (손실, 순서 뒤바뀜 가능)
- 단순 구조 (헤더 작음 → 빠름)
</details>
<details>
    <summary>장점</summary>

- 매우 빠름
- 지연 적음
- 오버헤드 적음
</details>
<details>
    <summary>단점</summary>

- 데이터 손실 가능
    - 재전송 없음
- 순서 보장 안됨
- 신뢰성 없음
    - 중요한 데이터에는 위험
</details>

### 5-3. TCP와 UDP 차이점
| 구분 | TCP | UDP |
| --- | --- | --- |
| 연결 방식 | 연결형 | 비연결형 |
| 속도 | 느림 | 빠름 |
| 신뢰성 | 높음 | 낮음 |
| 순서 보장 | O | X |
| 재전송 | O | X |
| 용도 | 웹, 파일 | 게임, 스트리밍 |

## 6. OSI 7계층 (OSI 7layer)
**OSI 7layer란?** 네트워크 통신 과정을 7단계로 나눈 모델    
**왜 나누는가?** 
- 복잡한 네트워크를 단계별로 나눠서 이해
- 문제 생기면 어디 문제인지 찾기 쉬움
- 개발/장비 역할을 명확히 분리  

**전체 구조?**
<details>
    <summary>7계층 응용(Application)</summary>

사용자랑 직접 연결되는 계층     
**특징** - 사용자 프로그램(브라우저,앱)이 사용  
**역할** - 웹 요청, 파일 전송, 이메일, DNS 요청 등   
**단위** - Data     
**ex)** HTTP, FTP
</details>
<details>
    <summary>6계층 표현(Presentation)</summary>
   
데이터 형식 처리 담당    
**특징** - 서로 다른 시스템 간 데이터 형식 통일 / 보안 기능 일부 포함   
**역할** - 암호화/복호화, 압축, 인코딩  
**ex)** 이미지 JPG -> 다른 형식 변환, HTTPS 암호화
</details>
<details>
    <summary>5계층 세션(Session)</summary>
   
연결 유지/관리 담당    
**특징** - 통신 흐름 제어, 연결 끊어지면 재연결 지원    
**역할** - 로그인 세션 유지, 통신 시작/종료 관리             
**ex)** 로그인 유지, 화상회의 연결 유지
</details>
<details>
    <summary>4계층 전송(Transport)</summary>
   
데이터 전달 책임 담당    
**특징** - 포트 번호 사용, 데이터 신뢰성 책임   
**역할** - TCP/UDP, 데이터 분할&재조립  
**단위** - 세그먼트(Segmet)  
**ex)** TCP(신뢰성), UDP(속도)
</details>
<details>
    <summary>3계층 네트워크(Network)</summary>
   
길 찾기 담당     
**특징** - 다른 네트워크로 데이터 전달 가능, 라우터 동작    
**역할** - IP주소, 라우팅    
**단위** - 패킷(Packet)   
**ex)** IP, ICMP, ARP
</details>
<details>
    <summary>2계층 데이터링크(Data Link)</summary>
   
같은 네트워크 안에서 전달 준비    
**특징** - 스위치 동작, 물리적 주소(MAC) 사용   
**역할** - MAC주소, 오류 검사
**단위** - 프레임(Frame)    
**ex)** 이더넷(Eternet), ARP
</details>
<details>
    <summary>1계층 물리(Physical)</summary>
   
전송 담당   
**특징** - 가장 하위 계층, 하드웨어 중심    
**역할** - 0과 1을 전기/빛 신호로 변환, 케이블과 전파 사용  
</details>

## 7. 브리지(Bridge)와 스위치(Switch)
**공통점?** 둘 다 MAC 주소 기반으로 데이터를 전달하는 2계층 장비

### 7-1. 브리지 (Bridge)
**브리지란?** 두 개 이상의 네트워크를 연결하면서, MAC 주소를 기준으로 데이터 흐름을 제어하는 장비   
**어디 계층?** OSI 2계층 - MAC 주소 사용, 프레임 단위 처리
<details>
    <summary>역할</summary>

- 네트워크 연결
    - LAN과  LAN을 연결
- 트래픽 필터링
    - MAC 주소를 확인 후 필요한 곳은 보내고 불필요한 곳은 차단
- 충돌 도메인 분리
    - 네트워크를 나눠서 충돌 줄임
</details>
<details>
    <summary>동작 원리</summary>

- 프레임 수신
    - 출발지 MAC 주소 확인
- MAC 테이블 학습
- 목적지 MAC 확인
    - 테이블에 있음 → 해당 포트로만 전송
    - 테이블에 없음 → 모든 포트로 전송
    - 같은 포트 → 전송 안 함
</details>
<details>
    <summary>특징</summary>

- MAC 주소 기반 동작 (IP 아님)
- 브로드캐스트는 통과
    - ARP 같은 건 그대로 전달 됨
- 학습 기능 있음
    - 자동으로 MAC 주소 기억
</details>
<details>
    <summary>장점</summary>

- 트래픽 감소
    - 필요한 곳만 보내서 효율
- 충돌 감소
    - 네트워크 분리 효과
- 성능 향상
    - 불필요한 데이터 줄어듦
</details>
<details>
    <summary>단점</summry>

- 브로드캐스트는 못 막음
    - 전체 전송은 그대로 퍼짐
- 속도 한계
    - 소프트웨어 기반이라 느림
- 네트워크 커지면 비효율
    - 테이블 관리 어려움
</details>

### 7-2. 스위치(Switch)

**스위치란?** MAC 주소를 기반으로 데이터를 필요한 장치에만 전달하는 2계층 네트워크 장비     
**어디 계층?** OSI 2계층 (일부는 3계층 기능도 있음)
<details>
    <summary>역할</summary>

- 데이터 전달
    - 목적지 MAC 주소 보고 해당 포트로 전송
- MAC 주소 학습
    - 어떤 MAC이 어느 포트에 있는지 기억
- 필터링
    - 같은 포트면 전송 안 함
- 충돌 도메인 분리
    - 각 포트가 독립적 → 충돌 거의 없음
</details>
<details>
    <summary>동작 원리</summary>

- 프레임 도착
    - 출발지 MAC 확인
- MAC 테이블 저장
    - 이 MAC → 이 포트 저장
- 목적지 MAC 확인
    - MAC 있음 → 해당 포트로만 전송
    - MAC 없음 → 모든 포트로 전송
    - 브로드캐스트 → 모든 포트로 전송
</details>
<details>
    <summary>특징</summary>

- MAC 주소 기반 (IP 아님)
- 각 포트 독립 (충돌 없음)
- 하드웨어 기반 처리 (빠른 속도)
- 브로드캐스트는 전달 (완전히 막지는 못함)
</details>
<details>
    <summary>장점</summary>

- 네트워크 성능 향상
    - 필요한 곳만 전송
- 충돌 없음
    - 각 포트 독립
- 확장성 좋음
    - 포트 추가로 쉽게 확장
</details>
<details>
    <summary>단점</summary>

- 브로드캐스트 문제 (많아지면 네트워크 느려짐)
- 보안 취약
    - MAC 스푸핑 가능
- 비용이 비쌈
</details>
<details>
    <summary>종류</summary>

- L2 스위치
    - MAC 기반
    - 기본 스위치
- L3 스위치
    - IP 기반 라우팅 가능
    - 라우터 기능 일부 포함
</details>

### 7-3. 브리지(Bridge)와 스위치의(Switch) 차이점

| 구분 | 브리지(Bridge) | 스위치(Switch) |
| --- | --- | --- |
| 계층 | 2계층 | 2계층 (L3도 있음) |
| 포트 수 | 적음 (보통 2~4개) | 많음 (수십 개 이상) |
| 처리 방식 | 소프트웨어 기반 | 하드웨어 기반 (ASIC) |
| 속도 | 느림 | 매우 빠름 |
| MAC 학습 | 있음 | 있음 |
| 충돌 도메인 | 분리 | 완전 분리 |
| 사용 환경 | 과거/소규모 | 현재 대부분 네트워크 |
