
# 4장~이후부터 어디까지 갈수 있을까?
1. 2부는 쿠버네티스의 객체에 대해서 알아본다. 
2. 실제 쿠버네티스를 쓸때는 pod를 한땀 한땀 실행하지는 않는다.
   - 4장은 포드를 실행시키는 객체에 대해서 알아보는 장이다.
   - 5장은 이러한 객체를 외부에서 사용할수 있게 해주는 서비스 객체에 대해서
   - 6장은 외부 저장소를 어떻게 쓸지에 대해서
   - 7장은 (외부,내부)설정데이터를 어떻게 할지에 대해서
   - 8장 인증에 대한내용같고, 9,10장은 pod실행시키는 다른방법에 대한 내용같다.

*3부는 아키텍쳐의 영역같다. 전체를 보는 시각을 가지도록 한다.
> 각 쿠버네티스 객체를 어떻게 쓸것인가를 다루는 듯?
## 2부
:memo: Table of Contents
- 4장 : 포드배포
    - 자동 실행
        - 레플리케이션컨트롤러(rc) : 지는 해
        - 디플로이먼트
          - 레플리카셋: 뜨는 해
        - 스테이트풀셋
    - 병렬 실행
        - 데몬셋
    - 1번 실행
        - 배치(Job)
          - pod를 여러개 만들어서 실행가능, 그중에서 병렬로 실행 가능. 
          - 중간에 병렬로 처리를 늘릴수도 있다.
          - 허용시간을 제한하도록 하자.
    - 주기적 실행
        - CronJob
- 5장 : 외부노출
    - 서비스
- 6장 : 볼륨
    - 외부디스크 사용 : mount개념?
    - git Repository
        - git으로 초기화
        - 사이드카 컨테이너
- 7장 : 설정데이터 
    - 환경변수
    - git Repository
    - 시크릿 : 지는해
    - ConfigMap : 뜨는 해
- 8장 : 인증 or 권한
    - 접근 권한/인증
    - 방화벽
    - 앰배서더 컨테이너
- 9장 : 디플로이먼트 
    - 롤링 : pod 수정(업데이트) 방법(? 전략)
    - 롤백, 롤아웃
- 10장 : 스테이터스풀

라이브러리
* :bulb: 라이브니스 프로브 : 컨테이너 살아있니?
  - HTTP GET 프로브
  - TCP 소켓 프로브
  - Exec 프로브
* :bulb: 레디네스 프로브

> 살아있는지 확인은 UDP를 사용하지 않나?

* 디플로이먼트 : 자동실행의 상위버전, 디플로이먼트를하면 자동실행 객체를 따로 생성할 필요없다. 
> 그말은 디플로이먼트는 자동실행위에서만 돌아간다는 말과 같다. 조금더 전략이 틀리다고 하는데.. 실행의 한 형태라고 생각하면 될것 같다.

---
4장에 주로 설명하는 레플리케이션컨트롤러와 레플리카셋은 일정의 pod를 유지시켜주는 객체이다. pod이 살아있는지 확인하는 것이 라이브니스 프로브이다.
- 그래서 레플리케이션컨트롤러와 레플라카셋에는 라이브니스 프로브가 설정되어있다.
> ㅇㅇ? 그런내용이 없는데? 꼭 라이브니스 프로브 대체제가 rc와 레플리카셋같다?

> 응? 리플리카 한것과 데몬셋이 중복되면 어떻게 되는거지?
> yml이든 yaml이든 상관없나보다.

### 라이브 프로브 실습

kubia-liveness-probe.yml
```yml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-liveness
spec:
  containers:
  - image: luksa/kubia-unhealthy
    name: kubia
    livenessProbe:
      httpGet:
        path: /
        port: 8080
```
```bash
$kubectl create -f kubia-liveness-probe.yml
$kubectl get po kubia-liveness
$kubectl describe po kubia-liveness #오류 확인
```
```yml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-liveness
spec:
  containers:
  - image: luksa/kubia-unhealthy
    name: kubia
    livenessProbe:
      httpGet:
        path: /
        port: 8080
      initialDelaySeconds: 15 #딜레이를 안넣으면 필히 오류가 발생한다.  
```
```bash
$kubectl delete po kubia-liveness
$kubectl create -f kubia-liveness-probe.yml
```

### rc 실습

kubia-rc.yaml
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
        - name: kubia
          image: luksa/kubia
          ports:
          - containerPort: 8080
