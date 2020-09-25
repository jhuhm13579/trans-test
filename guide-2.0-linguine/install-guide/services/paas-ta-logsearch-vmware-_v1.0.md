# Table of Contents

1. [문서 개요](paas-ta-logsearch-vmware-_v1.0.md#1)
   * [1.1. 목적](paas-ta-logsearch-vmware-_v1.0.md#2)
   * [1.2. 범위](paas-ta-logsearch-vmware-_v1.0.md#3)
   * [1.3. 참고](paas-ta-logsearch-vmware-_v1.0.md#4)
2. [InfluxDB & Grafana Release 배포](paas-ta-logsearch-vmware-_v1.0.md#5)
   * [2.1.  upload release](paas-ta-logsearch-vmware-_v1.0.md#6)
   * [2.2.  manifest 파일 설정](paas-ta-logsearch-vmware-_v1.0.md#7)
   * [2.3.  deploy](paas-ta-logsearch-vmware-_v1.0.md#8)
   * [2.4.  확인](paas-ta-logsearch-vmware-_v1.0.md#9)

 \# 1. 문서 개요

## 1.1. 목적

본 문서는 IaaS\(Infrastructure as a Service\) 중 하나인 Openstack 환경에서 모니터링 시스템의 주요 정보인 로그를 저장하기 위한 Elasticsearch, 수집하기 위한 LogStash 및 화면으로 보여주기 위한 Kibana를 설치하기 위한 가이드를 제공하는데 그 목적이 있다

 \#\#\# 1.2. 범위 본 문서는 Openstack 기반에 설치하기 위한 내용으로 한정되어 있다.

## 1.3. 참고

> [모니터링 시스템 Architecutre](https://github.com/OpenPaaSRnD/Documents-PaaSTA-2.0/blob/master/Use-Guide/PaaS-TA%20%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81%20%EC%8B%9C%EC%8A%A4%ED%85%9C%20Architecture.md)

 \# 2. Logsearch Release 배포 본 장에서는 Logsearch 서비스를 배포하는 방법에 대해서 기술하였다.

## 2.1.  upload "Logsearch" release

하단 링크로 접속하여 PaaS-TA 모니터링 패키지 파일을 다운로드하여, 압축해제한다.

> PaaS-TA 모니터링 : [https://paas-ta.kr/data/packages/2.0/PaaSTA-Monitoring.zip](https://paas-ta.kr/data/packages/2.0/PaaSTA-Monitoring.zip)   
>  다운로드 받은 PaaSTA-Monitoring.zip 파일을 압축해제한다.   
>  paasta-logsearch-2.0.tgz 파일을 확인한다.

다음의 명령어를 이용하여 릴리즈 파일을 bosh에 업로드한다.

$ bosh upload release paasta-logsearch-2.0.tgz

![](../../../.gitbook/assets/2-1-1%20%2823%29.png) ![](../../../.gitbook/assets/2-1-2%20%2815%29.png)

 \#\#\# 2.2. manifest 파일 설정 1. "Logsearch" 서비스가 배포되는 환경에 맞게 manifest 파일의 설정 정보를 수정한다. $ vi logsearch-release.yml \`\`\` --- name: paasta-logsearch \#deployment name compilation: cloud\_properties: cpu: 2 disk: 8192 ram: 2048 network: paasta-logsearch-network \#network name reuse\_compilation\_vms: true workers: 6 director\_uuid:  \#bosh uuid disk\_pools: - cloud\_properties: {} disk\_size: 1024 \#disk pool size name: elasticsearch\_master \#disk pool name - cloud\_properties: {} disk\_size: 20480 \#disk pool size name: elasticsearch\_data \#disk pool name - cloud\_properties: {} disk\_size: 1024 name: queue - cloud\_properties: {} disk\_size: 1024 name: cluster\_monitor jobs: - instances: 1 name: elasticsearch\_master \#job name networks: - name: paasta-logsearch-network \#network name static\_ips: - 10.30.152.120 \#local static ip persistent\_disk\_pool: elasticsearch\_master \#disk pool name properties: elasticsearch: node: allow\_data: false allow\_master: true resource\_pool: elasticsearch\_master \#resource name templates: - name: elasticsearch \#release job name release: paasta-logsearch \#release name update: max\_in\_flight: 1 - instances: 1 name: cluster\_monitor networks: - name: paasta-logsearch-network static\_ips: - 10.30.152.122 persistent\_disk\_pool: cluster\_monitor properties: curator: elasticsearch\_host: 10.30.152.120 \#elasticsearch master ip elasticsearch\_port: 9200 \#elasticsearch master port purge\_logs: retention\_period: 7 elasticsearch: cluster\_name: monitor master\_hosts: - 127.0.0.1 node: allow\_data: true allow\_master: true elasticsearch\_config: elasticsearch: host: 127.0.0.1 port: 9200 templates: - shards-and-replicas: '{ "template" : "\*", "order" : 99, "settings" : { "number\_of\_shards" : 1, "number\_of\_replicas" : 0 } }' - index\_template: /var/vcap/packages/logsearch-config/default-mappings.json kibana: elasticsearch: host: 127.0.0.1 port: 9200 memory\_limit: 30 port: 5601 wait\_for\_templates: - shards-and-replicas logstash\_ingestor: syslog: port: 514 logstash\_parser: filters: - monitor: /var/vcap/packages/logsearch-config/logstash-filters-monitor.conf nats\_to\_syslog: debug: true nats: machines: - 10.30.150.31 \# paasta-controller nats IP password: nats port: 4222 subject: '&gt;' user: nats syslog: host: 127.0.0.1 port: 514 redis: host: 127.0.0.1 maxmemory: 10 resource\_pool: cluster\_monitor \#resource name templates: - name: queue release: paasta-logsearch - name: parser release: paasta-logsearch - name: ingestor\_syslog release: paasta-logsearch - name: elasticsearch release: paasta-logsearch - name: elasticsearch\_config release: paasta-logsearch - name: curator release: paasta-logsearch - name: kibana release: paasta-logsearch - name: nats\_to\_syslog release: paasta-logsearch - instances: 1 name: queue networks: - name: paasta-logsearch-network static\_ips: - 10.30.152.123 persistent\_disk\_pool: queue resource\_pool: queue templates: - name: queue release: paasta-logsearch - instances: 1 name: maintenance networks: - name: paasta-logsearch-network resource\_pool: maintenance templates: - name: elasticsearch\_config release: paasta-logsearch - name: curator release: paasta-logsearch update: serial: true - instances: 3 name: elasticsearch\_data networks: - name: paasta-logsearch-network static\_ips: - 10.30.152.136 \#local static ip - 10.30.152.137 \#instance 개수와 static ip 개수 일치 - 10.30.152.138 persistent\_disk\_pool: elasticsearch\_data properties: elasticsearch: node: allow\_data: true allow\_master: false resource\_pool: elasticsearch\_data templates: - name: elasticsearch release: paasta-logsearch update: max\_in\_flight: 1 - instances: 1 name: kibana networks: - name: paasta-logsearch-network static\_ips: - 10.30.152.126 properties: route\_registrar: routes: - name: kibana port: 5601 registration\_interval: 20s uris: - kibana.monitoring.open-paas.com resource\_pool: kibana templates: - name: kibana release: paasta-logsearch update: max\_in\_flight: 1 - instances: 1 name: ingestor networks: - name: paasta-logsearch-network static\_ips: - 10.30.152.121 properties: logstash\_ingestor: debug: true relp: port: 2514 resource\_pool: ingestor templates: - name: ingestor\_syslog release: paasta-logsearch - name: ingestor\_relp release: paasta-logsearch - name: ingestor\_bosh release: paasta-logsearch update: max\_in\_flight: 1 - instances: 1 name: parser networks: - name: paasta-logsearch-network resource\_pool: parser templates: - name: parser release: paasta-logsearch - name: elasticsearch release: paasta-logsearch update: max\_in\_flight: 4 serial: false - instances: 1 name: ls-router networks: - name: paasta-logsearch-network static\_ips: - 10.30.152.125 - default: - dns - gateway name: external static\_ips: - "xxx.xxx.xxx.xxx" \#public ip properties: haproxy: \# haproxy 모듈의 Port Forwarding 정보 cluster\_monitor: backend\_servers: - 10.30.152.122 ingestor: backend\_servers: - 10.30.152.121 cf-service-backend\_servers: - 10.30.152.121 bosh-backend\_servers: - 10.30.152.121 \# grafana: \# backend\_servers: \# - 10.30.152.53 \# influxdb: \# backend\_servers: \# - 10.30.152.51 elastic: backend\_servers: - 10.30.152.120 kibana: backend\_servers: - 10.30.152.126 syslog\_server: 10.30.152.122 resource\_pool: haproxy templates: - name: haproxy release: paasta-logsearch update: max\_in\_flight: 1 networks: - name: paasta-logsearch-network \#internal network name subnets: - cloud\_properties: name: Internal dns: - 10.30.20.27 \#dns - 8.8.8.8 gateway: 10.30.20.23 \#gateway range: 10.30.0.0/16 \#static ip range reserved: - 10.30.0.1 - 10.30.152.109 \#reserved ip range static: - 10.30.152.110 - 10.30.152.150 \#available ip range type: manual - name: external \#external network name subnets: - cloud\_properties: name: External dns: - 10.30.20.27 \#dns - 8.8.8.8 gateway: "xxx.xxx.xxx.xxx" \#gateway range: "xxx.xxx.xxx.xxx"/24 \#public ip range static: - "xxx.xxx.xxx.xxx" \#available public ip type: manual properties: curator: elasticsearch: host: 10.30.152.120 port: 9200 elasticsearch: cluster\_name: logsearch exec: null master\_hosts: - 10.30.152.120 elasticsearch\_config: elasticsearch: host: 10.30.152.120 templates: - index\_template: /var/vcap/packages/logsearch-config/default-mappings.json kibana: elasticsearch: host: 10.30.152.120 port: 9200 logstash\_ingestor: debug: false logstash\_parser: debug: false redis: host: 10.30.152.123 releases: - name: paasta-logsearch \#release name version: 2.0 \#release version resource\_pools: - cloud\_properties: cpu: 1 disk: 4096 ram: 1024 env: bosh: password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0 name: elasticsearch\_master \#resource name network: paasta-logsearch-network \#networkname stemcell: name: bosh-vsphere-esxi-ubuntu-trusty-go\_agent \#stemcell name version: latest \#stemcell version - cloud\_properties: cpu: 1 disk: 10240 ram: 1024 env: bosh: password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0 name: elasticsearch\_data network: paasta-logsearch-network stemcell: name: bosh-vsphere-esxi-ubuntu-trusty-go\_agent version: latest - cloud\_properties: cpu: 1 disk: 2048 ram: 1024 env: bosh: password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0 name: ingestor network: paasta-logsearch-network stemcell: name: bosh-vsphere-esxi-ubuntu-trusty-go\_agent version: latest - cloud\_properties: cpu: 1 disk: 4096 ram: 2048 env: bosh: password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0 name: queue network: paasta-logsearch-network stemcell: name: bosh-vsphere-esxi-ubuntu-trusty-go\_agent version: latest - cloud\_properties: cpu: 2 disk: 4096 ram: 2048 env: bosh: password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0 name: parser network: paasta-logsearch-network stemcell: name: bosh-vsphere-esxi-ubuntu-trusty-go\_agent version: latest - cloud\_properties: cpu: 1 disk: 2048 ram: 1024 env: bosh: password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0 name: kibana network: paasta-logsearch-network stemcell: name: bosh-vsphere-esxi-ubuntu-trusty-go\_agent version: latest - cloud\_properties: cpu: 1 disk: 2048 ram: 1024 env: bosh: password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0 name: maintenance network: paasta-logsearch-network stemcell: name: bosh-vsphere-esxi-ubuntu-trusty-go\_agent version: latest - cloud\_properties: cpu: 2 disk: 4096 ram: 2048 env: bosh: password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0 name: cluster\_monitor network: paasta-logsearch-network stemcell: name: bosh-vsphere-esxi-ubuntu-trusty-go\_agent version: latest - cloud\_properties: cpu: 2 disk: 2048 ram: 1024 env: bosh: password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0 name: haproxy network: paasta-logsearch-network stemcell: name: bosh-vsphere-esxi-ubuntu-trusty-go\_agent version: latest - cloud\_properties: cpu: 1 disk: 2048 ram: 1024 env: bosh: password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0 name: errand network: paasta-logsearch-network stemcell: name: bosh-vsphere-esxi-ubuntu-trusty-go\_agent version: latest update: canaries: 1 canary\_watch\_time: 30000-600000 max\_errors: 1 max\_in\_flight: 1 serial: false update\_watch\_time: 5000-600000 \`\`\` 2. manifest 파일 지정 $ bosh deployment logsearch-release.yml !\[2-2-1\]

## 2.3.  배포

$ bosh -n deploy

![](../../../.gitbook/assets/2-3-1%20%2827%29.png) ![](../../../.gitbook/assets/2-3-2%20%2813%29.png)

## 2.4.  확인

$ bosh deployments

![](../../../.gitbook/assets/2-4-1%20%2815%29.png)
