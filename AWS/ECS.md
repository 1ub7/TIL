# ECS
**ECS(Elastic Container Service)란?** AWS에서 제공하는 완전관리형 컨테이너 오케스트레이션 서비스이다. 쿠버네티스가 하는 일을 AWS 전용 방식으로 더 단순하게 제공한다.

# 구성요소
## 논리 구성요소
**Cluster** : Task와 Service를 묶는 논리적인 그룹이다. 그 자체로는 컴퓨팅 자원을 갖지 않는다. Fargate는 등록할 자원이 필요 없고 EC2 방식은 ECS Agent가 설치된 EC2를 클러스터에 등록해야 Task를 띄울 수 있다.

Service나 Task를 실행하면 스케줄러가 조건에 맞는 실행 기반(Fargate 또는 Container Instance)을 골라 배치한다.

**Task** : Task Definition으로 실제 실행된 단위이다. K8s의 Pod와 가장 비슷하다. 같은 Task 안의 컨테이너는 네트워크를 공유한다. 

**Task Definition** : Task를 실행하기 위한 설정을 저장하고 있는 단위이고 컨테이너 실행 명세서(JSON)이다. `docker run`에 붙이던 옵션을 파일로 옮긴 것이라고 보면 된다. 

Task 레벨 설정과 컨테이너 레벨 설정이 나뉜다. 한 번 만들면 수정할 수 없고, 변경하면 새 리비전(`myapp:1`, `myapp:2`)이 생긴다. 배포는 새 리비전을 Service에 지정하는 것이고, 롤백은 이전 리비전으로 되돌리는 것이다.

**Service** : Task를 지속적으로 관리하는 단위이다. Cluster 내에서 지정한 desiredCount만큼 Task를 유지하고, 죽으면 새로 띄운다. ELB와 연동하면 Task가 생성/종료될 때 타깃 그룹에 자동으로 등록/해제한다. 

오토스케일링은 Service가 직접 하는 게 아니라 Application Auto Scaling이 desiredCount를 조절하는 방식이다.

## 실행 인프라
**Fargate** : 서버 없이 Task 단위로 격리된 마이크로 VM에서 실행한다. 호스트 접근이 불가능하고 Task마다 전용 커널 환경을 쓴다. 패치와 스케일링할 노드가 없다. 과금은 Task가 쓰는 vCPU/메모리 초 단위이다.

**EC2 Launch Type** : 직접 EC2를 운영하는 방식이다. 인스턴스는 ASG로 관리한다. Task 스케일링과 인스턴스 스케일링을 둘 다 신경 써야 해서 운영 부담이 크다. GPU, 특수 인스턴스가 필요하거나 대규모에서 비용을 최적화할 때 쓴다.

**Capacity Provider** : Task를 어떤 용량 풀에서 돌릴지를 추상화한 것이다. 종류는 `FARGATE`, `FARGATE_SPOT`, ASG 기반 EC2가 있다. EC2 방식에서는 Managed Scaling으로 Task 수요에 맞춰 ASG를 자동 조절할 수 있다.

# 역할
**스케줄링** : Service나 run-task 요청이 들어오면 Task를 어느 자원, 어느 AZ에 띄울지 결정한다. Fargate는 Task마다 격리된 마이크로 VM을 할당하고 지정한 서브넷들에 걸쳐 AZ를 분산한다. EC2는 CPU/메모리가 남는 Container Instance를 찾아 placement strategy에 따라 배치한다. 서브넷을 1개만 지정하면 AZ 분산 효과가 없다.

**자동 복구** : Service가 desiredCount와 실제 실행 중인 Task 수를 계속 비교해서 차이가 나면 새 Task를 띄워 맞춘다. essential 컨테이너 종료, 컨테이너 헬스체크 실패, ALB 헬스체크 실패를 죽었다고 판단한다. 프로세스는 살아 있는데 앱이 먹통인 경우는 헬스체크가 없으면 ECS가 감지하지 못하므로 헬스체크 설계가 복구 품질을 결정한다.

**무중단 롤링 배포와 롤백 처리** : 새 Task Definition 리비전으로 Service를 업데이트하면 새 Task를 먼저 띄우고 ALB 헬스체크를 통과한 뒤 기존 Task를 draining 후 종료한다. 기본값 minimumHealthyPercent=100이라 배포 중에도 정상 Task 수가 줄지 않는다. 서킷 브레이커를 켜면 배포가 연속 실패할 때 이전 리비전으로 자동 롤백되고 리비전이 불변이라 수동 롤백도 이전 리비전을 지정하면 끝난다. 롤백되는 건 앱 코드뿐이라 DB 스키마 변경은 되돌아가지 않는다.

# 특징
**선언적 상태 관리** : 원하는 상태를 선언하면 ECS가 현재 상태와 계속 비교하면서 차이가 나면 맞춘다. Task가 죽으면 새로 띄우고 배포 시에는 새 리비전으로 교체한다. 

**컴퓨팅 방식 선택 가능** : 서버를 직접 운영하지 않는 Fargate, 인스턴스를 직접 관리하는 EC2, 그리고 이 둘을 섞어 쓰는 Capacity Provider 중에서 워크로드에 맞게 고른다. 

**AWS 서비스와 밀착 통합** : 컨트롤 플레인은 AWS가 운영하고 무료이며 ALB, ECR, IAM, CloudWatch, Secrets Manager와 기본 기능으로 연결된다.

# 장단점
## 장점
**운영 단순** : 컨트롤 플레인은 AWS가 관리하고 무료이며 Fargate를 쓰면 서버 패치나 노드 스케일링도 신경 쓸 필요가 없다. K8s에 비해 배워야 할 개념이 적어서 빠르게 도입할 수 있다.

**AWS 서비스와 밀착 통합** : ALB 자동 등록, IAM Task Role, CloudWatch 로그, Secrets Manager 주입, ECR 연동이 기본 기능이라 별도 애드온 설치 없이 바로 붙는다.

**컨트롤 플레인 비용 없음** : 클러스터 관리 비용이 없고(EKS는 클러스터당 과금) 실제 쓴 Task 자원에만 과금된다. Fargate Spot이나 Graviton으로 추가 절감도 가능하다. 다만 Fargate 자체 단가는 높은 편이다.

## 단점
**AWS에 종속됨** : ECS 지식과 Task Definition은 다른 클라우드나 온프레미스에서 못 쓴다. 나중에 멀티클라우드나 이전을 하게 되면 전부 다시 짜야한다.

**생태계와 확장성이 K8s보다 약함** : Helm, ArgoCD 같은 방대한 생태계가 없고 StatefulSet이나 Ingress 같은 개념도 약하거나 없다. 복잡한 워크로드로 갈수록 한계가 드러난다.

**Fargate는 제어권이 낮고 단가가 높음** : 호스트 접근이 안되고 GPU나 특수 설정에 제약이 있다. 자원을 꽉 채워 쓰는 대규모 환경에서는 EC2보다 비용이 더 나올 수 있다.