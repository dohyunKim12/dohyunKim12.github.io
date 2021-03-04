---
layout: post
title: "SAN/NAS study(2)"
summary: Fibre Channel architecture
author: Dohyun Kim
date: 2021-02-09 21:30:00 -0400
category: Network
thumbnail: /assets/img/posts/san.jpeg
comments: true
---

SAN과 NAS 관리자 가이드 - 커티스 프레스톤(한빛미디어)

책 내용을 기반으로 작성하였습니다.

### Chapter2 - 파이버 채널 구조

2장에서는 SAN의 기반기술인 Fibre Channel이 무엇인지 알아보고, Fibre Channel의 변형된 topology를 살펴봄과 동시에 Fibre Channel의 SAN특성에 대해 알아본다.

먼저 최첨단 기술인 SAN에서 왜 Fibre Channel을 사용하는지 살펴보자.

Fibre Channel은 legacy LAN(전통적)과 parallel SCSI 구조의 성능과 논리적 장벽을 뛰어넘기 위해 사용된다.

    <Fibre Channel의 특징>

-   전형적 비네트워크 protocol (SCSI) 지원
-   중요한 백업, 다른 동작을 위한 bandwidth 보장 등 Qos(Quality of Service) 제공
-   Connection delay가 거의 없음
-   다양한 frame을 사용하는 '고효율, 고 대역폭, 낮은 연결지연' 전송으로 효율적 전송 가능
-   Host에 영향 없이 장치 설치, 제거가 가능 -> 이 기능은 고가용성 system에 좋음(down 빈도를 낮춘다.)

한마디로 많은 양의 data를 빠르게 전송할 수 있다는 특징을 갖는다.

