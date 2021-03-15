---
layout: post
title: "GMS UnitTest tutorial(4)"
summary: 새로운 module 추가 시 에러 대처.
author: Dohyun Kim
date: 2021-03-15 21:30:00 -0400
category: DesignPattern
thumbnail: /assets/img/posts/gitlab.png
comments: true
---

cpan 모듈 추가 시 설치 에러에 대한 대처 방법.

```Net::Interface``` module 설치 시 겪었던 오류와 해결방법을 정리한 글이다.

1. 먼저 meta cpan 사이트에 해당 모듈 검색.
2. 모듈이 존재한다면 ```cpanm::상위모듈::하위모듈```로 해당 모듈 설치
3. 모듈 존재하지 않는다면, GMS 안에서 사용하는 내부 module일 가능성. -> 해당 module을 사용하는 함수를 mock하고 다시 시도해보자.

만약, ```cpanm::상위모듈::하위모듈``` 명령어로 module 설치를 시도했으나, 

![image](https://user-images.githubusercontent.com/72643027/111115113-8148da00-85a7-11eb-9224-e483f4915edd.png){: width="70%" height="70%"}

이런식의 에러가 발생한다면, 다른 방법으로 설치를 시도해보자.
<br/>
<br/>

결론적으로 해결방법은 ```sudo apt-get install libnet-interface-perl``` 이었음.

최신버전 Ubuntu에서 lib이 잘 맞지 않아 발생하는 문제.

https://zoomadmin.com/HowToInstall/UbuntuPackage/libnet-interface-perl  
따라서 다음 규칙이 맞다면, 문제가 발생하는 다른 module들의 설치도 ```sudo apt-get install lib상위모듈-하위모듈-perl``` 로 설치가 가능하다.

* 참고 : CentOS나 Fedora에서는 perl module 설치 시 기본 명명 규칙이 ```perl-상위모듈-하위모듈``` 임에 주의!

unit-test code를 작성하다 필요에 의해 ```String::Random``` 모듈을 추가하게 되었다.  

나의 local에서는 ```cpanm String::Random```으로 간편히 추가하고 test를 진행할 수 있었으나,  
remote repository로 push 후 Gitlab의 CI/CD를 확인 해 보니 failure가 나왔다.

이유는 당연하게도 내 local에는 module이 설치되었으나, test하는 server에서는 해당 모듈이 설치되지 않았으므로 경로를 찾을 수 없다는 message가 출력된다.

![dependency-1](https://user-images.githubusercontent.com/72643027/110744755-8ba06680-827d-11eb-88d2-338842eebfc8.png){: width="90%" height="90%"} 

**dependency**를 관리하는 file을 수정해야 한다.

해결 방법.

1. ```GMS/.Jenkinsfile```에 해당 모듈 작성. (cpan이 아닌 perl로 설치해야 하기 때문에 ```perl-String-Random``` 이런식으로 적어준다.)
2. ```GMS/.gitlab-ci.yml```파일에 해당 모듈 작성. (마찬가지로 ```perl-String-Random``` 이런식으로 추가해줌.)
3. 최종적으로 권연구원님께 말씀드려 해당 모듈을 server에 설치할 수 있도록 한다.

![image](https://user-images.githubusercontent.com/72643027/110745103-20a35f80-827e-11eb-8598-9b7ccb20e6bd.png){: width="90%" height="90%"}

이후 정상적으로 CI/CD pipeline 통과.

* 참고사항 : 보통 CentOS-Base 에 기본적인 건 들어가 있음.  
```.gitlab-ci.yml``` 파일 상단에 before_script 부분에 
```
[base]
name=CentOS-$releasever - Base
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
baseurl=http://mirror.kakao.com/centos/$releasever/os/$basearch/
gpgcheck=0
enabled=0
```
를 추가하면 CI 돌아가는 건 확인가능!  

다만 해당 rpm이 들어가는게 fix되었는데,  
**abs.gluesys.com**에 없으면 직접 추가해야되서, **merge 된 이후 해당 list를 공유해야 한다!!**
