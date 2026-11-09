# Dependabot
**Dependabot이란?** GitHub에 내장된 의존성 관리 봇이다. 프로젝트가 쓰는 라이브러리, Docker 이미지, GitHub Actions 등의 버전을 감시하다가 문제가 있거나 새 버전이 나오면 알려주고 버전을 올리는 PR을 자동으로 만들어준다. 주요 기능은 아래와 같다.

 기능 | 하는 일 | 설정 
 --- | --- | --- 
 Dependabot alerts | 쓰고 있는 의존성에 알려진 취약점이 있으면 알림 | 저장소 설정에서 켜기 
 Dependabot security updates | 취약점을 고친 버전으로 올리는 PR 자동 생성 | 저장소 설정에서 켜기 
 Dependabot version updates | 취약점이 없어도 새 버전이 나오면 PR 생성 | `.github/dependabot.yml` 필요 

## 왜 필요할까?
**취약점 방치** : 널리 쓰는 라이브러리에 치명적 취약점이 터져도(예: 2021년 Log4Shell) 내가 그걸 쓰는지조차 모르는 경우가 많다.

**업그레이드 지옥** : 버전을 오래 방치하면 메이저 버전을 여러 단계 한꺼번에 올려야 해서 깨지는 곳이 너무 많아 결국 못 올리게 된다. 조금씩 자주 올리는 게 훨씬 싸다.

# 설정 예시
아래는 백엔드(Gradle) + Docker + GitHub Actions를 쓰는 프로젝트 기준 코드이다.

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "gradle"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Asia/Seoul"
    groups:
      minor-and-patch:
        update-types: ["minor", "patch"]
    cooldown:
      default-days: 3
      semver-major-days: 14
    open-pull-requests-limit: 5

  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

**package-ecosystem** : 어떤 종류의 의존성인가 (gradle, npm, docker, github-actions 등)

**directory** : 의존성 파일이 있는 위치

**schedule.interval** : 확인 주기. `daily`, `weekly`, `monthly`부터 `quarterly`, `yearly`, `cron` 까지 지원한다.

**groups** : 여러 의존성 업데이트를 PR 하나로 묶는다. 위 예시는 minor/patch를 한 PR로 묶고 major는 따로 받는다.

**cooldown** : 새 버전이 나와도 며칠 기다렸다가 PR을 만든다.

**open-pull-requests-limit** : 동시에 열려 있을 수 있는 PR 수. version updates 기본값은 5

## cooldown은 왜 쓸까?
바로 **공급망 공격** 때문이다. 유명 패키지 관리자 계정이 탈취돼서 악성 코드가 든 새 버전이 배포되는 사건이 실제로 계속 일어나고 있다. 보통 이런 버전은 며칠 안에 발견돼서 내려간다. 그래서 새 버전이 나오자마자 받지 않고 며칠 기다리는 것이다.

기본 동작은 새 버전을 바로 업데이트 대상으로 보지만 cooldown을 설정하면 새 릴리스가 대기 기간 안에 있을 경우 그 의존성을 건너뛴다. 그리고 중요한 점은 **cooldown은 version updates에만 적용되고 security updates에는 적용되지 않는다.** 보안 패치는 기다리지 않고 바로 받는다는 뜻이라 합리적이다.

# 주의할 점
**PR 폭탄** : 그냥 켜면 PR이 쏟아지고 결국 아무도 안 보고 방치하게 된다. groups로 묶고 open-pull-requests-limit로 개수를 제한해야 한다.

**테스트가 없으면 의미가 반감** : Dependabot은 버전만 올려줄 뿐 안 깨진다는 보장은 안 한다. CI에서 빌드 및 테스트가 돌아야 PR을 믿고 머지할 수 있다. 테스트 없이 자동 머지까지 걸면 위험하다. 

**Secrets 접근이 다름** : Dependabot이 만든 PR에서 도는 워크플로는 일반 Actions secrets를 못 읽는다. Dependabot용 secrets를 따로 등록해야 한다.

**`latest` 태그는 추적 못 함** : Dockerfile에서 `FROM eclipse-temurin:latest`처럼 쓰면 비교할 버전이 없다. `eclipse-temurin:21-jre`처럼 버전을 명시해야 Dependabot이 업데이트할 수 있다.