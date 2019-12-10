# 쿠버네티스 서비스

포드의 경우에 지정되는 IP가 랜덤하게 지정이 되고 리스타트 때마다 변하기 때문에 고정된 엔드포인트로 호출이 어렵고, 또한 여러 포드에 같은 애플리케이션을 운용할 경우 포드간의 로드밸런싱을 지원하기위해 서비스를 사용한다. 서비스는 지정된 IP로 생성이 가능하고 여러 포드를 묶어서 로드밸런싱이 가능하며 고유한 DNS 이름을 가질 수 있다.

## 1. 서비스 생성

서비스를 생성하는 방식은 두가지가 있다.

- 첫번째. kubectl expose 명령어를 통한 서비스 생성 

    ``` 
    $ kubectl expose deployment kubia --type=NodePort
    ```
- 두번째. YAML 디스크립터를 통한 서비스 생성 

    라벨 셀렉터를 이용하여 관리하고자하는 포드들을 정의할 수 있다.

    ```
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

## 2. 서비스 검색

## 3. 서비스 타입

서비스의 5가지 유형은 다음과 같다.

- ### **ClusterIP(기본값)**
    
    디폴트 설정으로, 서비스에 클러스터 IP (내부 IP)를 할당한다. 쿠버네티스 클러스터 내에서는 이 서비스에 접근이 가능하지만, 클러스터 외부에서는 외부 IP 를 할당  받지 못했기 때문에, 접근이 불가능하다. 

    ```
    apiVersion: v1
    kind: Service
    metadata:
        name: kubia
    spec:
        sessionAffinity: ClientIP
        ports:
        - port: 80
            targetPort: 8080
        selector:
            app: kubia       
    ```

- ### **NodePort**
        
    서비스 내부 클러스터 IP를 통해 액세스 되고 노드의 IP와 예약된 포트를 통해서도 액세스 된다. NodePort는 외부 클라이언트에서 접근 하기위해서는 방화벽 규칙을 외부에서 액세스 가능하도록 설정 해야 가능하다.

    ```
    apiVersion: v1
    kind: Service
    metadata:
        name: kubia-nodeport
    spec:
        type: NodePort
        ports:
        - port: 80
            targetPort: 8080
            nodePort: 30123
        selector:
            app: kubia       
    ```

- ### **LoadBalancer**
        
    로드밸런서는 자신만의 고유하면서 외부에서 액세스 가 가능한 IP주소를 가지고 모든 연결을 서비스로 리다이렉트한다. ( 클라이언트가 네트워크 부하 분산기의 IP 주소로 요청을 전송 )
    ```
    apiVersion: v1
    kind: Service
    metadata:
        name: kubia-loadbalancer
    spec:
        type: LoadBalancer
        ports:
        - port: 80
            targetPort: 8080
        selector:
            app: kubia
    ```

- ### **ExternalName**
    
    외부 DNS 이름에 대한 내부 별칭을 제공. 내부 클라이언트는 내부 DNS 이름을 사용하여 요청을 수행하고, 요청이 외부 이름으로 리디렉션된다.
    ```
    apiVersion: v1
    kind: Service
    metadata:
        name: external-service
    spec:
        type: ExternalName
        externalName: api.somecompany.com
        ports:
        - port: 80
    ```

- ### **Headless**
    
    기본적으로 서비스 연결은 임의로 선택된 하나의 포드로 전달 된다. 그러나 클라이언트를 모든 포드에 연결해야 할 때 혹은 포드 자체를 모든 포드에 연결해야 할때 헤드리스 서비스를 사용할 수 있다. 따라서 Headless 서비스는 Pod 그룹화가 필요하지만 안정적인 IP 주소가 필요하지 않은 경우에 사용할 수 있다. ( ? )
    ```
    apiVersion: v1
    kind: Service
    metadata:
        name: kubia-headless
    spec:
        clusterIP: None
        ports:
        - port: 80
            targetPort: 8080
        selector:
            app: kubia
    ```

- #### sessionAffinity: ClientIP 설정

    서비스가 포드들에 부하를 분살할때 디폴트는 포드간에 랜덤으로 부하를 분산하도록 한다. 특정 클라이언트가 특정 포드에 지속적으로 연결 되게 하려면 **sessionAffinity: ClientIP** 값을 설정하면 된다.

## 4. 인그레스 리소스

인그레스는 네트워크 스택(HTTP)의 애플리케이션 레이어에서 동작하고, 서비스가 할 수 없는 쿠키 기반 세션 친화성 기능을 제공한다.

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubia
spec:
  rules:
  - host: kubia.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kubia-nodeport
          servicePort: 80
```
    
## 5. 레디네스 프로브

레디네스 프로브는 클라이언트가 정상 상태인 포드하고만 통신하게 하고 시스템에 문제가 있다는 것을 알아차리지 못하게 한다.

```
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
        - name: http
        containerPort: 8080
        readinessProbe:
        exec:
            command:
            - ls
            - /var/ready
```

라이브니스 프로브처럼 레디네스 프로브에도 세 가지 유형이 있다.

- 첫번째. 프로세스를 실행시키는 exec 프로브
- 두번째. HTTP GET 요청을 컨테이너에게 보내고 응답의 HTTP 상태코드를 통해 상태를 확인하는 HTTP GET 프로브
- 세번째. 컨테이너의 지정된 포트로 TCP를 연결하는 TCP 소캣 프로브

---
## 쿠버네티스 용어

#### 노드(Node)
클러스터의 일부이며, 쿠버네티스에 속한 워커 머신. 

#### 클러스터(Cluster)
쿠버네티스에서 관리되는 컨테이너화 된 애플리케이션을 실행하는 노드 집합.

#### 에지 라우터(Edge router)
클러스터에 방화벽 정책을 적용하는 라우터. 이것은 클라우드 공급자 또는 물리적 하드웨어의 일부에서 관리하는 게이트웨이일 수 있다.

#### 클러스터 네트워크(Cluster network)
쿠버네티스 네트워킹 모델에 따라 클러스터 내부에서 통신을 용이하게 하는 논리적 또는 물리적 링크 집합.

#### 서비스(Service)
레이블 셀렉터를 사용해서 파드 집합을 식별하는 쿠버네티스 서비스(Service).

서비스의 5가지 유형은 다음과 같습니다.

---

### 참고사이트

조대협의블로그
- [쿠버네티스#7 서비스](https://bcho.tistory.com/1262)
- [쿠버네티스#8 인그레스](https://bcho.tistory.com/1263)

구글클라우드
- [서비스](https://cloud.google.com/kubernetes-engine/docs/concepts/service?hl=ko)
- [인그레스](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress?hl=ko)

쿠버네티스
- [인그레스](https://kubernetes.io/ko/docs/concepts/services-networking/ingress/#%ec%9d%b8%ea%b7%b8%eb%a0%88%ec%8a%a4%eb%9e%80)
