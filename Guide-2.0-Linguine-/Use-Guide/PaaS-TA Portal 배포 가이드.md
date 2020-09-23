## Table of Contents
- 1. [문서 개요](#1)
     - [1.1. 목적](#2)
     - [1.2. 범위](#3)
     - [1.3. 시스템 구성도](#4)
     - [1.4. 참고자료](#5)
- 2. [설치전 준비사항](#6)
     - [2.1. 기본설치 항목](#7)
     - [2.2. 사용자의 조직 생성 Flag 활성화](#23)
     - [2.3. 조직 및 공간 생성](#8)
     - [2.4. Redis 서비스 브로커 등록 및 활성화](#9)
     - [2.5. Redis 서비스 인스턴스 생성](#10)
     - [2.6. 사용자 제공 서비스 생성](#11)
     - [2.7. Portal Object Storage 설치 및 설정 변경](#12)
     - [2.8. Postgresql 기본 데이터 베이스 생성](#13)
- 3. [PaaS-TA 배포](#14)
     - [3.1. 포탈 Registration 배포](#15)
     - [3.2. 포탈 API 배포](#16)
     - [3.3. 포탈 APIV2 배포](#17)
     - [3.4. 사용자 포탈 배포](#18)
     - [3.5. 운영자 포탈 배포](#19)
     - [3.6. 앱 오토스케일러 배포](#19-1)
     - [3.7. 카탈로그 이미지 파일 업로드](#20)
- 4. [테스트 케이스 구동 가이드](#21)
     - [4.1. 테스트시 절차](#22)



# <div id='1'> 1. 문서 개요



### <div id='2'> 1.1. 목적
      
본 문서(Paas-TA Potal 배포 가이드)는 전자정부표준프레임워크 기반의 PaaS-TA 에서 배포 되는 Potal을 PaaS-TA를 이용하여 설치 하는 방법을 기술하였다.



### <div id='3'> 1.2. 범위 

배포 범위는 PaaS-TA Potal을 검증하기 위한 기본 설치 및 배포를 기준으로 작성하였다. 



### <div id='4'> 1.3. 시스템 구성도
본 문서의 설치된 시스템 구성도입니다. 사용자 포탈 1, 운영자 포탈 1, 포탈 API 1, 포탈 APIV2 1, 포탈 Registration 1, 포탈 오토스케일 1 로 최소사항을 구성하였다.

![portal_deploy_image_01]

<table>
  <tr>
    <td>어플리케이션명</td>
    <td>인스턴스 수</td>
    <td>메모리</td>
    <td>디스크</td>
  </tr>
  <tr>
    <td>portal-web-user</td>
    <td>N</td>
    <td>1GB</td>    
    <td>1GB Disk</td>
  </tr>
  <tr>
    <td>portal-web-admin</td>
    <td>N</td>
    <td>1GB</td>    
    <td>1GB Disk</td>
  </tr>
  <tr>
    <td>portal-api</td>
    <td>N</td>
    <td>1GB</td>    
    <td>1GB Disk</td>
  </tr>
  <tr>
    <td>portal-registration</td>
    <td>1</td>
    <td>512MB</td>    
    <td>1GB Disk</td>
  </tr>
  <tr>
    <td>portal-api-v2</td>
    <td>N</td>
    <td>1GB</td>    
    <td>1GB Disk</td>
  </tr>
  <tr>
    <td>app-autoscaler</td>
    <td>1</td>
    <td>1GB</td>    
    <td>1GB Disk</td>
  </tr>  
</table>



### <div id='5'> 1.4. 참고자료
[**http://bosh.io/docs**](http://bosh.io/docs)
[**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/)



# <div id='6'> 2. 설치전 준비사항



### <div id='7'> 2.1. 기본설치 항목

본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다. 서비스팩 설치를 위해서는 먼저 BOSH CLI 가 설치 되어 있어야 하고 BOSH에 로그인 및 target 설정이 되어 있어야 한다. BOSH CLI 가 설치 되어 있지 않을 경우 먼저 BOSH 설치 가이드 문서를 참고 하여 BOSH CLI를 설치 해야 한다.
PaaSTA-Portal 폴더에서 설치에 필요한 파일을 확인한다.

-	PaaS-TA에서 제공하는 포털 배포에 필요한 파일들을 다운받는다. (PaaSTA-Portal.zip)

- 다운로드 위치
>PaaSTA-Portal : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Portal.zip>**  
>PaaSTA-Deployment : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Deployment.zip>**



### <div id='23'>  2.2. 사용자의 조직 생성 Flag 활성화
PaaS-TA는 기본적으로 일반 사용자는 조직을 생성할 수 없도록 설정되어 있다. 포털 배포를 위해 조직 및 공간을 생성해야 하고 또 테스트를 구동하기 위해서도 필요하므로 사용자가 조직을 생성할 수 있도록 user_org_creation FLAG를 활성화 한다. FLAG 활성화를 위해서는 PaaS-TA 운영자 계정으로 로그인이 필요하다.

```
$ cf enable-feature-flag user_org_creation
```
```
Setting status of user_org_creation as admin...
OK

Feature user_org_creation Enabled.
```


### <div id='8'> 2.3. 조직 및 공간 생성

- PaaS-TA 어드민 계정으로 포탈을 배포할 조직 및 공간을 생성하거나 배포할 공간으로 target 설정을 한다.
```
$ cf create-org <조직명>
```
```
$ cf create-space <공간명>
```
```
$ cf target –o <조직명> -s <공간명>
```



### <div id='9'> 2.4. Redis 서비스 브로커 등록 및 활성화


- Redis 서비스 브로커를 확인한다.
```
$ cf service-brokers
```
```
Getting service brokers as admin...

name                     url
paasta-pinpoint-broker   http://10.30.70.82:8080
paasta-redis-broker      http://10.30.60.71:12350
```


- Redis 서비스 브로커가 생성되어 있지 않은 경우 [[**PaaS-TA Redis 서비스팩 설치 가이드**](https://github.com/OpenPaaSRnD/Documents-PaaSTA-2.0/blob/master/Service-Guide/NoSQL/PaaS-TA%20Redis%20%EC%84%9C%EB%B9%84%EC%8A%A4%ED%8C%A9%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C.md)]를 참고하여 Redis 서비스팩을 설치하고 서비스 브로커를 생성 및 활성화한다. 다음 명령어를 통해 Redis 서비스가 활성화 되었는지 확인할 수 있다.

```
$ cf marketplace
```
```
Getting services from marketplace in org OCP/ space prd as admin...
OK

service   plans                     description
redis    shared-vm, dedicated-vm   Redis service to provide a key-value store

TIP:  Use 'cf marketplace -s SERVICE' to view descriptions of individual plans of a given service.
```



###  <div id='10'> 2.5. Redis 서비스 인스턴스 생성

- Redis 서비스가 활성화 되면 서비스 인스턴스를 생성할 수 있다. 포털이 사용할 Redis 서비스의 서비스 인스턴스를 생성한다.
```
$ cf create-service redis shared-vm portal-redis-session
```
```
Creating service instance portal-redis-session in org OCP/ space prd as admin...
OK
```



### <div id='11'>  2.6. 유레카 사용자 제공 서비스 생성

- 사용자 서비스 eureka user provide service 등록

```
$ cf cups portal-eureka-service -p '{"uri":"http://portal-registration.115.68.46.186.xip.io"}'
```
##### Windows 환경에서 명령어 입력시
```
$ cf cups portal-eureka-service -p '{\"uri\":\"http://portal-registration.monitoring.open-paas.com\"}'
```

```
Creating user provided service portal-eureka-service in org OCP/ space prd as admin...
OK
```


- eureka user provide service 등록 확인
```
$ cf services
```
```
Getting services in org OCP/ space prd as admin...
OK

name                    service         plan        bound apps   last operation
portal-redis-session   redis           shared-vm                create succeeded
portal-eureka-service  user-provided
```



### <div id='12'>  2.7. Portal Object Storage 설치 및 설정 변경

포털은 파일 관리를 위해 Object Storage를 사용하기 때문에 PaaSTA 포털 Object Storage를 설치 하여야 한다.  [[**PaaS-TA 포탈 Obejct Storage 설치 가이드**](https://github.com/OpenPaaSRnD/Documents-PaaSTA-2.0/blob/master/Use-Guide/PaaS-TA%20%ED%8F%AC%ED%83%88%20Object%20Storage%20%EC%84%A4%EC%B9%98%20%EA%B0%80%EC%9D%B4%EB%93%9C.md)]를 참고하여 Object Storage를 설치한다.
Object Storage 설치가 완료되었다면, Portal API manifest.yml 파일에 설정된 값을 수정해야 한다. Object Storage 설치 시 입력한 값을 바탕으로 다음 항목의 값을 수정한다.

![portal_deploy_image_02]

※ 중괄호({}) 안의 값은 Object Storage의 deployment 파일 설정값

1.  spring_objectStorage_tenantName : {properties.keystone_tenantname}
2.  spring_objectStorage_username : {properties.keystone_username}
3.  spring_objectStorage_password : {properties.keystone_password}
4.  spring_objectStorage_authUrl : http://{properties.proxy_ip}:{properties.keystone_auth_port}/v2.0



###  <div id='13'> 2.8.  Postgresql 기본 데이터 베이스 생성
PaaS-TA-Portal 서비스를 하기 위해 배포 파일이 있는 PaaSTA-Portal/postgresql/의 portal-postgresql-init.sh, postgresql.sql을 실행하여야 한다.


※ postgresql.sql을 실행하기 전에 파일을 열어 관리자 계정의 정보를 Potal-DB에 삽입하는 SQL을 추가한다. 관리자 계정의 정보를 Portal DB에 삽입하지 않을 경우, 관리자 포털(Web Admin)에 로그인했을때, 관리자 메뉴에 접근 할 수 없는 오류가 발생할 수 있다. 

postgresql.sql 파일을 vi 에티더로 연다.
```
$ vi postgresql.sql
```

3958 라인으로 이동한다.
```
:3958
```

관리자 계정을 Portal DB의 user_detail 컬럼에 삽입하는 SQL을 추가한다. 현재 user_id를 '관리자 계정'이라는 값으로 삽입하도록 작성되어 있는데, Portal API 배포시 manifest.yml 파일에 입력한 관리자 ID와 동일한 값으로 변경한다. Portal API의 manifest.yml의 'cloudfoundry_user_admin_username' 값이 관리자 계정 ID가 된다. [[**3.2. 포탈 API 배포**](#16)] 를 참고한다.
```
INSERT INTO user_detail (user_id, status, tell_phone, zip_code, address, address_detail, user_name, admin_yn, refresh_token, auth_access_cnt, auth_access_time, img_path) VALUES ('관리자 계정 ID', '1', '01012345678', NULL, NULL, NULL, 'name', 'Y', NULL, 0, NULL, NULL);
```



- CloudFoundry내의 PostgreSql이 설치 되어있는 곳을 확인한다.

```
$ bosh vms paasta-controller-vsphere
```
```
Acting as user 'admin' on deployment 'cf-warden' on 'my-bosh'

Director task 1940

Task 1940 done

+---------------------------------------------------------------------------+---------+-----+-----------+----------------+
| VM                                                                        | State   | AZ  | VM Type   | IPs            |
+---------------------------------------------------------------------------+---------+-----+-----------+----------------+
| api_worker_z1/0 (73d2f2ed-9725-4693-bc5d-27a4ca6b2d08)                    | running | n/a | small_z1  | 10.244.0.104   |
| api_z1/0 (fdf37975-ecc0-482e-bdfc-1e1793da2909)                           | running | n/a | large_z1  | 10.244.0.103   |
| blobstore_z1/0 (e02a2870-3dae-4b86-9a6e-54131adb9ebd)                     | running | n/a | medium_z1 | 10.244.0.101   |
| consul_z1/0 (4bb07da6-8a0b-4f8c-aa1f-bcef51a878ec)                        | running | n/a | small_z1  | 10.244.0.154   |
| doppler_z1/0 (1b6be8af-5732-4ab5-b001-e2d6151200ec)                       | running | n/a | medium_z1 | 10.244.0.105   |
| etcd_z1/0 (5351e4fe-1f0d-4ae1-91cd-7888c27300c8)                          | running | n/a | medium_z1 | 10.244.0.142   |
| ha_proxy_z1/0 (23c154b7-0528-48f1-81c1-899022d74e02)                      | running | n/a | router_z1 | 10.244.0.134   |
|                                                                           |         |     |           | 115.68.151.188 |
| loggregator_trafficcontroller_z1/0 (60390a77-d02e-4c06-8bfd-370310b32833) | running | n/a | small_z1  | 10.244.0.106   |
| nats_z1/0 (cb8d1c32-359d-48ea-85f3-0b296244b525)                          | running | n/a | medium_z1 | 10.244.0.116   |
| postgres_z1/0 (dd765317-34a2-4f6f-8ac1-e724fd1a7165)                      | running | n/a | medium_z1 | 10.244.0.130   |
| router_z1/0 (996ecb9a-bc97-4730-9fee-2610ef978fdb)                        | running | n/a | router_z1 | 10.244.0.122   |
| router_z1/1 (a402f6c3-3563-4b74-a6d9-9db03ebf10c9)                        | running | n/a | router_z1 | 10.244.0.123   |
| uaa_z1/0 (b54d5358-48d3-4c1e-9f3c-2c23fb74f42b)                           | running | n/a | medium_z1 | 10.244.0.102   |
+---------------------------------------------------------------------------+---------+-----+-----------+----------------+

VMs total: 13
```


- 배포할 서버에 postgresql 폴더안의 파일을 업로드한다.
```
scp 파일명 vcap@서버ip://복사할 폴더위치
$ scp  * vcap@10.244.0.130:/home/vcap/
```

 
- PostgreSql 서버에 접속하여 파일을 확인한다.
```
$ ssh vcap@10.244.0.130
```
```
Unauthorized use is strictly prohibited. All access and activity

is subject to logging and monitoring.
vcap@10.244.0.130's password: 
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-59-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
Last login: Wed Jan 18 06:33:28 UTC 2017 from 10.30.20.23 on ssh
Last login: Wed Jan 18 06:33:55 2017 from 10.30.20.23
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
```


```
$ ll
```
```
total 224
drwxr-xr-x 3 vcap vcap   4096 Jan 18 06:33 ./
drwxr-xr-x 3 root root   4096 Jan 12 17:06 ../
-rw------- 1 vcap vcap     29 Jan 18 06:30 .bash_history
-rw-r--r-- 1 vcap vcap    220 Apr  9  2014 .bash_logout
-rw-r--r-- 1 vcap vcap   3708 Jan 12 17:06 .bashrc
drwx------ 2 vcap vcap   4096 Jan 18 06:25 .cache/
-rw-rw-r-- 1 vcap vcap    753 Jan 18 06:33 portal-postgresql-init.sh
-rw-rw-r-- 1 vcap vcap 192641 Jan 18 06:26 postgresql.sql
-rw-r--r-- 1 vcap vcap    675 Apr  9  2014 .profile
```


- 스크립트를 확인하여, 생성할 데이터베이스 명, postgresql vesrsion 및 사용자를 확인한다.
```
$ vi ./portal-postgresql-init.sh
```
```
#!/bin/bash

# ENVIRONMENTS
PSQL_VERSION=postgres-9.4.9
PSQL_USER=vcap
PSQL_PORT=5524
PSQL_BIN_DIR=/var/vcap/packages/$PSQL_VERSION/bin
INIT_SQL_DIR=.

# DROP
$PSQL_BIN_DIR/psql -U $PSQL_USER -p $PSQL_PORT -d postgres -c "DROP DATABASE \"portaldb\""
$PSQL_BIN_DIR/psql -U $PSQL_USER -p $PSQL_PORT -d postgres -c "DROP ROLE \"portaladmin\""

# CREATE
$PSQL_BIN_DIR/psql -U $PSQL_USER -p $PSQL_PORT -d postgres -c "CREATE ROLE \"portaladmin\""
$PSQL_BIN_DIR/psql -U $PSQL_USER -p $PSQL_PORT -d postgres -c "ALTER ROLE \"portaladmin\" WITH LOGIN PASSWORD 'admin'"
$PSQL_BIN_DIR/psql -U $PSQL_USER -p $PSQL_PORT -d postgres -c "CREATE DATABASE \"portaldb\""
$PSQL_BIN_DIR/psql -U portaladmin -p $PSQL_PORT -d portaldb -a -f $INIT_SQL_DIR/postgresql.sql
```

- 스크립트 파일을 실행한다.
(실행권한이 없을경우 .chmod +x 파일명을 실행하여 실행권한을 부여한다.)
```
$ ./portal-postgresql-init.sh
```
```
…….
--
ALTER TABLE ONLY web_ide_user
    ADD CONSTRAINT web_ide_user_primary PRIMARY KEY (org_name);
ALTER TABLE
--
-- Name: public; Type: ACL; Schema: -; Owner: vcap
--
REVOKE ALL ON SCHEMA public FROM PUBLIC;
psql:./postgresql.sql:4837: WARNING:  no privileges could be revoked for "public"
REVOKE
REVOKE ALL ON SCHEMA public FROM vcap;
psql:./postgresql.sql:4838: WARNING:  no privileges could be revoked for "public"
REVOKE
GRANT ALL ON SCHEMA public TO vcap;
psql:./postgresql.sql:4839: WARNING:  no privileges were granted for "public"
GRANT
GRANT ALL ON SCHEMA public TO PUBLIC;
psql:./postgresql.sql:4840: WARNING:  no privileges were granted for "public"
GRANT
--
-- PostgreSQL database dump complete
--
```

- PostgreSql 서버에 접속하여 기본데이터가 입력되었는지 확인한다.
```
$ /var/vcap/packages/postgres-9.4.9/bin/psql -U portaladmin -p 5524 -d portaldb
```
```
portaldb-> \du
```
```
                              List of roles
  Role name  |                   Attributes                   | Member of 
-------------+------------------------------------------------+-----------
 ccadmin     |                                                | {}
 diego       |                                                | {}
 portaladmin |                                                | {}
 uaaadmin    |                                                | {}
 vcap        | Superuser, Create role, Create DB, Replication | {}

```


- Postgresql Database 목록을 조회한다. 
```
$ /var/vcap/packages/postgres-9.4.9/bin/psql -U portaladmin -p 5524 -d portaldb
```
```
portaldb-> \l
```
```
                              List of databases
   Name    | Owner | Encoding |   Collate   |    Ctype    | Access privileges 
-----------+-------+----------+-------------+-------------+-------------------
 ccdb      | vcap  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 diego     | vcap  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 portaldb  | vcap  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | vcap  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | vcap  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/vcap          +
           |       |          |             |             | vcap=CTc/vcap
 template1 | vcap  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/vcap          +
           |       |          |             |             | vcap=CTc/vcap
 uaadb     | vcap  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 

```



# <div id='14'> 3.  PaaS-TA 배포



### <div id='15'>  3.1.  포탈 Registration 배포


- PaaS-TA-Portal 서비스를 하기위해 배포 파일이 있는 PaaSTA-Portal\registraion\에서 파일을 배포할 서버에 복사한다.


- manifest를 확인한다.
manifest는 components 요소 및 배포의 속성을 정의한 YAML  파일이다.
Deployment manifest 에는 sotfware를 설치 하기 위해서 어떤 name, memory, instance, host, path, buildpack, env등을 사용 할 것인지 정의가 되어 있다.


- portal-registration의 manifest.yml 확인 한다.

```
$ vi manifest.yml
```

```
yml
---
applications:
- name: portal-registration
  memory: 512M
  instances: 1
  host: portal-registration
  path: ./paasta-portal-registration-1.0.war
  buildpack: java_buildpack_offline
  env:
    eureka_instance_hostname: localhost
    eureka_client_registerWithEureka: false
    eureka_client_fetchRegistry: false
    eureka_client_healthcheck_enabled: true
    eureka_server_enableSelfPreservation: true

    server_port: 2221
```


- manifest.yml이 있는 폴더로 이동하여 cf push 명령어를 이용하여 배포한다.
```
$ cf push
```
```
Using manifest file /home/inception/bosh-space/kimdojun/paasta-portal/registration/manifest.yml

Creating app portal-registration in org OCP/ space prd as admin...
OK

Creating route portal-registration.115.68.46.186.xip.io...
OK

Binding portal-registration.115.68.46.186.xip.io to portal-registration...
OK

Uploading portal-registration...
Uploading app files from: /tmp/unzipped-app800851821
Uploading 921.7K, 113 files
Done uploading               
OK

Starting app portal-registration in org OCP/ space prd as admin...
Downloading java_buildpack_offline...
Downloaded java_buildpack_offline
Creating container
Successfully created container
Downloading app package...
Downloaded app package (34.4M)
Staging...
-----> Java Buildpack Version: v3.10 (offline) | https://github.com/cloudfoundry/java-buildpack.git#193d6b7
-----> Downloading Open Jdk JRE 1.8.0_111 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_111.tar.gz (found in cache)
      Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.5s)
-----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
      Memory Settings: -Xmx681574K -XX:MaxMetaspaceSize=104857K -Xss349K -Xms681574K -XX:MetaspaceSize=104857K
-----> Downloading Container Customizer 1.0.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-customizer/container-customizer-1.0.0_RELEASE.jar (found in cache)
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading droplet...
Uploading build artifacts cache...
Uploaded build artifacts cache (107B)
Uploaded droplet (79.5M)
Uploading complete
Destroying container
Successfully destroyed container

of 1 instances running, 1 starting
of 1 instances running, 1 starting
of 1 instances running, 1 starting
of 1 instances running

App started


OK

App portal-registration was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100% -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.WarLauncher`

Showing health and status for app portal-registration in org OCP/ space prd as admin...
OK

requested state: started
instances: 1/1
usage: 512M x 1 instances
urls: portal-registration.115.68.46.186.xip.io
last uploaded: Thu Jan 19 05:31:12 UTC 2017
stack: cflinuxfs2
buildpack: java_buildpack_offline

    state     since                    cpu    memory           disk         details
#0  running   2017-01-19 02:32:10 PM   0.0%   272.8M of 512M   162M of 1G
```

- 배포 확인
```
$ cf apps
```
```
Getting apps in org OCP/ space prd as admin...
OK

name                  requested state   instances   memory   disk   urls
portal-registration   started           1/1         512M     1G     portal-registration.115.68.46.186.xip.io
```



### <div id='16'> 3.2. 포탈 API 배포


- portal-api 의 manifest.yml확인
```
$ vi manifest.yml
```
```
yml
---
applications:
- name: portal-api                         # 앱 이름
  memory: 1536M                            # 앱 메모리 크기
  instances: 2                             # 앱 인스턴스 수
  host: portal-api                         # 앱과 바인드될 호스트
  path: ./paasta-portal-api-1.0.war        # war 파일 경로
  buildpack: java_buildpack_offline        # 앱이 사용할 빌드팩 이름
  services:                                # 앱과 바인드되는 서비스 목록
  - portal-eureka-service                  # 사용자 생성 서비스(UserProvided Service) 유레카
  env:
    SPRING_PROFILES_ACTIVE: prd
    spring_application_name: portal-api    # 앱 이름
    spring_jdbc: postgresql                # PaaS-TA 포털이 사용할 DBMS
    server_port: 2222
    cloudfoundry_cc_api_url: https://api.115.68.46.186.xip.io     # PaaS-TA CloudController의 api Url
    cloudfoundry_cc_api_uaaUrl: https://uaa.115.68.46.186.xip.io  # PaaS-TA Uaa Url

    # PaaS-TA Uaa 계정 정보
    cloudfoundry_user_admin_username: admin
    cloudfoundry_user_admin_password: admin
    cloudfoundry_user_uaaClient_clientId: login
    cloudfoundry_user_uaaClient_clientSecret: login-secret
    cloudfoundry_user_uaaClient_adminClientId: admin
    cloudfoundry_user_uaaClient_adminClientSecret: admin-secret
    cloudfoundry_user_uaaClient_loginClientId: login
    cloudfoundry_user_uaaClient_loginClientSecret: login-secret

    cloudfoundry_user_uaaClient_skipSSLValidation: true       # Uaa와 통신할 때, ssl 유효성 체크 skip 여부. 기본값 true이며 true 일때 유효성 체크 하지 않음
    cloudfoundry_authorization: cf-Authorization

    abacus_url: http://paasta-usage-reporting.115.68.46.186.xip.io/v1     # 모니터링 앱 Url

    # 스프링 시큐리티 계정 정보
    spring_security_username: admin
    spring_security_password: openpaasta

    # PaaS-TA CCDB 접속 정보. 
    # PaaS-TA Cloud Controller Deployment 파일인 paasta-controller-2.0-{IaaS 종류}.yml 파일을 참조하여 작성 
    spring_datasource_cc_driverClassName: org.postgresql.Driver
    spring_datasource_cc_url: jdbc:postgresql://10.30.150.42:5524/ccdb
    spring_datasource_cc_username: ccadmin
    spring_datasource_cc_password: admin

    # PaaS-TA 포털 DB 접속 정보.
    # PaaS-TA Cloud Controller Deployment 파일인 paasta-controller-2.0-{IaaS 종류}.yml 파일을 참조하여 작성 
    spring_datasource_portal_driverClassName: org.postgresql.Driver
    spring_datasource_portal_url: jdbc:postgresql://10.30.150.42:5524/portaldb
    spring_datasource_portal_username: portaladmin
    spring_datasource_portal_password: admin

    # PaaS-TA UAA DB 접속 정보.
    # PaaS-TA Cloud Controller Deployment 파일인 paasta-controller-2.0-{IaaS 종류}.yml 파일을 참조하여 작성 
    spring_datasource_uaa_driverClassName: org.postgresql.Driver
    spring_datasource_uaa_url: jdbc:postgresql://10.30.150.42:5524/uaadb
    spring_datasource_uaa_username: uaaadmin
    spring_datasource_uaa_password: admin

    # PaaS-TA 포털 Object Storage 접속 정보.
    # 포털 Object Storage Deployment 파일인 paasta_portal_object_storage_{IaaS 종류}_2.0.yml 파일을 참조하여 작성
    spring_objectStorage_tenantName: paasta-portal
    spring_objectStorage_username: paasta-portal
    spring_objectStorage_password: paasta
    spring_objectStorage_authUrl: http://10.30.131.12:5000/v2.0
    spring_objectStorage_container: portal-container
    
    # 포털 SMTP 정보
    spring_mail_smtp_host: smtp.gmail.com
    spring_mail_smtp_port: 465
    spring_mail_smtp_username: PaaS-TA 관리자
    spring_mail_smtp_password: openpasta!
    spring_mail_smtp_userEmail: openpasta@gmail.com
    spring_mail_smtp_properties_auth: true
    spring_mail_smtp_properties_starttls_enable: true
    spring_mail_smtp_properties_starttls_required: true
    spring_mail_smtp_properties_maximumTotalQps: 90
    spring_mail_smtp_properties_authUrl: http://portal-web-user.115.68.46.186.xip.io         # PaaS-TA 사용자 포털 Url
    spring_mail_smtp_properties_imgUrl: http://52.201.48.51:8080/v1/KEY_84586dfdc15e4f8b9c2a8e8090ed9810/portal-container/65bdc7f43e11433b8f17683f96c7e626.png                                    # PaaS-TA 로고 이미지 Url   
    spring_mail_smtp_properties_sFile: emailTemplate.html
    spring_mail_smtp_properties_subject: PaaS-TA User Potal 인증메일
    spring_mail_smtp_properties_contextUrl: user/authUser

    multipart_maxFileSize: 1000Mb
    multipart_maxRequestSize: 1000Mb

    # 사용자 생성 서비스(UserProvided Service) 유레카 접속 정보를 포털 API 앱의 환경정보에서 읽을 수 있도록 설정되어 있음.
    eureka_instance_hostname: ${vcap.application.uris[0]}
    eureka_instance_nonSecurePort: 80
    eureka_client_serviceUrl_defaultZone: ${vcap.services.portal-eureka-service.credentials.uri}/eureka/
```

- manifest.yml이 있는 폴더로 이동하여 cf push 명령어를 이용하여 배포한다.
```
$ cf push
```
```
Using manifest file /home/inception/bosh-space/kimdojun/paasta-portal/api/manifest.yml

Creating app portal-api in org OCP/ space prd as admin...
OK

Using route portal-api.115.68.46.186.xip.io
Binding portal-api.115.68.46.186.xip.io to portal-api...
OK

Uploading portal-api...
Uploading app files from: /tmp/unzipped-app140134101
Uploading 1.8M, 434 files
Done uploading               
OK
Binding service portal-eureka-service to app portal-api in org OCP/ space prd as admin...
OK

Starting app portal-api in org OCP/ space prd as admin...
Downloading java_buildpack_offline...
Downloaded java_buildpack_offline
Creating container
Successfully created container
Downloading app package...
Downloaded app package (67.3M)
Staging...
-----> Java Buildpack Version: v3.10 (offline) | https://github.com/cloudfoundry/java-buildpack.git#193d6b7
-----> Downloading Open Jdk JRE 1.8.0_111 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_111.tar.gz (found in cache)
      Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.4s)
-----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
      Memory Settings: -Xms1022361K -XX:MetaspaceSize=157286K -XX:MaxMetaspaceSize=157286K -Xss524K -Xmx1022361K
-----> Downloading Container Customizer 1.0.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-customizer/container-customizer-1.0.0_RELEASE.jar (found in cache)
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (109B)
Uploaded droplet (112.2M)
Uploading complete
Destroying container
Successfully destroyed container

of 2 instances running, 2 starting
of 2 instances running, 2 starting
of 2 instances running, 2 starting
of 2 instances running, 2 starting
of 2 instances running, 2 starting
of 2 instances running, 2 starting
of 2 instances running, 2 starting
of 2 instances running, 2 starting
of 2 instances running

App started


OK

App portal-api was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100% -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.WarLauncher`

Showing health and status for app portal-api in org OCP/ space prd as admin...
OK

requested state: started
instances: 2/2
usage: 1.5G x 2 instances
urls: portal-api.115.68.46.186.xip.io
last uploaded: Thu Jan 19 05:40:12 UTC 2017
stack: cflinuxfs2
buildpack: java_buildpack_offline

    state     since                    cpu      memory           disk           details
#0  running   2017-01-19 02:41:55 PM   35.1%    607.3M of 1.5G   200.8M of 1G
#1  running   2017-01-19 02:41:56 PM   122.8%   637.6M of 1.5G   200.8M of 1G
```

- 배포 확인
```
$ cf apps
```
```
Getting apps in org OCP/ space prd as admin...
OK

name                  requested state   instances   memory   disk   urls
portal-api           started           2/2         1.5G     1G     portal-api.115.68.46.186.xip.io
portal-registration  started           1/1         512M     1G     portal-registration.115.68.46.186.xip.io
```



### <div id='17'> 3.3. 포탈 APIV2 배포

배포 어플리케이션은 potal-api 어플리케이션과는 다른 cloudfoundry version의 api 서비스를 통한 기능개발을 위해 배포 되어야 하는 어플리케이션이다.


- portal-api-v2 의 manifest.yml확인
```
$ vi manifest.yml
```
```
yml
---
applications:
- name: portal-api-v2                       # 앱 이름
  memory: 1024M                             # 앱 메모리 크기
  instances: 1                              # 앱 인스턴스 수
  host: portal-api-v2                       # 앱과 바인드될 호스트
  path: ./paasta-portal-api-v2-1.0.war      # war 파일 경로
  buildpack: java_buildpack_offline         # 앱이 사용할 빌드팩 이름
  services:                                 # 앱과 바인드되는 서비스 목록
  - portal-eureka-service                   # 사용자 생성 서비스(UserProvided Service) 유레카
  env:
    spring_application_name: portal-api-v2  # 앱 이름
    server_port: 3333

    # 스프링 시큐리티 계정 정보
    security_user_name: admin
    security_user_password: openpaasta

    multipart_maxFileSize: 1000Mb
    multipart_maxRequestSize: 1000Mb

    # 사용자 생성 서비스(UserProvided Service) 유레카 접속 정보를 포털 API 앱의 환경정보에서 읽을 수 있도록 설정되어 있음.
    eureka_instance_hostname: ${vcap.application.uris[0]}
    eureka_client_serviceUrl_defaultZone: ${vcap.services.portal-eureka-service.credentials.uri}/eureka/
    

    cf_apiHost: api.115.68.46.186.xip.io     # PaaS-TA API 호스트
    cf_sslSkipValidation: true
    cf_clientId: cf
    #cf_clientSecret:
    cf_username: admin
    cf_password: admin

    # PaaS-TA CCDB 접속 정보
    # PaaS-TA Cloud Controller Deployment 파일인 paasta-controller-2.0-{IaaS 종류}.yml 파일을 참조하여 작성 
    datasource_cc_driverClassName: org.postgresql.Driver
    datasource_cc_url: jdbc:postgresql://10.30.150.42:5524/ccdb
    datasource_cc_username: ccadmin
    datasource_cc_password: admin

    # PaaS-TA 포털 DB 접속 정보
    # PaaS-TA Cloud Controller Deployment 파일인 paasta-controller-2.0-{IaaS 종류}.yml 파일을 참조하여 작성 
    datasource_portal_driverClassName: org.postgresql.Driver
    datasource_portal_url: jdbc:postgresql://10.30.150.42:5524/portaldb
    datasource_portal_username: portaladmin
    datasource_portal_password: admin

    # PaaS-TA 포털 Object Storage 접속 정보
    # 포털 Object Storage Deployment 파일인 paasta_portal_object_storage_{IaaS 종류}_2.0.yml 파일을 참조하여 작성
    datasource_uaa_driverClassName: org.postgresql.Driver
    datasource_uaa_url: jdbc:postgresql://10.30.150.42:5524/uaadb
    datasource_uaa_username: uaaadmin
    datasource_uaa_password: admin
```


- manifest.yml이 있는 폴더로 이동하여 cf push 명령어를 이용하여 배포한다.
```
$ cf push
```
```
Using manifest file /home/inception/bosh-space/kimdojun/paasta-portal/api-v2/manifest.yml

Creating app portal-api-v2 in org OCP/ space prd as admin...
OK

Using route portal-api-v2.115.68.46.186.xip.io
Binding portal-api-v2.115.68.46.186.xip.io to portal-api-v2...
OK

Uploading portal-api-v2...
Uploading app files from: /tmp/unzipped-app446087426
Uploading 1.8M, 266 files
Done uploading               
OK
Binding service portal-eureka-service to app portal-api-v2 in org OCP/ space prdas admin...
OK

Starting app portal-api-v2 in org OCP/ space prd as admin...
Downloading java_buildpack_offline...
Downloaded java_buildpack_offline
Creating container
Successfully created container
Downloading app package...
Downloaded app package (63.9M)
Staging...
-----> Java Buildpack Version: v3.10 (offline) | https://github.com/cloudfoundry/java-buildpack.git#193d6b7
-----> Downloading Open Jdk JRE 1.8.0_111 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_111.tar.gz (found in cache)
      Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.4s)
-----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
      Memory Settings: -XX:MetaspaceSize=105267K -Xmx684236K -XX:MaxMetaspaceSize=105267K -Xss350K -Xms684236K
-----> Downloading Container Customizer 1.0.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-customizer/container-customizer-1.0.0_RELEASE.jar (found in cache)
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (109B)
Uploaded droplet (108.9M)
Uploading complete
Destroying container
Successfully destroyed container

of 1 instances running, 1 starting
of 1 instances running, 1 starting
of 1 instances running, 1 starting
of 1 instances running, 1 starting
of 1 instances running

App started


OK

App portal-api-v2 was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100% -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORTeval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.WarLauncher`

Showing health and status for app portal-api-v2 in org OCP/ space prd as admin...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: portal-api-v2.115.68.46.186.xip.io
last uploaded: Thu Jan 19 05:46:46 UTC 2017
stack: cflinuxfs2
buildpack: java_buildpack_offline

    state     since                    cpu    memory         disk           details
#0  running   2017-01-19 02:48:02 PM   0.0%   370.2M of 1G   196.2M of 1G
```


- 배포 확인
```
$ cf apps
```
```
Getting apps in org OCP/ space prd as admin...
OK

name                  requested state   instances   memory   disk   urls
portal-api           started           2/2         1.5G     1G     portal-api.115.68.46.186.xip.io
portal-registration  started           1/1         512M     1G     portal-registration.115.68.46.186.xip.io
portal-api-v2        started           1/1         1G       1G     portal-api-v2.115.68.46.186.xip.io
```


### <div id='18'> 3.4. 사용자 포탈 배포


- manifest.yml확인
```
$ vi manifest.yml
```
```
yml
---
applications:
- name: portal-web-user
  memory: 1024M
  instances: 1
  host: portal-web-user
  path: ./paasta-portal-web-user-1.0.war
  buildpack: java_buildpack_offline
  services:
  - portal-eureka-service
  - portal-redis-session
  env:
    spring_application_name: portal-web-user
    spring_redis_host: ${vcap.services.portal-redis-session.credentials.host}
    spring_redis_password: ${vcap.services.portal-redis-session.credentials.password}
    spring_redis_port: ${vcap.services.portal-redis-session.credentials.port}
    spring_redis_timeout: 2100

    multipart_maxFileSize: 1000Mb
    multipart_maxRequestSize: 1000Mb

    server_port: 8080

    eureka_client_serviceUrl_defaultZone: ${vcap.services.portal-eureka-service.credentials.uri}/eureka/
    eureka_instance_hostname: ${vcap.application.uris[0]}

    # api manifest의 spring_security_username + ":" + spring_security_password를 Base64 인코딩하여 입력해야 합니다.
    # 예를 들어서 api manifest의 값이 아래와 같다면
    # spring_security_username: user
    # spring_security_password: password
    # user:password 를 인코딩해야합니다.
    # Base64 인코딩 사이트: http://www.convertstring.com/ko/EncodeDecode/Base64Encode
    paasta_portal_api_authorization_base64: Basic YWRtaW46b3BlbnBhYXN0YQ==
    paasta_portal_api_url: http://PORTAL-API

    ribbon_eureka_enabled: true
    ribbon_ConnectTimeout: 30000
    ribbon_ReadTimeout: 30000
    eureka_instance_hostname: ${vcap.application.uris[0]}
    eureka_instance_nonSecurePort: 80
    eureka_client_serviceUrl_defaultZone: ${vcap.services.portal-eureka-service.credentials.uri}/eureka/
```


- manifest.yml이 있는 폴더로 이동하여 cf push 명령어를 이용하여 배포한다.
```
$ cf push
```
```
Using manifest file /home/inception/bosh-space/kimdojun/paasta-portal/portal-web-user/manifest.yml

Creating app portal-web-user in org OCP/ space prd as admin...
OK

Creating route portal-web-user.115.68.46.186.xip.io...
OK

Binding portal-web-user.115.68.46.186.xip.ioto portal-web-user...
OK

Uploading portal-web-user...
Uploading app files from: /tmp/unzipped-app630250741
Uploading 9.6M, 3177 files
Done uploading               
OK
Binding service portal-eureka-serviceto app portal-web-user in org OCP/ space prd as admin...
OK
Binding service portal-redis-sessionto app portal-web-user in org OCP/ space prd as admin...
OK

Starting app portal-web-user in org OCP/ space prd as admin...
Downloading java_buildpack_offline...
Downloaded java_buildpack_offline
Creating container
Successfully created container
Downloading app package...
Downloaded app package (44.6M)
Staging...
-----> Java Buildpack Version: v3.10 (offline) | https://github.com/cloudfoundry/java-buildpack.git#193d6b7
-----> Downloading Open Jdk JRE 1.8.0_111 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_111.tar.gz (found in cache)
      Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.4s)
-----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
      Memory Settings: -Xss349K -Xmx681574K -XX:MaxMetaspaceSize=104857K -Xms681574K -XX:MetaspaceSize=104857K
-----> Downloading Container Customizer 1.0.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-customizer/container-customizer-1.0.0_RELEASE.jar (found in cache)
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (109B)
Uploaded droplet (88.4M)
Uploading complete
Destroying container
Successfully destroyed container

of 1 instances running, 1 starting
of 1 instances running, 1 starting
of 1 instances running, 1 starting
of 1 instances running, 1 starting
of 1 instances running

App started


OK

App portal-web-user was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100% -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.WarLauncher`

Showing health and status for app portal-web-user in org OCP/ space prd as admin...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: portal-web-user.115.68.46.186.xip.io
last uploaded: Thu Jan 19 05:54:09 UTC 2017
stack: cflinuxfs2
buildpack: java_buildpack_offline

    state     since                    cpu      memory         disk         details
#0  running   2017-01-19 02:55:25 PM   245.2%   469.8M of 1G   204M of 1G
```


- 배포 확인
```
$ cf apps
```
```
Getting apps in org OCP/ space prdas admin...
OK

name                  requested state   instances   memory   disk   urls
portal-api           started           2/2         1.5G     1G     portal-api.115.68.46.186.xip.io
portal-web-user      started           1/1         1G       1G     portal-web-user.115.68.46.186.xip.io
portal-registration  started           1/1         512M     1G     portal-registration.115.68.46.186.xip.io
portal-api-v2        started           1/1         1G       1G     portal-api-v2.115.68.46.186.xip.io
```



### <div id='19'> 3.5. 운영자 포탈 배포


- manifest.yml 확인
```
$ vi manifest.yml
```
```
yml
---
applications:
- name: portal-web-admin
  memory: 1024M
  instances: 1
  host: portal-web-admin
  path: ./paasta-portal-web-admin-1.0.war
  buildpack: java_buildpack_offline
  services:
  - portal-eureka-service
  - portal-redis-session
  env:
    spring_application_name: portal-web-admin
    spring_redis_host: ${vcap.services.portal-redis-session.credentials.host}
    spring_redis_password: ${vcap.services.portal-redis-session.credentials.password}
    spring_redis_port: ${vcap.services.portal-redis-session.credentials.port}
    spring_redis_timeout: 2100

    multipart_maxFileSize: 1000Mb
    multipart_maxRequestSize: 1000Mb

    server_port: 8090

    eureka_client_serviceUrl_defaultZone: ${vcap.services.portal-eureka-service.credentials.uri}/eureka/
    eureka_instance_hostname: ${vcap.application.uris[0]}

    # api manifest의 spring_security_username + ":" + spring_security_password를 Base64 인코딩하여 입력해야 합니다.
    # 예를 들어서 api manifest의 값이 아래와 같다면
    # spring_security_username: user
    # spring_security_password: password
    # user:password 를 인코딩해야합니다.
    # Base64 인코딩 사이트: http://www.convertstring.com/ko/EncodeDecode/Base64Encode
    paasta_portal_api_authorization_base64: Basic YWRtaW46b3BlbnBhYXN0YQ==
    paasta_portal_api_url: http://PORTAL-API

    ribbon_eureka_enabled: true
    ribbon_ConnectTimeout: 30000
    ribbon_ReadTimeout: 30000
    
   monitoringSite_url: http://115.68.151.183:3000
    monitoringSite_id: openpaas
    monitoringSite_password: openpaas
```


- manifest.yml이 있는 폴더로 이동하여 cf push 명령어를 이용하여 배포한다.

```
$ cf push
```
```
Using manifest file /home/inception/bosh-space/kimdojun/paasta-portal/portal-web-admin/manifest.yml

Creating app portal-web-admin in org OCP/ space prd as admin...
OK

Creating route portal-web-admin.115.68.46.186.xip.io...
OK

Binding portal-web-admin.115.68.46.186.xip.ioto portal-web-admin...
OK

Uploading portal-web-admin...
Uploading app files from: /tmp/unzipped-app035163477
Uploading 3.9M, 1824 files
Done uploading               
OK
Binding service portal-eureka-service to app portal-web-adminin org OCP/ space prd as admin...
OK
Binding service portal-redis-session to app portal-web-adminin org OCP/ space prd as admin...
OK

Starting app portal-web-adminin org OCP/ space prdas admin...
Downloading java_buildpack_offline...
Downloaded java_buildpack_offline
Creating container
Successfully created container
Downloading app package...
Downloaded app package (34.5M)
Staging...
-----> Java Buildpack Version: v3.10 (offline) | https://github.com/cloudfoundry/java-buildpack.git#193d6b7
-----> Downloading Open Jdk JRE 1.8.0_111 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_111.tar.gz (found in cache)
      Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.4s)
-----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
      Memory Settings: -Xmx681574K -XX:MaxMetaspaceSize=104857K -Xss349K -Xms681574K -XX:MetaspaceSize=104857K
-----> Downloading Container Customizer 1.0.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-customizer/container-customizer-1.0.0_RELEASE.jar (found in cache)
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (108B)
Uploaded droplet (79M)
Uploading complete
Destroying container
Successfully destroyed container

of 1 instances running, 1 starting
of 1 instances running, 1 starting
of 1 instances running, 1 starting
of 1 instances running, 1 starting
of 1 instances running

App started


OK

App portal-web-admin was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100% -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.WarLauncher`

Showing health and status for app portal-web-admin in org OCP/ space prd as admin...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: portal-web-admin.115.68.46.186.xip.io
last uploaded: Thu Jan 19 05:58:46 UTC 2017
stack: cflinuxfs2
buildpack: java_buildpack_offline

    state     since                    cpu      memory       disk           details
#0  running   2017-01-19 02:59:52 PM   225.9%   437M of 1G   178.7M of 1G
```



- 배포 확인
```
$ cf apps
```
```
Getting apps in org OCP/ space prd as admin...
OK

name                  requested state   instances   memory   disk   urls
portal-api           started           2/2         1.5G     1G     portal-api.115.68.46.186.xip.io
portal-web-user      started           1/1         1G       1G     portal-web-user.115.68.46.186.xip.io
portal-registration  started           1/1         512M     1G     portal-registration.115.68.46.186.xip.io
portal-web-admin     started           1/1         1G       1G     portal-web-admin.115.68.46.186.xip.io
portal-api-v2        started           1/1         1G       1G     portal-api-v2.115.68.46.186.xip.io
```





### <div id='19-1'> 3.6. 앱 오토스케일러 배포

portal-auto-scaler 배포는 cloudfoundry에 배포되어 있는 앱의 자동 스케일 업 서비스를 하기 위한 배포 어플리케이션이다.


- portal-auto-scaler 의 manifest.yml확인
```
$ vi manifest.yml
```
```
yml
---
applications:
- name: portal-auto-scaler                          # 앱 이름
  memory: 512M                                      # 앱 메모리 크기
  instances: 1                                      # 앱 인스턴스 수
  host: portal-auto-scaler                          # 앱과 바인드될 호스트
  path: ./passta-portal-auto-scaler-1.0.war         # war 파일 경로
  buildpack: java_buildpack_offline                 # 앱이 사용할 빌드팩 이름
  services:                                         # 앱과 바인드되는 서비스 목록
  - portal-eureka-service                           # 사용자 생성 서비스(UserProvided Service) 유레카
  - portal-redis-session
  env:
    SPRING_PROFILES_ACTIVE: prd

    spring_application_name: portal-auto-scaling    # 앱 이름
    spring_redis_host: ${vcap.services.portal-redis-session.credentials.host}
    spring_redis_password: ${vcap.services.portal-redis-session.credentials.password}
    spring_redis_port: ${vcap.services.portal-redis-session.credentials.port}
    spring_redis_timeout: 2100

    server_port: 8070

    eureka_client_serviceUrl_defaultZone: ${vcap.services.portal-eureka-service.credentials.uri}/eureka/
    eureka_instance_hostname: ${vcap.application.uris[0]}

    # api manifest의 spring_security_username + ":" + spring_security_password를 Base64 인코딩하여 입력해야 합니다.
    # 예를 들어서 api manifest의 값이 아래와 같다면
    # spring_security_username: user
    # spring_security_password: password
    # user:password 를 인코딩해야합니다.
    # Base64 인코딩 사이트: http://www.convertstring.com/ko/EncodeDecode/Base64Encode

    paasta_portal_api_authorization_base64: Basic YWRtaW46b3BlbnBhYXN0YQ==
    paasta_portal_api_url: http://PORTAL-API
    paasta_portal_api_monitor: http://115.68.46.190:9999

    ribbon_eureka_enabled: true
    ribbon_ConnectTimeout: 30000
    ribbon_ReadTimeout: 30000
    
    # paas-ta 설치시 등록한 어드민 아이디 / 패스워드
    cloudfoundry_user_admin_username: admin
    cloudfoundry_user_admin_password: xxxxx

```


- manifest.yml이 있는 폴더로 이동하여 cf push 명령어를 이용하여 배포한다.




```
$ cf push
```
```
Using manifest file /home/inception/bosh-space/kimdojun/paasta-portal/portal-auto-scaler/manifest.yml

Creating app portal-auto-scaler-v2 in org OCP/ space prd as admin...
OK

Using route portal-auto-scaler.115.68.46.186.xip.io
Binding portal-auto-scaler.115.68.46.186.xip.io to portal-auto-scaler...
OK

Uploading portal-auto-scaler...
Uploading app files from: /tmp/unzipped-app446087426
Uploading 1.8M, 266 files
Done uploading               
OK
Binding service portal-eureka-service to app portal-auto-scaler in org OCP/ space prdas admin...
OK

Starting app portal-auto-scaler in org OCP/ space prd as admin...
Downloading java_buildpack_offline...
Downloaded java_buildpack_offline
Creating container
Successfully created container
Downloading app package...
Downloaded app package (63.9M)
Staging...
-----> Java Buildpack Version: v3.10 (offline) | https://github.com/cloudfoundry/java-buildpack.git#193d6b7
-----> Downloading Open Jdk JRE 1.8.0_111 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_111.tar.gz (found in cache)
      Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.4s)
-----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
      Memory Settings: -XX:MetaspaceSize=105267K -Xmx684236K -XX:MaxMetaspaceSize=105267K -Xss350K -Xms684236K
-----> Downloading Container Customizer 1.0.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-customizer/container-customizer-1.0.0_RELEASE.jar (found in cache)
-----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading build artifacts cache...
Uploading droplet...
Uploaded build artifacts cache (109B)
Uploaded droplet (108.9M)
Uploading complete
Destroying container
Successfully destroyed container

of 1 instances running, 1 starting
of 1 instances running, 1 starting
of 1 instances running

App started

OK

App portal-auto-scaler was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100% -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY" && SERVER_PORT=$PORTeval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.WarLauncher`

Showing health and status for app portal-auto-scaler in org OCP/ space prd as admin...
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: portal-auto-scaling.115.68.46.186.xip.io
last uploaded: Thu Jan 19 05:46:46 UTC 2017
stack: cflinuxfs2
buildpack: java_buildpack_offline

    state     since                    cpu    memory         disk           details
#0  running   2017-01-19 02:48:02 PM   0.0%   240.2M of 512MG   196.2M of 512M
```


- 배포 확인

```
$ cf apps
```
```
Getting apps in org OCP/ space prd as admin...
OK

name                  requested state   instances   memory   disk   urls
portal-api           started           2/2         1.5G     1G     portal-api.115.68.46.186.xip.io
portal-registration  started           1/1         512M     1G     portal-registration.115.68.46.186.xip.io
portal-auto-scaling started           1/1         512M     1G     portal-registration.115.68.46.186.xip.io
portal-api-v2        started           1/1         1G       1G     portal-api-v2.115.68.46.186.xip.io
```



### <div id='20'> 3.6. 카탈로그 이미지 파일 업로드

PaaS-TA 포털에 기본 생성되는 카탈로그에 대한 이미지를 업로드 한다. 카탈로그 이미지 업로드는 운영자 포털을 통해서 진행하고 사용자 포털의 카탈로그 화면에서 이미지를 확인할 수 있다. 업로드할 이미지 파일은 '카탈로그 이미지' 폴더에서 확인할 수 있다. [[**PaaSTA 운영자 포털 가이드**](https://github.com/OpenPaaSRnD/Documents-PaaSTA-2.0/blob/master/Use-Guide/PaaS-TA%20%EC%9A%B4%EC%98%81%EC%9E%90%20%ED%8F%AC%ED%83%88%20%EA%B0%80%EC%9D%B4%EB%93%9C_v1.0.md)]의 [[**5.4 카탈로그 관리 서비스**](https://github.com/OpenPaaSRnD/Documents-PaaSTA-2.0/blob/master/Use-Guide/PaaS-TA%20%EC%9A%B4%EC%98%81%EC%9E%90%20%ED%8F%AC%ED%83%88%20%EA%B0%80%EC%9D%B4%EB%93%9C_v1.0.md#5.4)] 항목을 참고하여 각 카탈로그에 맞는 이미지를 업로드한다.




# <div id='21'>  4. 테스트 케이스 구동 가이드



### <div id='22'> 4.1.  테스트시 절차

PaaSTA Potal  JUnit 테스트 Class를 구동하기 위해 다음과 같은 작업이 필요하다.


1.  테스트 조직, 테스트 공간, 테스트 앱을 생성하여야 한다.
```
cf create-org [조직명]
cf create-space [공간명]
cf push [테스트 앱명]
```

##### 조직 생성
```
$ cf create-org app-test-org      
```
```
Creating org app-test-org as admin...
OK

Assigning role OrgManager to user admin in org app-test-org ...
OK
```

##### 공간 생성
```
$ cf create-space app-test-space  
```
```
Creating space app-test-space in org app-test-org as admin...
OK
Assigning role RoleSpaceManager to user admin in org app-test-org / space app-test-space as admin...
OK
Assigning role RoleSpaceDeveloper to user admin in org app-test-org / space app-test-space as admin...
OK
```


##### 테스트 앱 생성

```
$ cf push test-app
```
```
Creating app test-app in org OCP / space prd as admin...
OK

Creating route test-app.115.68.46.186.xip.io...
OK

Binding test-app.115.68.46.186.xip.io to test-app...
OK
```


2.  테스트 관리자 권한 사용자를 생성한다.
```
$ cf create-user [사용자아이디] [비밀번호]
```
```
$ cf create-user junit-test-user 1234
```


3.  테스트 사용자에 조직매니저 권한을 부여한다.
```
$ cf set-org-role junit-test-user app-test-org OrgManager
```


4.  테스트 사용자에 공간에 대한 권한을 부여한다.
```
$ cf set-space-role junit-test-user app-test-org app-test-space SpaceDeveloper
```


5.  조직이 없는 사용자를 생성한다.
```
$ cf create-user no-org-user 1234
```


6.	PaaS-TA 포털 레포지토리를 Clone 한다.
```
$ git clone https://github.com/OpenPaaSRnD/PaaS-TA-Portal.git
```


7.	PaaS-TA Porta API 디렉토리로 이동한다.
```
$ cd PaaS-TA-Portal/openPaasPaastaPortalApi
```


8.	현재의 PaaS-TA 설치 환경에 맞게 PaaS-TA 포털 API 설정을 변경한다. 현재 디렉토리 기준으로 변경이 필요한 파일의 경로는 'src/resources/application.yml' 이다. 굵은 글씨로 쓰여진 항목을 사용자의 설치 환경에 맞게 수정한다.

spring.profiles가 세가지로 분류되어 있다. 각 spring.profiles 별로 값을 다르게 사용할 수 있고 각 spring.profiles는 연속된 3개의 대시('---')로 구분한다. 예시에서는 spring.profiles 값이 local인 경우를 예로 설명하기 때문에 연속된 3개의 대시('---')로 구분했을때, spring.profiles 값이 local인 경우와 동일한 범위내에 있는 설정 값을 수정하도록 한다.

```
$ vi src/resources/application.yml
```
```
yaml
# Spring properties
spring:
  application:
     name: portal-api  # Service registers under this name
# jdbc 종류 (postgresql, mysql)
  jdbc: postgresql
#  jdbc: mysql

# HTTP Server
server:
  port: ${PORT:2222}   # HTTP (Tomcat) port

# CloudFoundry API Url
cloudfoundry:
  cc:
    api:
      url: https://api.115.68.46.186.xip.io       # PaaS-TA 플랫폼의 API url(수정 필요)
      uaaUrl: https://uaa.115.68.46.186.xip.io    # PaaS-TA 플랫폼의 UAA url(수정 필요)
  # CloudFoundry Login information
  user:
    admin:
      username: admin
      password: admin
    uaaClient:
      clientId: login
      clientSecret: login-secret
      adminClientId: admin
      adminClientSecret: admin-secret
      loginClientId: login
      loginClientSecret: login-secret
      skipSSLValidation: true
  authorization: cf-Authorization

abacus:
  url: http://paasta-usage-reporting.115.68.46.186.xip.io/v1

---
spring:
  profiles: local
  security:
    username: admin
    password: openpaasta
  datasource:
    cc:
      driverClassName: org.postgresql.Driver
      url: jdbc:postgresql://localhost:5524/ccdb        # PaaS-TA 플랫폼의 ccdb 접속 url(수정 필요)
      username: ccadmin
      password: admin
    portal:
      driverClassName: org.postgresql.Driver
      url: jdbc:postgresql://localhost:5524/portaldb    #(Portal DB로 PaaS-TA 플랫폼에 내장된 PostgreSQL을 사용할 경우) PaaS-TA 플랫폼의 portaldb 접속 url(수정 필요)
      username: portaladmin
      password: admin
      mysql:
        driverClassName: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/portaldb?autoReconnect=true&useUnicode=true&characterEncoding=utf8  # (Portal DB로 Mysql 서비스팩을 사용할 경우) MySQL 서비스팩의 portaldb 접속 url (수정 필요)
        username: portaladmin
        password: admin
    uaa:
      driverClassName: org.postgresql.Driver
      url: jdbc:postgresql://localhost:5524/uaadb      # PaaS-TA 플랫폼의 uaadb 접속 url (수정 필요)
      username: uaaadmin
      password: admin
  objectStorage:
    tenantName: paasta-portal
    username: paasta-portal
    password: paasta
    authUrl: http://localhost:5000/v2.0                # PaaS-TA Portal Object Storage AuthUrl (수정 필요)
    container: portal-container
  mail:
    smtp:
      host: smtp.gmail.com
      port: 465
      username: PaaS-TA 관리자
      password: openpasta!
      userEmail: openpasta@gmail.com
      properties:
        auth: true
        starttls:
          enable: true
          required: true
        maximumTotalQps: 90
        authUrl: http://localhost:8080
        imgUrl: http://52.201.48.51:8080/v1/KEY_84586dfdc15e4f8b9c2a8e8090ed9810/portal-container/65bdc7f43e11433b8f17683f96c7e626.png
        sFile: emailTemplate.html
        subject: PaaS-TA User Potal 인증메일(Local)
        contextUrl: user/authUser
multipart:
  maxFileSize: 1000Mb
  maxRequestSize: 1000Mb
eureka:
  instance:
    hostname: localhost
  client:
    serviceUrl:
      defaultZone: http://127.0.0.1:2221/eureka/           # defaultZone: http://[portal-registration의 접속 호스트]/eureka/ (수정 필요)
 logging:
  level:
    org.openpaas.paasta.portal.api.mapper: DEBUG
---
... 하단 생략 ...
```


9.	현재의 PaaS-TA 설치 환경에 맞게 PaaS-TA 포털 API 테스트 설정을 변경한다. 현재 디렉토리 기준으로 변경이 필요한 파일의 경로는 'src/test/resources/config.properties'이다.

```
#COMMON
test.apiTarget=https://api.115.68.46.186.xip.io      # PaaS-TA 플랫폼의 API url (수정 필요)
test.clientUserName=junit-test-user
test.clientUserPassword=1234
test.clientUserGuid=02aa0a5b-b8dd-4c4e-a6ec-d36586dd269a
test.adminUserName=admin

#ORG TEST
test.noOrgClientUserName=no-org-user
test.appTestOrg=app-test-org
test.appTestSpace=app-test-space
test.orgTestOrg=junit-org-test-org
test.noAuthTestOrg=junit-org-test-org-no-auth
test.nonexistentOrg=nonexistent-org
test.nonexistentOrNoAuthOrg=nonexistentOrNoAuthOrg
test.createTestOrg=junit-org-test-org-create

#SPACE TEST
test.testOrg=junit-space-test-org
test.testSpace=junit-test-space
test.createTestSpace=junit-space-test-space-create
test.noAuthTestSpace=junit-space-test-space-no-auth
test.noAuthSpaceTestOrg=junit-space-test-org-no-auth
test.nonexistentOrNoAuthSpace=test-nonexistent-no-auth-space
test.noContentTestOrg=junit-space-test-org-no-content

# APP TEST
test.testApp=test-app
test.nonexistentApp=nonexistentApp
test.domainName=115.68.46.186.xip.io
test.testHost=testhost

#SERVICE TEST
#COMMON
test.apiTarget=https://api.115.68.46.186.xip.io      # PaaS-TA 플랫폼의 API url (수정 필요)
test.clientUserName=junit-test-user
test.clientUserPassword=1234
test.clientUserGuid=02aa0a5b-b8dd-4c4e-a6ec-d36586dd269a
test.adminUserName=admin

#ORG TEST
test.noOrgClientUserName=no-org-user
test.appTestOrg=app-test-org
test.appTestSpace=app-test-space
test.orgTestOrg=junit-org-test-org
test.noAuthTestOrg=junit-org-test-org-no-auth
test.nonexistentOrg=nonexistent-org
test.nonexistentOrNoAuthOrg=nonexistentOrNoAuthOrg
test.createTestOrg=junit-org-test-org-create

#SPACE TEST
test.testOrg=junit-space-test-org
test.testSpace=junit-test-space
test.createTestSpace=junit-space-test-space-create
test.noAuthTestSpace=junit-space-test-space-no-auth
test.noAuthSpaceTestOrg=junit-space-test-org-no-auth
test.nonexistentOrNoAuthSpace=test-nonexistent-no-auth-space
test.noContentTestOrg=junit-space-test-org-no-content

# APP TEST
test.testApp=test-app
test.nonexistentApp=nonexistentApp
test.domainName=115.68.46.186.xip.io            # PaaS-TA 플랫폼의 기본 도메인 (수정 필요)
test.testHost=testhost

#SERVICE TEST
test.serviceTestOrg=junit-service-test-org
test.serviceTestSpace=junit-service-test-space
test.userProvidedInstanceName=user-provided-test-instance
test.createTestUP=user-provided-create-test

#COMMON CODE, CATALOG, MY QUESTION, USER MANAGEMENT
test.cf.authorization=cf-Authorization
test.admin.id=admin
test.admin.password=admin
test.org=catalog-test-org
test.space=catalog-test-space
test.file.path=./src/test/java/resources/images/test.jpg
test.domain.url=115.68.46.186.xip.io                     # PaaS-TA 플랫폼의 기본 도메인 (수정 필요)
test.java.build.pack=java_buildpack_offline
test.ruby.build.pack=ruby_buildpack
test.egov.build.pack=egov_buildpack

#COMMON
test.apiTarget=https://api.115.68.46.186.xip.io      # PaaS-TA 플랫폼의 API url (수정 필요)
test.clientUserName=junit-test-user
test.clientUserPassword=1234
test.clientUserGuid=02aa0a5b-b8dd-4c4e-a6ec-d36586dd269a
test.adminUserName=admin

#ORG TEST
test.noOrgClientUserName=no-org-user
test.appTestOrg=app-test-org
test.appTestSpace=app-test-space
test.orgTestOrg=junit-org-test-org
test.noAuthTestOrg=junit-org-test-org-no-auth
test.nonexistentOrg=nonexistent-org
test.nonexistentOrNoAuthOrg=nonexistentOrNoAuthOrg
test.createTestOrg=junit-org-test-org-create

#SPACE TEST
test.testOrg=junit-space-test-org
test.testSpace=junit-test-space
test.createTestSpace=junit-space-test-space-create
test.noAuthTestSpace=junit-space-test-space-no-auth
test.noAuthSpaceTestOrg=junit-space-test-org-no-auth
test.nonexistentOrNoAuthSpace=test-nonexistent-no-auth-space
test.noContentTestOrg=junit-space-test-org-no-content

# APP TEST
test.testApp=test-app
test.nonexistentApp=nonexistentApp
test.domainName=115.68.46.186.xip.io            # PaaS-TA 플랫폼의 기본 도메인 (수정 필요)
test.testHost=testhost

#SERVICE TEST
test.serviceTestOrg=junit-service-test-org
test.serviceTestSpace=junit-service-test-space
test.userProvidedInstanceName=user-provided-test-instance
test.createTestUP=user-provided-create-test

#COMMON CODE, CATALOG, MY QUESTION, USER MANAGEMENT
test.cf.authorization=cf-Authorization
test.admin.id=admin
test.admin.password=admin
test.org=catalog-test-org
test.space=catalog-test-space
test.file.path=./src/test/java/resources/images/test.jpg
test.domain.url=115.68.46.186.xip.io                     # PaaS-TA 플랫폼의 기본 도메인 (수정 필요)
test.java.build.pack=java_buildpack_offline
test.ruby.build.pack=ruby_buildpack
test.egov.build.pack=egov_buildpack
test.user.id.email.account=mingu@bluedigm.com

#USER TEST
test.insetTestId=insertUser
test.updateTestUser=updateUser
test.password=insertUser
test.newPassword=newPassword
test.testUser=junit-test-user
test.testUserPassword=1234

#Email TEST
test.inviteId=openpasta@gmail.com
test.userId=openpasta@gmail.com
test.emailId=aaa@gmail.com
test.resetPassword.html=resetPassword.html

#Email Org Role invite TEST

test.org-role-user=lij
test.org-role-user-guid=db040322-c831-4d51-b391-4f9ff8102dc9

#Service Broker
test.serviceBrokerName=redis-service-broker
test.serviceBrokerUsername=admin
test.serviceBrokerPassword=admin
test.serviceBrokerUrl=http://10.30.120.71:80        # PaaS-TA 플랫폼에 등록된 Redis 서비스 브로커 url (수정 필요)

#AuthorityGroup TEST
test.authorityGroup=testAuthGroup
test.authorityGroupForEachTest=testAuthGroup.forEachTest

#UaaCommon
test.uaaAdminClientId=admin
test.uaaAdminClientSecret=admin
test.uaaLoginClientId=login
test.uaaLoginClientSecret=admin
test.uaaTarget=https://uaa.115.68.46.186.xip.io      # PaaS-TA 플랫폼의 UAA url (수정 필요)
test.skipSSLValidation=true
```


10.	Gradle 테스트 수행

```
$ gradle -Plocation=local clean test 
```

[portal_deploy_image_01]:/images/paasta-portal/portal-deploy/portal_deploy_image_01.png
[portal_deploy_image_02]:/images/paasta-portal/portal-deploy/portal_deploy_image_02.png
