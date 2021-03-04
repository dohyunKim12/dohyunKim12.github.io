---
layout: post
title: "GMS UnitTest tutorial(2)"
summary: Write unit test code
author: Dohyun Kim
date: 2021-03-03 21:30:00 -0400
category: DesignPattern
thumbnail: /assets/img/posts/gitlab.png
comments: true
---

Summary
---
지난시간에 REST_API 명세를 살펴봄에 이어서 이번에는 직접 실질적인 Unit Test code를 작성해보는 시간을 가졌다.

GMS
---

먼저 시작 전에 ```MOCK_ETCD=1 prove -lvm -Ilibgms -I../GSM/lib``` 으로 test code 한번 돌려보고 시작하자.

All tests successful 로 잘 돌아간다는 멘트가 나왔다면, 시작할 수 있다.

오늘은 GMS 하위의 ```.gitlab-ci.yml``` 파일을 들여다 보는 것으로 시작한다.
```yaml
image: "centos:7" #어떤 docker 이미지를 사용할건지. (unit test 는 도커를 사용하기때문)

variables: # 환경변수
    GIRASOLE_ROOT: "/tmp/girasole"

before_script:
    - |
        echo "
        [anystor-e]
        name=AnyStor Enterprise Repository for production
        baseurl=http://abs.gluesys.com/repo/anystor/3.0/os/x86_64
        enabled=1
        gpgcheck=0

        [anystor-e-testing]
        name=AnyStor Enterprise Repository for development
        baseurl=http://abs.gluesys.com/repo/testing/3.0/os/x86_64
        enabled=1
        gpgcheck=0

        [anystor-e-debuginfo]
        name=AnyStor Enterprise Repository for DebugInfo
        baseurl=http://abs.gluesys.com/repo/anystor/3.0/debuginfo
        enabled=0
        gpgcheck=0

        [anystor-e-testing-debuginfo]
        name=AnyStor Enterprise Repository for DebugInfo
        baseurl=http://abs.gluesys.com/repo/testing/3.0/debuginfo
        enabled=0
        gpgcheck=0" > /etc/yum.repos.d/anystor-e.repo
    - yum clean all
    - >
        yum install -y
        git etcd jq rpm-build expect cifs-utils nfs-utils bonnie++ ntpdate
        perl-enum perl-Env perl-Test-Harness perl-Devel-Leak-Object
        perl-Devel-NYTProf perl-Devel-Cover perl-Test-Class-Moose
        perl-Test-MockModule perl-Mock-Sub perl-Array-Diff perl-XML-Smart
        perl-DateTime perl-DateTime-Format-Strptime
        perl-Module-Load perl-Module-Loaded
        perl-Mouse perl-MouseX-Foreign perl-MouseX-NativeTraits
        perl-Etcd perl-Coro perl-Coro-Multicore
        perl-Data-Compare perl-Data-Validator perl-Data-Validate-IP
        perl-Data-Dump perl-IPC-Cmd perl-File-chmod-Recursive
        perl-File-Copy-Recursive perl-Filesys-Statvfs perl-Socket6
        perl-Net-IP perl-Hash-Merge perl-Net-OpenSSH
        perl-AnyEvent perl-AnyEvent-HTTP
        perl-Mojolicious perl-Mojolicious-Plugin-OpenAPI
        perl-Sys-Syslog perl-MojoX-Log-Log4perl perl-Switch
        perl-Sys-Hostname-FQDN perl-IO-Interface perl-Digest-SHA
        perl-IO-Compress perl-libintl
        perl-Number-Bytes-Human perl-Filesys-Df perl-File-Slurp
        perl-Net-Netmask perl-Proc-Exists perl-String-Util perl-TimeDate
        perl-Mojo-JWT perl-CryptX perl-Crypt-AES-CTR perl-Crypt-OpenSSL-RSA
        samba-client cifs-utils perl-Tree-Simple
    - git clone http://jenkins:JzGRBbyEXa7tN-1zam6N@gitlab.gluesys.com/gitlab/potatogim/GSM.git ${GIRASOLE_ROOT} 
    #  클론을 받아오는데 계정에 대한 토큰을 미리 받아올 수 있음.( 클론 받아올 계정명:access token~~) 패스워드가 아니여도 이런식으로 clone 받을 수 있다.

# 여기서부터 실제 test.
unittest: # gitlab ci 에 들어가보면 나오는 unittest . (각  step 을 나타냄)
  script:
    - pwd
    - env
    - cover -delete
    - >
      MOCK_ETCD=1 TEST_VERBOSE=1 HARNESS_PERL_SWITCHES=-MDevel::Cover
      prove -lvm -Ilibgms -I${GIRASOLE_ROOT}/lib t/unit.t :: --statistics
    - cover -ignore_re '^t/prove' # ignore regular expression (예외 추가)
  artifacts: #artifacts (test 결과물 중 보관대상) 을 어떻게 보관할건지. 
    when: always #항상 만들것이다.
    paths:
      - cover_db # 
      - unit.log # unit test 돌렸을 때 log 남는것
    expire_in: 1 week   #일주일간 킵. 이후 만기.
```

이러한 형태로 존재하는 .gitlab-ci 야맬 파일을 유심히 볼 필요가 있다.  
주석으로 작성해 놓은 설명을 읽어보고 진행하자.