```

```bash
$kubectl delete rc kubia
$kubectl create -f kubia-rc.yaml
$kubectl get pods
$kubectl delete po kubia-v8znk
$kubectl get pods
$kubectl get rc
$kubectl describe rc kubia #오류 확인
$kubectl scale rc kubia --replicas=10 # 스케일업
$kubectl edit rc kubia #실행중인 rc 변경, 설정파일변경
$kubectl delete rc kubia --cascade=false #pod를 삭제하지 않고 rc지우기
```
* 일반적으로 rc를 지우면 그에 포함된 pod도 삭제된다.
    - 근데 의미가 있나? 관리가 안되자나?
    - 라벨 셀렉터로 다시 관리할수 있다고 한다.


> 컨테이너에서 자바 애플리케이션을 실행하는 경우에는 Exec 프로브 대신 HTTP GET 라이브니스 프로브를 사용해야 한다.       

### 레플리카셋 실습 - 진짜는 9장에서 진행
일반적으로 레플리카셋은 직접 생성하지 않는다. 그 대신 상위 수준의 디플로이먼트 리소스를 만들때 자동으로 생성한다. 
여기서는 그래도 레플리카셋이 어떤지 알아두기 위해서 억지 세트를 만들어서 진행해보겠다.

* 레플리카셋이 나아진점은 라벨 셀렉터의 표현식이 더 좋아졌다는 것이다.

> 실습을 하고 싶은데... v1의 버전을 몰로해야할지 몰라서 계속 오류 발생한다. 우선써놓기는 하자.

kubia-replicaset.yaml
```yaml
apiVersion: apps/v1beta2
kind: ReplicaSet #책에는 대문자로 안되어 있다.
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:      # 책에는 metadata안으로 되어있다.
      containers:
      - name: kubia
        image: luksa/kubia
```

```bash
$kubectl get rs
$kubectl describe rs
```

### 데몬셋 실습

ssd-monitor-daemonset.yaml
```yaml
# VI에서 자꾸 띄워쓰기가 멀찍이 되어서 정리안하고 해보았다.
apiVersion: apps/v1beta2
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
        selector:
                matchLabels:
                        app: ssd-monitor
        template:
                metadata:
                        labels:
                                app: ssd-monitor
                spec:
                        nodeSelector:
                                disk: ssd
                        containers:
                        - name: main
                          image: luksa/ssd-monitor
```

```bash
$kubectl get ds
$kubectl get node
# 결과 시작
NAME                                                STATUS   ROLES    AGE   VERSION
gke-standard-cluster-1-default-pool-3cc53fe8-5zp7   Ready    <none>   25h   v1.13.11-gke.14
gke-standard-cluster-1-default-pool-3cc53fe8-jr0f   Ready    <none>   25h   v1.13.11-gke.14
gke-standard-cluster-1-default-pool-3cc53fe8-pqtb   Ready    <none>   25h   v1.13.11-gke.14
# 결과 완료, 노드에 라벨 설정하해야한다.
$kubectl label node gke-standard-cluster-1-default-pool-3cc53fe8-5zp7 disk=ssd
```

### 배치(Job) 실습

export.yaml
```yaml
apiVersion: batch/v1
kind: Job
metadata:
        name: batch-job
spec:
        template:
                metadata:
                        labels:
                                app: batch-job
                spec:
                        restartPolicy: OnFailure
                        containers:
                                - name: main
                                  image: luksa/batch-job
```

``` bash
$kubectl create -f export.yaml
$kubectl get jobs
$kubectl get po
$kubectl get po -a
$kubectl logs batch-job-2sr49
$kubectl scale job [잡명칭] --replicas 3 #병렬처리로 변경(한번에 처리하는 수 정의)
```

### CronJob 실습

cronjob.yaml
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
        name: batch-job-every-fifteen-minutes
spec:
        schedule: "0,15,30,25 ***" #매이 작업은 매일 매시간 0,15,305분에 일한다는
        jobTemplate:
                spec:
                        template:
                                metadata:
                                        labels:
                                                app: periodic-batch-job
                                spec:
                                        restartPolicy: OnFailure
                                        containers:
                                                -name: main
                                                -image: luksa/batch-bjo
```

``` yaml
apiVersion: v1
kind: Service
metadata:
        name: kubia
spec:
        ports:
                - port: 80
                  targetPort: 8080
        selector:
                app: kubia
```                