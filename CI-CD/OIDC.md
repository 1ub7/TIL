# OIDC

**OIDC(OpenID Connect)란?** 이 사람(또는 이 시스템)이 누구인지를 증명하는 표준 인증 프로토콜이다. 쉽게 말해 OAuth 2.0 위에 신원 확인 기능을 얹은 것이다.

## ID Token

OIDC에서 발급하는 신분증으로 JWT 형식이다. 안에 아래 정보(claim)가 들어 있다.

| Claim | 의미 | 예시 |
| --- | --- | --- |
| `iss` | 누가 발급했나 | `https://token.actions.githubusercontent.com` |
| `sub` | 누구에 대한 토큰인가 | `repo:my-org/my-repo:ref:refs/heads/main` |
| `aud` | 누구에게 보여주려고 발급했나 | `sts.amazonaws.com` |
| `exp` | 언제 만료되나 | 발급 후 몇 분 |

## 어떻게 믿을 수 있을까?

발급자는 개인키로 토큰에 서명하고 공개키는 인터넷에 공개해둔다. 받는 쪽은 공개키로 서명을 검증해서 "진짜 그 발급자가 만든 토큰이고 중간에 안 바뀌었네?"를 확인한다. **발급자와 받는 쪽이 비밀번호를 미리 공유할 필요가 없다는 게 핵심이다.**

## DevOps에서 OIDC가 중요한 이유

DevOps에서 OIDC는 주로 **CI가 클라우드에 키 없이 접속하는 방법**으로 쓰인다.

**예전에는 Access Key를 Secrets에 저장**했다. GitHub Actions에서 AWS에 배포하려면 IAM 사용자의 Access Key를 만들어서 GitHub Secrets에 넣었다. 이 방식의 문제는 아래와 같다.

- **키가 영구적** : 직접 지우기 전까지 계속 유효하다.
- **유출되면 끝** : CI 러너가 털리면 키가 그대로 빠져나간다.
- **교체가 귀찮아서 안 함** : 몇 년째 같은 키를 쓰는 경우가 흔하다.

**OIDC를 쓰면 매번 임시 자격 증명을 발급**한다. GitHub이 "이 워크플로는 my-org/my-repo의 main 브랜치에서 돌고 있다"는 신분증을 발급하고 AWS가 그걸 검증해서 1시간짜리 임시 키를 내준다. **저장된 키가 아예 없어서** 털려도 금방 만료되고 정해진 저장소·브랜치에서만 쓸 수 있다.

## GitHub Actions → AWS 흐름

```
1. 워크플로 실행
2. GitHub OIDC 발급자에게 ID Token 요청
   → "repo:my-org/my-repo:ref:refs/heads/main" 이라는 신분증 발급
3. AWS STS에 신분증 제출 (AssumeRoleWithWebIdentity)
4. AWS가 검증
   - GitHub 공개키로 서명 확인
   - IAM Role 신뢰 정책 조건(aud, sub) 확인
5. 통과하면 임시 자격 증명 발급 (기본 1시간)
6. 그 자격 증명으로 배포
```

## 설정 3단계

**① AWS에 GitHub을 신뢰할 발급자로 등록 (IAM OIDC Provider)**

- Provider URL: `https://token.actions.githubusercontent.com`
- Audience: `sts.amazonaws.com`

계정당 한 번만 만들면 된다.

**② IAM Role의 신뢰 정책 작성** ← 보안의 핵심

```json
{
  "Effect": "Allow",
  "Principal": {
    "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
  },
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringEquals": {
      "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
    },
    "StringLike": {
      "token.actions.githubusercontent.com:sub": "repo:my-org/my-repo:ref:refs/heads/main"
    }
  }
}
```

GitHub이 발급한 신분증 중에서 **my-org/my-repo의 main 브랜치**인 것만 이 Role을 쓸 수 있다는 뜻이다.

**③ 워크플로에서 사용**

```yaml
permissions:
  id-token: write    # ID Token 발급 허용 (이게 없으면 토큰을 못 받음)
  contents: read

steps:
  - uses: aws-actions/configure-aws-credentials@<커밋-SHA>
    with:
      role-to-assume: arn:aws:iam::123456789012:role/github-deploy
      aws-region: ap-northeast-2
```

## `sub` 조건 패턴

어떤 상황에서 Role을 쓰게 할지 `sub`로 좁힌다.

| `sub` 값 | 허용 범위 |
| --- | --- |
| `repo:my-org/my-repo:ref:refs/heads/main` | main 브랜치에서만 |
| `repo:my-org/my-repo:environment:prod` | `prod` Environment를 쓰는 잡에서만 |
| `repo:my-org/my-repo:pull_request` | PR에서만 |
| `repo:my-org/my-repo:*` | 그 저장소 어디서든 |

실무 패턴은 **plan용 Role과 apply용 Role을 나누는 것**이다. PR에서는 읽기 권한만 있는 plan Role, main이나 prod Environment에서만 쓰기 권한 있는 apply Role을 쓴다.

## 주의할 점

**`sub` 조건을 빼먹거나 너무 넓게 쓰면 대참사가 일어날 수 있다.**

`aud`만 확인하고 `sub`를 안 걸면 **전 세계 아무 GitHub 저장소의 워크플로나** 내 Role을 쓸 수 있다. GitHub OIDC 발급자는 모든 GitHub 사용자에게 똑같은 방식으로 토큰을 발급하기 때문이다. `repo:my-org/*`처럼 조직 전체를 열어두는 것도 위험하다. 조직 내 아무 저장소에서나 운영 배포 권한을 쓸 수 있게 된다.
