# Dockerfile 을 통한 도커 이미지 생성 및 배포

요 며칠 spring 을 이용한 백엔드 서버와 react 를 이용한 프론트엔드 서버를 구축해서 배포해야할 일이 생겼다.

이 두개의 서버를 통합하여 도커이미지로 만들고, 해당 이미지를 배포하는 일련의 작업을 아래와 같이 진행하였다.

1. 우분투 도커이미지를 pull 하여 컨테이너를 만든다.
2. 우분투에 필요한 모듈들을 설치한다. (openjdk, nodejs, npm 등)
3. 빌드한 spring 의 jar 파일을 도커 컨테이너의 /home/spring/spring.jar 에 복사한다.
4. 빌드한 react 의 build 폴더를 도커 컨테이너의 /home/react/build 에 복사한다.
5. 위의 컨테이너를 커밋하여 이미지를 생성한다.
6. 생성된 이미지를 적절히 tag 를 붙여 docker hub 에 push 한다.
7. 해당 이미지를 pull 받아 실행하고, 컨테이너에 접속해 spring 과 react 프로그램을 실행한다.

위의 과정을 거치면 두개의 서버를 하나의 도커 이미지에서 관리할 수 있게 된다.

> 그러나, 위의 과정은 모든 명령어를 직접 타이핑 해야 한다는 단점이 있다. 멍청하게도 나는 이제껏 도커를 사용할 때 늘 이런식으로 사용해 왔다. 그렇게 오래 사용한건 아니지만... 이제 프로세스를 개선할 때가 되었다.

## Dockerfile 을 이용한 도커이미지 생성
도커는 Dockerfile 이라는 파일에 일련의 명령어를 입력하고, 이를 이용해 이미지를 build 할 수 있는 기능을 제공한다.

```
FROM ubuntu:20.04

WORKDIR /home

COPY ./backend/build/libs/backend-0.0.1-SNAPSHOT.jar /home/spring/spring.jar
COPY ./frontend/build /home/react/build

RUN apt update
RUN apt install openjdk-17-jdk -y
RUN apt install nodejs -y
RUN apt install npm -y
RUN npm install -g serve
```

이 파일은 내가 생성한 Dockerfile 로, 위에서 기술한 복잡한 과정 중 1 ~ 5번 작업을 한번에 할 수 있게 해준다.

하나하나 살펴보면,


`FROM ubuntu:20.04`  
내가 생성하려는 도커이미지의 베이스를 선택하는 것으로, 우분투 20.04 버전을 베이스 이미지로 하여 도커이미지를 생성하겠다는 뜻이다.

`WORKDIR`  
 도커이미지의 작업 디렉토리를 설정한다. -it 옵션을 통해 컨테이너에 접속하면 가장 처음으로 지정되는 디렉토리이다.
 
 `COPY`  
 내가 작업하는 컴퓨터(호스트) 에서 도커컨테이너로 파일을 복사한다. 이것과 유사한 `ADD` 라는 것도 있는데, 이는 URL 을 통한 파일의 추가도 가능한 것 같다. 자세한 것은 나중에 알아봐야겠다. 그리고, `COPY` 에서는 절대경로를 사용할 수 없다. 무조건 Dockerfile 이 있는 현재경로로 부터 상위 폴더에 한해서만 파일을 지정할 수 있다.
 
 `RUN`  
도커이미지를 빌드하면서 실행되는 명령어이다. 주로 필요한 모듈들을 설치하는데 사용한다.

이렇게 Dockerfile 을 생성하고, 해당 파일이 있는 경로에서 

```
docker build . -t docker_image:0.0.1
```
을 입력하면 docker_image:0.0.1 이라는 도커이미지가 생성된다. 해당 이미지에는  

* `ubuntu 20.04`
* `openjdk-17`
* `nodejs`
* `npm`
* `npm serve`

가 자동으로 설치되어 있다.

또한  컨테이너를 생성하면 해당 컨테이너 안에는  

* `/home/spring/spring.jar`
* `/home/react/build`

와 같은 파일들이 복사되어 있다.

명령어 한줄로 참 많은 일을 할 수 있다. 공부하는 귀찮음을 조금만 감내하면 훨씬 큰 귀찮음을 제거할 수 있다...

## Docker image 경량화
위에서 베이스 이미지로 사용한 `ubuntu:20.04` 는 용량이 꽤 크다. 일반적으로 사용하는 리눅스 os 이다 보니 많은 기능이 들어가 있기 때문이다.

위에서 생성한 도커이미지는 용량이 `1.2Gb` 가량 되었다. 이미지를 첨부하면 좋겠지만 몇시간 전에 작업한거라 남아있지 않다... 여튼 그정도 용량이었다.

