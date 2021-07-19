> ## 컨테이너 사용

* 컨테이너 생성 
  * docker create [option] [이미지이름:테그명]
  * `$ docker create --name webserver nginx:1.14`

* 컨테이너 실행
  * docker start [option] 컨테이너이름
  * `$ docker start webserver`

* 컨테이너 생성/실행 
  * docker run [option] [이미지이름:태그명]
  * `$ docker run --name webserver -d nginx:1.14`
  * `-d` : 백그라운드에서 실행
* 동작중인 컨테이너에 명령어 추가 실행
  * docker exec [option] 컨테이너이름
  * `$ docker exec -it webserver /bin/bash` 
  * `i` : interative, `t` : terminal
* 실행중인 컨테이너 목록 확인
  * docker ps [option]
  * `$ docker ps`
* 동작중인 컨테이너 중지
  * docker stop [option] 컨테이너이름
  * `$ docker stop webserver`
* 컨테이너 삭제
  * docker [option] 컨테이너이름
  * `$ docker rm -f webserver`
* 컨테이너에서 동작되고있는 프로세스 확인
  * `$ docker top`
  * `$ docker logs`  : log 보기
  * `$ docker logs -f` : 실시간 log 보기
* 컨테이너 내 필요한 정보만 찾기
  * `$ docker inspect --format '{{.NetworkSettings.IPAddress}}' webserver`  : webserver 컨테이너의 IP 정보만 조회
  * `alias conip(임의의 별명)="docker inspect --format '{{.NetworkSettings.IPAddress}}'"` : alias 등록해 놓으면 `conip webserver` 만으로 조회 가능

<br>

>  ### __컨테이너 리소스 제한__

Docker command를 통해 __CPU, Memory, Disk I/O__ 등의 리소스를 제한할 수 있다.

#### Memory 리소스 제한

| 옵션                 | 의미                                                         |
| -------------------- | ------------------------------------------------------------ |
| -m                   | 컨테이너가 사용할 최대 메모리 양을 지정                      |
| --memory-swap        | 컨테이너가 사용할 스왑 메모리 영역에 대한 설정 (디스크를 메모리처럼 사용)<br />메모리+스왑. 생략 시 메모리의 2배가 설정됨 |
| --memory-reservation | --memory 값보다 적은 값으로 구성하는 소프트 제한 값 설정     |
| --oom-kill-disable   | OOM Killer가 프로세스 kill 하지 못하도록 보호                |

* ` $ docker run -d -m 512m nginx:1.14`
  * 컨테이너 내 최대 512Mbite 사용 가능
* `$ docker run -d -m 1g --memory-reservation 500m nginx:1.14`
  * 최대 1Gbite 사용 가능, 최소 500Mbite 보장
* `$ docker run -d -m 200m --memory-swap 300m nginx:1.14`
  * 메모리 최대 200Mbite, 스왑 100Mbite 사용가능 (default 메모리의 2배값)
* `$ docker run -d -m 200m --oom-kill-disable nginx:1.14`
  * linux kunel에서 Out Of Memory Killer를 동작시켜 프로세스들을 중단시키것을 보호

#### CPU 리소스 제한

| 옵션          | 의미                                                         |
| ------------- | ------------------------------------------------------------ |
| --cpus        | 컨테이너에 할당할 CPU core 수를 지정<br />--cpus="1.5" 컨테이너가 최대 1.5개의 CPU파워 사용가능 |
| --cpuset-cpus | 컨테이너가 사용할 수 있는 CPU나 코어를 할당.<br />--cpuset-cpus=0-4 (cpu index는 0부터) |
| --cpu-share   | 컨테이너가 사용하는 CPU비중을 1024 값을 기반으로 설정 (상대적 가중치)<br />--cpu=share 2048 기본 값보다 두 배 많은 CPU 자원을 할당 |

* `$ docker run -d --cpus=".5" ubuntu:1.14`

  * CPU 코어 1개중 50% 까지 사용 가능

* `$ docker run -d --cpu-shares 2048 ubuntu:1.14`

  * 모든 컨테이너는 기본적으로 default 1024 cpu share값을 가지고 동작함
  * 설정함으로서 다른 컨테이너와의 상대적인 리소스를 할당하게 됌

* `$ docker run -d --cpuset-cpus 0-3 ubuntu:1.14`

  * 0~4번 코어에서 동작 가능

  

### Block I/O 제한

| 옵션                                        | 의미                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| --blkio-weight<br />--blkio-weight-device   | Block IO의 Quota를 설정할 수 있으며 100~1000까지 선택 <br />(default 500, 상대적인 가중치를 설정해주는 것임) |
| --device-read-bps<br />--device-write-bps   | 특정 디바이스에 대한 읽기와 쓰기 작업의 초당 제한을 kb, mb, gb 단위로 설정 |
| --device-read-iops<br />--device-write-iops | iops(컨테이너의 read/write 속도)의 쿼터를 설정한다.<br />초당 quota를 제한해서 I/O를 발생시킴(0 이상의 정수로 표기)<br />초당 데이터 전송량 = IOPS * 블럭크기(단위 데이터 용량) |

* `$ docker run -it --rm --blkio-weight 100 ubuntu:latest /bin/bash`

  * 다른 컨테이너(default 500)에 비해 1/5 수준의 block I/O만 할당

* `$ docker run -it --rm --device-write-bps /dev/vda:1mb ubuntu:latest /bin/bash`

  * /dev/vda에 저장할 때는 초당 최대 1mb로 전송 제한

* `$ docker run -it --rm --device-write-iops /dev/vda:100 ubuntu:latest /bin/bash`

  * /dev/vda에 대해 전송량을 제한

  ```bash
  # run과 함께 --rm 명령어를 사용하면 컨테이너 종료 해당 컨테이너 자동 삭제
  ```

  

