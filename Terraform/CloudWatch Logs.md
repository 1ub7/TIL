# CloudWatch Logs
**CloudWatch Logs란?** AWS가 제공하는 중앙 로그 수집/저장/검색 서비스이다. 서버마다 흩어져 있던 로그를 한 곳으로 모아서 저장하고 검색하고 알람까지 걸 수 있게 해준다. 구조는 아래 세 단계로 이해하면 된다.

 단위 | 의미 | 예시 
 --- | --- | --- 
 Log Group | 로그를 묶는 큰 단위. 보통 서비스나 앱 하나 | /myapp/backend/prod 
 Log Stream | 그룹 안에서 로그를 보내는 출처 하나 | 인스턴스 1대, 컨테이너 1개 
 Log Event | 로그 한 줄 | 2026-11-07 ERROR 결제 실패… 

보존 기간(Retention)은 **Log Group 단위**로 설정하고 그 안의 모든 Stream과 Event가 그 정책을 따른다.

## 왜 쓸까?

로그를 서버 안 파일이나 `docker logs`로만 보면 아래 문제가 생긴다.

- 서버나 컨테이너가 사라지면 로그도 같이 사라진다 (재배포, 오토스케일링 종료)
- 서버가 여러 대면 하나씩 SSH로 들어가서 봐야 한다.
- 로그가 디스크를 가득 채워 서버가 죽기도 한다.
- 검색이 `grep` 수준이다.

# 장점
**로그가 서버 수명과 분리된다** : 서버가 죽어도 로그가 남아 사후 분석이 가능하다. 컨테이너 환경에서는 사실상 필수이다.  

**중앙 집중** : 여러 서버의 로그를 한 곳에서 본다.

**Logs Insights로 쿼리 검색** : 로그를 JSON으로 찍으면 필드를 자동 인식한다.

**로그 기반 알람** : Metric Filter → CloudWatch Alarm → SNS → Slack/Discord 알림

**SSH 없이 로그 확인** : IAM 권한으로 로그만 보게 할 수 있어 운영 서버 접근을 줄인다.

**보존 기간 자동 관리** : 기간이 지나면 자동 삭제되고, 장기 보관은 S3로 내보낸다.

**Logs Insights로 쿼리 검색** : 로그를 JSON으로 찍으면 필드를 자동 인식한다.

ERROR가 포함된 최근 로그 50개 보기

```
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 50
```

5분 단위 에러 개수 세기 (JSON 로그에 `level` 필드가 있을 때)

```
filter level = "ERROR"
| stats count(*) by bin(5m)
```

# 단점

**비용** : 수집량(GB)만큼 과금된다. 아시아 리전은 미국보다 비싸다.

**검색 기능** : Datadog, ELK(OpenSearch), Grafana Loki 같은 전용 도구보다 투박하다는 평가가 많다.

**AWS 종속** : 다른 클라우드로 옮기면 로그 체계를 다시 짜야 한다.

**개인정보 노출** : 토큰, 비밀번호 등이 로그에 찍히면 중앙에 그대로 쌓인다.

## 과금 패턴

비용이 갑자기 많아지는 경우는 아래와 같다.

- 운영 환경에서 DEBUG 레벨 로그를 켜둠
- 보존 기간 기본값이 "만료 없음"이라 계속 쌓임
- 장애 시 에러 로그가 폭증하면서 요금도 폭증

→ **Log Group을 만들면 보존 기간부터 설정한다.**

# 연동 방법

 실행 환경 | 방법 
 --- | --- 
 EC2에 직접 실행 | CloudWatch Agent 설치 + 로그 파일 경로 지정 + EC2에 IAM Role 부여 
 EC2에서 Docker | Docker 로그 드라이버를 `awslogs`로 설정 
 ECS | 태스크 정의의 `logConfiguration`에 `awslogs` 드라이버 지정 
 EKS | Fluent Bit 같은 수집기를 DaemonSet으로 배포 

**원칙** : 앱은 로그를 파일이 아니라 stdout으로 찍고 수집은 인프라에 맡긴다. 가능하면 JSON 구조화 로그로 찍는다.