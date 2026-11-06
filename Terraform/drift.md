# Drift
**Drift(드리프트)란?** Terraform 코드와 실제 AWS 상태가 어긋난 것이다. 보통은 누군가가 Terraform을 거치지 않고 인프라를 직접 바꿨을 때 생긴다. 예시는 아래와 같다.

- 누군가 AWS 콘솔에서 직접 보안 그룹이나 설정을 바꿈
- main이 아닌 다른 브랜치에서 apply함
- 급하게 CLI로 고치고 코드에 반영하는 걸 잊음

이게 위험한 이유는 **plan을 꼼꼼히 안 보면 조용히 되돌려진다는 것**이다. 콘솔에서 고친 보안 설정이 누군가의 apply 한 번에 사라지거나 반대로 코드에 없는 열린 포트가 계속 남아 있어도 아무도 모른다. 

**Drift 감지**는 매일 아침 **main 기준으로 plan을 돌려서 변경이 하나라도 나오면 Disscord로 알려주는 것**이다. apply는 하지 않고 알리기만 하는 것이다.

# Terraform이 못 잡는 Drift
**state에 없는 리소스** : 콘솔에서 새로 만든 EC2나 S3 버킷은 Terraform이 존재 자체를 모른다. plan이 깨끗하게 나와도 계정안에는 관리 안되는 리소스가 돌고 있을 수도 있다. 비용이 새고 보안 구멍이 생기는 대표적인 원인이다.

**별도 리소스로 쪼개서 관리하는 규칙** : 

```hcl
resource "aws_security_group" "web" {
name   = "web"
vpc_id = var.vpc_id
}

resource "aws_vpc_security_group_ingress_rule" "https" {
security_group_id = aws_security_group.web.id
from_port         = 443
to_port           = 443
ip_protocol       = "tcp"
cidr_ipv4         = "0.0.0.0/0"
}
```

위 구조에서 누가 콘솔로 22번 포트 규칙을 추가하면 그 규칙은 state에 없는 새 리소스여서 Terraform은 못 잡는다. 반대로 `aws_security_group` 안에 `ingress {}` 블록을 직접 쓰면 그 목록 전체를 Terraform이 관리하기 때문에 추가된 규칙도 diff로 잡힌다. 대신 그 방식은 다른 리소스와 같이 쓰면 충돌하는 등 단점이 있다.

**`ignore_changes`로 빼둔 속성** : 일부러 추적을 끈거여서 당연히 안 잡힌다. 남발하면 drift를 숨기는 도구가 된다.

**코드에 안적은 선택(Optional) 속성** : 코드에 명시하지 않고 provider가 기본값을 알아서 채우는 속성은 바뀌어도 diff가 안 날 수 있다. 중요한 설정은 기본값이어도 코드에 명시하는게 안전하다.

**Terraform 바깥 레이어** : SSH로 들어가서 바꾼 nginx 설정, DB 안의 데이터, 클러스터 안에서 `kubectl` 로 바꾼 리소스 같은 건 Terraform 관할이 아니다.

# 해결하는 방법
| 상황 | 해결 | 명령 |
| --- | --- | --- |
| 수동 변경이 실수 | 코드 기준으로 되돌리기 | `terraform apply` |
| 수동 변경이 맞음 | 코드를 실제에 맞게 수정 → 
plan에 차이가 없어질 때까지 | `.tf` 수정 후 `terraform plan` 으로 “No changes” 확인 |
| 외부에서 바뀌는 게 정상 | 추적 예외 | `lifecycle { ignore_changes = [...] } |
| 콘솔로 새로 만든 리소스(계속 쓸 예정) | Terraform 관리로 편입 | import 블록 |
| 콘솔로 새로 만든 리소스(필요 없음) | 콘솔/CLI로 삭제 |  |

여기서 import 블록의 예시는 아래와 같다.

```hcl
import {
  to = aws_instance.legacy
  id = "i-0abc123def456"
}
```

# 예방하는 방법
**권한으로 막기** : 사람 계정은 운영 환경에서 ReadOnly를 해두는 게 좋다. apply는 CI의 IAM Role만 가능하게 한다. 장애 대응용 비상 계정은 따로 두되 쓰면 반드시 사후에 코드 반영하는 규칙 설계가 되어야한다.

**프로세스로 막기** : 모든 변경은 PR → CI가 plan 결과를 PR 코멘트로 → 리뷰 → main 머지 후 apply한다. apply는 main에서만 하게 하고 Atlantis 같은 도구가 이 흐름을 자동화해준다.