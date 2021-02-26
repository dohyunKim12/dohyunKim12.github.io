---
layout: post
title: "GMS UnitTest tutorial(1)"
summary: Writting openAPI
author: Dohyun Kim
date: 2021-02-24 21:30:00 -0400
category: DesignPattern
thumbnail: /assets/img/posts/gitlab.png
comments: true
---

Summary
---
회사의 Gitlab에서 GMS project를 fork 떠 와서 나의 개인 repository에 옮긴 다음, API 명세를 openAPI 형태에 맞춰 작성하고, 
구현한 API에 대하여 test하는 UnitTest code 작성, Controller를 작성하는 작업을 해 볼 것이다.<br/><br/>

GMS
---

먼저 GMS project에서 요구하는 library(perl module) 들은 Mojolicious를 포함하여 수없이 많다.  
Mojolicious를 설치하였고, GMS가 돌아가기 위한 적합한 환경임을 가정한다. (ver8.32)  
이 외에도 test를 돌리며 마주하는 수많은 dependencies setting과 module들은 사실상 GMS/build/required__packages에 나와있으나,  
이는 CentOS 기준이고 필자는 우분투 환경이었으므로(ver 20.04) cpanm 명령어로 일일이 module들을 설치해 가며 진행하였다.

팀장님께서 교육하시기를 openAPI의 기술적 의미는 사실상 swagger 라고 하셨다.   
이것이 무슨 말인지 처음엔 이해할 수 없었으나, 실습을 진행해보며 Swagger Editor를 사용해보니 바로 느낌이 왔다.

먼저 우리는 어떠한 SW제품을 개발하거나 유지보수를 할 때, 해당 API server가 어떠한 스팩을 가진 data를 주고 받는지에 대한 문서작업이 반드시 필요하다.  
이러한 문서작업은 시간소요가 상당한데, 이를 자동화하는 tool이 바로 swagger이고 openAPI이다.   

방법은 생각보다 간단하다. 그저 api.yaml 파일을 프로젝트 내에 생성하고 이를 Swagger Editor에 넣기만 하면,
API문서를 제공함과 동시에,  
debugging이 가능하고 작성한 API에 대해 method 별로 실행해 볼 수 있는 Curl code를 제공한다.

이제 대략적인 설명이 끝났으니, 실습으로 들어가보도록 하자.

먼저 dir를 하나 만들고 하위에 GMS와 GSM을 clone받는다.

