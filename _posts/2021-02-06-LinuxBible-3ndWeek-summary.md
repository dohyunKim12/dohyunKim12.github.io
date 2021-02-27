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


