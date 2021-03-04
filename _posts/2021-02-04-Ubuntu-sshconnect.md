---
layout: post
title: "Ubuntu - ssh로 root 접속하기"
summary: 
author: Dohyun Kim
date: 2021-02-04 11:59:28 -0400
category: Linux
thumbnail: /assets/img/posts/ubuntussh.png
comments: true
---

VMware에 Ubuntu-desktop 18.04를 설치하고,

사용하려고 보니...

내 컴퓨터의 해상도가 워낙 높은지라 VMware application자체의 디스플레이가 너무 작게 보였다. 사용하는데 불편 ㅠㅠ

![image](https://user-images.githubusercontent.com/72643027/109921378-1d98f400-7cff-11eb-97b9-90c99e435703.png){: width="100%" height="100%"}

그래서 local(우분투20.04)의 terminal로 ssh를 이용하여 접속을 시도.

VM에서

```
ip a
```

를 치면 IP주소가 나온다. 이 주소를 이용하여 ssh로 접속시도.

![image](https://user-images.githubusercontent.com/72643027/109921402-2ab5e300-7cff-11eb-8ca9-eea8c3f37176.png){: width="70%" height="70%"}

그런데, root로 접근이 실패. 다른 사용자 계정으로 접근은 가능했다. (이후 su - 로 root 권한을 얻는 것도 가능)

VM에서 root의 password를 제대로 설정해 주었고, ssh접속 시에도 password를 정확히 입력했음에도 permission denied가 뜨는 이유는 무엇일까?

바로 우분투 ssh설정에 처음부터 root로 접근하는 것을 막도록 설정되어 있었기 때문이다.

\*\* 수정방법

```
# vi /etc/ssh/sshd_config
```

![image](https://user-images.githubusercontent.com/72643027/109921502-55a03700-7cff-11eb-8142-5517754fc544.png){: width="100%" height="100%"}

/Permit을 쳐서 검색하다 보면 32번째 line쯤 설정값이 보임.

33 line에 PermitRootLogin prohibit-password 라는 설정값을 주석처리하고 

PermitRootLogin yes 로 변경한다.

이후 ssh 서비스를 재시작

```
# systemctl restart sshd
```

이후, ssh로 ```ssh root@ip주소``` 형식으로 접근하는 것이 가능해졌다.

\*참고(PermitRootLogin 설정값)

\- 관리자 계정인 root로 로그인을 허용하면 yes, 아니면 no, 기본값(prohibit-password)은 공개키 인증 방식이 아닌 ID/Passwd로 로그인 할 때만 금지한다.
