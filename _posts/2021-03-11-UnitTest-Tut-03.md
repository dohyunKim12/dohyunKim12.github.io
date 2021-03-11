---
layout: post
title: "GMS UnitTest tutorial(3)"
summary: 새로운 module 추가 시 방법.
author: Dohyun Kim
date: 2021-03-11 21:30:00 -0400
category: DesignPattern
thumbnail: /assets/img/posts/gitlab.png
comments: true
---

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
