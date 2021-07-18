# 도커

Repository for studying how to use docker in Windows environment

<br>



> 컨테이너란?

컨테이너는 하나의 Application 프로세스

컨테이너 1개 = Application 1개

완전히 독립된 프로세스로 동작함

image는 파일 형태로 저장, container는 실행중인 프로세스



> 컨테이너 동작과정

`$docker search nginx` # dockerd(도커 데몬: 도커 동작을 위한 플랫폼)에 nginx를 찾아달라고 요청함

dockerd은 `hub.docker.com` 에서 nginx 이미지 리스트를 출력해줌

`$docker pull nginx:latest` 명어를 통해서 허브에서 local로 이미지를 가져옴

`docker run -d --name web -p 80:80 nginx:latest` 명령어를 통해 컨테이너를 실행함

도커 플랫폼 위에 컨테이너가 실행되게 됌.



> 컨테이너 빌드 & 배포

* Dockerfile : 컨테이너를 build 하는데 도와주는 명령어 집합
  * text file로 Top-Down으로 해석
  * 컨테이너 이미지를 생성할 수 있는 **고유의 지시어(Instruction)** 을 가짐
  * 대소문자 구분은 없으나 가독성을 위해 사용

* Dockerfile 명령어

  * `#` : comment

  * `FROM` : 컨테이너의 base image(운영환경)

  * `MAINTAINER` : 이미지를 생성한 사람 정보

  * `LABEL` : 이미지에 컨테이너의 정보를 저장

  * `COPY` : 컨테이너 빌드시 호스트의 파일을 컨테이너로 복사

  * `ADD` : 컨테이너 빌드시 호스트의 파일(tar, url포함) 을 컨테이너로 복사\

  * `RUN` : 컨테이너 빌드를 위해 base image에서 실행할 commands

  * `WORKDIR` : 컨테이너 빌드 시 명령이 실행될 작업 디렉토리 설정

  * `ENV` : 환경변수 지정

  * `USER` : 명령 및 컨테이너 실행시 적용할 유저 설정

  * `VOLUME` : 파일 또는 디렉토리를 컨테이너의 데릭토리로 마운트

  * `EXPOSE` : 컨테이너 동작 시 외부에서 사용할 포트 지정

  * `CMD` : 컨테이너 동작 시 자동으로 실행할 서비스나 스크립트 지정

  * `ENTRYPOINT` : CMD와 함께 사용하면서 command 지정 시 사용

    ```bash
    # CMD vs ENTRYPOINT
    - CMD는 docker를 실핼할 때 파라미터를 변경하여 실행이 가능함
    - ENTRYPOINT는 docker를 실행할 때 파라미터를 변경할 수 없음
    ```

```dockerfile
# Dockerfile 예시

FROM ubuntu:14.04
MAINTAINER woqls226
LABEL "purpose"="practice"
RUN apt-get update
RUN apt-get install apache2 -y
ADD test.html /var/www/html
WORKDIR /var/www/html
RUN ["/bin/bash", "-c", "echo hello >> test2.html"]
EXPOSE 80
CMD apachectl -DFOREGROUND
```

* 컨테이너 배포
  * `$docker build -t [컨테이너명] .`  : 컨테이너 빌드
    * `.`은 빌드하고자 하는 컨테이너의 경로를 나타냄
  * `$docker login` : 도커 허브 로그인
  * `$docker push [컨테이너명]` : 컨테이너 배포

