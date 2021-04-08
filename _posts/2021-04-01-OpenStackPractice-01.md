---
layout: post
title: "OpenStack 실습(1)"
summary: Horizon Dashboard를 이용한 instace 생성 및 network 구축.
author: Dohyun Kim
date: 2021-04-01 21:30:00 -0400
category: Kubernetes
thumbnail: /assets/img/posts/OpenStack.png
comments: true
---

### Summary

- Horizon Dashboard (web server) 를 이용한 Instance 생성.

- Private network, Router 생성으로 VM instance 간 통신.

---

### 실습 1. 동일 Private network 상에서 VM Instance 생성과 통신.

- Private network 구축.  
Instance 생성 전에, 먼저 private network를 구축해 주자. 'Network' 탭의 '네트워크' 로 들어가서 '네트워크 생성'을 클릭. network 이름은 ```private-1```으로 하고, subnet과 subnet 세부 정보를 다음과 같이 설정한다. 

    ![image](https://user-images.githubusercontent.com/72643027/113235021-5962b180-92dd-11eb-8735-291a105bbf31.png){: width=70% height="70%"}

    ![image](https://user-images.githubusercontent.com/72643027/113235110-7dbe8e00-92dd-11eb-8b6f-679e8d80edfe.png)

    (subnet 세부 정보는 입력해주지 않아도 자동으로 할당되지만, 이번 실습에서는 다음과 같이 VM instance가 생성될 때 IP 주소를 10.0.0.50 ~ 10.0.0.200 사이 값으로 받게끔 직점 명시해주었다.)

    생성을 눌러 ```private-1``` network가 성공적으로 생성된 것을 확인했으면, VM instance를 생성한다.


- VM Instance 생성.   
먼저 Instance 생성이므로 'Compute'탭의 '인스턴스'로 들어가서 '인스턴스 시작' 으로 VM 2개를 생성한다. Instance 이름은 private-1 으로 하고 부팅 이미지는 ubuntu_16.04, Flavor는 m1.small, network는 private-1 로 선택하여 생성.   
(VM 접속을 위한 보안 그룹 설정은 기존에 default 보안그룹 설정을 완료하였으니 건너뛴다.)

    인스턴스 생성 후, Status가 ```Active```인지 확인한다.

    이제 2개의 VM instance를 private network (private-1) 에 생성했으니, 네트워크 토폴로지로 들어가서 연결상태를 확인한다.

    ![image](https://user-images.githubusercontent.com/72643027/113235760-a4c98f80-92de-11eb-8b09-588d234c87fd.png){: width=70% height="70%"}


    이렇게 두 개의 Instance가 하나의 private network에 잘 붙어 있다. 이 둘이 통신되는것은 당연하겠지만, 그래도 console로 확인을 해보자.

    지금 상태로는 당연히 나의 local에서 VM으로의 통신은 불가능하다. 나의 local pc는 DCN lab실의 network와 VPN으로 연결되어 있으나, Pulbic network로 가정한 192.168~~ 대의 network 를 사용하고 있기 때문이다. 결과는 이러하다. (통신 불가)

    ![image](https://user-images.githubusercontent.com/72643027/113236501-ee66aa00-92df-11eb-838e-8e0ef012b4ba.png){: width=70% height="70%"}


    Dashboard 상에서 VM instance의 console 에 들어갈 수 있다. 접속하여 다른 VM instance로 ping 을 날려보자. (통신 성공.)

    ![image](https://user-images.githubusercontent.com/72643027/113236468-ddb63400-92df-11eb-9c1d-a0e44b148f4d.png){: width=70% height="70%"}



### 실습 2. 다른 Private network 상에서 VM Instance 생성과 통신.

    새로운 실습을 위해 기존 VM 2개 중 하나를 삭제한다. (Instance 생성 MAX_LIMIT = 2 인 상태.)

- 새로운 Private network 구축.  
    실습 1에서 했던 방식과 동일한 방법으로 ```private-2``` private network를 생성한다. private-1 network 의 대역이 10.0.0.0/24 이었으므로 이번엔 10.0.100.0/24 로 구성하자.  
    Subnet 이름 역시 private-2_subnet 으로 해주고 Subnet 세부 정보에서 Pools 할당을 10.0.100.50,10.0.100.200 으로 설정해준다.

- 새로운 Private network (private-2) 에 VM instance 생성.  
    설정값들을 기존 VM instance 생성시와 동일하게 해주고, network 선택만 private-2로 해준다. '인스턴스 시작' 을 눌러 생성. private-2 에 잘 붙어 있나 network topology를 확인해 보자.

    ![image](https://user-images.githubusercontent.com/72643027/113237113-363a0100-92e1-11eb-9f08-7feb0db52ec2.png){: width=70% height="70%"}


    ![image](https://user-images.githubusercontent.com/72643027/113237160-54076600-92e1-11eb-81ca-062e902ad89b.png){: width=70% height="70%"}


    private network 2개에 각각 하나씩의 instance 들이 아주 잘 붙어 있다. 이 상황에서 각각의 VM 들은 당연히 통신이 불가능 하다. 이 둘 간의 통신을 가능하게 해 주기 위해 ```Router```를 생성하자.

- Public network에 Router 추가.  
    라우터 추가를 눌러 라우터를 추가한다. 이름은 router-1, 외부 network 선택에서 현재 public network 하나밖에 없으므로 그것을 선택해 준다.

    ![image](https://user-images.githubusercontent.com/72643027/113237359-a2b50000-92e1-11eb-9349-a6e9bec55db7.png){: width=70% height="70%"}


    Router icon을 클릭하여 Interface 를 추가해 준다. 먼저 subnet을 private-1 에 대해 추가해 준 다음, private-2 도 동일한 방식으로 추가해 준다. 그러면 private network 1, 2 모두에 다음과 같이 연결이 된다.

    ![image](https://user-images.githubusercontent.com/72643027/113244567-3b527c80-92f0-11eb-86fa-5f7495a8acdd.png){: width=70% height="70%"}


    이제 setting이 완료되어 두 vm 간 통신이 가능하다.

    