![image](https://user-images.githubusercontent.com/72643027/109925219-d31a7600-7d04-11eb-871f-c51c2cab90b4.png){: width="50%" height="50%"}

OSI 5 layer에 매칭되는 구조는 다음과 같다.

**FC-4(Protocol Mapping)**는 상위의 protocol과 통신하는 방법을 정의,

**FC-3(Common Service)**은 data striping같은 포트가 하나이상 필요한 app이 사용.

**FC-2(Framing)**은 OSI의 MediaAccessControl(Link layer)와 비슷하다. 상위계층(application layer)의 data를 frame으로 쪼개어 전송한다.

**FC-1(Order Set)**과 **FC-0(Physical)**은 OSI의 Physical layer와 비슷하다. Fibre Channel data 전송에 사용되는 다양한 매체를 정의한다.

'Fibre Channel network를 통해 SCSI tape-dirve로부터 SCSI disk로 data를 얻는 상황'을 가정하여 Fibre Channel이 어떻게 동작하는지 살펴보자.

1.  Server가 tape-drive에서 data를 검색하라고 요청.
2.  SCSI protocol이 정의한 대로 SCSI application은 그 data를 Fibre Channel network의 SCSI 맵으로 전달.
3.  FC-4 layer에서 맵은 application으로부터 data를 받는다.
4.  FC-3 에서 data를 암호화 및 압축한다.
5.  FC-2 에서 data를 frame으로 분할한다.
6.  FC-1 에서 분할된 frame을 인코딩한다.
7.  FC-0 Physical 매체를 통해 전달한다.
8.  Source(SCSI tape-drive) -> Destination(SCSI disk)으로 frame이 이동..
9.  SCSI disk의 port로 frame이 도착하면 (FC-0)
10.  FC-1 에서 도착한 frame을 디코딩한다.
11.  FC-2 에서 frame을 data로 reassemble 한다.
12.  이후 data를 application에 return.

이러한 과정은 직렬SCSI에서 동작하는 과정들이다. (Fibre Channel network) (Not Parallel SCSI)

직렬 SCSI는 단일 transport line을 통해 data를 송수신한다.

\- Fibre Channel Port -

기본적으로 N\_Port, F\_Port, E\_Port 세 가지 종류.

여기에 중재 루프(FC-AL)(FibreChannelArbitratedLoop) 기능을 추가한 NL\_Port, FL\_Port, G\_Port 등도 존재한다. 

-   N\_Port(Node Port) : 일반적으로 disk나 computer에 있는 포트. 
-   F\_Port(Fabric Port) : Switch에만 존재.
-   E\_Port : 대용량의 Fabric을 구성하기 위해 하나의 switch를 다른 switch로 연결하는 중간switch에 있는 확장 port이다.

이러한 구조에서 N\_Port는 E\_Port에 연결할 수 없고, N-N 또는 N-F, F-F 의 연결만 허용된다.

E Port는 기본적으로 switch에 존재하므로 switch끼리의 연결에 사용된다.

-   L\_Port : (중재 Loop에 참가할 수 있다는 의미) port가 L\_Port 뿐이라면 중재 loop하고만 연결할 수 있고, L\_Port 전용 port는 존재하지 않는다. L은 NL\_Port나 FL\_Port를 생성하기 위해 N 또는 F 의 끝에 추가된다.
-   NL\_Port : 중재 loop 기능이 있는 N\_Port. 즉 다른 node 또는 switch, 그리고 중재 loop에 연결가능.
-   FL\_Port : 중재 loop 기능이 있는 F\_Port. N, F, 그리고 중재 loop에도 연결가능.
-   G\_Port : 경우에 따라 E, FL, F Port로 동작할 수  있는 switch에만 존재하는 일반 port이다.

![image](https://user-images.githubusercontent.com/72643027/109925282-e594af80-7d04-11eb-89e1-4e7a1c71926f.png){: width="80%" height="80%"}


Fibre Channel Topolog & Port 종류와 연결형태

#### Fibre Channel Topology

-   Point-to-Point : 가장 저렴하고 간단. 말 그대로 두 node가 N-N 연결로 통신하는 형태.
-   Fabric : 각 N\_Port가 switch에 있는 F\_Port에 연결된 형태. switch에 연결된 node들은 그들이 장착된 port의 전체 대역폭을 사용할 수 있다. (성능은 좋으나 매우 비쌈.)
-   Loop : 일반적으로 loop는 여러 node들 끼리 맞물려 있는 형태이다. 논리적 문제 때문에 짧은 거리에서만 사용
-   FC-AL(Arbitrated Loop) : 중재 Loop. 보통 Hub를 사용한 FC-AL이 일반적이다. Hub는 loop를 생성하는 일을 한다. (node제거도 가능)

![image](https://user-images.githubusercontent.com/72643027/109925368-fe04ca00-7d04-11eb-9ed9-143fd43630bc.png){: width="30%" height="30%"}

FC-AL

![image](https://user-images.githubusercontent.com/72643027/109925404-0d841300-7d05-11eb-94a7-ca5bf470b07c.png){: width="30%" height="30%"}


HUB를 사용한 FC-AL(Commmon)

FC-AL 과 Fabric 은 다르다. 가장 큰 차이는 Fabric의 switch 대신 FC-AL에서는 hub를 사용한다는 것이다. FC-AL은 공유 매체이다. node가 loop로 data를 전송하려면 hub로부터 전송 권리를 반드시 중재해서 받아야 한다. (한번에 하나만 가능.)

Fabric + FC-AL

일반적으로 Fabric의 switch와 FC-AL의 HUB를 연결하여 Fabric과 FC-AL의 topology결합을 만들 수 있다. 이러한 결합으로 생성된 topology는 저렴한 FC-AL여러개를 고가의 Fabric에 연결하여 사용하므로 최적의 네트워크 효율을 낼 수 있다.

#### SAN 구축 블록

SAB의 주요 요소에는 HBA, Switch, HUB, Router, Disk-System, Tape-System, Cabling, SW 가 있다.

![image](https://user-images.githubusercontent.com/72643027/109925495-2f7d9580-7d05-11eb-86b6-31c87984cf7b.png){: width="70%" height="70%"}

SAN의 구조도. GBIC(gigabit interface converter)를 사용하여 Fibre를 연결한다.

![image](https://user-images.githubusercontent.com/72643027/109925641-56d46280-7d05-11eb-8152-3c353b1e2c01.png){: width="30%" height="30%"}


**HBA**(HostBusAdapter)와 GBIC을 통해 Server(Host)는 SAN에 연결된다. HBA는 fibre나 구리를 사용할 수 있고, Physical layer와 관계없이 Server를 SAN으로 연결한다. 

![image](https://user-images.githubusercontent.com/72643027/109925703-6784d880-7d05-11eb-92e9-939e430796e2.png){: width="30%" height="30%"}


dfdGBIC(GigabitInterfaceConverter)

**Switch**끼리 연결할 때는 AL이 아닌 Switch Fabric topology를 사용한다. 그림에서는 Host의 HBA를 이용해 HUB에 연결되고 HUB들을 FC(FibreChannel) switch를 이용해 연결하고 있다. 

**HUB**는 AL에만 관여한다. 장치가 HUB에 연결되면 HUB가 관리하던 AL이 초기화되고, 그 장치는 AL의 주소를 할당받으며 다시 Arbitrate를 시작한다.

Fibre Channel의 **Router**는 직렬 data stream을 parallel하게 변환하거나 또는 parallel한 data stream을 series하게 변환하기도 한다. ---> tape-drive같은 parallel SCSI 장치를 SAN에 plug할 수 있게 해준다.

**DiskSystem**은 크기와 모양이 다양하다. RAID를 어떻게 구성하느냐에 따라 달라진다. 

**Cable**은 광섬유 케이블이다. HybridFibreCoax에 이 Fibre cable이 사용된다.

**SW**는 protocol변환, 장치/경로 failover, load balancing과 같은 부분을 처리한다.
