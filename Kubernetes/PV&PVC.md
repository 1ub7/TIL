# PV
**PV(Persistent Volume)란?** 데이터를 저장할 볼륨이다. 볼륨을 생성하고 이를 클러스터에 등록한 것. 쿠버네티스 클러스터 외부 스토리지와 연결을 담당하는 리소스이기도 하다. 

관리자가 프로비저닝하거나 스토리지 클래스를 사용하여 동적으로 프로비저닝한 클러스터의 스토리지이다. 사용자 및 관리자에게 스토리지 사용 방법에서부터 **스토리지가 제공되는 방법에 대한 세부 사항을 추상화하는 API를 제공**한다. 컨트롤러 및 파드와 별개의 쿠버네티스 리소스이며, 파드의 생명주기와 별개로 작동한다.

```yaml
# 예시
apiVersion: v1
kind: PersistentVolume
metadata:
 name: my-pv
spec:
 capacity:
  storage: 10Gi
 accessModes:
  - ReadWriteOnce
 hostPath:
  path: /data/my-pv
```

# PVC
**PVC(Persistent Volume Claim)란?** 필요한 저장 공간/RW모드 등 요청사항을 기술한 명세로서 **PV에 전달하는 요청**이다. PV와 바인딩을 하는 목적으로 사용한다.

```yaml
# 예시
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: my-pvc
spec:
 accessModes:
  - ReadWriteOnce
 resources:
  requests:
   storage: 10Gi
```

PVC가 생성되면 **Kubernetes는 조건에 맞는 PV를 찾아서 자동으로 연결**한다. 이후 Pod는 PV를 직접 참조하지 않고 **PVC를 통해서 스토리지에 접근**한다.

```yaml
# 예시
apiVersion: v1
kind: Pod
spec:
 containers:
  - name: app
    volumeMounts:
    - mountPath: "/data"
      name: storage
 volumes:
  - name: storage
    persistentVolumeClaim:
     claimName: my-pvc
```

# accessModes 종류
| **모드** | **의미** |
| --- | --- |
| **ReadWriteOnce (RWO)** | 하나의 노드에서만 읽기/쓰기 가능 |
| **ReadOnlyMany (ROX)** | 여러 노드에서 읽기만 가능 |
| **ReadWriteMany (RWX)** | 여러 노드에서 읽기/쓰기 모두 가능 (NFS 등에서 지원) |

# 왜 필요할까?
컨테이너는 기본적으로 **일시적**이다. **Pod가 죽었다 다시 뜨면 그 안의 데이터도 함께 사라진다.** 데이터베이스처럼 데이터가 영구적으로 유지되어야 하는 애플리케이션을 다루려면 **별도의 저장 메커니즘이 필요한데 그게 PV/PVC**이다.

# PV/PVC의 생명주기
PV의 4가지 상태

| **상태** | **의미** |
| --- | --- |
| **Available** | 아직 어떤 PVC와도 바인딩되지 않은 사용 가능한 상태 |
| **Bound** | PVC와 바인딩되어 실제로 사용 중인 상태 |
| **Released** | PVC가 삭제됐지만 PV 자체는 아직 회수되지 않은 상태. 이전 데이터가 남아있을 수 있음 |
| **Failed** | 자동 회수 과정에서 오류가 발생한 상태 |

**Available** : 관리자가 PV를 만들어놓거나 StorageClass가 동적으로 생성한 직후의 상태이다. 아직 어떤 PVC도 이 PV를 점유하지 않은 **대기 중 상태**이다.

**Bound** : PVC가 생성되면 Kubernetes는 조건이 맞는 Available 상태의 PV를 찾아 자동으로 연결한다. 이 순간 **PV와 PVC 둘 다 상태가 Bound**로 바뀌고 **Pod는 이 PVC를 통해 실제 스토리지에 접근할 수 있게 된다.**

**Released** : 사용이 끝나서 PVC를 삭제하면 연결되어 있던 **PV는 바로 사라지지 않고 Released 상태**가 된다. 이 상태에서는 다른 PVC가 이 PV를 재사용할 수 없다. PV 안에는 이전 데이터가 아직 남아있는 경우가 많기 때문에 **안전을 위해 자동으로 재활용되지 않고 일단 묶어두는 것**이다.
(Released 이후엔 **Reclaim Policy**에 따라 처리하게 됨)

**Failed** : Released 상태에서 자동 회수 작업을 시도했는데 뭔가 실패했을 때 이 상태가 된다. 흔한 경우는 아니고 스토리지 백엔드 쪽 문제가 있을 때 발생한다.

# Reclaim Policy(회수 정책)
Released 상태의 PV를 어떻게 처리할지는 PV 생성 시 지정한 **persistentVolumeReclaimPolicy**에 따라 갈린다.

| **정책** | **Released 이후 동작** |
| --- | --- |
| Retain | PV와 데이터를 그대로 보존. 관리자가 수동으로 데이터 정리 후 PV를 삭제하거나 재사용 가능하게 만들어야 함 |
| Delete | PVC 삭제 시 PV와 실제 스토리지까지 자동으로 삭제됨. 동적 프로비저닝의 기본값 |
| Recycle | 데이터를 지우고 PV를 재사용 가능한 Available 상태로 되돌림 |

## 정적/동적 PV의 기본 Reclaim Policy 차이
**정적 프로비저닝 PV의 기본값은 Retain**이다.
**왜 Retain일까?** 관리자가 직접 만든 PV는 보통 의도적으로 준비해둔 중요한 스토리지인 경우가 많기 때문에 PVC가 삭제되더라도 데이터를 함부로 지우지 않고 일단 보존하는 쪽을 안전한 기본값으로 잡는 것이다.

**동적 프로비저닝 PV의 기본값은 Delete**이다.
**왜 Delete일까?** 동적으로 생성된 스토리지는 PVC와 생명주기를 함께하는 일회성 자원으로 취급되는 경우가 많기 때문이다. 클라우드 환경에서 PVC가 늘어날 때마다 자동으로 디스크가 계속 쌓이는 걸 막기 위한 설계이기도 하다.

# StorageClass
**StorageClass란?** 동적 프로비저닝을 가능하게 해주는 오브젝트이다. 즉, **이런 종류의 스토리지를 만들 때는 이런 방식으로 만들어라**라는 템플릿같은 개념인 것이다. 

**이게 왜 필요한가?** 정적 프로비저닝에서는 관리자가 PV를 하나하나 미리 만들어둬야 했다. 그런데 클러스터 규모가 커지면 요청이 들어올 때마다 관리자가 매번 PV를 수동으로 만드는 건 너무 비효율적이다. 그래서 **이 과정을 자동화해주는 게 StorageClass**이다. PVC가 특정 StorageClass를 지정해서 요청하면 그 StorageClass에 **정의된 방식대로 PV가 즉석에서 자동 생성**된다.

```yaml
# 예시
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
 name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
 type: gp3
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

`provisioner: kubernetes.io/aws-ebs` 
**provisioner이란?** 실제로 스토리지를 만들어주는 주체이다. 

`volumeBindingMode: WaitForFirstConsumer` 
**volumeBindingMode란?** PV를 언제 만들지 결정한다. **Immediate**는 PVC 생성 즉시 PV를 만들고, WaitForFirstConsumer는 Pod가 실제로 스케줄링될 때까지 PV 생성을 미룬다. Pod가 어느 노드에 뜰지 모르는 상태에서 미리 PV를 만들면 노드/영역이 안 맞을 수 있어서 **WaitForFirstConsumer**가 더 안전하고 널리 쓰인다.