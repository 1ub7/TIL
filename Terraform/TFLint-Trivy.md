# TFLint
**TFLint란?** Terraform 코드 전용 린터이다. 코드를 실행하지 않고 읽기만 해서 apply하면 실패할 코드나 관례에 어긋난 코드를 미리 찾아준다.

## 왜 필요할까?
Terraform 기본 검사 도구들은 각자 보는 범위가 좁다.

| 도구 | 보는 것 | 못 보는 것 |
| --- | --- | --- |
| `terraform fmt` | 들여쓰기, 정렬 | 내용 전부 |
| `terraform validate` | 문법, 타입, 필수 속성 누락 | 값이 실제로 유효한지 |
| `terraform plan`  | 실제 변경 내용 | 일부 오류는 apply 때야 AWS API가 거절 |

예를 들어 `instance_type = "t3.huuge"` 같은 오타는 문법상 문제가 없어서 validate를 통과한다. TFLint는 AWS에 실제로 존재하는 인스턴스 타입 목록을 알고 있어서 **apply 전에 이런 오류를 잡아준다.**

## 설정 파일 : .tflint.hcl
프로젝트 루트에 두는 설정 파일이다. 블록은 세 종류가 있다.

```hcl
# 예시
# 1. 전체 동작 설정
config {
  call_module_type = "local"   # 로컬 모듈(./modules/...)까지 검사
}

# 2. 플러그인
plugin "terraform" {
  enabled = true
  preset  = "recommended"      # 추천 규칙 묶음
}

plugin "aws" {
  enabled = true
  version = "x.y.z"            # GitHub 릴리스에서 최신 버전 확인 후 고정
  source  = "github.com/terraform-linters/tflint-ruleset-aws"
}

# 3. 개별 규칙 켜기/끄기/옵션
rule "terraform_naming_convention" {
  enabled = true
  format  = "snake_case"
}

rule "aws_resource_missing_tags" {
  enabled = true
  tags    = ["Environment", "Owner"]
}
```

| 블록 | 역할 |
| --- | --- |
| `config` | 모듈 검사 범위, 출력 형식 등 전역 설정 |
| `plugin` | 어떤 플러그인을 어떤 버전으로 쓸지 |
| `rule` | 규칙 하나하나를 켜고 끄거나 옵션 지정 |

| `call_module_type` 값 | 설명 |
| --- | --- |
| `local` | `source = "./modules/vpc"` 같은 로컬 모듈만 검사 (기본값) |
| `all` | 레지스트리 등 외부 모듈까지 검사 (terraform init으로 모듈을 먼저 받아야함) |
| `none` | 모듈 내부는 검사 안 함 |

### terraform 플러그인(recommended 프리셋)

| 규칙 | 잡는 것 |
| --- | --- |
| `terraform_unused_declarations` | 선언만 하고 안 쓰는 변수, local, data |
| `terraform_required_version` | `required_version` 누락 |
| `terraform_required_providers` | provider 버전 제약 누락 |
| `terraform_typed_variables` | 타입 없는 변수 |
| `terraform_module_pinned_source` | Git 모듈을 브랜치(main)로 참조 |
| `terraform_deprecated_interpolation` | `"${var.x}"` 같은 옛날 문법 |

### recommended에 없어서 직접 켜야 하는 것들

| 규칙 | 잡는 것 |
| --- | --- |
| `terraform_naming_convention` | 리소스/변수 이름 규칙 위반 |
| `terraform_documented_variables` | `description` 없는 변수 |
| `terraform_standard_module_structure` | `main.tf` , `variables.tf` , `outputs.tf` 구조 미준수 |

### aws 플러그인
| 규칙 | 잡는 것 | 기본 |
| --- | --- | --- |
| `aws_instance_invalid_type` | 존재하지 않는 인스턴스 타입 | 켜짐 |
| `aws_db_instance_invalid_type` | 존재하지 않는 RDS 클래스 | 켜짐 |
| `aws_instance_previous_type` | `t2` 같은 이전 세대 타입 | 켜짐 |
| `aws_resource_missing_tags` | 필수 태그 누락 | 꺼짐 |

