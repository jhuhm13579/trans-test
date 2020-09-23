## Table of Contents
1. [문서 개요](#1)
     * [1.1. 목적](#2)
     * [1.2. 범위](#3)
     * [1.3. 참고](#4)
2. [InfluxDB & Grafana Release 배포](#5)
     * [2.1.  upload release](#6)
     * [2.2.  manifest 파일 설정](#7)
     * [2.3.  deploy](#8)
     * [2.4.  확인](#9)

<div id='1'></div>

# 1. 문서 개요

<div id='2'></div>

### 1.1. 목적
      
본 문서는 IaaS(Infrastructure as a Service) 중 하나인 VMWare 환경에서 모니터링 시스템의 주요 정보인 시스템 metrics 를 저장히기 위한 InfluxDB와 View 화면을 제공하는 Grafana를 설치하기 위한 가이드를 제공하는데 그 목적이 있다.

<div id='3'></div>

### 1.2. 범위
      
본 문서는 VMWare 기반에 설치하기 위한 내용으로 한정되어 있다.

<div id='4'></div>

### 1.3. 참고  
      
> <a style="text-decoration:underline" href="https://github.com/OpenPaaSRnD/Documents-PaaSTA-2.0/blob/master/Use-Guide/PaaS-TA%20%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81%20%EC%8B%9C%EC%8A%A4%ED%85%9C%20Architecture.md">모니터링 시스템 Architecutre</a>

<div id='5'></div>

# 2.  InfluxDB & Grafana Release 배포

본 장에서는 InfluxDB와 Grafana 서비스를 배포하는 방법에 대해서 기술하였다.

<div id='6'></div>

### 2.1.  upload "InfluxDB & Grafana" release

하단 링크로 접속하여 PaaS-TA 모니터링 패키지 파일을 다운로드하여, 압축해제한다. 

>PaaS-TA 모니터링 : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Monitoring.zip>** <br>
>다운로드 받은 PaaSTA-Monitoring.zip 파일을 압축해제한다.<br>
>paasta-influxdb-grafana-2.0.tgz 파일을 확인한다. <br>

다음의 명령어를 이용하여 릴리즈 파일을 bosh에 업로드한다.

$ bosh upload release paasta-influxdb-grafana-2.0.tgz

<kbd>![2-1-1]</kbd>
<kbd>![2-1-2]</kbd>

<div id='7'></div>

### 2.2.  manifest 파일 설정

> <a style="text-decoration:underline" href="https://github.com/OpenPaaSRnD/Documents-PaaSTA-2.0/blob/master/Use-Guide/PaaS-TA%20%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81%20DB%20%EB%B0%8F%20Metrics%20%EA%B0%80%EC%9D%B4%EB%93%9C.md">InfluxDB 참조</a>

1. "InfluxDB & Grafana" 서비스가 배포되는 환경에 맞게 manifest 파일의 설정 정보를 수정한다.

$ vi influxdb-grafana-release.yml

```
---
name: paasta-influxdb-grafana									#deployment name

compilation:
  cloud_properties:
  cloud_properties:
    cpu: 2
    disk: 8192
    ram: 2048
  network: paasta-influxdb-grafana-net				            #network name
  reuse_compilation_vms: true
  workers: 2
 
director_uuid: <%= `bosh status --uuid` %>						#bosh uuid

releases:
- name: paasta-influxdb-grafana									#release name
  version: latest  												#release version

disk_pools:
- cloud_properties:
    type: gp2
  disk_size: 10240 												#influxdb disk pool size
  name: influx_data

jobs:
- name: influxdb												#influxdb service name
  templates:
  - name: influxdb																		
    release: paasta-influxdb-grafana
  instances: 1
  resource_pool: influx											#resource name
  persistent_disk_pool: influx_data								#disk pool name
  networks:
  - default:
    - gateway
    - dns
    name: paasta-influxdb-grafana-net							#network name
    static_ips:
    - 10.10.152.51												#static IP
  - name: elastic												#external network(public) name
    static_ips:
    - "xxx.xxx.xxx.xxx"											#external IP (public)
  properties:
    influxdb:
      database: cf_metric_db									#default database
      user: root												#admin account
      password: root											#admin password
      replication: 1      														
      ips: 10.30.152.51											#local IP
      
- name: grafana													#grafana service name
  templates:
  - name: grafana
    release: paasta-influxdb-grafana
  instances: 1
  resource_pool: grafana										#resoure name		
  networks:
  - default:
    - gateway
    - dns
    name: paasta-influxdb-grafana-net													
    static_ips:																				
    - 10.30.152.53												#local IP			
  - name: elastic												#external network(public) name
    static_ips:
    - "xxx.xxx.xxx.xxx"                                         #public ip

  properties:
    grafana:
      listen_port: 3000											#grafana listen port
      admin_username: admin										#grafana admin account
      admin_password: admin 									#grafana admin password
      users: 
        allow_sign_up: true
        auto_assign_organization: true
        
      datasource:
        url: http://10.30.152.51:8086/							#database url
        name: Grafanadb							
        database_type: Influxdb									#database type	
        user: root												#database admin account
        password: root											#database admin password
        database_name: cf_metric_db								#default database
        
networks:
- name: paasta-influxdb-grafana-net				                #internal network name
  subnets:
  - cloud_properties:
      name: Internal
    dns:
    - 10.30.20.27												#dns
    - 8.8.8.8
    gateway: 10.30.20.23									    #gateway
    range: 10.30.0.0/16										    #static ip range
    reserved:
    - 10.30.0.1 - 10.30.152.49						            #reserved ip range
    static:
    - 10.30.152.50 - 10.30.152.55					            #available ip range
  type: manual

- name: elastic													#external network name
  subnets:
  - cloud_properties:
      name: External
    dns:
    - 10.30.20.27												#dns
    - 8.8.8.8
    gateway: "xxx.xxx.xxx.xxx"							    	#gateway
    range: "xxx.xxx.xxx.xxx"/24							        #external ip range (public)
    static:
    - "xxx.xxx.xxx.xxx"											#available public ip
    - "xxx.xxx.xxx.xxx"
  type: manual

 
resource_pools:
- cloud_properties:
    cpu: 4
    disk: 32768
    ram: 16384
  env:
    bosh:
      password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0
  name: influx													#resource name
  network: paasta-influxdb-grafana-net							#network name
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent			    #stemcell name
    version: latest												#stemcell version

- cloud_properties:
    cpu: 1
    disk: 4096
    ram: 1024
  env:
    bosh:
      password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0
  name: grafana													#resource name
  network: paasta-influxdb-grafana-net							#network name
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent			    #stemcell name
    version: latest												#stemcell version

update:
  canaries: 1
  canary_watch_time: 1000-180000
  max_in_flight: 1
  serial: true
  update_watch_time: 1000-180000
```

2. manifest 파일 지정

$ bosh deployment influxdb-grafana-release.yml

<kbd>![2-2-1]</kbd>

<div id='8'></div>

### 2.3.  배포

$ bosh -n deploy 

<kbd>![2-3-1]</kbd>
<kbd>![2-3-2]</kbd>

<div id='9'></div>

### 2.4.  확인

$ bosh deployments 

<kbd>![2-4-1]</kbd>


[2-1-1]:images/influxdb-grafana/2-1-1.png
[2-1-2]:images/influxdb-grafana/2-1-2.png
[2-2-1]:images/influxdb-grafana/2-2-1.png
[2-3-1]:images/influxdb-grafana/2-3-1.png
[2-3-2]:images/influxdb-grafana/2-3-2.png
[2-4-1]:images/influxdb-grafana/2-4-1.png
