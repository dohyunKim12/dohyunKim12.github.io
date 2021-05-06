---
layout: post
title: "Kubernetes 실습(2)"
summary: Docker image 생성 및 배포과정 실습.
author: Dohyun Kim
date: 2021-03-18 21:30:00 -0400
category: Kubernetes
thumbnail: /assets/img/posts/Kubernetes.png
comments: true
---

### Summary

Docker에서 Dockerfile 을 이용해 Container image를 생성하고  
어떻게 배포하는지 과정을 살펴본다.

---

#### Docker Image 란?
- service 운영에 필요한 server program, source code 및 library, compile된 실행 파일을 묶는 형태,  
    특정 process를 실행하기 위한 모든 file과 설정값을 지닌 것.  
- 하나의 image는 여러 container를 생성할 수 있고, Container 삭제 시에도 이미지는 남아있음.
- docker image 들은 github와 유사한 ```DockerHub```를 통해 형상관리 및 배포가 가능함.
- Image 생성 시, Dockerfile 이라는 파일로 이미지 생성.
- Dockerfile 에는 source와 함께 의존성 package 등 사용했던 설정 파일을 version 관리 하기 쉽게 명시됨.


#### Layer 란?
- Layer는 기존 image에 추가적인 파일이 필요할 때, 다시 삭제 후 새롭게 생성하는 방식이 아닌 해당 file을 추가하는 개념.
- Image는 여러 개의 Read-Only Layer로 구성되어 있고, file이 추가되면 새로운 RW layer가 생성됨. (생성된 RW layer는 작업 완료 후 반영하여 version update 시 RO layer로 바뀐다.)
- 여러 개의 Layer를 묶어, 하나의 file-system 으로 사용할 수 있게 해줌.

