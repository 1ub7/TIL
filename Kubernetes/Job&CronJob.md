kubernetes에서 파드는 일반적으로 지속적으로 수행되도록하는 목적을 가지고 있다. 하지만 로그 데이터 처리나 데이터 백업같은 데이터 처리 작업이나 외부 API를 통해 데이터 수집이나 동기화 작업과 같은 배치 작업들같이 일회성을 가지고 수행되는 작업들도 존재한다. 이러한 작업들도 Job과 CronJob을 사용해 Kubernetes 클러스터 내에서 실행될 수 있다.

# Job

**Job이란?** 지속적으로 실행되는 서비스가 특정 동작을 수행하고 종료하는 작업을 정의하기 위한 리소스이다. Pod를 통해서 작업을 수행하고 지정된 작업이 성공적으로 종료될때까지 수행을 반복한다. 내부적으로 Pod를 생성하여 작업을 수행하게 되는데 **Pod의 상태가 Running이 아닌 Completed가 되는 것이 최종상태이다.** 실패 시 재시작, 작업 수행 횟수 지정, 동시 실행 작업 수 지정 등의 세부 옵션도 제공한다.

즉, 일회성 작업을 수행하는 리소스이다.

```yaml
# 예시
apiVersion: batch/v1
kind: Job
metadata: 
 name: my-job
spec: 
 completions: 5
 parallelism: 2
 backoffLimit: 4
 template: 
  spec:
   containers: 
    - name: worker
      image: busybox
      command: ["echo", "hello"]
   restartPolicy: Never
   
   # echo hello를 실행하는 Pod를 동시에 최대 2개씩 띄워서
   # 총 5번 성공할 때까지 실행하고 실패하면 최대 4번까지 재시도한다.
```

## Job의 주요 특징

**완료 보장 :** Pod가 실패하면 성공할 때까지 **자동으로 재시도**한다. Deployment처럼 계속 떠있는 게 목적이 아니라 **한 번 실행해서 끝까지 완료시키는 것**이 목표다.

**병렬 처리 지원 :** `completions`로 총 몇 번 성공해야 완료로 볼지, `parallelism`으로 동시에 몇 개 Pod를 띄울지 지정할 수 있다.

**restartPolicy 제한 :** Job은 언젠가 끝나야 하는 작업이라 `Always`는 허용되지 않고 `Never` 또는 `OnFailure`만 사용 가능하다.

# CronJob

**CronJob이란?** 주기적으로 특정 동작을 수행하고 종료하는 작업을 정의하기 위한 리소스이다. **리눅스의 크론 스케줄링 방법을 그대로 사용**한다. 그래서 생성과 동시에 실행되지 않고 Cron식에 맞게 실행된다. 내부적으로 Job을 생성해 작업을 수행한다. 그리고 Job은 내부적으로 Pod를 만든다. **주기적으로 데이터를 백업하거나 데이터 점검 및 알림 전송 등의 목적으로 사용한다.** 주기적인 배치 작업이나 일정을 가진 작업을 처리하는 데 매우 유용하다.

즉, Job을 Cron 식에 맞게 주기적으로 수행할 수 있도록 하는 리소스이다.

```yaml
# 예시
apiVersion: batch/v1
kind: CronJob
metadata:
 name: my-cronjob
spec: 
 schedule: "0 2 * * *"
 jobTemplate:
  spec:
   template:
    spec:
     containers:
      - name: backup
        image: my-backup-image
     restartPolicy: OnFailure
     
# 매일 새벽 2시에 백업 작업용 Job을 자동으로 하나 만들고 그 Job이 
# my-backup-image로 Pod를 띄워서 실행함. 실패하면 재시작
```

## CronJob의 주요 특징

**Cron 표현식으로 스케줄 관리 : 리눅스 cron과 동일한 문법**(`분 시 일 월 요일`)으로 반복 주기를 지정한다. 예시로 `"0 2 * * *"`는 매일 새벽 2시 실행인 것이다.

**실행 시점마다 Job을 자동 생성 :** CronJob이 직접 Pod를 만드는 게 아니라 **스케줄이 될 때마다 Job을 새로 만들고 그 Job이 Pod를 생성**한다. 즉 CronJob은 **Job의 생성기 역할**이다.

**이력 관리 및 동시 실행 정책 :** 과거 실행 기록을 몇 개까지 남길지, 이전 실행이 끝나기 전에 다음 스케줄이 오면 어떻게 할지를 설정할 수 있다.