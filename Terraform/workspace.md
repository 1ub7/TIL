# Workspace

**workspace란?** 같은 코드(같은 디렉터리)로 state를 여러 개 갖게 해주는 기능이다. 원래 디렉터리 하나에는 state가 하나이다. 그래서 같은 코드로 dev와 prod를 따로 만들고 싶으면 디렉터리를 복사해야 했다.

Workspace를 쓰면 **코드는 그대로 두고 state만 바꿔 끼워서 여러 환경**을 만들 수 있다.

```
같은 코드 (main.tf)
   ├── workspace: default → state A → 리소스 세트 A
   ├── workspace: dev     → state B → 리소스 세트 B
   └── workspace: prod    → state C → 리소스 세트 C
```

아무 설정 안 해도 처음엔 `default`라는 workspace에서 작업하고 있는 것이다.

## 명령어

```bash
terraform workspace list          # 목록 (* 표시가 현재 workspace)
terraform workspace new dev       # 생성 + 이동
terraform workspace select prod   # 이동
terraform workspace show          # 현재 workspace 이름
terraform workspace delete dev    # 삭제 (리소스가 남아 있으면 거부됨)
```

## 코드에서 workspace 이름 쓰기

`terraform.workspace`로 현재 workspace 이름을 꺼낼 수 있다.

```hcl
locals {
  instance_type = {
    dev  = "t3.micro"
    prod = "m5.large"
  }
}

resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = local.instance_type[terraform.workspace]

  tags = {
    Name = "web-${terraform.workspace}"   # web-dev, web-prod
  }
}
```

## state 저장 위치

| backend | 저장 위치 |
| --- | --- |
| 로컬 | `terraform.tfstate.d/<workspace 이름>/terraform.tfstate` |
| S3 | `env:/<workspace 이름>/<key>` (default만 원래 key 그대로) |

S3 backend면 **같은 버킷 안에 경로만 다르게** 저장된다.

# 문제점

**① 지금 어느 workspace인지 헷갈린다**

terraform apply를 실행했는데 명령어에 환경 이름이 안 드러나니까 알고 보니 prod였다는 사고가 실제로 난다. 디렉터리 방식은 cd envs/prod처럼 경로에 환경이 보이는데 workspace는 workspace show를 쳐봐야 안다.

**② 환경끼리 제대로 분리가 안 된다**

모든 workspace가 **같은 backend, 같은 자격 증명**을 쓴다. dev를 apply할 수 있는 사람이 prod도 apply할 수 있고 state가 같은 버킷에 있으니 권한을 나누기 어렵다. HashiCorp 공식 문서도 **자격 증명이나 접근 권한을 엄격히 분리해야 하는 환경에는 CLI workspace가 적합하지 않다**고 말한다.

**③ 코드에 환경 분기가 퍼진다**

terraform.workspace == "prod" ? ... : ... 같은 조건이 여기저기 생긴다. 모듈화 공부할 때 **모듈 안에서 환경 이름으로 분기하지 마**라고 한 것과 같은 문제이다. 환경이 늘어날수록 코드가 지저분해진다.

**④ 환경마다 구조가 다르면 못 쓴다**

**prod에만 WAF와 멀티 AZ RDS가 있다**처럼 리소스 구성 자체가 다르면 결국 `count = terraform.workspace == "prod" ? 1 : 0` 같은 코드가 잔뜩 늘어난다.

# Workspace VS 디렉터리 분리

|  | Workspace | 디렉터리 분리 |
| --- | --- | --- |
| **코드** | 하나 | 환경마다 (모듈로 중복 최소화) |
| **state** | 같은 backend, 경로만 다름 | 환경마다 완전히 분리 가능 |
| **권한 분리** | 어려움 | 쉬움 (backend/Role을 다르게) |
| **실수 위험** | 현재 workspace 착각 | 경로에 환경이 보여서 낮음 |
| **환경별 구조 차이** | 조건문 늘어남 | 자연스럽게 처리 |

그래서 실무에서 dev / staging / prod 처럼 중요한 환경 분리는 디렉터리 방식을 더 많이 쓰고 workspace는 아래와 같은 곳에 쓰인다.

- 구조가 완전히 같은 임시 환경 : 기능 브랜치마다 미리보기 환경 띄우기
- 개발자별 샌드박스 : `alice` , `bob` workspace로 각자 테스트 환경