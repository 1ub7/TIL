# CloudFront
**CloudFront란?** AWS의 CDN(Content Delivery Network) 서비스이다. 전 세계에 깔린 AWS 엣지 서버에 콘텐츠를 캐시해두고 사용자는 원본 서버 대신 **가장 가까운 엣지**에서 받아가게 하는 것이다. 아래는 핵심 용어들이다.

 용어 | 의미 
 --- | --- 
 Distribution | CloudFront 설정 단위. 하나 만들면 `d1234.cloudfront.net` 같은 도메인이 생김 
 Origin | 원본 콘텐츠가 있는 곳. S3, ALB, EC2, 외부 서버 등 
 Edge Location | 사용자와 가까운 캐시 서버. 한국에도 여러 곳 있다. 
 Cache Behavior | 경로별 규칙. `/api/*` 는 캐시 안 함. `/static/*` 는 1년 캐시처럼 나눔 
 TTL | 캐시를 얼마나 오래 유지할지 
 Cache Key | 캐시를 구분하는 기준. 기본은 URL이고 쿼리스트링/헤더/쿠키를 추가할 수 있음 
 Invalidation | 캐시를 강제로 지우는 것 

## 왜 쓸까?
**속도** : 가까운 엣지에서 응답하니까 빠르다.

**오리진 부하 감소** : 요청 대부분을 엣지가 처리하니 서버가 덜 맞는다.

**비용** : S3나 ALB 같은 AWS 오리진에서 CloudFront로 가는 데이터 전송은 무료라서 S3에서 직접 서빙하는 것보다 싸지는 경우가 많다.

**보안** : DDoS 기본 방어, WAF 연결, 그리고 S3를 비공개로 두고 CloudFront로만 접근하게 만들 수 있다. 

# 흔한 실무 구조
1. 프론트엔드 정적 배포 : S3 + CloudFront

국내에서 React, Vue 같은 프론트엔드를 배포할 때 거의 정석인 구조이다. 

```text
사용자 → CloudFront → S3 (비공개 버킷)
              ↑
         OAC로 CloudFront만 S3 접근 허용
```

- **OAC(Origin Access Control)**: S3 버킷을 public으로 열지 않고 CloudFront만 읽을 수 있게 하는 설정이다. 예전엔 OAI라는 방식을 썼는데 지금은 **OAC가 권장**이다.
- **SPA 라우팅 문제**: `/mypage` 같은 경로로 새로고침하면 S3에 그 파일이 없어서 403/404가 난다. CloudFront의 커스텀 에러 응답으로 `index.html`을 돌려주게 설정해서 해결한다.

2. 프론트 + 백엔드를 한 도메인

```text
example.com/*      → S3 (프론트, 캐시 O)
example.com/api/*  → ALB (백엔드, 캐시 X)
```

Cache Behavior로 경로별로 오리진을 나눈다. 한 도메인이라 CORS 문제도 줄어든다.

## 캐시 무효화 vs 파일명 버저닝
배포했는데 사용자한테 옛날 화면이 보이는 건 캐시 때문인데 이 해결 방법이 두 가지 있다. 

- **Invalidation**: 배포할 때마다 `/*` 캐시 삭제. 간단하지만 반영에 시간이 걸리고 일정 개수를 넘으면 과금된다.
- **파일명 버저닝**: 빌드 도구가 `main.a1b2c3.js`처럼 내용이 바뀌면 파일명도 바뀌게 만든다. 이러면 JS·CSS는 1년 캐시해도 안전하고 `index.html`만 짧게 캐시하거나 무효화하면 된다. **실무에서는 이게 정석이다.**

# 주의할 점
**인증서는 us-east-1에서 발급** : CloudFront에 붙일 ACM 인증서는 버지니아(us-east-1) 리전에서 만들어야 한다. 서울 리전에서 만들면 목록에 안 뜬다.

**캐시 키 실수 = 개인정보 유출** : 사용자마다 응답이 다른 API를 캐시하면 A의 응답이 B에게 보일 수 있다. 개인화된 응답은 캐시하지 않는다. 반대로 헤더·쿠키를 전부 캐시 키에 넣으면 캐시가 거의 안 맞아 CloudFront를 쓰는 의미가 없어진다.

**오리진 직접 접근 차단** : CloudFront를 붙여도 ALB나 S3에 직접 접근이 열려 있으면 WAF를 우회해 공격받을 수 있다.