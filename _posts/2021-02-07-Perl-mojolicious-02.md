---
layout: post
title: "Perl Mojolicious Tutorial(2)"
summary: How to use Mojolicious web framework
author: Dohyun Kim
date: 2021-02-07 21:30:00 -0400
category: DesignPattern
thumbnail: /assets/img/posts/mojoliciousThumb.png
comments: true
---


지난시간에 이어 Mojolicious web framework 사용법에 대해 알아보자.

(https://docs.mojolicious.org/Mojolicious/Guides/Growing)

Perl - Mojolicious tutorial(1)  
(https://dohyunkim12.github.io/designpattern/2021/02/25/Perl-mojoliciousTut-01/#/)


이렇게 지난시간에 간단히 username와 password를 parameter로 받아 Model에서 data를 처리(username 과 password를 비교)하여 web상에 json형태로 메세지를 띄워보는 작업을 해보았다.

오늘은 이러한 login module을 조금 더 우리가 실제 사용하는 web page답게 만들어보는 작업을 해보자.
 
![image](https://user-images.githubusercontent.com/72643027/109927456-9b60fd80-7d07-11eb-9cec-ac2cda29f110.png){: width="70%" height="70%"}

크게 할 작업을 설명하자면, Users Controller와 main app 내용을 수정하고, html template을 추가하여 login module을 만들 것이다. 

![image](https://user-images.githubusercontent.com/72643027/109927488-a74cbf80-7d07-11eb-94c9-dfb870120303.png){: width="80%" height="80%"}


또한, 개발자 옵션과 페이지 소스 보기 모드를 이용하여 HTTP, mojo의 set-cookie, html에 간략히 알아 볼 것이다.

그리고 perl에서 제공하는 prove -l 옵션을 통한 code test또한 해보자.

먼저 main app을 다음과 같이 수정한다.

```perl
package MyApp;
use Mojo::Base 'Mojolicious', -signatures;
use Mojo::Util;
use MyApp::Model::Users;

our $users;

# This method will run once at server start
sub startup ($self) {

  # Load configuration from config file
  my $config = $self->plugin('NotYAMLConfig');

  # Configure the application
  $self->secrets($config->{secrets});
    
  $self->helper(
      users => sub{
          if(!$users)
          {
              $users = MyApp::Model::Users->new();
          }
          return $users;
      }
  );
  # Router
  my $r = $self->routes;

  # Normal route to controller
  $r->any('/')->to('Users#index')->name('index'); #(컨트롤러 # 메서드)any:HTTP GET이든, POST든 어떤 method형태도 받는다.
  $r->get('/hello')->to('example#hello'); #(컨트롤러 # 메서드)

  my $auth_route = $r->under(
     '/',
     sub{
         my $c = shift;
  
        if ($c->session('user')){ #user라는게 있으면 로그인된걸로가정
            return 1;
        }
        $c->redirect_to('index'); #index template으로 redirect시킴.
 
        return undef;
    }
 );

 $auth_route->get('/protected');
 $auth_route->get('/logout')->to('Users#logout');
}

1;
```

Controller 하위의 Users.pm을 다음과 같이 수정한다. main app 코드에 대한 설명은 Users Controller와 함께 하겠다.

```perl
package MyApp::Controller::Users;

use strict;
use warnings;

use Mojo::Base 'Mojolicious::Controller';


sub index
{
    my $c = shift;

    my $user = $c->param('user') || '';
    my $pass = $c->param('pass') || '';

    #Check password
    if (!$c->users->check($user, $pass))
    {
        return $c->render();    # check 실패시 바로 render를 탐. (template아래에 index를 mapping시켜줌)
    }

    # Store username in session
    $c->session(user => $user);

    #Store a friendly message for the next page in fla
    $c->flash(message => 'Thanks for logging in.'); # 한번 깜빡.

    # Redirct to protected page with a 302 resposne
    $c->redirect_to('protected');   # prtected라는 uri로 리다이렉트. template밑의 protected로 바로 렌더링 해버림.
}
sub logout
{
    my $c = shift;

    #Expire and in turn clear session automatically.
    $c->session(expires => 1);    

    # Redirect to main page with a 302 response. (302는 page redirection을 나타냄)
    $c->redirect_to('index');
}
1;
```

<main app (MyApp.pm), Users Controller (Users.pm) 코드설명>

먼저 Users Controller로 routing하는 부분에서 welcome메소드를 index메소드로 이름을 바꾸었다. (이유는 template폴더 하위에 새롭게 작성될 index.html과 맵핑되기 위해서.)

deafult로 '/'를 전달했을 경우 어떻게 되는지 살펴보자. 먼저 Users Controller의 index메소드를 호출하게 된다. 

index 메서드에서는 기존의 welcome과 동일하게 user와 pass를 check하고(check-Model에서 담당) 만약 check를 통과하면 session에 접근해서 user값을 저장, message를 띄우고 protected라는 URI로 redirect를 하게 된다. 

이때 session이란, 지난시간에 설명한 mojolicious의 특성인데 set-cookies메소드를 말한다.

check를 무사히 통과하여 로그인 했을 경우, 로그인 된 창(/protected)과 로그아웃(/logout) 기능을 구현하기 위해 routing - auth\_route 부분을 추가하였다. 

under메소드는 perl에서 지원하는 문법인데, under부분이 가장 먼저 실행된다. 따라서 기존에 $r이 Users Controller에서 어떠한 값으로 redirecting 되었는지에 따라 작동한다. $r이 Controller에서 session값을 가져왔고, user항목이 존재하면 1로 바로 return되고, 없으면 index template으로 redirect 시킨다.

또한 Users Controller에서 protected로 redirecting 시킨 부분을 처리한다. (protected.html로 렌더링)

여기까지 진행했으면, 이제 rendering을 적절히 시켜주기 위해 protected.html과 index.html을 작성한다.

각각은 temlpate directory 하위에 작성해주어야 하며, index.html의 경우 main app에서 'Users#index' 로 호출하기 때문에 template/users 하위에 존재해야 한다.(users라는 directory를 template하위에 만들자.) (tree참고)

protected.html.ep을 다음과 같이 작성한다.

```html
% layout 'default';
% if (my $msg = flash 'message') {
  <b><%= $msg %></b><br>
% }
Welcome <%= session 'user' %>.<br>
%= link_to Logout => 'logout'
```

layout은 template/layout의 default로 설정하였고, flash 'message'가 존재하면 msg를 띄우도록 작성하였다. (flash는 session에 username이 존재하지 않을 시 Users Controller의 index메소드로 들어오게 되고, check를 통과하면 flash msg가 설정된다. 따라서 html의 2번째 line은 로그인 한 적 없는(session기록에 없는) 사용자가 최초로 로그인에 성공 시 True값을 가질 것이다.

index.html.ep를 다음과 같이 작성한다.

```html
% layout 'default';
%= form_for index => begin
  % if (param 'user') {
    <b>Wrong name or password, please try again.</b><br>
  % }
  Name:<br>
  %= text_field 'user'
  <br>Password:<br>
  %= password_field 'pass'
  <br>
  %= submit_button 'Login'
% end
```

layout은 마찬가지로 template/layout의 default로 설정, %= 부터 %까지는 inline code block.

우선 Name과 Password라는 box를 만들고 text\_field, password\_field에 각각 user, pass를 받는다. submit\_button으로 Login 버튼 또한 만들어준다.

ID와 PW로 로그인 시도, 만약 check를 통과하지 못하면 다시 index.html.ep로 렌더링 될 것이고, 그 때엔 user라는 text\_field가 존재하기 때문에 if문에 걸리게 된다. (Wrong name or password, please try again. 출력)

(check 통과시 index.html이 아닌 protected.html로 렌더링 됨.)

페이지 title을 'Login Manager'로 표시하기 위해 default.html.ep도 수정해주자.

```html
  <head><title><%= title %></title></head>
```

부분에서 title을 수정해준다.

```html
  <head><title>Login Manager</title></head>
```

이제 Login Module이 완성되었다.

Users Model에 존재하는 USERS dictionary 값과 비교하여, 입력한 ID와 PW가 일치하지 않으면 다시 index.html로 rendering됨과 동시에 'Wrong username~~' 메세지가 띄워질 것이다.

일치하면(check메서드를 통과) session에 user값이 없다면 flash msg('Thanks for logging in')가 최초 1회 띄워질 것이고,

이후에는 메세지 없이 protected.html에 따라 화면에 띄워질 것이다.

확인해 보자.

![image](https://user-images.githubusercontent.com/72643027/109927571-c4818e00-7d07-11eb-986c-c0ffc65814c0.png){: width="70%" height="70%"}


![image](https://user-images.githubusercontent.com/72643027/109927614-d5ca9a80-7d07-11eb-87f4-9f47da253437.png){: width="70%" height="70%"}


![image](https://user-images.githubusercontent.com/72643027/109927629-da8f4e80-7d07-11eb-9fc7-d5108a34018b.png){: width="70%" height="70%"}


HTTP request & response와 302 redirect가 어떻게 이루어지는지 좀 더 자세히 살펴보자.

web page - 마우스 우클릭으로 '검사'를 누르거나 F12버튼을 눌러 개발자 모드로 들어온다.

![image](https://user-images.githubusercontent.com/72643027/109927664-e549e380-7d07-11eb-9a7c-eaa9047f9592.png){: width="90%" height="90%"}


맨 상단의 Application - cookies를 살펴보면 처음 login할 때, cookies가 설정되는 것을 볼 수 있다.

logout시에는 cookies가 expire되면서 없어진다.

Network tab에 들어가보면, HTTP request와 response가 어떻게 이루어지는지 조금 더 자세히 볼 수 있다.

302-redirect가 어떻게 이루어지는지 살펴보자.

![image](https://user-images.githubusercontent.com/72643027/109927675-e9760100-7d07-11eb-8d8b-c8b2804782e8.png){: width="90%" height="90%"}


처음 ID와 PW를 정확히 입력하였을 때, 302-redirection이 발생하여 protected로 넘어간다. 이후, set-cookies를 통해 session에 값이 존재하게 되므로 302-redirection없이 바로 protect로 mapping되게 된다. 

logout 시에는 다시 index로 redirect하며 session을 지우게 된다.

### < testing >

이제 test code를 작성하여 만들어진 Login module이 주어진 test조건에서 정상 동작하는지 살펴보자.

t 폴더 하위의 basic.t 를 다음과 같이 수정한다.

```perl
use Mojo::Base -strict;

use Test::More;
use Test::Mojo;

#include application
use MyApp;

my $t = Test::Mojo->new(MyApp->new()); #test객체

# Allow 302 redirect responses  #302로 redirect하는것을 생략.
$t->ua->max_redirects(1);   #ua == UserAgent (Client의 max)redirects를 1로 지정. (redirect를 너무 많이 하지 못하게.)

#Test if the HTML login form exists
$t->get_ok('/')     #맨 root에다가 GET이 잘 들어오는지 .
    ->status_is(200)
  ->element_exists('form input[name="user"]')   #html code에서(source) form아래input이 있고 이름이 user인가?
  ->element_exists('form input[name="pass"]')
  ->element_exists('form input[type="submit"]');

#Test login with valid credentials
$t->post_ok('/' => form => {user => 'sebastian', pass => 'secr3t'}) #form이 보여지지않게 body에 실림. POST메소드의 특징. value를 채워주는 user와pass를 보냄.
  ->status_is(200)
  ->text_like('html body' => qr/Welcome sebastian/);    #text matching을 함. welcome ~~ 이 떳는지. html 아래 body 의 텍스트를 확인해서 매칭함. 

# Test accessing a protected page
$t->get_ok('/protected')
    ->status_is(200)
    ->text_like('a' => qr/Logout/);     #마찬가지로 Logout 텍스트를 비교. a는 anker tag를 뜻함. (이러한 anker tag가 있는지)

# Test if HTML login form shows up again after logout
$t->get_ok('/logout')   #마지막으로 logout시 확인.
  ->status_is(200)
  ->element_exists('form input[name="user"]')
  ->element_exists('form input[name="pass"]')
  ->element_exists('form input[type="submit"]');    #이러이러한 항목들이존재하는지.확인

done_testing();
```

test객체로 main app을 사용할 것이기 때문에 MyApp을 import하고 t변수를 MyApp->new()로 생성한다.

그리고, 302-redirect을 생각해주지 않기 위해 max\_redirects를 1로 지정한다. 이렇게 하면 302 redirect하는 과정이 보이지 않게 된다. (200 ok 에 대한 부분만 검사하게 됨) 이러한 방법은 사실 보다 정확한 검사를 위한 방법은 아니다. (이후 과정을 생각하며 302를 포함하는 test code를 작성해보도록 하자.)

이후 내용은 각각의 element들이 존재하는지 확인하고, html body를 확인하여 일치하는지에 따라 login module이 정상 작동함을 판단한다.

```
$ prove -l -v
```

로 test code 실행결과를 확인할 수 있다. (prove -l : perl에서 제공하는 test기능 명령어와 option)  
  
![image](https://user-images.githubusercontent.com/72643027/109927737-fe529480-7d07-11eb-9ddc-917c77a819ad.png){: width="70%" height="70%"}


