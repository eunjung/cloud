# 스프링부트를 이미지로 만들기
context
1. 로그를 외부파일로 뺀다.
2. maven, gradle가 아닌 docker로 이미지를 만든다.
    - 소스 빌드는 빌드 툴에서 진행
    - docker 이미지 만드는 빌드만 docker에서 진행

## 스프링부트 images 생성 간단히 해보기
1. spring.io에서 스프링부트생성
2. gradle build
3. vi Dockfile
4. docker build -t web-boot .
5. docker images : 이미지 생성확인
6. docker run -p 5000:8080 web-boot

Dockerfile
``` 
FROM openjdk:8-jdk-alpine
VOLUME /tmp
COPY build/libs/demo-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

* [참조](https://zetawiki.com/wiki/%EB%8F%84%EC%BB%A4_%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EB%A1%9C_%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8_%EB%9D%84%EC%9A%B0%EA%B8%B0)

## volume 빼기
1. 스프링부트 log path변경
    - application.properties에 다음 설정 추가
    - Dockerfile의 volume와 패스를 맞추어야 함
    > logging.path=/tmp
2. 이미지 재생성
    1. gradle build
    2. docker build -t web-boot .
    3. docker run -p 5000:8080 web-boot

*  /tmp/spring.log에 로그쌓이는것 tail로 확인완료    
> /tmp/spring.log에 쌓이는 것 확인

## 기타
1. 이미지 수정은? 
    > 그냥 빌드 다시하면 된다.
2. aws workspace에서 도커를 해보려고 했는데 hyper-x가 허용이 안되서 실패!