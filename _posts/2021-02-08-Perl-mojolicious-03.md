---
layout: post
title: "Perl Mojolicious Tutorial(3)"
summary: How to use Mojolicious web framework
author: Dohyun Kim
date: 2021-02-08 21:30:00 -0400
category: DesignPattern
thumbnail: /assets/img/posts/mojoliciousThumb.png
comments: true
---

(https://dohyunkim12.github.io/designpattern/2021/03/04/UnitTest-Tut-02/#/)

(https://docs.mojolicious.org/Mojolicious/Guides/Growing)

Perl - Mojolicious web framework를 이용하여 login module을 만들어 보았다.

이번 Posting에서는 살짝 불완전했던 test code를 수정하고 다시 testing 해 볼 것이다.

t/basic.t 에 test code를 작성하고 testing 해 보았는데, 한가지 살짝 찜찜했던 점은 302-redirect를 고려하지 않고 testing 했다는 점이다.

그래서 내용을 살짝 수정해 302-redirect 또한 잘 이루어지는지 살펴보았다.

t/basic.t 를 다음과 같이 수정했다.

```perl
use Mojo::Base -strict;

use Test::More;
use Test::Mojo;

#include application
use MyApp;

my $t = Test::Mojo->new(MyApp->new()); #test객체

# Allow 302 redirect responses  #302로 redirect하는것을 생략.
#$t->ua->max_redirects(1);   #ua == UserAgent (Client의 max)redirects를 1로 지정. (redirect를 너무 많이 하지 못하게.)

#Test if the HTML login form exists
$t->get_ok('/')     #맨 root에다가 GET이 잘 들어오는지 .
    ->status_is(200)
  ->element_exists('form input[name="user"]')   #html code에서(source) form아래input이 있고 이름이 user인가?
  ->element_exists('form input[name="pass"]')
  ->element_exists('form input[type="submit"]');

#Test login with valid credentials
$t->post_ok('/' => form => {user => 'sebastian', pass => 'secr3t'}) #form이 보여지지않게 body에 실림. POST메소드의 특징. value를 채워주는 user와pass를 보냄.
  ->status_is(302);
$t->get_ok('/protected')
  ->status_is(200)
  ->text_like('html body' => qr/Welcome sebastian/)    #text matching을 함. welcome ~~ 이 떳는지. html 아래 body 의 텍스트를 확인해서 매칭함. 
  ->text_like('a' => qr/Logout/);     #마찬가지로 Logout 텍스트를 비교. a는 anker tag를 뜻함. (이러한 anker tag가 있는지)

# Test if HTML login form shows up again after logout
$t->get_ok('/logout')   #마지막으로 logout시 확인.
  ->status_is(302);
$t->get_ok('/')
  ->status_is(200)
  ->element_exists('form input[name="user"]')
  ->element_exists('form input[name="pass"]')
  ->element_exists('form input[type="submit"]');    #이러이러한 항목들이존재하는지.확인

done_testing();
```

가장 중요한 max\_redirects(1) 부분을 주석처리하고, 나머지 내용을 수정하였다. (이렇게 하면 302-redirect 에 대한 부분을 고려해야 함.)

기존 내용과 simulation 상황은 동일하되, 302-redirection에 대한 부분만 추가해주었다.

sim 내용은, IP:port(3000번) 으로 접속한 뒤, 적절한 ID/PW를 적어넣고 login, 이후 logout 링크를 클릭하는 것이다.

이 작업을 했을 때, HTTP request, response가 적절히 이루어지는지에 대한 test code를 작성한 것이다.

![image](https://user-images.githubusercontent.com/72643027/109929275-a157de00-7d09-11eb-9cbb-15edbf712644.png){: width="90%" height="90%"}

단계를 수행하면서 HTTP network의 이동은 이와 같다. (처음 페이지가 열릴 때 deafult로 '/'를 HTTP GET으로 요청했을 때의 network상태는 나타나있지 않음)

ID/PW를 적어넣고 login 버튼을 클릭하면 302-redirection이 발생한다. 그에 대해 status\_is(302) 로 확인해주었다. 그리고 바로 redirect - protected 로 넘어가게 되며 200ok를 반환한다.(HTTP response) 이 값에 대해서도 test code에서 확인한다.

이후 logout 버튼을 클릭하게 되면 또다시 302-redirection이 발생하고, index로 렌더링되며 session을 지웠기 때문에 최초 로그인 페이지로 넘어가게 된다. 이러한 과정들을 test code에서 확인한다.

![image](https://user-images.githubusercontent.com/72643027/109929409-c1879d00-7d09-11eb-9c9e-68b825366a4a.png){: width="90%" height="90%"}


```
$ prove -l -v
```

로 test code를 실행해보았을 때, 모든 경우에 대해 통과하였다.