먼저 어떤 part의 unit-test code를 작성할 건지 한번 coverage 정보를 살펴보자.


gitlab - CI/CD 에 올라온 Pipelines 를 보면, 맨 우측에 다운로드 버튼이 있고 ```artifact```를 다운할 수 있게 되어있다.

zip 파일을 다운받아서 압축 해제 후, 안의 ```coverage.html``` 파일을 열어보면 다음과 같은 화면이 띄워진다.

![image](https://user-images.githubusercontent.com/72643027/109814640-14157a80-7c72-11eb-89e0-e48715ab57b5.png){: width="90%" height="90%"}

여기서 많은 것들 중, libgms-Account-Local pm 을 살펴보자.  
오늘은 이 Account::Local 에서도 check 메서드에 대한 test code를 작성해 볼 것이다.

![image](https://user-images.githubusercontent.com/72643027/109815136-a87fdd00-7c72-11eb-95a9-0bd67a5123da.png){: width="90%" height="90%"}

![image](https://user-images.githubusercontent.com/72643027/109895864-56bb6f00-7cd3-11eb-9bc6-ccb193c55a34.png)

먼저 다음과 같이 표시되는 table에서 각 column 이 의미하는 바가 뭐지 살펴보자.  
- **stmt** : statement. 단일 code, 단일 구문에 대한 cover.
- **bran** : branch. 분기문에 대한 cover.
- **cond** : condition. 조건 자체에 대한 cover.
- **sub** &nbsp; : subroutine. 호출이 안되는 함수가 있느지 확인.

이 중에서 밑으로 쭉 내리다가 만나는 빨간색 part (아직 test code 작성되지 않은 부분) check 메서드의 cond 부분에 under line이 그어져 있는 부분을 클릭해 보자.

![image](https://user-images.githubusercontent.com/72643027/109896347-1f998d80-7cd4-11eb-9d08-737cbcd1a4c6.png)

이런 식으로 화면이 띄워진다. libgms 하위의 Account::Local 의 66 line으로 가보자.

![image](https://user-images.githubusercontent.com/72643027/109897992-ddbe1680-7cd6-11eb-9db3-f26c083ef714.png)

이렇게 분기문이 존재한다. 이 부분을 검증해주는 code가 존재하지 않는다는 뜻이다.

그럼 이 부분에 대해 test code를 작성해 보자.

```GMS/t/lib/Test/``` 하위에 ```Account``` directory 를 생성하고, 그 안에 ```Local.pm``` 파일을 생성하자.  

기본적으로 test를 하기 위한 template은 다른 아무 test 파일을 참조하여 가져온다.  

Account::Local 의 check 메서드를 검증하는 test code이다. 따라서 subroutine 명으로 ```test_check``` 를 작성한다.

![image](https://user-images.githubusercontent.com/72643027/109897260-ca5e7b80-7cd5-11eb-8283-f000e6dcf88b.png)

이제 test_check 메소드만 작성하면 된다.

자. 어떻게 생각이 흘러가야 할까? 먼저 Account::Local의 check method를 살펴보자. 

Account::Local 의 check에서는 분기문으로 passwd_file, group_file, shawdow_file 이 각각 읽히는지를 확인하고 있다.   
이 파일들이 무엇인지부터 확인해야 겠다.

Account::Local에서는 
```
Account::Local::Config
Account::Local::User
Account::Local::Group 
Account::Local::Subgroup
``` 
들을 상속받고 있다.  

상위 경로로 올라가서 ```Account/Local``` 하위로 들어가보자.  
User.pm 과 Group.pm 등 확인해보고 싶은 파일들이 존재한다.  
각각에서 살펴보니 역시 ```passwd_file, group_file, shawdow_file``` 속성들이 무엇인지 정의하고 있다.

![image](https://user-images.githubusercontent.com/72643027/109898538-cc293e80-7cd7-11eb-809e-ad221a4fdc0e.png)

이제 이 파일들이 무엇인지도 확인하였으니, test code를 작성해 보자.

```perl
sub test_check : Tests(no_plan)
{
    my $self = shift;


    mkdir('/tmp/etc'); # /etc/passwd 같은 파일을 변경하게 되면 안되므로, /tmp 디렉토리 하위에 etc dir를 생성하고 작업한다.
    my $fh;

    # Mock_up

    my $obj = Account::Local->new( # Class Object를 생성할 때, 각 속성들을 parameter로 전달하여 설정한다.
        passwd_file => '/tmp/etc/passwd',
        shadow_file => '/tmp/etc/shadow',
        group_file => '/tmp/etc/group',
    );

    $obj->check(); # Account::Local::check() 호출.

    cmp_ok($obj->check(), '==', 0, 'check()_success');  # cmp_ok(실행값, 비교 연산자, 기댓값, test description)
    
    foreach my $file ('/tmp/etc/passwd', '/tmp/etc/shadow', '/tmp/etc/group')
    # for문 순회로 각 파일들을 하나씩 unlink(파일 삭제) 해봄.
    # 이로써 파일을 하나씩 못 읽는 
    {
        unlink($file);
        cmp_ok($obj->check(), '==', -1, 'check()_failure');
        open($fh, '>', $file);
        close($fh);
    }
}
```

