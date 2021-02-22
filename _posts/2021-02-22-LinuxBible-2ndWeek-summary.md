---
layout: post
title: "LinuxBible 2nd week Summary"
summary: CH07~CH11
author: Dohyun Kim
date: 2021-02-22 11:59:28 -0400
categories: Linux
thumbnail: ![image](https://user-images.githubusercontent.com/72643027/108458588-54f7b180-72b8-11eb-91ac-0409e2ea22de.png)
---

# LinuxBible Summary(CH07~CH11)

CH07(Writing Simple Shell Scripts)
---------------------------------
Shell script? : 쉘 명령어를 모아 script파일로 작성한 것.   
=> 같은 명령어를 다른 환경에서 반복해서 입력하지 않아도 됨.
ex) 고객의 서버에 회사의 sw제품 설치하는 과정을 scirpt파일로 만들면 편하다. 

- **#!** : shebang(쉬뱅)이라고 함. script파일을 실행할 program위치 명시.
- **변수** : 사용시 대입연산자(=) 앞, 뒤에 공백(space)가 없어야 한다.  
    변수에는 문자열, 숫자값 등이 저장될 수 있다.<br/>
    변수에 명령의 출력이 포함될 수 있다. (ex - MyDate=$(date), MyDate=\`date\`, echo ${MyDate})
- **read** : 사용자에게 입력메세지를 출력하며 값을 입력받아 변수에 저장한다. (python의 input() 함수와 동일.)   
(zsh 쉘에서는 오류.. #!/usr/bin/bash 로 실행하니 정상작동함.)   
    ```bash
    read -p "Enter your Kor, Eng, Math score:  " kor eng mat  # 쉼표 아닌 띄어쓰기로 구분
    echo "Your Korean score is $kor, English score is $eng, Math score is $mat "
    ```
- **산술 계산** : shell script에서는 변수의 자료형을 정의하지 않고 사용. 
(변수를 문자열이나 text로 처리하지만 산술 시 정수로 취급하여 계산됨.)
    ```bash
    NUM='1024'
    let RESULT=$NUM/16
    # == RESULT=$NUM/16
    # == RESULT=`expr $NUM / 16`
    # == RESULT=`echo "$NUM / 16" | bc`   (bc는 calculator app)
    ``` 
- **조건문** : 조건문(if, else) 역시 비슷한 방법으로 사용할 수 있다.   
문법적인 부분을 예제로 설명.
    ```bash
    VAR=1
    if [ $VAR -eq 0 ]  
    then
        echo "The varibale is 1"
    elif [ $VAR -eq 2 ]
    then
        echo "The variable is 2"
    else
        echo "The variable is ??"
    fi
    ```
    여기서 if문 사용시 [ ] 사이에는 여백이 있어야 한다. 
    그리고 { } 이런 curly braces대신 then과 fi(finish)를 사용한다. (else에는 then을 안붙여도 됨.)   
    file과 함께 사용할 수 있는 operator들이 있다. (-d _file_) : directory인지 검사. (-f _file_) : 파일인지 검사 등.   
    **equal 연산자** :  variable 값을 비교할 때 == 기호를 쓰지 않고 = 기호를 쓴다.   
    대입 시에는 띄어쓰기를 하지 않으므로 대입연산자와 구별 할 수 있다. (-eq, -ge, -gt, -lt 등도 지원.)   
    **||** : directory 존재하지 않을 시, directory를 생성하는 데 쓰임.
    ```bash
    dirname="/tmp/testdir"
    [ -d "$dirname" ] || mkdir "$dirname"
    ```
    **case**명령어도 동일하게 사용가능. endcase 대신 case를 거꾸로 한 esac를 써 줘야 한다.   
    형태 (여기서 VAR는 꼭 variable 일 필요가 없음.)
    ```bash
    case "VAR" in
        RESULT1)
            { body };;
        RESULT2)
            { body };;
        *)
            { body };;
    esac

    ```
- **반복문** : 반복문(for, while) 역시 지원한다.   
    **for**문 형태
    ```bash
    for VAR in LIST
    do
        { body }
    done
    ```
    **while**문 형태
    ```bash
    while condition
    do
        { body }
    done
    ``` 
    <br/>
이외에도 여러가지 명령어를 조합하여 효율적인 동작이 가능하다. 
```
$ cat $HOME/boottime.txt | sed 's/Server/Subal/g' > $HOME/fixed_file.txt
```
이 code는 home경로 밑에 boottime.txt 파일에서 Server라는 문자열을 Subal이라는 문자열로 치환하여 fixed_file.txt에 저장한다.
(fixed_file은 새로 생성.)

예제 - script 파일로 짠 간단 전화번호부
```bash
#!/usr/bin/bash

