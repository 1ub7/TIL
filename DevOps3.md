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

## 4-3. 애니캐스트 (Anycast)

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
