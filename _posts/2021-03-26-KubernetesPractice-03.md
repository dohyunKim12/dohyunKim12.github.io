---
layout: post
title: "Kubernetes 실습(3)"
summary: DockerHub image Push & PUll
author: Dohyun Kim
date: 2021-03-18 21:30:00 -0400
category: Kubernetes
thumbnail: /assets/img/posts/Kubernetes.png
comments: true
---

### Summary

- DockerHub로 image를 push 하고, 다른 node에서 image를 pull해봄.  

- DockerHub를 이용하여 Kubernetes Cluster에 POD 생성.

- Kubernetes Cluster에 POD, Deployment, Service 생성 및 배포

---

#### DockerHub 에 image Push, Pull
- PUSH: (push 작업은 저번에 time 이미지를 만들었던 worker node에서 작업한다.)
    + DockerHub 로그인 ```docker login```
    + docker image tag 걸기  
        ```docker tag [이미지명] [DockerHub ID]/[repository명]:[tag명]```  
        repository명은 DockerHub에 생성할 repository 이름을 작성해주면 됨.
    + docker image push
        ```docker push [DockerHub ID]/[repository명]:[tag명]```

    이렇게 해주면 DockerHub에 실제로 repository가 생성되고, image가 올라간다. (시간 조금 소요)

    ![image](https://user-images.githubusercontent.com/72643027/112596257-eaa1d600-8e4e-11eb-83a6-50e985351678.png){: width="80%" height="80%"}

- PULL: (pull 은 반대로 time 이미지가 아직 존재하지 않는 master node에서 진행.)
    + docker image pull
        ```docker pull [DockerHub ID]/[repository명]:[tag명]```

성공적으로 time (1.0 ver) image 가 push 되고 pull 받아졌다.

#### Kubernetes 명령어 - kubectl
```kubectl [Command] [Type]/[Name] [Flags]```  
- Command : 자원에 실행하려는 동작
    + ```create``` : 자원 생성
    + ```get``` : 정보 가져오기
    + ```describe``` : 자세한 상태 정보
    + ```delete``` : 자원 삭제 
    + ```logs``` : describe 명령어와 함께 오류 처리 시 사용

- Type : 자원의 type
    + ```pod``` : 하나의 container 또는 비슷한 container들의 집합.
    + ```service``` : pod는 DHCP로 IP가 할당되어 계속 IP가 바뀌게 되는데, service가 anchor 역할을 하게 되어(service는 고정IP) pod로의 접근을 가능하게 해 준다. 즉 pod를 가리키는 포인터 역할을 함.
    + ```namespace``` : 논리적으로 pod들을 구분하기 위함.
    + ```deployment``` : pod가 특정상태를 벗어나게 되면, 다시 정상상태로 되돌려줌.

- Name : 자원 이름
- Flag : 옵션


#### Kubernetes yaml file
- apiVersion : 이 object를 생성하기 위해 사용할 kubernetes API version을 명세.
- kind : 어떤 종류의 object를 생성하고자 하는 지를 나타냄. (Pod, Deployment, Service, ...)
- metadata : object에 이름을 부여하고 object를 Unique(유일)하게 구분지어 줄 data.
    + name : 동일한 namespace 상에서 유일한 값.
    + labels : 특정 kubernetes object만 나열하거나 검색할 때 쓰이는 **key-value** 값.
    + spec : 생성할 object의 구체적인 내용을 정의. (spec에 대한 format 은 object 종류마다 다르다.)  
        <자주 사용되는 항목>
        - containers : pod에는 1개 이상의 container를 포함가능. 원하는 container 수 만큼 정의.
        - image : pull 받아올 docker image의 주소
        - imagePullPolicy : IfNotPresent (local에 없으면 pull), Never, Always
        - replicas : 원하는 pod 갯수 정의
        - selector : controller가 어떤 pod를 감시해야 하는지.
        - template : 새 pod를 launching 하는데 사용할 template.

---

### Kubernetes yaml file 작성 & 배포 (POD)
- kubernetes pod yaml 파일 생성 (test_pod.yaml)  
```
apiVersion: v1
kind: Pod
metadata:
  name: time
  labels:
    app: time
  namespace: default
spec:
  containers:
  - name: time
    image: dohyunkim12/time:1.0
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 8888
```

- Pod 생성  
    ```kubectl apply -f test_pod.yaml```
- Pod 확인  
    ```kubectl get pod -o wide | grep time``` ( -o wide option은 Pod에 대한 전체적인 정보를 표시해줌. IP주소 포함.)
- Pod 동작 확인  
    ```curl http://pod_ip:8888```

    ![image](https://user-images.githubusercontent.com/72643027/112599183-f099b600-8e52-11eb-8509-eb3f71db680e.png){: width="50%" height="50%"}


*만약, 동작이 정상적으로 되지 않는다면 kubectl이나 kube-flannel 등이 문제일 수 있음. (time image는 정상이라는 가정 하에)

이럴 경우, ```kubectl get pod -A``` 해서 pod상태들을 확인해 보면, READY 되지 않은 pod들이 보임. 예를 들어 kube-flannel 이 정상작동하지 않는 경우, 이전에 작성했던 kube-flannel.yml 파일이 문제일 수 있음(syntax error). 따라서 파일을 다시 작성할 경우, ```kubectl delete -f kube-flannel.yml```로 먼저 pod를 삭제한 다음, 파일 수정 후, 다시 ```kubectl apply -f kube-flannel.yml```로 Pod를 생성시켜줌. 이후 ```kubectl get pod -A``` 로 다시 상태 확인.

<br/>

### Kubernetes yaml file 작성 & 배포 (Deployment)
- kubernetes pod yaml 파일 생성 (test_deployment.yaml)
``` apiVersion: v1 kind: Deployment metadata: name: time labels: app: time namespace: default spec: selector: matchLabels:
      app: time
  replicas: 2   # 포드 생성 수. (과부하시 분산을 위한 목적)
  template:
    metadata:
      labels:
        app: time
    spec:
      containers:
      - name: time
        image: dohyunkim12/time:1.0
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8888
```
- Deployment 생성  
    ```kubectl apply -f test_deployment.yaml```
- Deployment 확인  
    ```kubectl get deployment```
- Pod 동작 확인  
    ```kubectl get pod```

![image](https://user-images.githubusercontent.com/72643027/112600741-eaa4d480-8e54-11eb-8bd8-5d7da1fb172c.png){: width="50%" height="50%"}

### Kubernetes yaml file 작성 & 배포 (Service)
- kubernetes pod yaml 파일 생성 (test_service.yaml)  
```
apiVersion: v1
kind: Service
metadata:
  name: time
spec:
  selector:
    app: time
  ports:
  - name: http
    protocol: TCP
    port: 8888
    targetPort: 8888
```

- Service 생성  
    ```kubectl apply -f test_service.yaml```
- Kubernetes Service 확인  
    ```kubectl get service```
- Service 요청 확인  
    ```curl http://ClusterIP:port```

![image](https://user-images.githubusercontent.com/72643027/112601310-a6fe9a80-8e55-11eb-9b42-23c6eb7f52a0.png){: width="50%" height="50%"}


