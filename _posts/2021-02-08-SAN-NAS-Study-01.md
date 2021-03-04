---
layout: post
title: "SAN/NAS study(1)"
summary: SAN, NAS basic architecture
author: Dohyun Kim
date: 2021-02-08 21:30:00 -0400
category: Network
thumbnail: /assets/img/posts/san.jpeg
comments: true
---

SAN과 NAS 관리자 가이드 - 커티스 프레스톤(한빛미디어) 

책 내용을 기반으로 작성하였습니다.

### **Chapter1 - SAN과 NAS란?**

컴퓨터가 개발되고 운영되면서 많은 개발자들은 디스크를 쉽게 공유하는 방법에 대해 연구하였다. 결과 슈가르트가 정의한 SASI(Shugart Associates System Interface)를 기반으로 하여 SCSI(Small Computer System Interface)가 탄생하였다.

결과적으로 SAN과 NAS는 각각

**SCSI**가 **SAN**으로 발전한 형태,

**NFS, CIFS**가 **NAS**로 발전한 형태이다.

따라서 SCSI, NFS, CIFS가 무엇인지 알아보고, 진화된 형태의 SAN과 NAS는 이전의 SCSI, NFS, CIFS보다 어떠한 장점을 가지고 있는지 알아보자.

**< SCSI >**

SCSI는 SASI에서 진화된 구조로 병렬(parallel)형태를 띄고 있다. 

![image](https://user-images.githubusercontent.com/72643027/109924154-34414a00-7d03-11eb-9920-ab5549d8acf9.png){: width="90%" height="90%"}

이러한 형태의 물리적 cable이다.

![image](https://user-images.githubusercontent.com/72643027/109924187-41f6cf80-7d03-11eb-8278-0216712bba32.png){: width="50%" height="50%"}


이러한 구리선 기반의 SPI(SCSI Parallel Interface)는 물리적, 전기적 특성때문에 여러 한계점이 존재한다.

(SPI = 기존의 병렬구조를 취하고 있는 SCSI)

-   하나의 BUS에 장치를 16개만 연결할 수 있다.
-   SPI는 케이블 길이에 제한을 받는다. (SCSI 체인에 부착되는 각 장치가 사용가능한 전체 길이를 짧게 만든다.)

크게 이러한 제약사항이 존재하는데, 이를 극복한 것이 '직렬 Fiber Channel Protocol'이다.

마찬가지로 SCSI이지만, SPI와 달리 하나의 경로로 보내고 다른 하나로 받는 직렬 protocol이다.

직렬 Fiber Channel Protocol은 SPI에 대비하여 여러 장점들을 갖는다.

-   거리 : 이론적으로 장치간 거리 제한이 없어짐.
-   속도 : 단일 Gbit 채널 속도는 SPI보다 느리지만, 더 넓은 대역폭 사용을 위해 '다중 Fiber Channel Linking'을 할 수 있다.                -> 이렇게 할 경우 SPI보다 빠름.(~40mbps)
-   하나의 computer에 수백만 개 장치 연결 : 이론상 SPI 보다 직렬 SCSI의 adapter에 연결할 수 있는 장치 수가 백만 배 많다.              -> 자원 공유에 유리.

이제 SAN을 정의할 수 있다. **SAN**은 '직렬 SCSI protocol을 이용해 통신하는 두 개 이상의 장치'이다. 다시말해, SAN은 데이터를 전송하기 위해 직렬 SCSI protocol을 사용하는 네트워크 인 것이다.

SAN사용 이전과 이후는 '백업 및 복구'에서 많은 변화 양상을 보인다.

초기에는 각 서버에 조그맣게 달린 tape-drive를 이용하여 백업했다. 다루는 data양이 많지 않았기 때문에 System에 부담이 없었다. 그러나 점차 System이 커지고 처리하는 data양이 방대해지면서 이 방식으로 백업하기에는 부담이 커졌다. 

\-> 방안1) 미디어 서버를 이용 : 각 서버가 네트워크를 통해 data를 전달하지 않고, 자체 로컬 tape-drive로 백업 가능.(bandwidth문제는 해결.) 그러나 충분히 큰 tape-library를 대여하는데 많은 비용이 들어감. 비효율적.

\-> 방안2) 단일 library를 다중 host와 연결 : 하나의 tape-library를 많은 host들이 공유하는 것이다. 이 방식으로 비용은 절감할 수 있으나, 비효율적 문제가 남아있다.

