---
layout: post
title: "Network 스터디(1)"
summary: Linux(CentOS7) 에서 network setting
author: Dohyun Kim
date: 2021-02-03 21:30:00 -0400
category: Network
thumbnail: /assets/img/posts/network.png
comments: true
---

Linux(CentOS7) 환경에서 Computer Networking이 어떻게 설정되고, 동작하는지 알아보는 스터디 과정.

최종 궁극적 목표로는, 실 장비(서버)를 만지며 네트워크를 구성하고 회사의 sw제품을 이해하여 원하는 대로 network를 자유자재로 구성할 수 있는 것.

첫째 날(2021.1.28) 

-   가상머신(VMware또는 VirtualBox)에 CentOS7을 2대 설치하고, network setting 해보기.
-   VMware 자체 설정으로 network 대역폭 나눠보기.

먼저, VMware를 사용함에 있어서 실제와 가상의 개념을 나누면 더욱 헷갈린다. 우리가 현재 사용하는 것은 가상의 machine이지만, 그냥 physical 하게 존재한다고 생각하고 다루는 것이 편리하다.

VMware Workstation에서 제공하는 네트워크 장치는 크게 (Network Adapter, VMnet Network Switch, DHCP Server, NAT)가 있다. 

1) **Network Adapter(Network Interface Controller)** - 흔히 LAN(Local Area Network)카드라 부르는 장치. VMware를 설치함으로써 생성되는 network adpater는 크게 2가지인데, 하나는 VM에 장착된 network adapter, 다른 하나는 Host computer에 가상으로 장착된 network adapter(VMnet1, Vmnet8)이다. 필요에 따라 추가, 제거가 가능하며 이 둘이 VMnetwork Switch에 연결되어 통신한다.

2) **VMnet Network Swtich** - 내부적으로 VMnet이라는 총 255개(window ver - 10개)의 network switch를 제공한다. network adapter와의 연결 관계는 어댑터가
- Bridge mode   ->  VMnet0

- Host-only         ->  VMnet1

- NAT                   ->  VMnet8

와 같이 각 모드에 따라 VMnet에 연결된다. (255개의 switch 중 VMnet0, VMnet1, VMnet8을 전용 switch로 설정, 이외의 VMnet2, VMnet3.... 등은 Custom 스위치로 사용자가 직접 설정.)

![image](https://user-images.githubusercontent.com/72643027/109156613-622f0780-77b4-11eb-9a07-6e67bf0ffc31.png){: width="70%" height="70%"}

3) **DHCP Server** \- Dynamic Host Configuration Protocol의 약자로 동적으로 호스트를 설정 할 수 있는 dhcp 서버를 제공한다. 이 서버가 제공되므로 TCP/IP 설정이 자동으로 할당되고 관리됨. 우리가 네트워크에 연결하기 위해서는 IP주소, subnet mask, Gateway, DNS주소 등을 일일이 모두 설정해 주어야 하는데 **(/etc/sysconfig/network-scripts/ifcfg-ens33)** 이런 것들을 자동으로 할당, 관리해줌. 이러한 DHCP Server는 기본적으로 VMnet1, VMnet8 스위치에 연결됨. (필요시 추가, 제거 가능)

4) **NAT** - Network Address Translation. 사설IP와 공인 IP의 변경을 담당한다. networking에서 IP주소가 10. 또는 172. 또는 192.으로 시작하는 경우 사설 IP이다. 사설 IP로는 같은 network아래에 있는 host들끼리 통신이 가능하지만, 바깥(internet)과 통신할 경우 그러한 사설 IP가 공인 IP로 변경되어 나간다. (충돌 없이,, IP를 식별자로써 이용할 수 있도록)  이때, 사설 IP와 공인 IP 간의 변경을 해주는 것이 NAT이다. switch를 거치면서 translation이 이루어짐.

<VMware network 연결방식>

Bridged, NAT, Host-only 세가지 방식이 있음.

(https://unabated.tistory.com/entry/VMware-%EC%9D%98-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EA%B5%AC%EC%84%B1%EA%B3%BC-%EC%97%B0%EA%B2%B0%EC%9D%98-%EC%9D%B4%ED%95%B4)

여기까지, VMware에 대해 기본적인 network 개념을 이해한 후, 

CentOS7 이미지를 다운로드하여 VMware에 설치해 보았다. (이미지 - [www.centos.org/download/](https://www.centos.org/download/) 에서 x86\_64 DVD iso다운)

설치하고 www.google.com에  ping 테스트(정상).

![image](https://user-images.githubusercontent.com/72643027/109157012-dec1e600-77b4-11eb-99d0-fb4467bd0ddb.png)

현재 NAT으로 adapter가 설정되어 있으므로 외부와 통신할 수 있는 상태이다.

![image](https://user-images.githubusercontent.com/72643027/109157055-e84b4e00-77b4-11eb-8d81-8447226997cf.png)

CentOS\_second라는 이름으로 centos를 하나 더 생성, ens33 파일을 dhcp(자동)이 아닌 static(수동)으로 IP주소 설정 후

network 재부팅하여 ping 찍어봄.

network 재부팅 명령

```
systemctl restart network
```

  
이후, VMware에서 network adapter를 각각 하나씩 추가하여, 네트워크 대역폭을 나눠보았다. network adapter는 Host-only로 설정. (ens33 - NAT, ens37 - HostOnly)

network settings

```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

![image](https://user-images.githubusercontent.com/72643027/109157080-ef725c00-77b4-11eb-91d4-98c6cbfa9894.png)

![image](https://user-images.githubusercontent.com/72643027/109157138-fd27e180-77b4-11eb-938e-24c795e59733.png)

앞으로 ens37 IP addr를 이용해 Host OS(우분투)의 shell에서 ssh로 접속하여 사용할 것이다.

(Centos1 => 192.168.140.100  Centos2 => 192.168.140.200)

여기까지 완료하면, VM 두대는 현재 다음과 같이 연결되어 있는 상태이다.

![image](https://user-images.githubusercontent.com/72643027/109157213-13ce3880-77b5-11eb-9b44-0aaa409ce1ae.png)

