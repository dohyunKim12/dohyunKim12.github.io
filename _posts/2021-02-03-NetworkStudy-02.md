---
layout: post
title: "Network 스터디(2)"
summary: NFS(Network File System)
author: Dohyun Kim
date: 2021-02-03 21:30:00 -0400
category: Network
thumbnail: /assets/img/posts/network.png
comments: true
---

### 2번째 Keyword - NFS

어떠한 서비스인지 공부해보고 VM장비(CentOS7) 2개를 가지고 서로 서비스 해보기.

'''systemctl''' 명령어도 추가로 공부해보자.

NFS(Network File System)란?

\- Client가 network상의 파일을 직접 연결된 storage에 접근하는 방식과 비슷한 방식으로 접근하도록 도와줌. ex - 파일 탐색기에서 ondrive 또는 googledrive가 보이는 것 처럼 그러한 파일 공유 storage service를 해준다.

마음대로 저장하거나 수정하는 것을 가능하게 해 주는 Server/Client 응용 프로그램. 사용자 system에는 nfs client가 설치되어 있어야 하고, server로 사용될 컴퓨터에는 NFS server가 설치되어 있어야 함.

특징: 다른 서버에 있는 partition을 마치 내 local 영역인 것 처럼 사용하기 때문에 보안에 취약. (보안에 민감한 회사는 nfs사용을 추천하지 않음)

실습내용: VMware로 생성한 두 개의 CentOS7 에서 하나를 nfs-server, 하나를 nfs-client로 구성하여 nfs가 잘 동작하는지 확인.

1) 먼저 centos7\_first(VM)을 nfs-server로, centos7\_second를 nfs-client로 사용하기로 함.

![image](https://user-images.githubusercontent.com/72643027/109922703-22f73e00-7d01-11eb-9872-c14af23ad9af.png){: width="100%" height="100%"}

2) 왼쪽의 가상머신(server)에 nfs 패키지(nfs-utils)를 설치

```
# sudo yum install nfs-utils
```

3) nfs-server 서비스 가동 및 재부팅 시 활성화

```
# systemctl start nfs-server
# systemctl enable nfs-server
```

4) nfs-server에서 공유할 폴더 및 파일 생성

![image](https://user-images.githubusercontent.com/72643027/109922730-2e4a6980-7d01-11eb-9daf-ff50b829d43d.png){: width="50%" height="50%"}

5) export 할 directory를 설정.

```
# vi /etc/exports
```

![image](https://user-images.githubusercontent.com/72643027/109922756-386c6800-7d01-11eb-9b44-a8892daa2e83.png){: width="30%" height="30%"}


\*    //접속을 허용할 client의 IP주소

(rw,sync,no\_root\_squash)    //권한, 동기화 설정, root가 아닌 사용자로부터 접근 가능하도록 설정

//공백 있으면 안됨.

6) export 수정내용 반영

```
# exportfs -r
```

7) NFS에 대해 방화벽 허가(server측에서만 하면 됨)

```
# firewall-cmd --permanent --add-service=nfs
# firewall-cmd --reload
# firewall-cmd --list-all
```

// firewall에 nfs서비스를 허용하도록 설정(영구적)

// firewall 재시작

// firewall list출력

![image](https://user-images.githubusercontent.com/72643027/109922806-47531a80-7d01-11eb-8144-d6737ccb7e8f.png){: width="50%" height="50%"}


services에 nfs가 추가되어 있는 것을 확인.

8) NFS공유 확인

```
# showmount -e
```

![image](https://user-images.githubusercontent.com/72643027/109923171-b9c3fa80-7d01-11eb-855b-ac4535f0c457.png){: width="30%" height="30%"}


9) NFS - **client측** mount 설정. (여기서부터는 오른쪽 VM으로 진행)

NFS 패키지 설치

```
# yum install nfs-utils
```

10) export 지점 확인

```
# showmount -e 192.168.140.100
```

![image](https://user-images.githubusercontent.com/72643027/109922860-5c2fae00-7d01-11eb-9132-2f7c62810c75.png){: width="30%" height="30%"}


server 주소인 192.168.140.100을 기준으로 mount 확인. (err발생원인 확인 못함.)

![image](https://user-images.githubusercontent.com/72643027/109922940-78cbe600-7d01-11eb-8dbc-f63a0d01c54b.png){: width="30%" height="30%"}


server, client 측 모두 rcpbind, nfs 재시작 하였으나 오류 해결 못함.

11) 마운트

```
# mount -t nfs -o sync 192.168.140.100:/sharing /nfsdir
```

// server IP를 기반으로 mount. sharing은 서버에서 nfs로 공유설정 해 놓은 dir, nfsdir는 client에서 server와 동기화시킬 dir.

// 이 때 server의 ip주소 192.168.140.100은 2개의 network-adapter(ens33, ens37)중 ens37(host-only)의 주소임. ens33의 주소로 적어도 무방하다.(마운트 잘 됨)

![image](https://user-images.githubusercontent.com/72643027/109923243-d2341500-7d01-11eb-9c1e-dd2afb5790b2.png){: width="30%" height="30%"}


결과: mount는 성공(공유기능엔 문제없음)

\*\*\* 추가적으로 nfs-client가 어떤 network-adapter로 통신하는지 알아보기 위해 nfs-server(centos7\_first)의 export script 수정해봄.

\-> Bridge, NAT, Host-only 모드에 따라 각 조건에 맞춰 되는 경우가 있고 되지 않는 경우가 있음.

\*\*\* 추가적으로 VM이 아닌 우분투(Physical Host)에서 nfs-server에 접근 시도(centos7\_first)

```
sudo apt-get install nfs-kernel-server
```

```
mount -t nfs -o sync 192.168.140.100:/sharing /home/dohyunkim/nfs
```

```
showmount -e 192.168.140.100
```

\*\*\* 같은 network내의 다른 host와 nfs서비스 가능한지 실험.
