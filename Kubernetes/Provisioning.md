# Provisioning
**프로비저닝(Provisioning)이란?** 어떠한 지식이나 자원 등을 미리 준비하고 요청이 들어왔을 때, 해당 요청에 맞게 **제공**하는 것을 의미한다. 즉, **사용자 혹은 비지니스 요구사항에 맞게 할당, 배치, 배포하여 시스템을 사용가능하도록 준비하는 절차**를 뜻한다.

# Provisioning의 종류
**스토리지 프로비저닝 :** 필요한 저장 공간을 요청받아 실제 디스크 볼륨을 할당해주는 과정이다. 쿠버네티스에서는 PVC 요청이 들어오면 StorageClass가 자동으로 PV를 만들어주는 동적 프로비저닝이 핵심 개념이다.

**서버 프로비저닝 :** 물리 서버나 VM을 새로 만들고 OS, 미들웨어까지 설치해서 실제 서비스를 올릴 수 있는 상태로 준비하는 과정이다. 온프레미스 k3s 서버를 새로 구축할 때 딱 이 과정을 거치게 된다.

**클라우드 프로비저닝 :** AWS/GCP 같은 클라우드에서 컴퓨팅·네트워크·스토리지 자원을 API 호출만으로 즉시 할당/회수하는 과정이다. EKS 클러스터를 새로 만들거나 오토스케일링으로 노드를 늘리는 것도 여기 해당한다.

**네트워크 프로비저닝 :** 장비나 서비스에 IP, 라우팅, 보안 규칙 등을 설정해서 통신 가능하게 만드는 과정이다. 쿠버네티스의 Service, Ingress가 트래픽을 라우팅할 수 있게 되는 것도 결국 이 네트워크 프로비저닝의 연장선이다.

# Provisioning 방식
## Static Provisioning
**정적 프로비저닝(Static Provisioning)이란?** 관리자가 미리 PV를 만들어두고 PVC 요청이 들어오면 조건에 맞는 걸 찾아서 매칭해주는 방식이다.

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

관리자가 스토리지 용량, 접근 모드 등을 미리 정해서 PV를 등록해두고 PVC가 생성되면 **Kubernetes가 조건에 맞는 PV를 찾아 자동으로 바인딩**한다. 어떤 스토리지가 어떻게 쓰이는지 통제하기 쉽지만 매번 관리자가 수동으로 PV를 만들어둬야 해서 번거롭고 수요를 못 맞추면 자원이 남거나 부족해진다.

## Dynamic Provisioning
**동적 프로비저닝(Dynamic Provisioning)이란?** PVC가 생성되는 시점에 자동으로 PV가 만들어지는 방식이다.

```yaml
# 예시
apiVersion: v1
kind: PersistenVolumeClaim
metadata: 
 name: my-pvc
spec:
 accessModes:
  - ReadWriteOnce
 storageClassName: standard
  resources:
   requests:
    storage: 10Gi
```

관리자가 PV를 미리 만들어둘 필요 없이 **PVC 요청만 하면 StorageClass가 알아서 PV를 즉석에서 생성**한다. 관리자 개입 없이 즉시 자원 확보가 가능하고 확장성이 뛰어나지만 자동 생성이다 보니 어떤 스토리지가 얼마나 쓰이는지 **세밀한 통제는 상대적으로 어렵다.**