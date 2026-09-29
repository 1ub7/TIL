# SSM
**SSM(Systems Manager)이란?** EC2 인스턴스나 온프레미스 서버를 SSH 없이 한곳에서 관리하는 AWS 서비스 묶음이다. 하나의 기능이 아닌 여러 도구의 모음이라서 "SSM 쓴다"라고 하면 보통 그중 한두 개를 쓴다는 뜻이다. 
(즉,AWS 리소스를 원격으로 관리하고 자동화할 수 있는 통합 관리 서비스)

# 동작원리
SSM의 핵심은 서버에 설치된 **SSM Agent**이다. 

(1) 에이전트가 AWS의 SSM 엔드포인트로 먼저 바깥으로 연결을 맺는다.  

(2) AWS 쪽에서 명령을 내리면 이 연결을 통해 에이전트가 받아서 실행한다.

(3) 그래서 서버에 인바운드 포트를 열 필요가 없다

## 동작 조건
SSM Agent가 동작하려면 아래 3가지 조건이 필요하다.

**에이전트 설치** : Amazon Linux, Ubuntu 등 주요 AMI에는 기본으로 설치돼 있고 그 외 OS나 온프레미스 서버는 직접 설치해야 한다.

**IAM 권한** : 인스턴스에 `AmazonSSMManagedInstanceCore` 정책이 붙은 인스턴스 프로파일(IAM Role)이 연결돼 있어야 한다.

**네트워크 경로** : 인스턴스가 SSM 엔드포인트에 닿을 수 있어야 한다. 프라이빗 서브넷이라면 NAT Gateway를 거치거나, VPC 엔드포인트를 만들어야한다. (ex: `ssm`, `ssmmessages`, `ec2messages`)

# 주요 기능
**Parameter Store** : 설정값과 비밀값을 저장하는 계층형 Key-value 저장소이다. 경로 형태로 관리하며 경로 단위로 IAM 권한을 줄 수 있다. (DB 비밀번호나 API 키를 코드나 이미지에 하드코딩하지 않고 런타임에 조회한다.)

타입은 `String`, `StringList`, `SecureString`(KMS 암호화)이다.

**Session Manager** : SSH 없이 브라우저나 CLI로 서버 셸에 접속하는 기능이다. SSM Agent가 AWS로 아웃바운드 연결을 하는 구조여서 22번 포트를 안열어도 된다.

접근은 IAM으로 통제하고 접속 기록은 CloudTrail에 남는다. 세션 로그는 S3나 CloudWatch Logs에 저장할 수 있다.

**Run Command** : 여러 서버에 원격으로 명령을 한 번에 실행하는 기능이다. 인스턴스 ID나 태그를 대상으로 지정한다. 실행할 작업은 SSM Document로 정의한다. 

실행 결과는 콘솔, S3, CloudWatch에서 확인한다. 서버마다 SSH로 들어가 수작업하던 일을 자동화하고 이력을 남긴다.

**Patch Manager** : OS 보안 패치를 정해진 기준과 일정에 따라 자동 적용하는 기능이다. Patch Baseline을 정하고 Maintenance Window에 맞춰 OS 패치를 자동 적용한다. 패치 준수 여부를 리포트로 확인할 수도 있다.