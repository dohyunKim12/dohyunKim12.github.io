---
layout: post
title: "LinuxBible 3nd week Summary"
summary: CH12~CH15
author: Dohyun Kim
date: 2021-02-26 16:25:00 -0400
category: Linux
thumbnail: /assets/img/posts/LinuxBible.png
comments: true
---

# LinuxBible Summary(CH12~CH15)

CH12(Managing Disks and Filesystems)
---------------------------------
### LVM(Logical Volume Management)
LVM은 크게 PV(Physical Volume), VG(Volume Group), LV(Logical Volume) 으로 구분되어 있다.  
- PV(Physical Volume) - PV는 실제 물리 디스크를 LVM용도로 초기화한 물리 디스크 또는 파티션. 예를 들어 /dev/sda, /dev/sda1 등이 PV에 속함.  

![image](https://user-images.githubusercontent.com/72643027/109271295-a96bd580-7852-11eb-8314-b1f3b16add6f.png){: width="30%" height="30%"}

- VG(Volume Group) - VG는 하나 이상의 PV를 가지고 만든 물리적 volume의 집합. PV는 단 하나의 VG에만 포함될 수 있다. VG를 만든다는 것은 논리적 volume을 할당할 수 있는 disk 공간 풀 을 생성하는 것이다.

![image](https://user-images.githubusercontent.com/72643027/109271589-167f6b00-7853-11eb-9b71-534909c2479a.png){: width="30%" height="30%"}

- LV(Logical Volume) - LV는 사용자가 직접 다루는 논리적인 volume 공간. 여러 종류가 존재한다.
    + linear volume
    + RAID volume
    + striped volume
    + mirror volume
    + thin volume  

![image](https://user-images.githubusercontent.com/72643027/109271849-6d854000-7853-11eb-8686-386377784c51.png){: width="30%" height="30%"} 

대략 이렇게 존재한다고 생각할 수 있다.

이러한 LVM이 존재하여 우리는 볼륨 관리를 좀 더 쉽게 할 수 있다.

```fdisk -l``` 로 disk 위치 확인. 

![image](https://user-images.githubusercontent.com/72643027/109274616-0ec1c580-7857-11eb-8adf-e4febc807adb.png){: width="70%" height="70%"}

주로 사용되는 디스크는```/dev/nvme0n1``` 임을 확인하였다. 476.96 기가의 메모리를 갖고 있다. 

실습을 위해 32 기가 USB를 장착.

![image](https://user-images.githubusercontent.com/72643027/109277654-f5227d00-785a-11eb-9074-de6254fdb204.png){: width="50%" height="50%"}

이렇게 /dev/sda 로 disk 위치가 잡혔다. 

파티션을 다음과 같이 10기가씩 3개로 나누었다.

![image](https://user-images.githubusercontent.com/72643027/109278959-83e3c980-785c-11eb-94e3-36a284fb1b4c.png){: width="70%" height="70%"}

![image](https://user-images.githubusercontent.com/72643027/109279779-82ff6780-785d-11eb-9ae8-93c28da12469.png){: width="50%" height="50%"}

이렇게 자동으로 sda1, sda2, sda3 으로 설정되었다.

#### 1. PV 생성
```fdisk /dev/sda``` 명령어로 각 disk들의 타입을 리눅스의 LVM으로 변경.

![image](https://user-images.githubusercontent.com/72643027/109280866-e76ef680-785e-11eb-8c4e-592c0ae4f557.png){: width="50%" height="50%"}

이후 ```pvcreate /dev/sda1 /dev/sda2 /dev/sda3```으로 각 볼륨들을 PV로 만들어줌. 

#### 2. VG 로 합치기

```vgcreate myGroup /dev/sda1 /dev/sda2 /dev/sda3```로 Volume Group 생성.

```vgdisplay```로 Volume Group이 잘 생성되었는지 확인.

![image](https://user-images.githubusercontent.com/72643027/109281747-07eb8080-7860-11eb-8f29-ec7d428c340c.png){: width="50%" height="50%"}

#### 3. LV 만들기

```lvcreate```로 Volume Group의 파티션 생성. (하드디스크 조작 명령어는 fdisk)

![image](https://user-images.githubusercontent.com/72643027/109282287-ad065900-7860-11eb-8a41-3541bbeffb87.png){: width="50%" height="50%"}

이렇게 Physical Volume 10GB 3개를 하나의 Volume Group으로 합친 뒤(30GB), Logical Volume 500MB 6개로 나누어 보았다.(3GB)

결과 확인.

![image](https://user-images.githubusercontent.com/72643027/109282717-3f0e6180-7861-11eb-84d6-f6c000eda41f.png){: width="50%" height="50%"}

#### 4. FileSystem Format

```mkfs -t ext4 /dev/myGroup/myLV1``` 으로 파일시스템 포맷을 해줌. (myLV2,3,4,5,6 도 동일)

![image](https://user-images.githubusercontent.com/72643027/109283322-fe631800-7861-11eb-80ef-202298aca227.png){: width="50%" height="50%"}

#### 5. Mount

```mkdir /LVMdata``` 로 root 하위에 directory를 하나 생성.

```mount /dev/myGroup/myLV1 /LVMdata``` 로 myLV1을 LVMdata에 mount. (myLV2,3,4,5,6은 동일 폴더에 같이 마운트 할 수 없음. 다른 공간에 mount해야한다.)

---
이외에도 ```df -h /LVMdata```로 해당 디렉토리의 여유공간을 확인하여 LV에 공간부족시 ```lvextend -L + unmount 없이 추가확장이 가능하다.


CH13(Understanding Server Administration)
---------------------------------
서버 설정은 보통 /etc 디렉토리 하위에 .conf 파일로 존재.

REHL 에서 사용하는 RedhatPackageManager(RPM)은 사용성보다 보안과 안정성에 focus를 둔다.


**SSH(Secure SHell) Remote Access**  
```ssh```: 원격 로그인, 원격 실행.  
```scp```: client - server 간 파일 복사  
```sftp```: 원격 파일시스템에서의 FTP 제공  

##### scp의 단점
- 속성이 사라짐(파일의 수정시간 등)
- Symbolic Link가 사라짐(바로가기 링크 복사시 실제 원본 파일이 복사됨)
- 필요이상으로 많이 복사됨.

-> ```rsync```로 극복.  
```
rsync Options
-a: archive mode(-rlptgoD)
    -r: recursive
    -l: symbolic link 복사
    -p: preserve permissions
    -t: preserve modification times
    -g: preserve group
    -o: preserve owner(su only)
    -D: same as --devices --specials
        --devices: preserve device files(su only)
        --specials: preserve special files
-v: verbose mode(상세 정보 끌고옴)
```

##### SSH public key authentication
```ssh -keygen``` 으로 공개키/개인키 쌍 생성 가능.

사용예 : github에 나의 ssh 공개키를 등록해놓고 git clone 시 ssh 공개키, 개인키 인증을 사용하여 clone받는다.

##### rsyslog
```rsyslogd```: system log daemon. 여러 프로그램의 log message를 받아 적절한 log 파일에 작성.   
로그파일을 더욱 편하게 관리할 수 있다.

```/etc/rsyslog.conf``` 파일을 살펴보면 

![image](https://user-images.githubusercontent.com/72643027/109303186-c7026480-787d-11eb-8368-e180eba86056.png){: width="70%" height="70%"}

module(load) 로 모듈을 불러오는데,  
```imuxsock``` 모듈은 local system의 메세지를 받는다.  
```imklog``` 모듈은 --MARK-- 메세지를 로그 처리한다.  
```imjournal``` 모듈은 rlogind가 systemd journal에 access 할 수 있게 해준다.

Log 파일들을 다른 PC로 전송하기 위해서는 ```local```과 ```remote``` 모두 ```rsyslog.conf```파일을 수정해야 함.  
client 측 -> log file의 이름을 ```@loghost``` 로 변경.  
server 측 -> ```ModLoad imudp.so``` (UDP), ```ModLoad imtcp.so```(TCP) 부분 주석해제하여 port 514번을 열어준다.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ```firewalld``` 명령어로 방화벽 해제.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ```systemctl restart rsyslog.service``` 명령으로 재시작.

##### sar(system activity reporter)
```sar``` 명령어로 시스템 활동을 확인할 수 있음.
```
sar Options
    -u: CPU 사용량
    -d: disk 활동량
    -n 단위시간 횟수: 단위시간마다 횟수만큼 반복하여 data를 읽어옴.
```

CH14(Administering Networking)
---------------------------------
이번 장에서는 Linux의 네트워크 구성과 관리에 대하여 다룬다.  

```ip a``` 명령어로 network interface 정보를 알 수 있다.  

![image](https://user-images.githubusercontent.com/72643027/109410349-3e9ad580-79dd-11eb-8f0f-0984135ce4cb.png){: width="50%" height="50%"}

이와 같이 출력되는 화면에서 ```lo```는 loopback interface로 local에서 local으로 연결할 때 network 명령을 실행하는 interface이다.

```wlo1```은 wireless network interface로 외부와의 연결에 사용된다.

이와 같은 정보는 ```ifconfig``` 명령으로도 확인할 수 있다. 추가적으로 RX(수신 패킷), TX(송신 패킷), 유실된 패킷 정보를 포함한다.

```ping``` : 원격 IP와의 연결 확인. ( ex : ```ping www.google.com```)

```route``` : Route 정보 확인 가능.   
![image](https://user-images.githubusercontent.com/72643027/109410502-8ff79480-79de-11eb-96ab-6a38f7acaf31.png){: width="50%" height="50%"}

#### network setting
- CentOS : ```/etc/sysconfig/network-scripts``` 하위에 존재  
    network 설정 변경 후 ```systemctl restart network```
- Ubuntu : ```/etc/network/iinterfaces``` 하위에 존재   
    network 설정 변경 후 ```systemctl restart NetworkManager```

network setting을 변경 시에는
1) ```systemctl stop NetworkManager```
2) ```systemctl disable NetworkManager```
3) ```vi /etc/network/interfaces``` 로 설정파일 수정
4) ```systemctl restart networking```
5) ```systemctl enable networking```

이와 같은 순서로 설정해준다.

이외의 network 관련 파일들
- ```/etc/sysconfig/network``` : RHEL에만 존재. Gateway 설정할 때 사용
- ```/etc/hostname``` : 호스트명 설정파일
- ```/etc/hosts``` : DNS없이 hostname을 IP주소로 mapping하기 위한 파일
- ```/etc/resolv.conf``` : DNS서버와 검색 Domain이 저장되있는 파일. (NetworkManager를 반드시 끄고 편집해야 한다.)

#### network bonding, bonding mode
Bonding - NIC(Network Interface Card)를 여러 개 본딩하여(논리적으로 묶음) NIC 개수만큼 대역폭을 확장하는 기술.

- mode 0 : Round-robin  
    첫 번째 가능한 slave부터 마지막까지 순차적으로 전송. load balancing과 failover를 제공한다.

- mode 1 : Active-backup
    bond에서 하나의 slave만 활성화된다. 다른 slave는 standby 상태로 대기하다 활성중인 slave가 fail된 경우 standby slave가 활성화됨.

- mode 2 : balance-xor(load balancing + failover)  
    Round-robin과 비슷하지만 xor연산을 이용하여 목적지 Mac과 근원지 Mac을 이용하여 분배한다.

Fault tolerance (Active-backup) 모드의 경우 하나의 slave만 활성화 되고 다른 slave들은 대기상태에 있다가 활성중인 slave가 fail되면 그 때 활성화된다.   
이러한 방식은 평상시 load balancing (부하분산)을 제공하지 않으며 많은 data를 다루는 기업형 서버에는 적합하지 않다. 

---

이 외에도 network setting을 이용하면 Linux system을 
- Router 로 사용
- DHCP server로 사용
- DNS server로 사용
- proxy server로 사용   

가능하다. 

CH15(Starting and Stopping Services)
---------------------------------
```ps -ef``` : 현재 실행중인 process list 출력.

#### service 시작 / 중지: systemctl 명령어 이용.  

- ```systemctl start [service name]``` : service 시작  
- ```systemctl stop [service name]``` : service 중지  
- ```systemctl restart [service name]``` : service 재시작  
    ex) ```systemctl restart Network-Manager``` 

- ```systemctl enable [service name]``` : 관련 서비스를 /etc/systemd/system/ 하위에 link file 을 생성  
    부팅 시 service 를 자동 시작. 
- ```systemctl disable [service name]``` : 해당 link file을 삭제  
    부팅 시 service 자동 시작 해제.
- ```systemctl status [service name]``` : service 상태 확인.

#### Run Level

- **level 0 poweroff.target (halt)** : system 종료를 의미.   

- **level 1 rescue.target (Signle user mode)** : 시스템 복원모드. 기본적으로 관리자 권한을 얻음.  
주로 filesystem 점검 또는 관리자 암호 변경 시 사용.

- **level 2 multi-user.target (Multiuser mode)** : NFS를 지원하지 않는 다중 사용자 모드.

- **level 3 multi-user.target (Full multiuser mode)** : 일반적인 쉘 기반 Interface 를 가진 다중 사용자 모드. GUI를 지원하지 않는다.

- **level 5 graphical.target (X11)** : level 3 + GUI.

- **level 6 reboot.target (reboot)** : system reboot.

현재 system의 runlevel을 알고 싶다면 ```sudo systemctl get-default```입력.

![image](https://user-images.githubusercontent.com/72643027/109462686-3a3aef00-7aa7-11eb-8a81-159ab48081cd.png){: width="50%" height="50%"}

일반적으로 사용하는 mode는 level5임. runlevel을 level3으로 변경하기 위해서는   
```sudo systemctl set-default multi-user.target```와 같이 입력 후 system 재부팅.


