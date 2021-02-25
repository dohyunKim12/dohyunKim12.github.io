---
layout: post
title: "Perl Mojolicious Tutorial(1)"
summary: How to use Mojolicious web framework
author: Dohyun Kim
date: 2021-02-24 21:30:00 -0400
category: DesignPattern
thumbnail: /assets/img/posts/mojoliciousThumb.png
comments: true
---

GMS project - unitTest code 작성을 위해서는 

MVC pattern을 기반으로 동작하는 시스템구조와 그를 구현하는 **웹프레임워크에 대한 이해**가 필요하다.

Web framework는 Python-Django, Java-Spring 등 여러가지가 있지만 GMS에서는 perl언어를 이용하여 개발, web framework도 perl의 **Mojolicious**를 사용한다.

https://docs.mojolicious.org/Mojolicious/Guides/Growing


Perl - Mojolicious는 역시 REST Api를 이용하며(REpresentational State Transfer : REST Api를 통해 요청이 수행될 때 리소스 상태에 대한 표현을 요청자에게 전송. 이 정보 표현은 HTTP: JSON, HTML, XLT 등 여러 형식으로 전달될 수 있다.)

Perl의 Mojolicious는 또한 다음과 같이 MVC(Model, View, Controller) 구조를 기반으로 설계한다.

