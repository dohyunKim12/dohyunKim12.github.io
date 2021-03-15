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

<br/>
<br/>
<br/>

### Tip  

```apt-cache search [package name]```  
```apt-cache dumpavail | grep [package name]```  

명령으로 설치 가능한 list에 비슷한 이름을 검색하여 확인해 보는 방법도 활용하자!!

![image](https://user-images.githubusercontent.com/72643027/111116399-4fd10e00-85a9-11eb-94c9-97363e8b3ecd.png){: width="70%" height="70%"}

![image](https://user-images.githubusercontent.com/72643027/111116498-70996380-85a9-11eb-8e8f-0fbd07d97fe4.png){: width="70%" height="70%"}
