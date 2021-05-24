---
layout: post
title: "Kubernetes 실습(4)"
summary: Prometheus 설치, Query 및 Grafana 연동
author: Dohyun Kim
date: 2021-04-12 21:30:00 -0400
category: Kubernetes
thumbnail: /assets/img/posts/Kubernetes.png
comments: true
---

### Summary

- Prometheus 설치

- Prometheus Query

- Grafana 연동

- Grafana Dashboard 생성

---

#### Prometheus 설치

Prometheus 는 OpenSource 기반 Monitoring system으로, Kubernetes node들의 상태를 monitoring 한다. (master node에서 동작.)

간단한 text 형식으로 Metric을 쉽게 노출하고, Grafana 같은 대시보드 시스템에서 graph로 표시 가능하다.

PromQL Query Language를 사용한다.

![image](https://user-images.githubusercontent.com/72643027/114351152-29e95a00-9ba5-11eb-97c2-3155128dc89e.png){: width="70%" height="70%"}

설치 전 먼저 ```kubectl get nodes```로 쿠버네티스 클러스터 상태가 정상인지 확인한다.

![image](https://user-images.githubusercontent.com/72643027/114351359-746ad680-9ba5-11eb-956e-f27cffe04658.png){: width="70%" height="70%"}

!참고 ```kubectl get nodes```에서 ready 안 뜰 경우, 확인하는 법.  
1. docker, kubelet restart.
2. 1 해도 안될 경우 config 파일 다시 복사떠와서 설정.(1주차 내용에 존재.)
3. master, worker node로 설정한 node의 호스트명 확인. JOIN 당시의 호스트명을 전달해서 인식하기 때문에 node의 호스트명이 바뀌면 인식하지 못함.
4. ```calico```문제 => 방화벽 off 로 해결.
5. pod 들이 ```pending```되는 이유는, 배포할 node가 없다고 판단했기 때문.(master node는 애초에 배포할 node라고 가정하지 않음.) => ```kubectl get nodes```로 worker node를 ready 상태로 만드는 것이 우선.  

<br/>

- Helm 설치.

Helm 차트는 복잡한 Kubernetes app들을 편리하게 정의하여 설치하거나 upgrade가 가능하게 해 주는 tool이다. 한마디로 Kubernetes Package managing tool.

```
curl -fsSL -o helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 777 helm.sh
./helm.sh
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

- Helm Chart git clone  
```
yum install -y git
git clone https://github.com/helm/charts.git
```

- Prometheus values 파일 수정. (Persistant Volume 사용하지 않고 설치)
```
cd charts/stable/prometheus/
vi values.yaml
```
values.yaml 파일에서 persistantVolume: 하위의 enabled: 를 ```false```로 설정. (3군데 수정필요)

- Prometheus 설치
```
helm install prometheus stable/prometheus -f charts/stable/prometheus/values.yaml
```

```kubectl get pods```로 설치 확인.

![image](https://user-images.githubusercontent.com/72643027/114353326-dfb5a800-9ba7-11eb-8c5a-b2c2b4c86a40.png){: width="70%" height="70%"}

- jq 설치. (Query response를 보기 편하게 해 주는 tool)
```
yum install -y epel-release
yum install snapd
systemctl enable --now snapd.socket
ln -s /var/lib/snapd/snap /snap     #symbolic link 생성.
snap install jq
yum install jq
```

<br/>

#### Prometheus Query (PromQL)

- Metric 출력 형태  
```<Metric 이름>{<Lable key>=<Lable 값>, <Lable key>=<Lable 값> ... }<Metric 값>[<timestamp>]```
- Data type
    1. Instant Vector : 같은 시간대를 갖는 Metric 값들의 집합.
    2. Range Vector : 특정 시간 동안의 Metric 값들을 배열로 가진 집합.
- Query 형식  
    ```curl http://[prometheus-server ip]/[end point] -d query="[query]"```  
    Instant Query - end point: /api/v1/query  
    Range Query - end point: /api/v1/query_range
- Test  
```
curl http://[prometheus-server ip]/api/v1/query -d query="node_cpu_seconds_total"```
curl http://[prometheus-server ip]/api/v1/query -d query="node_memory_MemTotal_bytes" | jq
```

직접 Prometheus Query 를 날려보기 위해, prometheus-server의 IP를 확인해 보자.

![image](https://user-images.githubusercontent.com/72643027/114356831-007ffc80-9bac-11eb-8c39-26c631f33d5f.png){: width="70%" height="70%"}

이렇게 pod와 service의 IP를 확인할 수 있는데, 이 중 prometheus-server의 service Cluster-IP인 ```10.98.162.91```을 사용한다. (이유는 pod의 IP주소는 유동적이고 service는 그러한 유동적인 pod의 IP를 찾을 수 있는 anchor역할을 하기 때문.)

![image](https://user-images.githubusercontent.com/72643027/114358126-894b6800-9bad-11eb-8979-abe84a314d3e.png){: width="70%" height="70%"}

이처럼 확인이 가능하다.

- 대표적으로 많이 사용하는 Query
    + CPU 사용률: Node => node_cpu_seconds_total  
                  Container => container_cpu_usage_seconds_total
    + RAM 사용률: Node => node_memory_MemTotal_bytes  
                  Container => container_memory_working_set_bytes
    + Node, Container 상태: Node => kube_node_status_condition  
                            Container => kube_pod_status_ready

<br/>

#### Grafana 연동

- charts/stable/grafana/values.yaml 파일 수정

![image](https://user-images.githubusercontent.com/72643027/114358817-505fc300-9bae-11eb-94f8-bcf0812418b6.png){: width="70%" height="70%"}

- Grafana 설치
```helm install grafana stable/grafana -f charts/stable/grafana/values.yaml```

![image](https://user-images.githubusercontent.com/72643027/114359141-a0d72080-9bae-11eb-9b4c-279043cbe409.png){: width="70%" height="70%"}

![image](https://user-images.githubusercontent.com/72643027/114359187-aaf91f00-9bae-11eb-84f0-8c681cc74ea8.png){: width="70%" height="70%"}

- Grafana service 외부 IP 할당
```
kubectl edit svc/grafana
```

clusterIP 하위에 externalIPs 추가 후 IP 할당. (자신의 node IP주소 기입.)  
port 부분에 원하는 port 설정.

![image](https://user-images.githubusercontent.com/72643027/114359392-dda31780-9bae-11eb-84ea-c07d8b484463.png){: width="70%" height="70%"}

- Grafana 접속

browser에서 externalIP port로 접속.  
ID, Passwd를 svc/grafana에서 설정한 ID/PW로 로그인.

- Prometheus와 연동

Grafana dashboard에서 ```Configuration - Data Sources - Add data source```

Prometheus 선택, URL에 Prometheus server IP 입력.

![image](https://user-images.githubusercontent.com/72643027/114360284-cf093000-9baf-11eb-9284-55d7b27dd889.png){: width="70%" height="70%"}

Save & Test 진행.

<br/>

- 외부 Dashboard import.

https://grafana.com/grafana/dashboards 에서 원하는 dashboard 선택 후, import  - url 붙여넣고 Load.

Dashboard 선택 후 확인.

![image](https://user-images.githubusercontent.com/72643027/114360746-55257680-9bb0-11eb-852a-54eb858664be.png){: width="70%" height="70%"}