![image](https://user-images.githubusercontent.com/72643027/112418646-3da35c80-8d6d-11eb-9074-96b1b4ff3a91.png){: width="80%" height="80%"}

![image](https://user-images.githubusercontent.com/72643027/112418888-b4d8f080-8d6d-11eb-8ef8-ba752cb860c3.png){: width="60%" height="60%"}


##### Read Write Layer (Container Layer)  
- 변경 가능함.
- Image Layer에서 추가적으로 작업이 이루어지는 Layer.
- 추가적 작업이 끝나면 Image Layer(RO layer)로 만들 수 있음.

##### Read Only Layer (Image Layer)
- 설정값을 변경할 수 없는 Layer.
- Container가 생성되기 전, 필요 구성 요소를 갖춘다.

---
<br/>


### Docker 명령어
- ```docker ps``` 실행중인 Container들을 확인. (-a option : 종료된 container들까지 확인 가능.)
- ```docker image ls``` or ```docker images``` Local repository에 존재하는 docker image들 확인.
- ```docker search [image]``` DockerHub에 있는 image 검색.
- ```docker create [option] [image명]``` 컨테이너 생성.(image가 저장소에 있어야만 가능)(생성만 함. run은 생성하고 실행까지.)(--name option : container에게 별칭 부여)
- ```docker start [option] [container 별칭 or container ID]``` 컨테이너 실행.
- ```docker stop [container ID]``` 컨테이너 종료.
- ```docker attach [option] [container 별칭]``` 실행되고 있는 container에 접근할 때 사용.
- ```docker run [option] [container 별칭 or container ID]``` 이미지가 존재하지 않으면 DockerHub에서 image pull 한 뒤, 컨테이너를 실행시킨다.  
    option : 
    + -i : container의 표준 입출력을 사용.
    + -t : 단말 device를 사용.
    + -d : container를 생성하고 background 에서 실행.
- ```docker build --tag "[태그명]" [dockerfile 위치]``` dockerfile 을 이용하여 image 생성.
- ```docker exec [option] container명 command``` 주로 동작중인 container shell에 접근할 때 사용. (ex: docker exec -it 컨테이너명 /bin/bash)  
- ```docker rm container명``` Container를 삭제.
- ```docker rmi container이미지명``` Container image를 삭제.  
    (```docker rmi $(docker images -f "dangling=true" -q)``` : <none> 과 같이 오류로 image명이 없는 것들을 삭제.)
- ```docker logs container명``` Container에 문제가 생겼을 경우, 확인하는 log.  
<br/>


### Dockerfile 문법
- FROM : 새로운 build 단계를 준비하고, 다음 명령어들의 기본 image 를 지정.  
    ```FROM [--platform=<platform>] <images> [AS <name>]```
- ARG : Dockerfile 에서 FROM 앞에 올 수 있는 유일한 명령어.  
    ARG 명령어로 선언된 변수를 사용할 수 있음.
- RUN : 현재 image의 위인 새 Layer(Container layer, RW layer) 에서 실행됨.   
    실행된 후 결과를 커밋하고 커밋된 image의 결과물은 Dockerfile의 다음 step에서 사용됨.  
    ```RUN <command>``` (기본적으로 linux 에서는 /bin/sh 로 실행.)
    ```RUN ['executable', 'param1', 'param2'] ```
- CMD : 실행 중인 Container에 기본 환경을 제공하기 위함. Dockerfile에서 단 하나의 CMD 명령어만 존재 가능하다.  
    ```CMD ['executable', 'param1', 'param2']```  
    ```CMD ['param1', 'param2']```  
    ```CMD command param1 param```
- LABEL : image 에 metadata를 추가함. (Key-Value)
- EXPOSE : Docker에게 Container가 Runtime에서 어떤 network port에 대기중인지 알려줌. Container를 실행할 때 -p flag로 expose 된 port를 개방.
- ENV : 환경 변수를 Key-Value로 설정.   
    ```ENV <key> <value>```  
    ```ENV <key1>=<value1> <key2>=<value2>```
- ADD : 새 file, directory, remote file의 URL주소 등 <src>로 받고 image의 파일 system의 <dest>에 추가.  
    ```ADD [--chown=<user>;<group>]<src>...<dest>```

### DockerHub Image Pull, Push
- Push
    1. DockerHub login ```docker login```
    2. Docker Image tag 걸기 ```docker tag [이미지명] [DockerHub ID]/[repository 명]:[태그명]```
    3. Docker Image Push ```docker push [DockerHub ID]/[repository 명]:[태그명]```
- Pull
    Docker Image Pull ```docker pull [DockerHub ID]/[repository 명]:[태그명]```  
<br/>

---
### 실습(1)

#### Nginx Web Server 구축
- Nginx image pull  
    ```docker pull nginx```
- image 확인  
    ```docker images | grep nginx```

![image](https://user-images.githubusercontent.com/72643027/112425511-8b25c680-8d79-11eb-9af6-2be188ebbb44.png){: width="60%" height="60%"}


- Container 이름을 ```nginx-test``` 로 정한 후 port 80 으로 가동.  
    ```docker run --name nginx-test -d -p 8080:80 nginx``` (-d : run background, -p : publish (port 지정))
- nginx-test Container 실행 확인  
    ```docker ps | grep nginx```
- nginx-test Container 상태 확인  
    ```docker stats nginx-test --no-stream``` (--no-stream : 상태 한번 띄우고 종료.(not waited))

![image](https://user-images.githubusercontent.com/72643027/112426596-937f0100-8d7b-11eb-94ad-f7df221d7862.png){: width="60%" height="60%"}


- Web Server 확인  
    ```curl http://localhost:8080```

![image](https://user-images.githubusercontent.com/72643027/112426816-ece73000-8d7b-11eb-86fc-0c736e4cc999.png){: width="60%" height="60%"}


- Web server Container 접속  
    ```docker exec -it nginx-test /bin/bash```
- Web Server Container의 index file 확인. (index file 위치 : /usr/share/nginx/html)  
    ```cat /usr/share/nginx/html/index.html```

![image](https://user-images.githubusercontent.com/72643027/112428156-143efc80-8d7e-11eb-9cfe-8289ec73f702.png){: width="60%" height="60%"}
 

<br/>

---
### 실습(2)

#### Web Container 생성
test directory 생성 후, ```time.py```, ```Dockerfile```, ```requirement``` file 작성. (기능 : 현재 시간을 반환해주는 web server)  

![image](https://user-images.githubusercontent.com/72643027/112428492-98917f80-8d7e-11eb-9f19-f5c7bbe1f7d6.png){: width="60%" height="60%"}


time.py 
```python
from flask import Flask, request
import datetime

app = Flask(__name__)

def get_cur_time():
        return str(datetime.datetime.now()) + "\n"

@app.route("/",methods=["POST","GET"])
def main():
        if request:
                return get_cur_time()

if __name__ == "__main__":
        app.run(host='0.0.0.0',port=8888,threaded=True)
```
<br/>

Dockerfile 
```
FROM centos:7

RUN yum update -y && yum install -y python3 \
python3-pip

COPY ./requirement.txt /app/requirement.txt

WORKDIR /app

RUN python3 -m pip install pip --upgrade

RUN pip3 install -r requirement.txt

COPY . /app

CMD ["python3","time.py"]
```
<br/>

requirement.txt
```
flask
```
<br/>

#### Web Container Dockerfile 생성
- test directory 확인

![image](https://user-images.githubusercontent.com/72643027/112430355-603f7080-8d81-11eb-95af-97b8bcef84eb.png){: width="60%" height="60%"}


- Docker Image build  
    ```docker build --tag time:1.0 .``` (--tag option으로 이미지명 뒤에 : 을 붙여 version을 명시할 수 있음.) (명시하지 않으면 자동으로 latest로 설정됨.)

![image](https://user-images.githubusercontent.com/72643027/112431751-43a43800-8d83-11eb-9b13-cb912ade0b57.png){: width="60%" height="60%"}


- 만들어진 Docker image 확인  
    ```docker images | grep time```
- 만들어진 image 로 docker Conatiner 생성.  
    ```docker run --name time-web -d -p 8888:8888 time:1.0```
- docker Conatiner 확인  
    ```docker ps | grep time``` (만약, ps에서 안뜨고 ps -a (종료된 상태) 로 검색된다면 오류. docker logs를 살펴보거나 test directory 하위 파일들의 문법오류를 check.) 
- Web Container 작동 확인  
    ```curl http://127.0.0.1:8888```

![image](https://user-images.githubusercontent.com/72643027/112432364-258b0780-8d84-11eb-84da-a49d8963f440.png){: width="60%" height="60%"}  

<br/>

--- 

#### 마무리

정리해보자면, 
1. ```docker build --tag 이미지명:version .``` 으로 이미지를 생성.(현재 test dir 의 Dockerfile, time.py, requirement.txt 를 기반으로)  
2. ```docker run``` 해서 docker 이미지 기반으로 container가 실행됨.
3. ```docker ps | grep 컨테이너명``` 으로 잘 돌아가고 있는지 확인.
4. ```curl http://127.0.0.1:8888``` 으로 time-web 컨테이너의 작동을 직접 확인.

잘 되지 않는 경우 ```docker logs 컨테이너명``` 으로 로그 확인하여 에러 해결.

다음시간에는, 만든 image를 DockerHub 에 배포하고 Kubernetes를 통해 다른 Worker node에 배포해 보는 작업을 해 볼 것이다. 