### 리소스 모니터링

* __docker monitoring commands__
  * `docker stat[옵션] [컨테이너이름]` : 실행중인 컨테이너의 런타임 통계를 확인
  * `docker event` : 도커 호스트의 실시간 event 정보를 수집해서 출력
    * `docker events -f container=<NAME>`
  * cAdvisor : 도커 모니터링 툴
    * https://github.com/google/cadvisor/



### 실습

> 메모리 리소스 제한

메모리 부하 확인할 dockerfile 생성

* docker build -t stress .
  * stress : 부하 테스트 프로그램

* `docker run -m 100m --memory-swap 100m stress:latest stress --vm 1 --vm-bytes 90m -t 5s`
  * stress 컨테이너 메모리 2코어 100% 사용 및 100Mbyte 메모리 부여(스왑메모리 0) 5초간 작업 로드 90Mbyte 일으킴 -> 잘 작동함

```bash
$ docker run -m 100m --memory-swap 100m stress:latest stress --vm 1 --vm-bytes 90m -t 5s
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: info: [1] successful run completed in 5s
```



* `docker run -m 100m --memory-swap 100m stress:latest stress --vm 1 --vm-bytes 150m -t 5s`
  * 최대 메모리 100M인데 150M 로드를 부하하면 -> 동작 FAIL

```bash
$ docker run -m 100m --memory-swap 100m stress:latest stress --vm 1 --vm-bytes 150m -t 5s
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: FAIL: [1] (415) <-- worker 8 got signal 9
stress: WARN: [1] (417) now reaping child worker processes
stress: FAIL: [1] (421) kill error: No such process
stress: FAIL: [1] (451) failed run completed in 0s
```



* `docker run -m 100m stress:latest stress --vm 1 --vm-bytes 90m -t 5s`
  * swap size를 생략하면 (default 값 : 메모리의 2배) 설정되어 잘 작동함

```b
$ docker run -m 100m stress:latest stress --vm 1 --vm-bytes 90m -t 5s
stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
stress: info: [1] successful run completed in 5s
```

<br/>

> CPU 리소스 제한

#### 1. CPU 코어 설정

* 컨테이너 별로 작업 로드를 분산시킬 수 있음

* `$ docker run --cpuset-cpus 1 --name c1 -d stress stress --cpu 1`

![image-20210718220602065](C:\Users\user\JIKMAN\docker-exercise\img\image-20210718220602065.png)

​		`htop` 커멘드를 통해 2번째(index 1) CPU에 작업 로드 100% 일으키고 있는 것을 확인 가능

	- `$ docker run --cpuset-cpus 0 --name c1 -d stress stress --cpu 1`

![image-20210718221024461](C:\Users\user\JIKMAN\docker-exercise\img\image-20210718221024461.png)

​		1번째 CPU에서 작업하고 있는 것을 확인 가능

#### 2. CPU 리소스 가중치 설정

컨테이너 별로 CPU 가중치를 할당하여 실행하도록 구성

* `$ docker run -c 2048 --name cload1 -d stress:latest`

* `$ docker run --name cload2 -d stress:latest` (default : 1024)
* `$ docker run -c 512 --name cload3 -d stress:latest`
* `$ docker stats` CPU 사용량 등 확인 가능

![image-20210718221909893](C:\Users\user\JIKMAN\docker-exercise\img\image-20210718221909893.png)

사용하고 있는 컨테이너별 cpu 가중치를 확인할 수 있음

(CPU돌아가는 소리가 엄청나서 컴퓨터 터지는줄...)

#### 3. Block I/O 제한

* `$ docker run -it --rm --device-write-iops /dev/xvda:10 ubuntu:latest /bin/bash`

  * `$ dd if=/dev/zero of=filel bs=1M count=10 oflag=direct`
    * `if` : 지정 경로 내에 `of` : 지정된 파일을 만들고  `count=10` : 10블록 크기만큼 `bs=1M` : 한번에 1M씩 읽어들여 출력, `oflag=direct` : buffer cache 사용 않고 다이렉트로 I/O
      즉, 10M를 읽고 출력하는데 걸리는 시간을 측정하기 위해 사용

  ```bash
  10+0 records in
  10+0 records out
  10485760 bytes (10 MB, 10 MiB) copied, 1.01403 s, 10.3 MB/s
  
  # 초당 10.3 MB
  ```

  

* `$ docker run -it --rm --device-write-iops /dev/xvda:100 ubuntu:latest /bin/bash`

  * `$ dd if=/dev/zero of=filel bs=1M count=10 oflag=direct`

  ```bash
  10+0 records in
  10+0 records out
  10485760 bytes (10 MB, 10 MiB) copied, 0.0412287 s, 254 MB/s
  
  # 초당 254 MB
  ```

  block IO 설정을 통해 확연한 속도 차이를 확인할 수 있음

#### 4. cAdvisor

* https://github.com/google/cadvisor/

  ```bash
  # Quick start
  
  VERSION=v0.36.0 # use the latest release version from https://github.com/google/cadvisor/releases
  sudo docker run \
    --volume=/:/rootfs:ro \
    --volume=/var/run:/var/run:ro \
    --volume=/sys:/sys:ro \
    --volume=/var/lib/docker/:/var/lib/docker:ro \
    --volume=/dev/disk/:/dev/disk:ro \
    --publish=8080:8080 \
    --detach=true \
    --name=cadvisor \
    --privileged \
    --device=/dev/kmsg \
    gcr.io/cadvisor/cadvisor:$VERSION
  ```

* UI를 통해 확인 가능
* cAdvisor는 Kubernates에 포함되어 있음

![image-20210718224637640](C:\Users\user\JIKMAN\docker-exercise\img\image-20210718224637640.png)

