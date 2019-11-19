
  <a href="https://github.com/OSD-STUDY/cloud/edit/master/pod.md">
    <img alt="Documentation" src="https://img.shields.io/badge/documentation-yes-brightgreen.svg" target="_blank" />
  </a>

# [POD](https://kubernetes.io/ko/docs/concepts/workloads/pods/pod-overview/)

 :bulb: 파드는 쿠버네티스 애플리케이션의 기본 실행 단위이다. 쿠버네티스 객체 모델 중 만들고 배포할 수 있는 가장 작고 간단한 단위이다.

## :memo: 목차
- [준비 작업](#준비-작업)
  - [컨테이너 만들기](#Container-만들기)(NodeJs)
    - [package.json](#package.json)
    - [server.js](#server.js)
    - [Dockerfile](#Dockerfile)
    - 도커허브에 올리기 : docker push
  - [GKE 엔진만들기](https://console.cloud.google.com/)
    - 아래 참조
- [포드만들기](#포드-만들기)
  - [pod만들어보기](#일단-포드-만들기)
    - [설정파일](#설정파일)
    - [실행](#pod-만들기)
  - [나의 도커로 pod만들어보기](#나의-docker로-포드-만들기)(도커허브)
  - [포트포워딩](#포드-포트포워딩-하기)
- 레이블과 셀렉터
  - [레이블 있는 포드 생성](#레이블-있는-포드-생성)
  - [레이블 조회와 수정](#레이블-조회와-수정)

> GKE(구글 쿠버네티스 엔진) 만들기
>> 사이트 들어가서 `컴퓨팅->Kubernetes Engin -> 클러스터`에서 클러스터 만들기후에 `연결`클릭, 아래 스크립스를 사용

* 사용한 다음에는 지우도록 한다. 돈 나간다.
* 모바일과 패드에서는 사용성이 좋지 않다. 대신 패드에서는 앱을 다운로드 받아서 쓰도록 하자.
* 삭제한뒤에 다시 만들었는데... 파일이 그대로 있다. 귀신?

> 컨테이너만들기 예제 : https://nodejs.org/de/docs/guides/nodejs-docker-webapp/

---

### :gem: 준비 작업

#### [Container 만들기]

##### package.json

```json
{
  "name": "docker_web_app",
  "version": "1.0.0",
  "description": "Node.js on Docker",
  "author": "First Last <first.last@example.com>",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.16.1"
  }
}
```

##### server.js
```javascript
"use strict";

const express = require("express");

// Constants
const PORT = 8080;
const HOST = "0.0.0.0";

// App
const app = express();
app.get("/", (req, res) => {
  res.send("Hello world\n");
});

app.listen(PORT, HOST);
console.log(`Running on http://${HOST}:${PORT}`);
```

##### Dockerfile
``` Dockerfile
FROM node:10

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 8080
CMD [ "node", "server.js" ]
```
#### shell 명령
```bash
$docker build -t dev2ranger/node-web-app .
$docker build -t [본인계정명]/node-web-app .

$docker push dev2ranger/node-web-app
$docker push [본인계정명]/node-web-app
```
> deity98/node-web-app

---

###  :gem: 포드 만들기
:bulb: GCP에 접속 하셔서 위 파일을 vi를 이용해서 생성하신 다음에 아래의 명령을 실행해주세요.
#### 일단 포드 만들기
##### 설정파일
myapp-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container1
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
```
##### pod 만들기
```bash
$kubectl create -f myapp-pod.yaml
$kubectl get pods
```

#### 나의 docker로 포드 만들기

nodejs-manual.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodejs-manual
spec:
  containers:
  - image: dev2ranger/node-web-app
    name: nodejs
    ports:
    - containerPort: 8080
      protocol: TCP
```



```bash
$kubectl create -f nodejs-manual.yaml
```
#### 포드 포트포워딩 하기

```bash
$kubectl port-forward nodejs-manual 8080:8080 &
# 클라우드 콘솔 상단에 '웹 미리보기'로 들어가서 페이지가 열리는지 확인
$ps -eaf | grep port # 여기서 pid 확인, 
$kill -9 <pid>
```

#### 포드 기본 명령어

```bash
# 포드 리스트 확인
$kubectl get pods
$kubectl get pod myapp-pod
$kubectl get pod myapp-pod -o wide
$kubectl get pod myapp-pod -o yaml
# 포드 내용 확인
$kubectl describe pod myapp-pod
# 포드의 레이블까지 내용 확인
$kubectl get pod --show-labels
# 포드 삭제하기
$kubectl detele pod <pod name>
$kubectl delete pod --all

# 실행중인 포드에서 shell 실행하기
$kubectl exec -it <pod name> -- /bin/bash

# 포드의 로그 확인하기
$kubectl logs nodejs-manual
```

#### 연습

YAML을 사용하여 도커이미지 jenkins로 Jenkins-manual 포드를 생성하기 데헷!

jenkins-manual-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
name: jenkins-manual
spec:
containers:
- image: jenkins
  name: jenkins
  ports: - containerPort: 8080
  protocol: TCP
```

Jenkins 포드에서 curl 명령어로 로컬호스트:8080 접속하기

``` bash
$kubectl exec -it jenkins-manual -- curl 127.0.0.1:8080

# Jenkins 포트를 8888로 포트포워딩하기
$kubectl port-forward jenkins-manual 8888:8080 &

# 현재 Jenkins-manual의 설정을 yaml로 출력하기
$kubectl get pod jenkins-manual -o yaml
```

# :gem: 레이블과 셀렉터

### 레이블 있는 포드 생성

nodejs-pod-v2.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nodejs-manual2
  labels:
    creation_method: manual
    env: pod
spec:
  containers:
  - image: dev2ranger/node-web-app
    name: nodejs
    ports:
    - containerPort: 8080
      protocol: TCP
```

생성 및 생성 확인

```bash
$kubectl create -f nodejs-pod-v2.yaml
$kubectl get pod
```

#### 레이블 조회와 수정

레이블을 확인하는 다양한 방법들

```bash
$kubectl get pod --show-labels
### 결과
NAME READY STATUS RESTARTS AGE LABELS
jenkins-manual 1/1 Running 0 19m <none>
nodejs-manual 1/1 Running 0 67s creation_method=manual,env=pod
###

$kubectl get pod -L env
### 결과
NAME READY STATUS RESTARTS AGE ENV
jenkins-manual 1/1 Running 0 20m
nodejs-manual 1/1 Running 0 113s pod
###

$kubectl get pod -L env,creation_method
### 결과
NAME READY STATUS RESTARTS AGE ENV CREATION_METHOD
jenkins-manual 1/1 Running 0 20m
nodejs-manual 1/1 Running 0 2m8s pod manual
### 

# 레이블 추가하기
$kubectl label pod nodejs-manual test=foo
# 레이블 수정하기
$kubectl label pod nodejs-manual test=foo1 --overwrite

# 레이블 이용한 필터링 검색
## env label 이 있는거
$kubectl get pod --show-labels -l 'env'
## env lagel 이 없는거
$kubectl get pod --show-labels -l '!env'
## env label 의 값이 test 인거
$kubectl get pod --show-labels -l 'env!=test'

# env label의 값이 test가 아니면서 rel label의 값이 beta인거
$kubectl get pod --show-labels -l 'env!=test,rel=beta'
```

연습..!

YAML 파일을 사용하여 app=nginx 라벨을 가진 nginx container 를 가진 포드를 생성해보기
