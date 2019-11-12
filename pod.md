  <a href="https://github.com/OSD-STUDY/cloud/edit/master/pod.md">
    <img alt="Documentation" src="https://img.shields.io/badge/documentation-yes-brightgreen.svg" target="_blank" />
  </a>
# POD

### 준비 작업

#### Container 만들 준비

예제 site
https://nodejs.org/de/docs/guides/nodejs-docker-webapp/

##### Nodejs

package.json

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

```
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

```shell
$ docker build -t dev2ranger/node-web-app .
$ docker build -t [본인계정명]/node-web-app .

$ docker push dev2ranger/node-web-app
$ docker push [본인계정명]/node-web-app
```

## POD

### 포드 만들기

#### 포드 예제

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

#### 포드 정보 확인하기

```
kubectl get pod myapp-pod
kubectl get pod myapp-pod -o wide
kubectl get pod myapp-pod -o yaml
kubectl describe pod myapp-pod
```

#### 포드 만들어보기

nodejs-pod.yaml

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

#### 포드 포트포워딩 하기

```
kubectl port-forward nodejs-manual 8080:8080 &
# 클라우드 콘솔 상단에 웹 미리보기로 들어가서 페이지가 열리는지 확인
ps -eaf | grep port # 여기서 pid 확인
kill -9 <pid>
```

#### 포드 기본 명령어

포드의 레이블까지 내용 확인

```
kubectl get pod --show-labels
```

포드 삭제하기

```
kubectl detele pod <pod name>
kubectl delete pod --all
```

실행중인 포드에서 shell 실행하기

```
kubectl exec -it <pod name> -- /bin/bash
```

포드의 로그 확인하기

```
kubectl logs nodejs-manual
```

#### 연습

YAML을 사용하여 도커이미지 jenkins로 Jenkins-manual 포드를 생성하기 데헷!

# 레이블과 셀렉터

### 레이블 있는 포드 생성

nodejs-pod-v2.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: nodejs-manual
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

```
$ kubectl create -f nodejs-pod-v2.yaml
$ kubectl get pod
```

#### 레이블 조회와 수정

레이블을 확인하는 다양한 방법들

```
\$ kubectl get pod --show-labels
NAME READY STATUS RESTARTS AGE LABELS
jenkins-manual 1/1 Running 0 19m <none>
nodejs-manual 1/1 Running 0 67s creation_method=manual,env=pod
```

```
\$ kubectl get pod -L env
NAME READY STATUS RESTARTS AGE ENV
jenkins-manual 1/1 Running 0 20m
nodejs-manual 1/1 Running 0 113s pod
```

```
\$ kubectl get pod -L env,creation_method
NAME READY STATUS RESTARTS AGE ENV CREATION_METHOD
jenkins-manual 1/1 Running 0 20m
nodejs-manual 1/1 Running 0 2m8s pod manual
```

#### 레이블 추가하기

```
kubectl label pod nodejs-manual test=foo
```

#### 레이블 수정하기

```
kubectl label pod nodejs-manual test=foo1 --overwrite
```

#### 필터링 검색

```
* env label 이 있는거
kubectl get pod --show-labels -l ‘env'
* env lagel 이 없는거
kubectl get pod --show-labels -l '!env’
* env label 의 값이 test 인거
kubectl get pod --show-labels -l 'env!=test’
* env label의 값이 test가 아니면서 rel label의 값이 beta인거
kubectl get pod --show-labels -l 'env!=test,rel=beta'
```

연습..!

YAML 파일을 사용하여 app=nginx 라벨을 가진 nginx container 를 가진 포드를 생성해보기
