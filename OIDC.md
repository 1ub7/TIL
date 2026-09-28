# OIDC
**OIDC(OpenID Connect)란?** OAuth 2.0 위에 인증계층을 얹은 표준이다. 

# ID Token 구조
ID Token은 **JWT**이고 `.`으로 구분된 3부분으로 이루어짐

```
header.payload.signature
```

## Header
```json
{
  "alg": "RS256",
  "kid": "abc123",
  "typ": "JWT"
}
```

**alg** : 서명 알고리즘이다. RS256은 비대칭키 방식으로 OP가 개인키로 서명하고 RP는 공개키로 검증한다.

**kid** : 서명에 사용한 키의 ID이다. OP의 JWKS(공개키 목록)에서 어떤 키로 검증할지 찾는 데 씀

## Payload
```json
{
  "iss": "https://accounts.google.com",
  "sub": "10769150350006150715113082367",
  "aud": "my-client-id",
  "exp": 1759050000,
  "iat": 1759046400,
  "nonce": "n-0S6_WzA2Mj",
  "email": "student@gsm.hs.kr"
}
```

**iss** : 토큰을 발급한 OP의 URL      
**sub** : 사용자의 고유 ID (발급자 안에서만 유일)  
**aud** : 이 토큰을 받을 대상   
**exp** : 만료 시각     
**nonce** : RP가 로그인 요청 시 생성해 보낸 랜덤 값이다. OP가 이를 ID Token에 그대로 넣어주고 RP는 자기가 보낸 값과 일치하는지 확인한다. (재전송 공격 방지)

## Signature
`header.payload`를 OP의 개인키로 서명한 값을 말한다. 내용이 1바이트라도 바뀌면 검증에 실패한다.

# ID Token 검증
(1) **서명 검증** (kid로 JWKS에서 공개키 조회)  
(2) **iss** : 신뢰하는 OP인가   
(3) **aud** : 내 client_id인가 → 빠뜨리면 다른 서비스용 토큰으로 로그인되는 취약점  
(4) **exp** : 만료되지 않았는가     
(5) **nonce** : 내가 보낸 값과 일치하는가

# Access Token과 ID Token
구분 | ID Token | Access Token
--- | --- | ---
용도 | 클라이언트가 누가 로그인했나 확인 | API(리소스 서버) 호출 권한
수신자 | RP | API 서버
형식 | 항상 JWT | JWT일 수도, 불투명 문자열일 수도 있음

# DevOps 사용 사례
**GitHub Actions -> AWS** : Accesss Key 없이 워크플로가 OIDC 토큰으로 임시 자격증명을 받는다.   

**K8s API 서버 인증** : Keycloak, Dex 등을 연동하고 토큰의 `groups` claim을 RBAC에 매핑한다.   

**ArgoCD, Grafana, Vault SSO** : OIDC로 사내 IdP와 연동한다.