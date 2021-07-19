> ### 컨테이너 볼륨이란?

* 컨테이너 이미지는 **read-only**(수정이 불가능한 상태)
* 컨테이너 또한 __read-only__
* 컨테이너에 추가되는 데이터들은 별도의 RW(Read-Write) 레이어에 저장됨

* 도커는 Union File System(Overlay) 구조로 작동하여 RO와 RW가 하나인 것처럼 동작(RO 위에 RW가 쌓이는 구조로 작동함)
* 예를 들어 mysql 컨테이너가 RO(Read-Only)로 작동하면 그 위에 데이터 들이 RW 레이어(`/var/lib/mysql`)에 쌓임
* 그러나 __컨테이너를 삭제해 버리면 RO와 RW 전부 사라져 복원할 수 없다!!!__
* __"컨테이너에서 만든 데이터를 영구적 보존"__ : Docker Host에 특정 저장 공간을 만들어주는 방법?? -> 볼륨마운트
* `$ docker run -d --name db -v /dbdata:/var/lib/mysql \`
  `-e MYSLQ_ALLOW_EMPTY_PASSWORD=pass mysql:latest`
* 컨테이너를 삭제해도 `/dbdata`는 남아 있기 때문에, 나중에 다시 컨테이너를 다시 실행해도 쌓아놨던 동일한 데이터를 가지고 service를 실행 가능

<br>

> #### volume 옵션 사용

* -v <host path>:<container mout path>
* -v <host path>:<container mout path>:<read write mode>
* -v <container mount path>
  * `$ docker run -d -v /dbdata:/var/lib/mysql -e MYSQL...`
    * 해당 커멘드는 컨테이너가 docker host의 directory를 수정하게됌.
      보안이슈 등으로 인해 권장되지는 않음 (default : `rw`)
  * `$ docker run -d -v /webdata:/var/www/html:ro httpd:latest`
    *  `ro` : /webdata의 데이터를 이용해 service를 실행하나
       apache는 host에 있는 데이터를 수정하지 못하도록 보호하면서 마운트
  * `$ docker run -d -v /var/lib/mysql -e MYSQL...`
    * 임의의 컨테이너 디렉토리를 넣어주어 자동마운트 시켜줌
* __"컨테이너 끼리의 데이터 공유가 가능해짐"__
  * 컨테이너의 data를 host /webdata에 저장
  * /webdata 데이터를 readonly형식으로 다른 웹 서버로 docker run하여 service

<br/>

## 실습

> __-v <host path>:<container mout path>__

1.  `$ docker run -d --name db -v /dbdata:/var/lib/mysql -e \ MYSQL_ROOT_PASSWORD=pass mysql:latest`

2. `$ docker exec -it db /bin/bash` 
   * db 컨테이너 안으로 들어감

3. `root@d09da1f9a493:/# mysql -u root -ppass`

   * `root@d09da1f9a493:/#`  : db 터미널

   * db에 접속함.

4. `mysql> show databases;`

   * db list가 보여짐

![image-20210719002824854](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210719002824854.png)

5. `mysql> create database docker;`

   * docker 라는 database를 생성

6. `$ cd /dbdata/`

   * 만들어둔 dbdata로 들어감

   * `ls` 보면 아까 만들어둔 docker db 생성된 것 확인 가능

7. `$ docker rm -f db`

   * docker 컨테이너를 삭제해버림

   * RO, RW 전부 사라졌으니 volume mout 해준 data 남아 있나 확인해보자

8. `ls /dbdata/`

   * data 그대로 남아있음

   * 컨테이너를 또 만들어도 이 데이터를 그대로 다시 사용 가능함

> __-v <container mount path>__

컨테이너 path만 써도 데이터가 영구보존이 될까?

어디에 저장이 되는것인지 알아보자

1. `$ docker run -d --name db -v /var/lib/mysql -e \ MYSQL_ROOT_PASSWORD=pass mysql:latest`

2. `docker inspect`

   * ```bash
     jungik@DESKTOP-J10OVLO:/$ docker inspect db | grep -E "Source|Destination"*
                     "Source": "/var/lib/docker/volumes/d08d80221de3456e25ec13a762b55098a8ebf8f41fbdbf697b217c9c349faa84/_data",
                     "Destination": "/var/lib/mysql",
                     
     # Source : volume mount 되는 directory
     # Destination : container data가 저장되는 위치
     # volume mount 된 것을 확인할 수 있음
     ```

   

3. `docker volume ls`

   * volume 목록 확인

4. `docker volume rm <해당UUID>`

   * 필요없는 volume 삭제