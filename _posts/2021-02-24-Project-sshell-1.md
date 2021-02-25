---
layout: post
title: "Project sshell(1)"
summary: architecture, layout, REST API
author: Dohyun Kim
date: 2021-02-24 21:09:00 -0400
categories: DesignPattern
thumbnail: /assets/img/posts/sshell.png
comments: true
---


# sshell(숭쉘) 
made by _Team sshell_   
https://github.com/gluesys/sshell   

대략적인 architecture, layout 짜고 functional API 까지 작성된 상태.

이번 posting에서는 plack 기반으로 짜여진 기존 앱(showmetheshell)을 분석하고
우리가 아는 perl의 Mojolicious형태로 바꿔보는 작업을 할 것이다. 
사실상 인후씨가 기존 앱 구조를 파악하고 나에게 가르쳐준 내용을 바탕으로 
복습 차원에서 써보는 글이다. **(출처 - InhooLee's Brain)**

먼저 기존 앱의 구조부터 분석을 해 보자.   
기존 앱은 전체적인 앱 구조를 구성하는데 Perl의 'Plack' library를 이용하였다. Front-end의 library로는 jquery를 이용하였다. 그리고 socket으로 socket.io를 이용하였다.

기존 앱에서는 MVC의 전반적인 부분을 showmetheshell.psgi 파일에서 모두 처리하고 있다. 파일을 살펴보면 PocketIO를 사용하여 Handler에게 인스턴스 값을 던지고 있다.

![image](https://user-images.githubusercontent.com/72643027/108490676-58a22d00-72e6-11eb-952f-4aba5cecbf18.png){: width="70%" height="70%"}

정적 file들을 관리하는 htdocs 하위의 'WebSocketMain.swf', 'WebSocketMainInsecure.swf', 'socket.io.min.js' file들이 바로 socket.io를 불러와서 사용하는 부분이다.  

우리는 이번 project(sshell)에서 Plack이 아닌 Perl-Mojoicious를 이용하여 앱을 짤 것이며, 이러한 환경에서 socket.io는 잘 지원하지 않는 것 같았다.
때문에 socket.io대신 WebSocket을 사용하기로 하였고,   
이번 posting에서 다룰 내용이 바로 이부분이다. 
- Plack -> Mojolicious
- Socket.io -> WebSocket

먼저 인후씨가 알려준 내용을 복습하기 위해 작업한 directory(sshell) 은 냅두고, 
```perl
$ mojo generate app sshell2
```
로 sshell2 를 만들었다.

![image](https://user-images.githubusercontent.com/72643027/108491783-b2efbd80-72e7-11eb-8372-233df7198db6.png){: width="70%" height="70%"}


만들자 마자 초기 상태는 이러하다. 기본적인 MVC 구조를 갖춰진 상태로 app이 생성된다.   
여기서 이제 기존 구현된 앱을 가져와서 구조만 조금 바꿔볼 것이다.

먼저 기존 앱에서 정적 file들을 관리하는 htdocs 하위의 내용들을 동일 성격을 가진 public dir 밑으로 가져온다. 
이 때, Socket.io는 쓰지 않을 것이기 때문에 위에서 언급했던 관련 파일은 제외하고 가져온다.

웹 터미널을 어떻게 구현했는지 아직 모르기 때문에 기존 앱의 lib dir 밑에 있는 파일들도 모조리 cp해서 lib 하위에 넣는다.

그리고 기존 앱의 template 하위의 index.caml 파일도 가져오자. (가져올 때 이름은 index(마음대로).html.ep로 변경하고 example 하위에 넣는다.)

![image](https://user-images.githubusercontent.com/72643027/108497870-71631080-72ef-11eb-89ae-608a398d9a2f.png){: width="70%" height="70%"}


가져온 후의 계층도는 이러하다.   

### Client - side WebSocket setting
이제 templates/example 하위의 index.html을 수정해보자.
```html
<!doctype html><html>
    <head>
        <title>Showmetheshell</title>
        <link rel="stylesheet" href="/styles.css" type="text/css" />
        <script type="text/javascript" src="/jquery.js"></script>
        <script type="text/javascript" src="/jquery.json.js"></script>
        <script type="text/javascript" src="/shell.js"></script>
        <script type="text/javascript">
//            $(document).ready(function() {
//                var socket = io.connect();
//                var shell  = new Shell();
//
//                socket.on('message', function(data) {
//                    if (typeof data == 'undefined' || data === null)
//                        return;
//
//                    data = $.evalJSON(data);
//
//                    if (data.type == 'history') {
//                        for (var i = 0; i < data.history.length; i++) {
//                            shell.updateRow(i + 1, data.history[i]);
//                        }
//                    }
//                    else if (data.type == 'row') {
//                        shell.updateRow(data.row, data.text);
//                    }
//                    else if (data.type == 'cursor') {
//                        //shell.moveCursor(data.x, data.y);
//                    }
//                });
//
//                shell.onsend = function(message) {
//                    socket.send(message);
//                };
//
//                socket.on('connect', function() {
//                    $('#disconnect').html('<a href="#">Disconnect</a>');
//                    $('#disconnect > a').click(function() {
//                        socket.disconnect();
//                        return false;
//                    });
//
//                    $('#container').html('Connected. Initializing...');
//
//                    shell.init();
//                });
//
//                socket.on('disconnect', function() {
//                    shell.close();
//                    $('#disconnect').html('');
//                    $('#container').html('Disconnected');
//                });
//
//                socket.on('connect_failed', function() {
//                    $('#disconnect').html('');
//                    $('#container').html('Connect failed');
//                });
//
//                socket.connect();
                $(document).ready(function()){
                    var socket = new WebSocket("ws://localhost:3000"); // WebSocket 객체생성.
                    var shell = new Shell();
                    socket.onopen = function(e){ // WebSocket이 연결되었을 때 호출되는 method. 
                        $('#disconnect').html('<a href="#">Disconnect</a>');
                        console.log('connected')  // WebSocket이 연결되었는지 log찍어봄.
                        shell.init();
                    };
                    socket.onmessage = function(e) { // Server로부터 data 수신될 때, e를 받으며 onmessage 호출됨.
                        if(typeof e == 'undefined' || e === null) return;
                        var msg = JSON.parse(e.data);

                        switch(msg.type){
                            case 'history':
                                for( var i = 0; i< msg.history.length; i++){
                                    shell.updateRow(i + 1, msg.history[i]);
                                }
                                break;
                            case 'row':
                                shell.updateRow(msg.row, msg.text);
                                break;
                            case 'cursor':
                                break;
                        }
                    };
                    shell.onsend = function(message) { socket.send(message) }; // Server로 data 전송하는 send() 메소드는 socket.io와 WebSocket이 동일. 그대로 사용하였다.
            });
        </script>
    </head>
    <body>
        <div id="disconnect"></div>
        <div id="container">
            Connecting...
        </div>
    </body>
</html>
```
먼저 WebSocket 사용법을 다음 링크로 접속하여 숙지하자.   
https://developer.mozilla.org/ko/docs/WebSockets/Writing_WebSocket_client_applications   
그리고 기존에 Socket.io를 이용하지 않고 WebSocket객체를 생성해서 메소드의 작동방식에 따라 정의해 주면 된다.   
switch - case는 parameter로 전달받은 e의 data 속성을 parse하여 기존 if, elsif, else 로 이루어진 부분을 변경했을 뿐이다.   

이로써 javascript를 이용하여 Client 측 WebSocket 설정은 끝났다.<br/><br/>

### Server - side WebSocket setting
이제 perl로 Server측 WebSocket 설정하는 부분을 작성해 보자.

먼저 HTTP GET으로 '/'를 받았을 때, 우리가 작성한 template/example/index.html.ep 로 routing 해 줘야 하므로    
lib/sshell2.pm 파일의 route 부분을 수정한다.
```perl
$r->get('/')->to('example#index');
```
그리고 다음으로 Controller/Example.pm을 작성해 보자.
```perl
package sshell2::Controller::Example;
use Mojo::Base 'Mojolicious::Controller', -signatures;

my $root;

BEGIN {
    use File::Basename ();
    use File::Spec     ();

    $root = File::Basename::dirname(__FILE__);
    $root = File::Spec->rel2abs($root);

    unshift @INC, "$root/../../../lib";
}

use lib 'lib';

use Plack::Builder;
use Plack::App::File;

use Handler;
use Text::Caml;
use PocketIO;
use Terminal;
use Terminal::Ascii2Html;
use JSON ();

# This action will render a template
my %clients;
sub index ($self) {

    # Render template "example/index.html.ep" with message
    #	my $log = $self->app->log;
    #   $clients{$id} = $self->tx;

    $self->render(msg => 'Welcome to the Mojolicious real-time web framework!');

    my $cmd = $self->{cmd};
    $self->{ascii2html} = Terminal::Ascii2Html->new;

    my $terminal = Terminal->new(
        cmd            => $cmd,
        on_row_changed => sub {
            my ($terminal, $row, $text) = @_;

            $log->debug("TEXT : $terminal $row $text");

            $text = $self->{ascii2html}->htmlify($text);

            my $message = JSON->new->encode(
                {type => 'row', row => $row, text => $text});
            $self->send($message);
        },
        on_cursor_move => sub {
            my ($terminal, $x, $y) = @_;

            my $message =
            JSON->new->encode({type => 'cursor', x => $x, y => $y});
            $self->send($message);
        },
        on_finished => sub {
            my $terminal = shift;

            $self->close;
        }
    );

    $self->on(
        message => sub {
            my ($self, $message) = @_;

            my $json = JSON->new;

            eval { $message = $json->decode($message); };
            return if !$message || $@;

            my $type = $message->{type};
            if ($type eq 'key') {
                my $buffer;

                my $code = $message->{code};

                $terminal->key($code);
            }
            elsif ($type eq 'action') {
                $terminal->move($message->{action});
            }
            else {
                warn "Unknown type '$type'";
            }
        }
    );

    $self->on(
        disconnect => sub {
        }
    );

    $terminal->start;


    builder {
        mount '/socket.io/static/flashsocket/WebSocketMain.swf' =>
        Plack::App::File->new(file => "$root/htdocs/WebSocketMain.swf");

        mount '/socket.io/static/flashsocket/WebSocketMainInsecure.swf' =>
        Plack::App::File->new(file => "$root/htdocs/WebSocketMainInsecure.swf");

        enable "Static",
        path => qr/\.(?:js|css|jpe?g|gif|png|html?|swf|ico)$/,
        root => 'htdocs';

        mount '/socket.io' =>
        PocketIO->new(instance => Handler->new(cmd => '/bin/bash'));

        mount '/' => builder {
            enable "Static",
            path => qr/\.(?:js|css|jpe?g|gif|png|html?|swf|ico)$/,
            root => "$root/public";

        };
    };
};

1;
```
showmetheshell.psgi 과 terminal.pm  의 틀을 가져와서 example의 index메소드를 정의하였다.
