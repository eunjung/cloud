# 우선 써놓기

## 스터디
깃헙
 > 계정 : 
 	- 공용메일
 	- 개인

깃헙 : 숙제... 
	소스 리파지토리 	
	도커 리파지토리
CI

프로젝트 -> 도커    -->   쿠버네티스
       -> CI/CD


## 시간

1일차 OT
	1. 전체 흐름 살펴보기

2일차 
osd 공용공간 : github, https://github.com/OSD-STUDY
	1. 도커 간단히 살펴보기
	- 숙제 : 프로젝트를 도커 이미지 만들어서 hub 올리기


*AUTOMATED란?
> https://novemberde.github.io/2017/04/02/Docker_8.html

OFFICIAL
> 공식지원되는 것



프로젝트 - 도커/쿠버네이티스 - 서버 / 클라우드 서버


1. 흐름살펴보기
2. 도커
	- 인트로
	- 프로젝트 도커이미지 만들기/배포
	- CI를 통한 배포, 젠킨스
3. 쿠버네이티스




## 오늘 해본것

1. 간단히 docker 써보기

docker images
docker search mysql
docker pull mysql:latest
docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=mysql mysql
docker ps -a
docker stop mysql
docker rm mysql
docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=mysql mysql
docker logs -f --tail 10 mysql
docker exec -it mysql /bin/bash
docker exec -it mysql mysql -uroot -pmysqql