비효율적 문제를 어떻게 해결할 수 있을까? SAN을 이용하므로써 아래의 것들이 가능해진다.

-   대용량 db서버가 하나의 local-tape drive로 백업하고 그 tape-dirve를 다른 대용량 서버가 사용할 수 있다.
-   대용량 db를 사용하는 서버의 cpu를 통해 data를 전달하지 않고, 그 disk를 백업한 다른 서버가 대용량 db서버의 disk를 볼 수 있다.
-   disk와 tape-dirve가 다른 서버의 cpu를 거치지 않고 data를 디스크에서 tape로 직접 전달하는 방식으로 연결된다.

***
< NFS, CIFS > **

**CIFS (or SMB)**(ServerMessageBlock) : IBM과 Microsoft가 개발한 file system. **Microsoft window 운영체제**에서 기본적으로 쓰이는 파일 공유 protocol. 여러 버전을 거치면서 최근에 CIFS(CommonInternetFileSystem)으로 명칭이 바뀜. 

**NFS** : Sun Microsystems가 개발한 file system. Unix의 기본 app이 됨. (NFS로 윈도우 파일을 공유하기는 쉽지 않다.) 

NFS는 TCP가 아닌 UDP를 기반으로 하는 protocol이다. 따라서 재전송을 처리하는 데에 문제가 존재한다. 

NFS, CIFS모두 자신의 local 파일 탐색기에서 네트워크로 연결된 storage를 일반 directory처럼 보여주고, 사용할 수 있게 해준다. 그러나 다음과 같은 문제점이 발생할 수 있다.

-   NFS, CIFS 서버를 사용하는 입장에서 NFS, CIFS에 대한 전문 지식이 필요할 수 있다. (일반적으로 전문지식이 없다고 가정)
-   대부분의 환경에는 UNIX, window 환경이 혼합되어 있다. (unix - NFS, window - CIFS 를 지원하는 상황에서, unix와 window간의 파일 공유를 함에 있어서 문제가 발생할 수 있다.) 
-   성능 - NFS나 CIFS 서버의 뒤에 위치한 (network상으로 연결되어 있는)disk는 local에 존재하는 disk 만큼 빠르게 사용할 수 없다.

이러한 문제들의 해결을 위해 NAS(NetworkAttachedStorage)가 탄생하였다.

**NAS**(NetworkAttachedStorage)는 NFS, CIFS 또는 DAFS를 통한 파일 공유를 전용으로 하는 컴퓨터 또는 장치이다. (일반적으로 Storage Server)

NAS는 host-swappable RAID를 이용해 가용성을 향상시키고 교정 유지보수에 필요한 시간을 줄였다. 또한 NAS는 NFS, CIFS를 제공하는 단일 서버로서 그 일을 수행해야하는 다른 서버의 수를 감소시킨다. 그리고 NFS와 CIFS모두를 통해 같은 directory를 마운트 할 수 있게 되었다. 

요약. NAS vs SAN

**NAS**

![image](https://user-images.githubusercontent.com/72643027/109924229-4fac5500-7d03-11eb-9225-0c43bafe424b.png){: width="70%" height="70%"}

**SAN**

![image](https://user-images.githubusercontent.com/72643027/109924279-5d61da80-7d03-11eb-92a7-38b5a67f66b9.png){: width="70%" height="70%"}

<br/>

|   |NAS    | SAN |
|----|-----|----|
|protocol | NFS / CIFS | 직렬 SCSI|
|접속장치 | LAN 스위치 | Fiber channel 스위치|
|공유 항목 | tape-disk, 또는 백업파일 전체&nbsp;&nbsp;  | 파일|
|파일시스템 관리| File Server | Application Server|
|접속 속도 결정 요인&nbsp; &nbsp; | LAN과 채널 속도에 의해 좌우됨 | 채널 속도에 의해 좌우됨|
