# 도커에 대한 이해
빠른 가상머신, 연계도 쉬운 가상머신
> svn이 그냥 가상머신이라면 도커는 git과 같은 가상머신이다.
>> git은 빠르고, 상호환적이며, 커뮤니케이션요소를 가지고 있다. docker도 기존 가상머신의 불편한점을 개선? 아니 혁신하여서 나왔다.

## 컨테이너 기반의 오픈소스 가상화 플랫폼
도커는 프로세스 격리 방식
- 기존의 가상화 방식은 주로 OS를 가상화
- 이러한 상황을 개선하기 위해 CPU의 가상화 기술(HVM) 등장, 아래 두가지 방식
    - KVM(Kernel-based Virtual Machine)
    - Xen(반가상화 Paravirtualization방식)
    - OpenStack이나 AWS, Rackspace같은 클라우드 서비스에서 가상 컴퓨팅 기술의 기반
- 프로세스 격리 방식 : 전가상화든 반가상화든 추가적인 OS를 설치하여 가상화하는 방법은 어쨋든 성능문제가 있었고 이를 개선하기 위해 프로세스를 격리 
- 컨테이너라는 개념은 도커가 처음 만든 것이 아님

* 도커의 기본 네트워크 모드는 Bridge모드

## 도커의 주요 개념
컨테이너와 이미지가 주요 개념, docker hub는 이미지를 저장할수 있는 공간일 뿐이다.
### 1. 컨테이너
컨테이너는 이미지를 실행한 상태라고 볼 수 있고 추가되거나 변하는 값은 컨테이너에 저장
- `docker ps`로 나오는 컨테이너 정보가 어떻게 되는지 알아두도록 합시다.
- 이미지만들기 : `Dockerfile`파일을 만들어서 `build` 명령어로 만든다.
### 2. 이미지
이미지는 컨테이너 실행에 필요한 파일과 설정값등을 포함하고 있는 것으로 상태값을 가지지 않고 변하지 않습니다(Immutable)
- `docker images`에서 나오는 것이 무엇을 의미하는 알아두도록 합시다.
### 3. docker hub
도커 이미지는 Docker hub에 등록하거나 Docker Registry 저장소를 직접 만들어 관리할 수 있습니다.
> 개인이 관리하는 docker hub같은 것이 있을까? 주로 무엇이 있을까?
### 4. 레이어(layer) 저장방식
`1.OS` 위에 `2.ngix`를 설치하고 그위에 `3.web`을 만들어서 사용하는 것을 사용합니다. 총 3개의 이미지를 사용했지요? 이것을 모두 각자 설정하는 것이 아닙니다. 레이어 방식을 설정해놓으면 저절로 다운로드 받게 되어있습니다.
- 실제로 Dockerfile에 서 From으로 되어있는게 상위의 레이어를 말하게 되며, 컨테이너를 띄울때 각각의 레이어에 해당하는 이미지를 다운로드 받게 됩니다.
### 5. 컨테이너 실행
그냥 실행은 run을 하면 되고 이미지들끼리 의존적인것은 compose를 이용해서 실행하도록 한다.
- compose : `Docker-compose.yml`파을을 만들어서 `docker-compose up`를 만들어서 사용한다.
> 의존적인 이미지들을 설정해놓고, 한꺼번에 쓰는 것이다.

- 컨테이너내부가 아닌 외부로 빼야할것들 : volumn, log

---

* [도커란](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)

* [도커-간단히-실행](https://blog.naver.com/PostView.nhn?blogId=alice_k106&logNo=220646382977&parentCategoryNo=7&categoryNo=&viewDate=&isShowPopularPosts=true&from=search)

* [도커-명령어](https://nicewoong.github.io/development/2017/10/09/basic-usage-for-docker/)

* [도커-명령어](https://www.soday.net/wp/archives/371)

* [도커-컴포즈](http://raccoonyy.github.io/docker-usages-for-dev-environment-setup/)