![image](https://user-images.githubusercontent.com/72643027/109002781-fe400c80-76e9-11eb-8b34-9b2aa89d476c.png){: width="70%" height="70%"}

GMS 하위에서 ```MOCK_ETCD=1 prove -l -v -m -Ilibgms -I../GSM/lib```
으로 test를 진행해본다.

![image](https://user-images.githubusercontent.com/72643027/109003149-6b53a200-76ea-11eb-8656-531eabc579c4.png){: width="70%" height="70%"}

이와 같이 All tests success가 나와야 정상. 그렇지 않으면 fork 해 온 project에 문제가 있거나  
필요한 module이 설치되지 않아서이다.

이제 정적파일을 관리하는 dir인 public 하위에 api.yaml 파일을 수정해보자.

API를 작성하는데, REST API의 표준인 OpenAPI (ver 3.0) 의 명세를 따라 작성한다.<br/><br/>

OpenAPI
---

HTTP method에 대해 살펴보면  

| HTTP method &nbsp;&nbsp; | function|
|-------------|---------|
| GET        |자원의 조회|
|POST | 자원에 대한 모든 비멱등 연산과 정보를 은닉할 필요가 있는 조회, 자원에 종속되지 않는 명령형 API|
|PUT | 자원의 교체 혹은 수정(all)|
|PATCH| 자원의 일부만을 수정 |
|DELETE | 자원의 삭제|
{:.table-striped}<br/>

와 같이 사용된다. (출처 - gluesys redmine) 

작성할 API들은 다음과 같다.

| HTTP method &nbsp;&nbsp;  | URI     | Description|
|-------------|---------|------------|
|GET | ```/api/v3/bikes```|바이크들의 목록을 조회.|
|GET | ```/api/v3/bikes/{id}```|이름을 통해 특정 바이크의 정보를 조회.|
|POST| ```/api/v3/bikes```|바이크를 추가.|
|POST| ```/api/v3/bikes/search```  |조건에 해당하는 바이크들을 검색.|
|PUT | ```/api/v3/bikes/{id}```|이름을 통해 특정 바이크의 정보를 수정(교체).|
|PATCH | ```/api/v3/bikes/{id}```|이름을 통해 특정 바이크의 정보 일부분을 수정.|
|DELETE | ```/api/v3/bikes/{id}```|이름을 통해 바이크를 삭제.|
{:.mbtablestyle}<br/>

참고로 API 경로(URI)에서 자원을 나타낼 때에는 복수형으로 명명한다.

이제 이 API들을 api.yaml 파일에 작성하는데, ```paths```하위에 추가하고 각 method들에 대해 HTTP request와 response를 작성해 줄 것이다.

![image](https://user-images.githubusercontent.com/72643027/109005521-617f6e00-76ed-11eb-8ed6-24625a94aa60.png){: width="70%" height="70%"}

api.yaml 파일을 열었더니 다음과 같이 보인다. 이제 paths 하위에 URI를 작성한다. api.yaml을 작성시에는 indent에 굉장히 주의를 기울여야 한다. (현재 yaml파일에는 indent가 띄어쓰기 2칸으로 설정되어 있음.)

paths 하위에 ```/bikes``` 부터 순차적으로 ```bikes/search``` ```bikes/{id}``` 와 같이 URI를 적어주고, 또 그 하위에 ```get``` ```post``` ```put``` ```patch``` ```delete``` 와 같은 method들을 적어준다. 각 method들의 하위에는 request와 response에 대한 내용을 작성한다. 

request를 정의하는 방법에는 2가지가 있다.
- parameters:  ```/?key:value``` 의 형태로 url로 넘김.
- requestBody: ```post``` 로 숨겨서 넘김   

method마다 각각을 적절히 사용하여 정의해 보자. (vi에서 작업해도 되나, Swagger Editor 이용 시 debugging측면에서 좀 더 수월)

본 tutorial_1 에서는 모든 URI를 작성하지 않고 가장 기본적인 GET과 POST에 대한 부분만 간단히 작성해 볼 것이다. 

GMS에서는 각 method의 하위에 공통적으로 들어가야할 field들이 존재한다.
각 field가 의미하는 바는 다음과 같다.

{:class="table table-bordered"}
|field 명| Description|
|--------|-----------|
|x-mojo-to  |x로 시작하는 것은 확장 키워드. Mojolicious에서 제공. <br/> 어떤 API controller에서 이 API요청을 처리할 것인지 정의. <br/> ```컨트롤러#메서드```|
|operationId &nbsp; &nbsp; |각 API에 대한 식별자. 전역적으로 고유한 값이어야 함.|
|summary|API의 간략한 설명.|
|description  |API에 대한 자세한 설명. 최대한 명확하고 디테일하게 적어줘야 한다.|<br/>

이 field들은 공통적으로 모든 method 정의 시 들어가야 한다.

먼저 get method에 대한 부분이다. 
```yaml
paths:
  /bikes:
    get:
      x-mojo-to: 'Bike#list_by_page'
      operationId: listBikesByPage
      summary: List Bikes by page
      description: >
          ### LIST bike by page  
          list bikes by well
          ---
      parameters:   # request 에 대한 정의.
          # 매개변수의 위치를 지정
        - in: query #어디서 받아올 건지.(어떤 방식인지)
          # 매개변수의 이름을 지정
          name: offset
          # 매개변수의 자료형을 가리키는 스키마를 정의
          schema:
            # 자료형은 정수형
            type: integer
            # 최소값은 0
            minimum: 0
            # 기본값은 0
            default: 0
          # 필수 매개변수 여부
          required: false
          # 매개변수의 설명
          description: The number of items to skip before starting to collect the result set
        - in: query
          name: limit
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
          required: false
          description: The numbers of items to return

      responses:   # response 에 대한 정의.
        # HTTP 응답 코드
        200:
          # 이 응답에 대한 설명
          description: Bike page retrieval successful
          # 응답 내용(content) 정의를 시작
          content:
            # 응답 내용의 유형 (Content-Type: application/json과 등가)
            application/json:
              # 이 유형에 대한 스키마를 정의 (자세한 내용은 아래에서)
              schema:
                # 응답의 자료형 (여기에선 배열)
                type: array
                # 배열의 원소를 나타내는 스키마 혹은 자료형
                items:
                  # 배열 원소의 자료형 (여기에선 객체)
                  type: object
                  # 배열 원소인 객체들이 갖는 속성
                  properties:
                    # 각 배열 원소들은 id, name, birthdate라는 속성을 갖는 객체
                    id:
                      # id 속성은 정수형(integer) 데이터
                      type: integer
                    name:
                      # name 속성은 문자열(string) 데이터
                      type: string
                    birthdate:
                      # birthdate 속성은 yyyy-mm-dd의 날짜 형식(date)을 따르는 문자열(string) 데이터
                      type: string
                      format: date
        # 지정되지 않은 모든 응답(여기에선 앞서 정의한 200을 제외한 나머지)
        default:
          description: Unexpected error
```
이렇게 작성될 수 있다.  
OpenAPI에서는 모든 data의 서술이 schema를 통해 이루어진다. request와 response에 모두 schema를 정의해주었다. 

get method의 request정의 부분에는 **parameters**를 이용하여 HTTP request의 **URI / header** 등에 있는 매개변수를 나타내었고,  
**requestBody**이용하여 정의 시 HTTP request의 **본문(body)** 에 있는 매개변수를 나타낼 수 있다.  
response 부분에는 content를 작성하여 응답 내용을 정의하였다.  
이 때 'MIME 표준에 의한 content의 유형' 에는 다음과 같은 것들이 존재한다.

|Contents                           |Description|
|-----------------------------------|-----------|
|application/json|JSON format의 data|
|application/xml|XML format의 data|
|application/x-www-form-urlencoded &nbsp; &nbsp;  |RFC1738에 따르는 URL형식으로 encoding된 data.|
|multipart/form-data|Binary data. (주로 file 등)|
|text/plain|단순 text data|
|text/html|HTML data|
|application/pdf|PDF data|
|image/png|PNG format의 img|<br/>

또한 **parameters**를 이용하여 request 정의 시 4가지 위치가 존재한다.

|위치|Description|
|----|-----------|
|query|URI 뒤에 따라오는 질의에서 매개변수를 가져오도록 지정. <br/> ex) ```/api/v3/bikes?id=r6``` 의  경우```r6``` 가 ```id``` 매개변수의 값이 됨.|
|path|URI경로 상에서 ```{...}``` 위치 지정자를 통해 매개변수를 가져오도록 지정. <br/> ex) ```/api/v3/bikes/{id}``` 로 지정시 ```/api/v3/bikes/r6``` 요청이 발생했을 대 r6가 id 매개변수의 값이 됨.
|header  &nbsp; &nbsp; |수신한 요청의 HTTP header에 지정된 키값 쌍에서 매개변수를 가져오도록 지정.|
|cookie  |수신한 요청의 HTTP cookie 에서 매개변수를 가져오도록 지정.|<br/>

이제 post method의 request와 response를 정의해 보자.
```yaml
    # HTTP 메서드가 POST인 /api/v3/bikes API
    post:
      x-mojo-to: 'Bike#create'
      operationId: createBike
      summary: Create a bike
      description: >
         ### Create a bike
         create a bike as well!
         ---

      requestBody:  # request 에 대한 정의.
        # 본문 내용 정의 시작
        content:
          # 본문 내용의 형식 정의 (Content-Type: application/json과 등가)
          application/json:
            # 스키마를 통해 본문 내용의 자료형 정의 시작
            schema:
              # 자료형은 객체
              type: object
              # 객체 자료형일 경우, 어떤 속성들이 있는지 정의하기 시작
              properties:
               # 속성 이름
                id:
                  # 속성의 자료형
                  type: integer
                  # 속성의 설명
                  description: The ID of a bike.
                name:
                  type: string
                  description: The human readable name of a bike.
                birthdate:
                  type: string
                  format: date
      responses:
        # HTTP 응답 코드
        201:
          # 이 응답에 대한 설명
          description: Bike created successfully
```

이렇게 이번에는 requestBody를 이용하여 request를 정의, 
response는 단순하게 201-Created 로 응답하며 description 만 작성해두었다.

또 하나 알아볼 것이 있다. 앞서 정의한 get method의 response 부분을 살펴 보면, 지금까지 작성한 code로는 반환 시 entity의 정보만을 갖고 있다.  

실제 API의 반환에서는 이 반환 data(entity)에 기본 응답(BasicResponse)를 합친 형태로 최종 반환을 해야 한다. 이 떄, **allOf** 키워드를 사용한다.

그리고 이미 만들어진 기본 응답(BasicResponse)template schema을 사용할 것이므로, 참조하기 위해 
참조 키워드 **$ref** 를 사용한다.

따라서, get method - response를 다음과 같이 다시 작성하면 
```yaml
      responses:   # response 에 대한 정의.
        # HTTP 응답 코드
        200:
          # 이 응답에 대한 설명
          description: Bike page retrieval successful
          # 응답 내용(content) 정의를 시작
          content:
            # 응답 내용의 유형 (Content-Type: application/json과 등가)
            application/json:
              # 이 유형에 대한 스키마를 정의 (자세한 내용은 아래에서)
              schema:
                allOf:
                  - $ref: '#/components/schemas/BasicResponse'
                  - type: object
                    properties:
                      entity:
                        type: array
                        # 배열의 원소를 나타내는 스키마 혹은 자료형
                        items:
                          # 배열 원소의 자료형 (여기에선 객체)
                          type: object
                          # 배열 원소인 객체들이 갖는 속성
                          properties:
                            # 각 배열 원소들은 id, name, birthdate라는 속성을 갖는 객체
                            id:
                              # id 속성은 정수형(integer) 데이터
                              type: integer
                            name:
                              # name 속성은 문자열(string) 데이터
                              type: string
                            birthdate:
                              # birthdate 속성은 yyyy-mm-dd의 날짜 형식(date)을 따르는 문자열(string) 데이터
                              type: string
                              format: date
```

이제 최종 반환값에 entity값과 BasicResponse가 포함되어 명확하게 나오는 것을 알 수 있다.

![image](https://user-images.githubusercontent.com/72643027/109095753-afcd5500-775f-11eb-9282-48236fd55c7b.png){: width="70%" height="70%"}

에러 없이 **Swagger Editor**에 API 문서가 잘 보여야 한다.   
여기까지 확인했으면, 한번 확인 차원에서 test code를 돌려보자. (작성한 API를 test하는 것은 아님.)

잘 돌아가는지 확인했다면, 이어서 다음 명령어를 실행하여 작성한 API 가 실제 구동되는 환경으로 setting되었는지 확인한다.
```perl -Ilib -Ilibgms -I../GSM/lib script/gms help``` 하여 살펴보면 Commands 중 routes라는 명령어가 있다. 

```perl -Ilib -Ilibgms -I../GSM/lib script/gms routes```  로 API 가 잘 추가되었는지 살펴본다.  
(실제로는 이렇게 하면 모든 API 들이 출력되기 때문에 grep을 함께 이용하는 것이 편리.)

![image](https://user-images.githubusercontent.com/72643027/109105574-91705500-7771-11eb-9266-ad7bb73d8e05.png){: width="70%" height="70%"}

이렇게 포함 된 것이 확인되었다면 이제 Bike API에 대해 Test code를 작성하고 Controller (Bike class)를 작성해 보자.

먼저 TestCode 작성부터 선행되어야 한다. Test Code를 작성할 때는 항상 실패지점부터 시작한다.

t/lib/Test/ 하위에 Bike.pm 파일을 하나 생성한다. 템플릿이 존재하지 않으니, 다른 test code에서 test를 위한 code 앞부분만 살짝 가져오자.

![image](https://user-images.githubusercontent.com/72643027/109106353-ed87a900-7772-11eb-912c-99211600b17c.png){: width="70%" height="70%"}

이 상태에서 이제 test code를 작성한다. 먼저 정의한 get method에서 x-mojo-to 키워드로 Bike 컨트롤러의 list_by_page 라는 메소드로 넘겨주기로 정의하였으므로,  
    sub로 list_by_bike 라는 API에 대해 test하는 method를 작성한다.

```perl
sub test_list_by_page
{
    my $self = shift;

    my $t = $self->t->get_ok('/api/bikes');
    diag(explain($t->tx->res->json));

    $t->status_is(200)
      ->json_is('/entity/0/id' => 0)
      ->json_is('/entity/0/name' => 'r6')
      ->json_is('/entity/0/birthdate' => '2021-01-01');
}
```
**diag**는 test 수행 시 전달받는 json형태의 data를 보여주면서 디버깅을 좀 더 쉽게 해준다. (에러 메세지 포함)

test code는 작성하였고, 이제 Controller를 작성해보자.  
먼저 작성한 test code는 list_by_page 이기 때문에 그에 맞게 Controller를 작성해본다.

```perl
package GMS::Controller::Bike;

use v5.14;

use strict;
use warnings;
use utf8;

use Mouse;
use namespace::clean -except => 'meta'; #메모리 아끼려고 넣음.

extends 'GMS::Controller';  # GMS의 Controller를 상속받아와 재정의.

sub list_by_page
{
    my $self = shift;
    my $bike = {
        id => 0,
        name => 'r6',
        birthdate => '2021-01-01',
    };
    $self->render(openapi => [$bike]); 
}
sub create
{
    my $self = shift;
}

__PACKAGE__->meta->make_immutable(); # PACKAGE의 metadata를 immute로 만들겠다. (데이터 아끼기 위함)

1;

=encoding utf8

=head1 NAME

GMS::Controller::Bike - Bike management API controller

=head1 SYNOPSIS

=head1 DESCRIPTION

=head1 COPYRIGHT AND LICENSE

Copyright 2015-2020 Gluesys Co., Ltd. All right reserved.

=head1 SEE ALSO

=cut
```

이렇게 작성한다. 정의한 api.yaml에 먼저 정의해놓은 method는 get과 post임을 기억하자. get 은 Bike#list_by_page로, post는 Bike#create 로 각각 rendering 하고 있다.

list_by_page에 대한 test code만 작성된 상태이므로 method도 list_by_page를 먼저 구현해보았다. <br/><br/>

UnitTest
---
이제 ```MOCK_ETCD=1 prove -lvm -Ilibgms -I../GSM/lib``` 으로 test를 진행해 보자.

![image](https://user-images.githubusercontent.com/72643027/109112422-f92c9d00-777d-11eb-9e79-28f696523e11.png){: width="70%" height="70%"}

이와 같이 모두 성공함을 볼 수 있다. (만약 실패하더라도 diag keyword 덕분에 data를 볼 수 있어서 code를 찾아 얼마든지 디버깅 가능하다.)

마지막으로 ```prove```명령어로 test 시, 추가적인 option들에 대해 알아보자.

```MOCK_ETCD=1 prove -lvm -Ilibgms -I../GSM/lib :: --statistics``` 이처럼 입력 시 모든 test class 갯수와 인스턴스, 메소드 수를 포함해 Total test 값들을 볼 수 있다.
![image](https://user-images.githubusercontent.com/72643027/109113169-26c61600-777f-11eb-847c-9921d24ca5f9.png){: width="70%" height="70%"}

```MOCK_ETCD=1 prove -lvm -Ilibgms -I../GSM/lib :: --class Test::Bike``` 다음과 같이 입력하면 Test::Bike에 해당하는 클래스만 test 가 가능하다.
![image](https://user-images.githubusercontent.com/72643027/109113316-6856c100-777f-11eb-8e37-42de1a014206.png){: width="70%" height="70%"}

이렇게 작성한 API에 대해 Test가 통과했다.

--- 

이후 과정으로는 이제 이 code를 Commit 하여 Gitlab에 올리게 되면,

![image](https://user-images.githubusercontent.com/72643027/109119600-3e55cc80-7788-11eb-9f0d-250ffd5c2ab7.png){: width="90%" height="90%"}

![image](https://user-images.githubusercontent.com/72643027/109119769-7826d300-7788-11eb-8915-e2d0c27e9272.png){: width="90%" height="90%"}

이렇게 Gitlab의 CI/CD 에서 자동적으로 Test를 진행하게 된다. (새로 작성한 API 뿐만 아니라 전체 Unit Test 들을 진행하게 됨.)

결과 : job succeeded

---

다음 Posting에서 추가적으로 다뤄볼 부분
- 남은 method들을 똑같은 방식으로 전부 request, response를 정의.
- Controller와 method rendering 하는 부분 작성.
- Test 확인.
- allOf 이외에 컴포넌트의 선택적 적용에 사용되는 ```oneOf``` ```anyOf``` 키워드 공부하기.



