# 도커 이미지파일 어떻게까지 만들수 있는지에 대하여...
context
1. spring boot를 바탕으로 이미지파일 만들어보기
2. maven과 gradle로 만들어보기

> 이상 = local을 넘어서 서버에 이미지 실행까지 
>>현실 = 알아보는 정도까지만... 실행도 제대로 안되는
>>> 실행파일까지해서 다음 git에 올려 놓도록 하겠습니다. 믿어주시길... 

목차
1. [docker로 만들기](#도커-익숙해지기)
2. [빌드 툴로 만들기](#툴을-통한-배포)
3. [실제 사용절차](#실제-사용사례)
4. [번외편](#잡담)

---

3. jib으로
4. 배포툴에서 진행
5. 서버에서 이미지로 실행

## 도커 익숙해지기
간단히 해보기
1. docker란
2. docker 원리
3. docker 명령어 익숙해지기
    - docker run
    - docker ps
    - docker images
4. docker 이미지 만들어보기
    - Dockfile에 만들기
```
FROM ubuntu:16.04
ADD test.txt /
```
> docker build -t myubuntu .


## 툴을 통한 배포
### Docker 컨테이너 내에서 Maven 빌드를 실행하는방법
1. `https://start.spring.io/`에서 maven으로 스프링부트 파일만들기
2. 도커파일 만들기 
3. 도커실행
> docker build -t demo .

```도커파일
# Create a Dockerfile
# Build stage
#
FROM maven:3.6.0-jdk-11-slim AS build
COPY src /home/app/src
COPY pom.xml /home/app
RUN mvn -f /home/app/pom.xml clean package
```

```
#
# Package stage
#
FROM openjdk:11-jre-slim
COPY --from=build /home/app/target/demo-0.0.1-SNAPSHOT.jar /usr/local/lib/demo.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/usr/local/lib/demo.jar"]
```

### maven 사용
- 우선은 실패. 오류남
- 플러그인 라이브러리 com.spotify를 주로 사용하는 것으로 보임
1. 스프링부트 파일 만들기
2. Dockerfile 만들기
- src/main/docker
3. pom.xml 수정
4. maven 실행
> mvn package docker:build 

``` 
<!-- maven pom.xml-->
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>0.3.8</version>
    <configuration>
    <imageName>example</imageName>
    <dockerDirectory>src/main/docker</dockerDirectory>
    <!-- copy the service's jar file from target into the root directory of the image --> 
    <resources>
        <resource>
            <targetPath>/</targetPath>
            <directory>${project.build.directory}</directory>
            <include>${project.build.finalName}.jar</include>
        </resource>
    </resources>
    </configuration>
</plugin>
```


### gradle 
- 여기도 실패. ㅎㅎㅎㅎ
1. 스프링부트 파일 만들기
2. Dockerfile 만들기
- src/main/docker
3. gradle.build 수정
4. gradle 실행
> gradle build buildDocker

```
-- add build.gradle

buildscript {
    repositories {
        mavenCentral()
    }

    dependencies {
        classpath "se.transmode.gradle:gradle-docker:1.2"  
    }
}


apply plugin:'docker'
apply plugin:'eclipse'

task buildDocker(type:Docker,dependsOn:build){
	applicationName=jar.baseName
	dockerfile=file('src/main/docker/DockerFile')
	doFirst{
		copy{
			from jar
			into stageDir
		}
	}
}

```

### 구글에서 만든 Jib 사용
maven과 gradle 모두 쓰는 플러그인

### 기타
오류가 발생한다. 왜인지 찾아봐야할 것 같다.
#### maven의 경우 `fabric8 Docker plugin`도 있다고 한다.

## 실제 사용사례
### 로컬
개발하는 로컬은 image를 만들필요가 없다.
### 서버
1. CI로 소스 패키징과 docker 빌드 분리
> CI는 주로 젠킨스 사용

---

* 도커에서 maven실행 : https://stackoverflow.com/questions/27767264/how-to-dockerize-maven-project-and-how-many-ways-to-accomplish-it

* maven으로 이미지만들기 : http://wonwoo.ml/index.php/post/268

* gracle로 이미지만들기 : https://blog.naver.com/PostView.nhn?blogId=sharplee7&logNo=221485570132

* 구글 jib 예제 : https://github.com/GoogleContainerTools/jib/tree/master/examples/spring-boot

* jib 소개 : https://github.com/GoogleContainerTools/jib


---

## 잡담

### 도커-컴포즈
의존적인 컨테이너를 설정한다.
실제로는 각각의 컨테이너가 구성되어있는 것을 알수 있다.
- 정의 : 컨테이너 여럿을 사용하는 도커 애플리케이션을 정의하고 실행하는 도구
#### wiki.js
- docker 컴포즈로 되어있는 예제이다.
- 실행 : docker-compose up -d
- 컴포즈의 기본파일명 : docker-compose.yml
#### 스케일 아웃을 고려했을때
- pot으로 분리

스케일 아웃을 고려했을때는 컴포즈를 쓰는것 같지 않다.

* 도커 컴포즈 자세한 설명글 : http://raccoonyy.github.io/docker-usages-for-dev-environment-setup/

---

inspect로 도커가 어떻게 구성되어있는지 볼수 있다.

도커 외부에 존재할것들
1. 볼륨
    - 컨테이너가 없어지더라도 데이터는 살아 있어야지.
2. 로그
    - 그래야 컨테이너 올라갈때 오류나더라도 무슨 오류인지 알수 있다.

---

* 쉘 : 파워라인
* Quiver
* 젠킨스 X
    > 도커 기반
* k8