하지만 조금 서치해보니, 도커이미지의 용량을 줄일 수 있는 방법들이 몇가지 있었다. 그중 내가 적용한것은 한가지이고, 나머지에 대해서는 다음에 알아보도록 하자.

```
FROM alpine:3.15.0

WORKDIR /home

COPY ./backend/build/libs/backend-0.0.1-SNAPSHOT.jar /home/spring/spring.jar
COPY ./frontend/build /home/react/build

RUN apk update
RUN apk add coreutils
RUN apk add openjdk17
RUN apk add nodejs
RUN apk add npm
RUN npm install -g serve
```
이번에는 베이스이미지로 `alpine:3.15.0` 을 사용하였다. `alpine` 은 경량화된 리눅스 os 로, 서버 등의 목적으로 가벼운 os 기능만 필요할 경우 사용하도록 만들어졌다.

`alpine:3.15.0` 도커이미지의 용량은 무려 `5Mb` 정도다... 진짜 가볍다.

우분투가 아니기 때문에, `RUN` 에서 사용한 명령어도 조금 다른 것을 볼 수 있다.

`apt install` 대신 `apk add` 가 사용되었다.

이렇게 빌드한 이미지는 대략 `390Mb` 정도이다. 거의 용량이 3배가량 차이나는 것을 볼 수 있다. 여기서 더 줄일 수 있다고 하니, 실로 가슴이 웅장해진다..


## Entrypoint 를 이용한 프로세스 자동실행
이제 도커이미지에서 컨테이너를 만들고, 컨테이너 안에서 `spring` 과 `react` 를 실행하여 서버를 구축해야한다.

이제까지의 나는 직접 이미지에서 

```
docker run -d -it -p 8080:8080 -p 3000:3000 --name server docker_image:0.0.1
```  

컨테이너를 실행

```
docker exec -it server /bin/sh
```
컨테이너에 접속

```
nohup java -jar /home/spring/spring.jar &    # nohup 을 통해 백그라운드로 실행
```

`spring` 실행

```
nohup npx serve -s /home/react/build & 		# nohup 을 통해 백그라운드로 실행
```

`react 실행`

의 과정을 거쳐 서버를 실행시켰다. 이짓을 하고있었다니...

`Dockerfile` 의 `EntryPoint` 를 이용하면 컨테이너를 시작할 때 바로 해당 프로세스들을 실행시킬 수 있다.


```
FROM alpine:3.15.0

WORKDIR /home

COPY ./backend/build/libs/backend-0.0.1-SNAPSHOT.jar /home/spring/spring.jar
COPY ./frontend/build /home/react/build
COPY ./run_server.sh /home/run_server.sh			# spring 과 react 를 실행시키는 shell script

RUN apk update
RUN apk add coreutils
RUN apk add openjdk17
RUN apk add nodejs
RUN apk add npm
RUN npm install -g serve

ENTRYPOINT sh /home/run_server.sh; sh				# shell script 실행
```

`DockerFile` 에서 `Entrypoint` 를 이용하면 해당 명령어를 컨테이너가 실행될 때 실행하게 된다.

`Entrypoint` 는 총 두가지의 경우에 실행하게 된다.

* docker run 을 통해 컨테이너가 시작 될 때
* docker start 를 통해 컨테이너가 중지되었다가 다시 시작될 때

비슷한 명령어로 `CMD` 가 있는데, 이를 통하면 해당 명령어가 docker run 을 통해 시작될 때만 실행된다. 컨테이너가 재시작 될 때마다 프로세스가 시작되어야 하므로, `Entrypoint` 가 더 낫다.

`Entrypoint` 와 `CMD` 는 조금 더 디테일한 차이가 있긴 하던데, 정확히 찾아 보지는 않았다. 다음에 한번 찾아보자.


### 중요 포인트!
이것때문에 진짜 많이 해멨다.  `Entrypoint` 에 

```
sh /home/run_server.sh
```

라고 입력했더니 컨테이너를 run 하면 자꾸 shutdown 이 되더라. 이것저것 시도해보았는데 해결이 안돼 삽질을 반복하다가 찾아냈다.

```
sh /home/run_server.sh   -> 해당 명령어를 수행하고 나면, 터미널을 종료한다.
```

정확한 이유는 모르겠지만, 터미널 명령어를 수행하고 나면 터미널 자체가 종료되는 현상이 있는듯 했다. 그로인해 컨테이너가 shutdown 되고 있었다..

```
sh /home/run_server.sh; sh
or
sh /home/run_server.sh; sleep infinity
```
위의 코드 둘 중 하나를 적용하면 해결되는 문제였다. 터미널이 계속 유지될 수 있도록 만드는 기믹인 것 같다. 정확한 작동방식에 대해서는 아직 잘 모르겠으나, 오늘은 시간으 늦었으므로 이쯤에서 정리해야겠다.