## 명령어
```bash
tflint --init                    # 플러그인 다운로드 (설정 바꾸면 다시 실행)
tflint                           # 현재 디렉터리 검사
tflint --recursive               # 하위 디렉터리 전부 검사
tflint --var-file=prod.tfvars    # 변수 값을 넣고 검사
tflint --fix                     # 자동 수정 가능한 규칙은 고쳐줌
tflint --minimum-failure-severity=error   # error만 실패 처리, warning은 통과
tflint --format=compact          # 출력 형식 변경 (json, sarif 등도 가능)
```

## 한계점
- **보안은 거의 안 본다** : SG에 `0.0.0.0/0`로 SSH를 열어도 값 자체는 유효해서 통과한다. 이건 Trivy 같은 보안 스캐너의 몫이다.
- **실행 시점 값은 모른다** : data 소스나 다른 리소스 출력에서 오는 값은 검사를 건너뛴다.
- **apply 성공을 보장하진 않는다** : 권한 부족, 서비스 한도, 이름 중복은 여전히 apply 때 터진다.

# Trivy
**Trivy란?** Aqua Security가 만든 오픈소스 범용 보안 스캐너이다. 도구 하나로 컨테이너 이미지, 코드 저장소, IaC 파일, 쿠버네티스 클러스터까지 스캔한다.

## 핵심 구조
**대상(Target)**

| 명령 | 대상 |
| --- | --- |
| `trivy image` | 컨테이너 이미지 |
| `trivy fs` | 로컬 디렉터리 |
| `trivy repo` | Git 저장소 |
| `trivy config` | IaC 파일만 (Terraform, Dockerfile, K8s YAML 등) |
| `trivy k8s` | 실행 중인 쿠버네티스 클러스터 |
| `trivy sbom` | SBOM 파일 |

**스캐너(Scanner)**

| 스캐너 | 찾는 것 |
| --- | --- |
| `vuln` | 패키지/라이브러리의 알려진 취약점(CVE) |
| `misconfig` | 위험한 설정 (보안그룹 전체 개방, root 실행 등) |
| `secret` | 하드코딩된 비밀번호, AWS 키, 토큰 |
| `license` | 라이선스 문제 (상업적 사용 불가 라이선스 등) |

## 왜 필요할까?

지금까지의 검사 도구들은 **코드가 맞냐**만 보고 **안전하냐**는 안 본다.

| 도구 | 보는 것 | 못 보는 것 |
| --- | --- | --- |
| `terraform validate` | 문법, 타입, 필수 속성 누락 | 값이 실제로 유효한지, 보안 |
| TFLint | 값이 유효한지, 관례 | 보안 설정 |
| 코드 리뷰 | 로직, 설계 | 라이브러리 수백 개의 취약점, 놓치기 쉬운 설정 실수 |
| **Trivy** | 취약점(CVE), 위험한 설정, 하드코딩된 시크릿 | 제로데이, 실행 중 공격 |

예를 들어 `cidr_blocks = ["0.0.0.0/0"]`로 SSH를 전 세계에 열어도 값 자체는 유효해서 validate와 TFLint를 모두 통과한다. 또 백엔드 이미지 하나에도 패키지가 수백 개 들어 있고 새 취약점은 매일 공개되기 때문에 사람이 일일이 확인할 수 없다. Trivy는 이런 보안 문제를 **배포 전에 매번 같은 기준으로 잡아준다.**

## 사용 예시
### 이미지 스캔
```bash
trivy image eclipse-temurin:21-jre
```

결과는 아래처럼 패키지별로 나온다.