![img](https://user-images.githubusercontent.com/72643027/108144274-ae22e200-710c-11eb-816f-ea7828e5edba.png){: width="70%" height="70%"}

**Model**: data를 처리, **Controller**: 실제 logic이 담겨있는 부분, **View**: 처리된 data를 view를 통해 보여줌.

![img (1)](https://user-images.githubusercontent.com/72643027/108144419-f8a45e80-710c-11eb-87e6-25ea581fffa3.png){: width="70%" height="70%"}


이런식으로 항상 **request와 response들은 Controller를 통해 주고받음.**

**REST**가 어떻게 동작하는지 보여주는 예.

![img (2)](https://user-images.githubusercontent.com/72643027/108144682-79635a80-710d-11eb-9f04-db9cc878cc4f.png){: width="70%" height="70%"}

![img (3)](https://user-images.githubusercontent.com/72643027/108144663-736d7980-710d-11eb-9b93-fe1bd04f8119.png){: width="70%" height="70%"}


HTTP protocol은 Stateless protocol이다. 즉 상태정보를 저장하지 않는다. 이러한 관점에서 우리는 **Session**이라는 것이 Client측이 아닌 Server측의 정보를 저장한다고 바라봐야 한다. 실제로 Web Server는 Client가 이전에 무엇을 요청했는지에 대한 정보를 저장하지 않기 때문에 이전 검색 기록에 대해 알 수도 없다. 그러나 Web Framework는 이러한 기록들을 Set-Cookie라는 메소드를 이용해 서버측에 저장한다. 

![img (4)](https://user-images.githubusercontent.com/72643027/108144661-72d4e300-710d-11eb-90a7-2814f0cf34a3.png){: width="70%" height="70%"}


이제 나의 local환경(ubuntu)에서 Mojolicious를 이용하여 간단하게 web framework의 동작을 확인해 보자.

먼저 작업할 directory에 가서 Mojolicious 모듈을 다운받는다. (이전에 cpan을 설치해야 한다.)

```
$ cpan Mojolicious
```

작업할 directory에서 **mojo generate app **명령어로 MyApp이라는 app을 생성한다.

```
$ mojo generate app MyApp
```

![img (5)](https://user-images.githubusercontent.com/72643027/108144660-723c4c80-710d-11eb-8752-ef8a631b61f7.png){: width="70%" height="70%"}


폴더 계층도를 보면, MVC의 대략적인 구조가 갖춰진 형태로 여러 폴더 및 파일들이 생성되었다. 

자. 여기서 script밑의 my\_app실행파일을 daemon으로 실행시키면 바로 web framework가 동작한다. 한번 실행시켜보자.

```
$ ./script/my_app daemon
```

![img (6)](https://user-images.githubusercontent.com/72643027/108144657-723c4c80-710d-11eb-828a-629c6a9d690d.png){: width="70%" height="70%"}


만약 포트 3000번이 사용중이어서 create할 수 없다는 문구가 나온다면, 

**$ fuser -n tcp 3000** 명령어로 tcp 포트3000을 사용하는 process를 검색하여 kill 한 후, 다시 daemon을 실행시키면 된다.

![img (7)](https://user-images.githubusercontent.com/72643027/108144655-71a3b600-710d-11eb-8873-2f461e4ef659.png){: width="70%" height="70%"}


이렇게 열리는 페이지의 Welcome to the Mojolicious real-time web framework! 문구는 어떻게 뜨는 것일까?

![img (8)](https://user-images.githubusercontent.com/72643027/108144654-710b1f80-710d-11eb-9f40-3b4cf1312ea3.png){: width="70%" height="70%"}


위의 그림을 다시 보자. 모든 Request와 Response는 Controller를 통해 전송, 수신된다. 이 점을 기억하자.

그리고 **폴더의 계층구조**(tree 명령어로 찍은 계층도)를 기억하고 **계속적으로 확인**하며 살펴보자.

최종적으로 모든 기능을 구현했을 때 예상되는 폴더 및 파일구조는 이러하다.

![img (9)](https://user-images.githubusercontent.com/72643027/108144653-70728900-710d-11eb-877e-02c687313353.png){: width="70%" height="70%"}


우선 daemon으로 script폴더의 my\_app을 실행시켰으므로 my\_app 스크립트가 어떻게 구성되어있는지 살펴보자.

![img (10)](https://user-images.githubusercontent.com/72643027/108144651-70728900-710d-11eb-8f90-13a5da55bb07.png){: width="70%" height="70%"}


 my\_app 스크립트의 start\_app 이 가리키고 있는 'MyApp.pm'에 들어가 보자.

![img (11)](https://user-images.githubusercontent.com/72643027/108144647-6fd9f280-710d-11eb-9f52-ba9f1bf40c53.png){: width="70%" height="70%"}


이와 같이 작성되어 있다. 여기서 주의해서 볼 부분은 startup 메소드에서 router ($r)을 선언하고 get메소드로 '/'를 받았을 때 어떻게 처리하는지를 명시하고 있는 17번째 line이다. 보면 'example의 welcome으로 전달하라' 라는 내용을 나타내고 있다.

'example#welcome' 은 'example' 이라는 Controller 에 가서 'welcome' 메소드를 호출하라 는 의미이다.

그럼 **Controller의 Example.pm**으로 가보자.

![img (12)](https://user-images.githubusercontent.com/72643027/108144645-6fd9f280-710d-11eb-8571-623e289d0595.png){: width="70%" height="70%"}


welcome 메소드에서 드디어 rendering하는 부분을 찾았다. msg로 'Welcome to the Mojolicious~~' 를 보내고 있다. 

이것이 IP:port로 접속했을 때 나타나는 이유이다. 

자 그러면 우리가 HTTP GET으로 '/'만 보내는 상황을 확인했으니, 다른 parameter를 전달할 때 어떻게 반응하는지 살펴보고 적절한 response를 받을 수 있도록 설정해 보자.

start\_app 이 MyApp.pm을 가리키고 있으니 다시 **MyApp.pm**에 접근하여 수정한다.

```perl
$r->get('/hello')->to('example#hello')
```

이렇게 controller로 route하는 부분 밑에 '/hello'를 parameter로 전달받았을 경우에 대해 작성해보자.

example 컨트롤러의 hello메서드를 호출하도록 설정하였다.

그럼 다시 Example.pm에 가서 hello 메서드를 추가해줘야겠다.

![img (13)](https://user-images.githubusercontent.com/72643027/108144644-6f415c00-710d-11eb-8a9d-07b9ae503458.png){: width="70%" height="70%"}


이렇게 hello메소드를 정의하면 IP:port/hello 로 접근하였을 때 해당 controller의 메소드가 호출되어 Hello! 메세지가 html로 띄워질 것이다. 확인해 보자.

![img (14)](https://user-images.githubusercontent.com/72643027/108144637-6ea8c580-710d-11eb-9c63-571bf9d92ce1.png){: width="70%" height="70%"}


이러한 html메세지가 띄워지는 과정에서도 한가지 확인할 것이 있다. 

example의 welcome메소드에서 msg를 정의하는 부분이 있었는데, 이것이 어떻게 html로 변환되어 표현되었을지 궁금하다. 살펴보자.

my\_app/template/example/ 폴더에 정의되어 있다. 하위의 welcome.html.ep를 살펴보자.

![img (15)](https://user-images.githubusercontent.com/72643027/108144634-6e102f00-710d-11eb-84e1-f61e783b7220.png){: width="70%" height="70%"}


3번째 line에서 msg를 어떻게 치환하는지 명시되어 있다. <%= ~~~~~~ %> 부분을 보자. 이러한 문법은 html자체에 있는 게 아니라 template언어이다. (mojo 에서 치환을 함.)

그리고 밑에 4 line부터"This page was generated from~~~" 메세지를 띄우게 되어 있다.

my\_app/template/layouts/ 하위의 default.html.ep 라는 파일을 살펴보자.

![img (16)](https://user-images.githubusercontent.com/72643027/108144633-6e102f00-710d-11eb-85b8-796b4d947715.png){: width="70%" height="70%"}


여기까지 되었으면 다음 단계로 넘어가자.

MVC 에서 data를 처리하는 Model에 해당하는 부분을 작성해 보자. HTTP GET으로 '/'을 넘겨줄 때, User controller의 welcome 메서드에서 받도록 해 볼 것이다. 따라서 my\_app/lib/MyApp.pm 을 다음과 같이 수정한다.

![img (17)](https://user-images.githubusercontent.com/72643027/108144631-6d779880-710d-11eb-8505-62349d0471d6.png){: width="70%" height="70%"}


그리고 Controller 디렉토리의 하위 파일로 Users.pm을 생성, 작성한다.

```perl
package MyApp::Controller::Users;

use strict;
use warnings;

use Mojo::Base 'Mojolicious::Controller';


sub welcome
{
    my $c = shift;

    my $user = $c->param('user') || '';
    my $pass = $c->param('pass') || '';

    if ($c->users->check($user, $pass))
    {
        return $c->render(json => {msg => 'Welcome $user.'});
    }

    $c->render(json => {msg =>'Wrong username or password.'});
}
1;
```

Users - Controller를 이처럼 작성하였고, Users - Model 부분을 작성해야 한다.

lib/MyApp하위에 Model directory를 생성, Model하위에 Users.pm을 생성하고 check 메소드를 정의하여 data를 처리하는 부분을 구현할 것이다.

```perl
package MyApp::Model::Users;

use strict; #엄격한 coding rule 적용
use warnings;   #문제가 도힐 법한 code들을 알려줌.

my $USERS = {
    joel => 'las3rs',
    marcus => 'lulz',
    sebastian => 'secr3t',
};
sub new {   #Moose, Mouse이용하지 않음. 원래 perl문법대로 생성.
    my $class = shift;
    return bless({}, $class);
}
sub check{
    my $self = shift;
    my $user = shift;
    my $pass = shift;
    # Success
    return 1 if $USERS->{$user} && $USERS->{$user} eq $pass;
    # Fail
    return undef;
}
```

그리고 이렇게 작성된 Users Model을 main app에서 사용한다고 명시해줘야 한다. main app 파일을 수정하자.

![img (18)](https://user-images.githubusercontent.com/72643027/108144627-6cdf0200-710d-11eb-8033-5a2821bc20d6.png){: width="70%" height="70%"}

![img (19)](https://user-images.githubusercontent.com/72643027/108144626-6c466b80-710d-11eb-9b97-7ba18bef25a5.png){: width="70%" height="70%"}


페이지에서 "msg" 부분이 나타나게 되는 과정을 살펴보면, (tree 참고)

먼저 우리는 script 하위의 my\_app을 daemon으로 실행시키면서 lib밑의 MyApp.pm(main app)을 실행한다.

그런데 우리는 main app에서 HTTP GET으로 '/' (default)를 받으면 Users컨트롤러의 Welcome메소드로 넘겨주도록 설정하였다. 때문에 Users컨트롤러의 welcome메소드를 살펴봐야 한다.

![img (20)](https://user-images.githubusercontent.com/72643027/108144625-6badd500-710d-11eb-8b43-677c8f0e7e8e.png){: width="70%" height="70%"}


users는 main app에서 처음에 Users Model(class) 값으로 초기화한 전역변수이다.<br/> data를 처리하는 Model에서 check method를 통해 user와 password값이 일치하는지 확인하고, 일치하면 1, 그렇지 않으면 undef값을 return하므로
Users Controller의 welcome에서는 if문을 통해 일치할 경우 "Welcome $user"메세지를 json형태로 띄우고 return한다.

그렇지 않으면 "wrong username or password"문구를 json형태로 띄운다.<br/>
(이 tutorial에서 rendering은 곧 "메세지나 특정 객체를 web상테 띄운다"는 의미로 생각하면 된다. )

그럼 parameter로 user와 pass를 다음과 같이 보내주면 어떻게될까? 

(parameter는 welcome메서드에서 $c->param('user') ,   $c->param('pass') 로 각각 받고 있다.)

![img (21)](https://user-images.githubusercontent.com/72643027/108144621-6b153e80-710d-11eb-8882-450fd26fbbdd.png){: width="70%" height="70%"}


이렇게 간단하게 MVC 형태로 web framework를 작성하고, 전달하는 parameter에 따라 어떻게 web상으로 구현되는지 확인해 보았다.

다음 시간에는 조금 더 login module답게 꾸며보고 개발자 옵션과 소스 코드를 살펴보며 html을 간단히 수정해보는 작업을 할 것이다.
