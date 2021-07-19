> 네트워크 간 연결 연습

* `$ mkdir webdata`
* `$ echo "<h1>Docker Volume-Learn</h1>" > index.html`
  * webdata 폴더를 만들어 html index를 저장해줌

* `$ docker run -d --name web -p 80:80 -v \ /webdata:/usr/share/nginx/html:ro nginx:1.14`
  * webdata 폴더에 볼륨마운트 시켜줌

* `$ docker ps`

```bash
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS         PORTS                               NAMES
8f4f3236e15b   nginx:1.14   "nginx -g 'daemon of…"   4 minutes ago   Up 4 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp   web

# ngnix 컨테이너가 실행 중임을 확인
```

* localhost:80 서버가 통신중인 것 확인할 수 있음

![image-20210719212612672](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210719212612672.png)

> 네트워크간 연결 연습 2

`df -h /` : 디스크 사용량을 모니터링 할 때 사용하는 커멘드

해당 커멘드를 10초마다 갱신해서 서버에 전송해보자

```bash
#!/bin/bash
mkdir -p /webdata
while true
do
        df -h / > /webdata/index.html
        sleep 10
done

# df.sh 파일을 생성 -> 서버에 전송할 html 파일
```

```bash
FROM ubuntu:18.04
ADD df.sh /bin/df.sh
RUN chmod +x /bin/df.sh
ENTRYPOINT ["/bin/df.sh"]

# dockerfile을 만들어줌. 서버에 df.sh가 실행되도록 명령을 전송하는 역할
```

* df.sh 와 dockerfile을 bulid함

  * `docker build -t wjddlr0303/df:latest .`

* `docker run -d --name web2 -v /webdata:/usr/share/nginx/html:ro -p 80:80 nginx:1.14`

  * 로컬 호스트에서 실행되는 것을 확인할 수 있음

  ![image-20210719214617092](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210719214617092.png)



### 컨테이너 간 통신

---

docker를 host에 설치한 후 host의 network interface를 살펴보면, docker0라는 virtual interface가 있는 것을 볼 수 있다.

* IP 는 자동으로 172.17.42.1 로 설정 되며 16 bit netmask(255.255.0.0) 로 설정된다. 

* 이 IP는 DHCP를 통해 할당 받는 것은 아니며, docker 내부 로직에 의해 자동 할당 받는 것이다.

* docker0 는 일반적인 interface가 아니며, virtual ethernet bridge 이다.

![docker0-network](https://jonnung.dev/images/docker_network.png)

docker가 설치되면 docker0라는 bridge가 생성되며, container가 running 될때마다 vethXXXX라는 이름의 interface가 attach되는 형태이다.

```bash
root@~~:# brctl show docker0

bridge name      bridge id                STP    enabled   interfaces

docker0          8000.d67973326669        no               veth0e580ab                                                                              veth77faee0

# 2개의 컨테이너가 running 중인 상태
```