| Library | Vulnerability | Severity | Status | Installed | Fixed |
| --- | --- | --- | --- | --- | --- |
| libssl3 | CVE-XXXX-XXXX | HIGH | fixed | 3.0.13-1 | 3.0.13-2 |
| zlib | CVE-YYYY-YYYY | MEDIUM | affected | 1.2.13 |  |

| Status | 의미 |
| --- | --- |
| `fixed` | 패치된 버전이 있다 → 버전을 올리면 해결된다 |
| `affected` | 아직 패치가 없다 → 지금 당장은 고칠 방법이 없다 |

대부분의 취약점은 OS 패키지에서 나온다. 그래서 **베이스 이미지를 최신으로 올리거나** `alpine`·distroless처럼 **작은 이미지로 바꾸거나 멀티 스테이지 빌드**로 최종 이미지에 필요한 것만 남기면 취약점이 크게 줄어든다.

### IaC 설정 스캔
```bash
trivy config .
```

```hcl
resource "aws_security_group_rule" "ssh" {
  type              = "ingress"
  from_port         = 22
  to_port           = 22
  protocol          = "tcp"
  cidr_blocks       = ["0.0.0.0/0"]   # 전 세계에 SSH 개방 → 경고
  security_group_id = aws_security_group.web.id
}
```

```docker
FROM node:latest          # latest 태그 사용 → 경고
COPY . .
CMD ["node", "app.js"]    # USER 지정 없음 = root로 실행 → 경고
```

## CI 옵션
```bash
trivy image \
  --severity HIGH,CRITICAL \
  --ignore-unfixed \
  --exit-code 1 \
  myapp:1.0
```

| 옵션 | 의미 |
| --- | --- |
| `--severity` | 볼 심각도를 지정한다 |
| `--exit-code 1` | 발견되면 종료 코드 1을 반환해 CI를 실패시킨다 (기본값은 발견해도 0) |
| `--ignore-unfixed` | 패치가 없는(`affected`) 취약점은 제외한다 |
| `--format` | `table`, `json`, `sarif` 등. `sarif`로 뽑으면 GitHub Security 탭에 결과가 올라간다 |
- **`-exit-code 1`이 핵심이다.** 기본값은 취약점이 나와도 0을 반환해서 이 옵션이 없으면 CI는 결과를 출력만 하고 그냥 통과한다.
- **`-ignore-unfixed`를 쓰는 이유** : 패치가 없는 취약점까지 막으면 내가 할 수 있는 게 없는데도 배포가 계속 막힌다. 고칠 수 있는 것만 막고 패치가 나오면 그때 잡히게 한다.

**처음엔 `CRITICAL`만 막는다.** 처음 돌리면 결과가 수십~수백 개 나온다. 전부 막으면 아무것도 배포 못 하고 결국 스캔 자체를 꺼버리게 된다. `CRITICAL`부터 정리하고 단계적으로 `HIGH`까지 올린다.

## 한계점
- **알려진 취약점만 잡는다** : 취약점 DB와 대조하는 방식이라 아직 공개되지 않은 제로데이는 못 잡는다. DB가 오래되면 새 취약점도 놓친다.
- **심각도 ≠ 실제 위험도** : CRITICAL이라도 그 기능을 우리 코드가 안 쓰면 실제 위험은 낮고 MEDIUM이라도 외부에 노출된 기능이면 위험할 수 있다. 결과를 기계적으로 따르지 말고 판단해야 한다.
- **노이즈가 많다** : 결과가 너무 많으면 사람들이 무시하기 시작한다. 예외 처리와 단계적 기준이 도구 설정보다 중요하다.
- **Trivy 자체도 공격 대상이다** : 2026년 3월 Trivy와 `trivy-action`이 공급망 공격을 당해 악성 버전이 배포됐다. CI에서 쓸 때는 Action을 태그가 아니라 **커밋 SHA로 고정**하고 스캔 잡에는 최소 권한만 준다.