PHONELIST=~/.phonelist.txt

if [ $# -lt 1 ]     # parameter가 하나보다 적으면(0개이면)
then echo "Whose phone number do you want?"
    exit 1
fi

if [ $1 = "new" ]   # 첫 번째 parameter로 new를 받았다면
then shift          # shift로 new를 먹음.(씹음)
    echo $* >> $PHONELIST   # .phonelist.txt에 파라미터전체를 저장.(shift로 씹은 new 제외)
    echo $* added to database.  # console창에 echo로 메세지 출력.
    exit 0
fi

# 여기까지도 종료가 안되고 실행중이면, 다음단계로 이동.(전화번호를 찾는 작업)

if [ ! -s $PHONELIST ]  # PHONELIST에 아무 문자열도 존재하지 않으면.
then echo "no names in the phone list yet!!"
    exit 1
else
    grep -i -q "$*" $PHONELIST  # -q옵션으로 조용히 찾음(콘솔화면에 출력없이)
    if [ $? -ne 0 ]  # grep 실행한 결과가 $?에 저장되었는데 이게 0(정상종료) 가 아니면.
    then echo "Sorry, $* was not found in the phone list"
        exit 1
    else
        grep -i "$*" $PHONELIST
    fi
fi

exit 0
```

CH08(Learning System Administration)
---------------------------------
chapter8 에서는 관리자로서 시스템을 사용하는 방법을 배운다.  
root 권한이 필요할 때는 언제인가?
- Filesystem 관리 시
- Software 설치 시 
- Graphical windows tool 관리 시
- User accounts 생성 시
- Network interfaces 관리 시(현재는 일반 사용자들에게도 허용하는 경우 있음.)
- Servers (웹 서버, 파일 서버, dns 서버, 메일 서버 등) 확인 시
- Firewall 또는 user access list 등 설정 시 

#### Graphical Admin tools 사용 
**system-config** file : GUI tool은 system-config 로 시작하는 파일들로 구성. 이를 통해 system을 관리한다.
```
$ sudo find / -name 'system-config-*'
```
![image](https://user-images.githubusercontent.com/72643027/108335286-1d382d80-7216-11eb-86fc-5b281fd58365.png)

root 계정을 포함한 사용자 계정 설정들을 보고 싶으면 /etc/passwd 파일을 살펴보면 된다.  
**su 와 su -** : su는 substitute user의 줄임말로 현 사용자를 로그아웃 하지 않고, 다른 사용자의 권한을 획득할 때 사용.   
```bash
$ su [변경하고자 하는 사용자 ID]
```
이렇게 사용하고 뒤에 ID를 입력하지 않으면 su root 와 동일하게 동작한다.   
su - 는 su -l(--login) 과 동일한 명령이다. 즉 사용자 계정이 하나인 환경에서는 su --login root 와 동일. 
su와 su - 에서 - 의 유무는 환경변수와 워킹디렉토리에 영향을 줌. 
계정이 1개인 환경에서 su와 su - 모두 root권한을 얻게되지만 su는 pwd를 현재(su 하기 전)로 유지하고 su - 는 pwd를 /root로 변경한다.

**관리자가 접근할 만한 구성 파일**   
- sbin/ : 시스템 부팅에 필요한 command 존재.   
- /usr/sbin : 사용자 계정관리에 관한 command들이 존재(ex-useradd)
- /etc : 대부분의 기본적인 Linux System 구성파일들이 존재.
- /etc/cron\* : 시간에 대한 동작을 정의한 file을 담은 dir.
- /etc/cups : CommonUnixPrintingSystems (printing systems를 구성하는 dir)
- /etc/default : 다양한 utility에 대한 default값들이 존재.(grub, docker, ssh, slack, nfs ...)
- /etc/httpd : 아파치 웹 서버에 대한 동작방식을 구성하는 file들 존재.
- /etc/init.d : 서비스 daemon이 동작하는 directory.
- /etc/security : computer에 대한 default 보안 setting을 구성하는 파일들이 존재.
- /etc/sysconfig : (우분투에는 없음) 주로 network setting 등이 존재.
- /etc/systemd : 부팅과정과 시스템 서비스 관리 등의 파일들 포함.

**journalctl** : systemd의 journal을 보기 위한 명령어. 부팅 시 kernel과 모든 systemd가 관리하는 서비스들은 그들의 상태와 에러 메세지를 journal에 기록한다.

CH10(Getting and Managing Software)
---------------------------------
SW를 다운받는 데에는 패키지 형식이 이용된다. 기존의 tarball 방식에서 DEB, RPM 과 같은 복잡한 패키지 방식으로 변경됨.   
DEB(.deb) 은 데비안 계열 패키지. APT(Advanced Packaging Tool)과 함께 사용된다.   
데비안 계열 리눅스란 GNU Debian project에서 만들어 배포하는 공개 운영체제. (ex 우분투)   
package 설치 및 업그레이드 시 apt 툴을 사용하면 의존성 문제나 여러 설정들을 자동으로 해줌.

RPM(.rpm) 은 레드햇 계열 패키지. RPM(Redhat Package Manager)과 같이 사용된다.   
레드햇 배포판에는 대표적으로 Fedora, CentOS가 있고 기업 환경에서 선호한다.   
YUM으로 RPM패키지를 관리하면 RPM패키지의 의존성 문제를 해결하는 등 여러 점에서 유리하다.

#### YUM 명령으로 SW 관리하기
- Package 검색
    ```
    # yum search editor
    ```
    'editor' 가 들어간 패키지명을 전부 출력.   

- Package 정보 출력
    ```
    # yum info python
    ```
    'python' 패키지에 대한 정보 출력.(Version, Release, Size ...)
    
- Package 확인   
    **list** : 패키지 목록 확인  
    options 
    + **available** : 설치된 패키지의 리스트 확인   
    + **installed** : 설치된 패키지 전체 목록 보기  
    + **all** : 설치가 가능한 모든 패키지 목록 출력
    ```
    # yum list installed py*      # py로 시작하는 설치된 패키지 전부 출력.
    ```
    ![image](https://user-images.githubusercontent.com/72643027/108359156-f9371500-7232-11eb-9aba-0834bff7cb9a.png)

- Package 설치
    ```
    # yum install zsh
    ```
    zsh(지셸) 설치

- Package 삭제
    ```
    # yum erase zsh
    ```
    zsh(지셸) 삭제

- Package history 관리  
    **history** : Yum package에 대한 최소 설치, update내역을 나타냄.  
    options
    + **info** : History-number에 해당하는 명령의 상세 내역을 확인.
    + **undo** : 함께 입력한 history number에 해당하는 작업을 되돌림.

- Package update   
    ```
    # yum update zsh
    ```
    뒤에 zsh 처럼 패키지명을 입력하지 않으면 전체 업그레이드를 진행.

CH11(Managing User Accounts)
---------------------------------
chapter11 에서는 사용자 계정 관리에 대해 알아본다.   
#### Common Env. User Management
**useradd** : 계정만 생성하며 기타 계정정보를 수동으로 생성.   
**adduser** : 실행 시 기본 계정 정보를 자동으로 생성.   
```
# useradd [option] [계졍명]
# adduser [option] [계정명]
# options
-c [Comment] // /etc/passwd에 사용자 설명 추가.
-d [Home] // home directory 위치 지정.
-e [Expiredate] // 지정된 날짜에 사용자 계정 삭제.
-f [Inactive] //  password 만기 후 계정 영구 삭제 기간.
-u [User ID] // 지정된 값으로 UID 설정.
-s [Shell] // 사용자의 login shell 지정.
-G [Groups] // 생성하는 계정을 그룹에 추가.
-M [No Create HomeDIR] // 홈 디렉토리 생성하지 않음.
```

/etc/passwd 파일의 속성값들   
```
# cat /etc/passwd | grep 'dohyunkim'
dohyunkim:x:1000:1000:dohyunkim,,,:/home/dohyunkim:/usr/bin/zsh

(User명:password(encrypted):UID:GID(primary):comment:HomeDIR:LoginShell)
```
/etc/shadow 파일 안에는 사용자 인증에 필요한 암호 정보가 존재한다.   
(password 변경 일, 로그인 차단 일 수, 로그인 사용을 금지하는 일 수 등)   

계정 생성 이후에도 **usermod** 명령어로 사용자 계정정보를 수정(특정 그룹에 추가하는 등), 
**userdel** 명령어로 사용자 계정정보를 삭제 등의 처리를 할 수 있다.

#### Managing Users in the Enterprise

##### More flexible - ACL(AccessControlList)
기존의 linux filesystem에서는 각 file은 오직 하나의 user, 하나의 group에만 할당될 수 있었음.
현재는 ACL을 이용하여 특정 file을 특정 user에 대해 접근권한을 줄 수 있다.

**ACL Feature**
- Fedora, Redhat Linux에서 ACL은 기본적으로 제공됨.
- ACL에 file추가: ```setfacl```
- 특정 file에 대해 ACL설정 확인: ```getfacl```
- file을 ACL에 set 하기 위해서는 그 file의 실제 소유자여야 함.   

```setfacl -m u:user:권한 file명``` 으로 ACL 설정.   
```getfacl file명```으로 ACL 설정값 출력.   
```setfacl -x u:user```으로 user의 ACL권한 해제.   

**Enabling ACL**   
기본적으로 ACL이 제공되나, OS 설치 이후 추가된 disk 등에 대해서는 ACL이 불가능 할 수 있음. 다음 세 가지 방법으로 해결.   
1. /etc/fstab 파일의 5번째 field에 acl 옵션을 추가. -> 부팅 시 파일 시스템을 자동으로 mount 하게됨.   
```$ tutne2fs -o acl /dev/sdc1``` (맨 마지막 parameter는 새로 추가된 filesystem 위치)
2. /etc/fstab 파일 수정. etx4 뒤에 acl을 삽입해준다. -> acl이 자동, 혹은 수동으로 mount 되는 것에 관계없이 작동.
![image](https://user-images.githubusercontent.com/72643027/108447475-43f07580-72a3-11eb-8d98-06b38f7f5732.png)
3. filesystem을 mount 할 때 수동으로 mount 명령어에 acl옵션 추가. -> 수동으로 mount    
```$ mount -o acl /dev/sdc1 /var/stuff```

**Team 협업을 위한 directory 추가**   
```$ chmod xxxx filename```으로 퍼미션을 변경.  
이 때 최상위 bit가 4이면 UID 설정, **2이면 GID 설정**, 1이면 Sticky bit 설정, 0이면 그냥 평소에 쓰던 형식.   
예제를 통해 어떠한 방식으로 동작하는 지 알아보자.   
```
# groupadd -g 301 sales     # GID=301로 sales라는 group 추가.
# usermod -aG sales sara    # sara라는 user를 sales group에 추가.
# mkdir /mnt/salestools     # /mnt하위에 salestools라는 dir 생성.(추후 이 폴더가 sales팀의 공유 폴더가 될 것임.)
# chgrp sales /mnt/salestools   # salestools directory 소유 group을 sales로 변경.
# chmod 2775 /mnt/salestoos     # salestools directory의 GID를 설정(최상위 bit 2) (group-sales로 설정됨.)

# mkdir /mnt/salesFoos      # 비교할 directory도 생성.
# chgrp sales /mnt/salesFoos
# chmod 775 /mnt/ salesFoos # salesFoos dir는 chmod로 그냥 775를 줬음.
```
```
$ su - sara     # sara 계정으로 로그인
$ cd /mnt/salestools 
$ touch s ss sss

$ cd /mnt/salesFoos
$ touch s
```
![image](https://user-images.githubusercontent.com/72643027/108451081-8ae16980-72a9-11eb-94b5-b4928af6285e.png)
/mnt 하위의 salestools는 group이 sales로 지정되었음. (chmod 2775)   
drwxrw**s**r-x: salestools 하위 생성되는 file들은 GID가 sales로 설정되면서 sales 그룹에 속해있는 누구나 rw 가능해짐.   

반면, salesFoos 는 GID설정이 없었으므로, sara가 하위에 파일을 생성하더라도 그 파일의 group은 sara로 생성됨. (팀이 파일을 공유할 수 없다.)

#### LDAP(LightweightDirectoryAccessProtocol)
LDAP는 현재 Linux에서 Directory service 를 위한 표준이다.   
- 사용자, system, network, service, app 등의 정보를 조회 또는 관리
- 회사에서 구성원의 조직도나 팀별 email주소 등을 LDAP service로 관리
- 흔하게는 특정 영역에서 이용자명과 passwd를 확인하여 인증하는 용도로 사용

![image](https://user-images.githubusercontent.com/72643027/108458588-54f7b180-72b8-11eb-91ac-0409e2ea22de.png)

이와 같은 상황에서 login 을 시도하려는 host는 중앙집중화된 인증 서버를 통해 user account의 인증을 받는다.   
이처럼 인증하는 과정에 LDAP이라는 protocol이 이용된다.
