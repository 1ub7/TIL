# Docker Swarm
**Docker Swarm이란?** Docker에 기본으로 내장되어 있는 컨테이너 오케스트레이션 도구이다. 즉, **여러 대의 서버(호스트)를 하나의 클러스터로 묶어서 컨테이너를 자동으로 분산 배포하고 관리해주는 도구**이다.

```powershell
docker run -d nginx
```

위에 명령어를 친다면 컴퓨터 안에만 컨테이너가 뜬다. Swarm은 이걸 여러 대의 서버로 확장해서 마치 한 대의 큰 서버를 다루듯이 여러 서버에 컨테이너를 분산시켜준다.

# Docker Swarm이 왜 필요한 이유
**트래픽 분산이 필요해서 :** 서버 한 대의 자원(CPU, 메모리)만으로는 늘어나는 트래픽을 감당하기 어렵다. 여러 서버에 컨테이너를 나눠 띄우고 요청도 분산시켜야 한다.

**장애가 나도 서비스가 죽지 않아야 해서 :** 컨테이너가 서버 한 대에만 떠있으면 그 서버가 장애 나는 순간 서비스 전체가 멈춘다. 여러 서버에 분산되어 있으면 한 대가 죽어도 나머지가 서비스를 이어갈 수 있다.

**수동 배포는 서버가 늘어날수록 감당이 안 돼서 :** 서버가 3대, 5대, 10대로 늘어날 때마다 하나하나 접속해서 배포하는 건 비효율적이다. "이 이미지로 컨테이너 5개를 띄워라"라고 한 번만 명령하면, 나머지는 자동으로 처리해주는 도구가 필요하다.

# 핵심 구성 요소
## Node

**Node란?** Swarm 클러스터에 참여하는 서버 한 대를 말한다. 물리 서버든 가상 머신이든 상관없다. 도커 엔진이 설치되어 있고 `docker swarm join` 으로 클러스터에 합류한 컴퓨터 하나하나가 전부 Node이다.

`docker node ls` 를 매니저 노드에서 실행하면 클러스터에 속한 모든 노드 목록을 볼 수 있다.

```powershell
# 출력 예시
ID                HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS
abc123 *          node1      Ready     Active         Leader
def456            node2      Ready     Active         
ghi789            node3      Ready     Active         Reachable
```

여기서 `*` 가 붙은 노드가 **지금 명령어를 실행하고 있는 노드**다. 하나의 Node는 Manager 역할과 Worker 역할 중 하나를 맡는데 경우에 따라 **한 노드가 두 역할을 동시에 가질 수도 있다.**

## Manager Node
**Manager Node란?** 클러스터 전체를 관리하고 컨테이너를 어느 노드에 배치할지 결정하는 노드이다. Swarm의 두뇌 역할인 셈이다.

```powershell
docker swarm init --advertise-addr <매니저의 IP>
```

이 명령어를 실행한 노드가 첫 번째의 Manager가 된다. 이 명령을 치면 아래처럼 Worker가 합류할 때 쓸 토큰이 나온다.

```powershell
docker swarm join --token SWMTKN-1-xxxxx <매니저IP>:2377
```

**여러 개의 Manager를 둘 수도 있다.** 

그 이유는 **고가용성** 때문이다. Manager가 딱 1개뿐이면 그 Manager가 죽는 순간 클러스터 전체를 관리할 사람이 없어져서 큰일난다. 그래서 보통 Manager를 **3개, 5개처럼 홀수 개**를 둔다.

**왜 굳이 홀수여야할까?** Manager들끼리는 **Raft 합의 알고리즘**으로 누가 진짜 리더(Leader)인지 투표해서 정하는데 **짝수면 동점 상황이 나올 수 있어서 홀수로 구성하는 게 안전**하다.

## Worker Node
**Worker Node란?** Manager의 지시를 받아서 실제로 컨테이너(Task)를 실행만 하는 노드이다. 스스로 판단해서 뭘 하지 않고 오직 Manager가 시키는 대로 컨테이너를 띄우고 상태를 보고한다.

```powershell
docker swarm join --token <워커용 토큰> <매니저IP>:2377
```

**기본적으로 Worker는 관리 권한이 없다.** 즉, Worker 노드에서는 `docker service create` 같은 클러스터 **관리 명령어를 실행할 수 없다**. 오직 컨테이너를 실행하는 **실무자 역할**만 한다.

## Service
**Service란?** “이 이미지로, 이런 설정으로, 몇 개의 컨테이너를 클러스터에 띄워라”라는 선언적 정의이다. Service 자체는 컨테이너가 아니라 **원하는 상태(Desired state)에 대한 명세**라고 생각하면 된다.

```powershell
docker service create --name web --replicas 3 -p 80:80 nginx
```

이 명령을 실행하면 **Manager가 적절한 Worker Node들에 자동으로 배치**한다.

Service의 핵심은 상태를 계속 유지해준다는 점이다. 만약 컨테이너 하나가 죽으면 Swarm은 그걸 감지해 자동으로 새 컨테이너를 다른 노드에 띄워서 다시 맞춰준다. → **Self-healing**이다.

```powershell
docker service scale web=5
```

이렇게 하면 replicas 개수를 5개로 늘릴 수도 있다. Service 하나로 컨테이너 개수 조절, 이미지 업데이트, 롤백까지 전부 관리 가능하다.

## Task
**Task란?** Service가 실제로 각 노드에 배치한 컨테이너 하나하나의 실행 단위이다. Service가 3개 띄우라고 선언하면 **실제로 생성되는 개별 컨테이너가 3개가 각각 하나의 Task**이다.

```powershell
docker service ps web
```

이 명령어로 `web` 이라는 Service에 속한 Task들을 확인할 수 있다.

```powershell
ID          NAME     IMAGE   NODE    DESIRED STATE   CURRENT STATE
task1       web.1    nginx   node1   Running         Running 2 minutes ago
task2       web.2    nginx   node2   Running         Running 2 minutes ago
task3       web.3    nginx   node3   Running         Running 2 minutes ago
```

각 Task는 정확히 하나의 컨테이너에 대응되고, 한번 특정 노드에 배치되면 그 노드에서 실행된다. 만약 `node2` 가 죽으면 `task2` 는 **Failed 상태**가 되고 **Manager는 새로운 Task를 다른 살아있는 노드에 생성해서 replicas 수를 다시 맞춘다.**