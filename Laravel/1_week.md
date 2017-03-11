#01 :: 라라벨 스터디

##Study
###Laravel 설치
- composer 사용 (컴포저 설치 후 컴포저를 통한 라라벨 프로젝트 생성)
  ```
  composer create-project --prefer-dist laravel/laravel blog
  ```

- installer 사용 (컴포저로 인스톨러 설치 후 이스톨러를 사용한 라라벨 프로젝트 생성)
  ```
  composer global require "laravel/installer"
  laravel new blog
  # 터미널에서 laravel 명령어 사용을 위해 $HOME/.composer/vendor/bin을 환경변수 $PATH에 등록해줘야 합니다.
  ```


###composer를 통한 의존성 관리
composer를 사용하여 php의존성을 관리할 수 있다.
install, update 명령어를 사용할 수 있는데 각각 사용케이스를 구분하자면 관리자와 사용자로 구분할 수 있을 것 같다.
```composer update```는 관리자용 명령어
(.json을 토대로 업데이트하고 lock을 갱신)
```composer install```는 사용자용 명령어
(.lock을 토대로 설치하고 없으면 .json으로 설치 진행 후 .lock생성)

많은 의존성 모듈은 packagist.org 에 올라와 있지만 사설 패키지를 composer로 관리해야할 경우도 있다.
> Q 회사에서 만든 사설 컴포저 패키지(PHP 라이브러리)는 어떻게 컴포저로 가져올 수 있냐?

> A 두 가지 방법이 있습니다. 첫째 사내에서 쓰는 사설 패키지스트를 운영하는 방법(설치형 소스가 공개되어 있습니다.). 둘째, 로컬이나 사설 깃허브 저장소에 올려놓고 가져오는 방법이 있습니다. 두번째 방법은 `composer private repository` 라는 키워드로 검색하시면 됩니다.

###namespace
보통 회사 이름\프로젝트\기능\하위기능 이름으로 합니다. e.g. `Tmon\FooProject\Command`
개인프로젝트일 경우 회사 이름 대신 깃허브 ID를 사용합니다.
일반적으로.. 그렇다는 겁니다.
디렉토리 구조는 보통 루트만 맵핑시키고 그 하위는 네임스페이스와 폴더 구조를 동일하게 가져갑니다. `app`을 `Tmon`에 맵핑시키겠죠.
namespace는 사용
use는 선언
같은클래스는 use 네임스페이스 as 사용할별칭
네임스페이스가있는 클래스 내부에서 전역멤버사용시 '\'붙여줘야지 php전역 멤버를찾고 '\'안하면 해당클래스의 네임스페이스 내부에서 찾음
\Exception();

###psr-4 autoload
오토로드는 컴포저로 설치한 애들 및 composer.json의 "autoload" : { 에설정된 애들을 한방에 불러주는 것
오토로드는 require 파일경로를 하나로 묵어주는거고 use는 네임스페이스경로만
 뭐가 오토로더 대상인지는 vendor/하위 애들은 다 불려진다고 보면됨,
 라라벨 경우는 composer.json에 "autoload" : "psr-4":{}리스트들로 오토로드 되게 해줄 수 있습.
기존 require(클래스풀경로) 시 해당 클래스가 다른 클래스나 인터페이스를 사용하고있으면 그애들까지 require를 다해줘야함 개불편
composer에서 오토로더 깔아서 라라벨 아닌데서도 사용가능
근데 php7이면 그냥사용되는건가?? psr-은 php버전에 탑재되어 있는건가?
오토로더 사용명세
APP:\\ : \app



##TIP
- __멀티바이트를 고려한 DB생성__
mysql DB생성시 utf8mb4 인코딩으로(mb4가 이모티콘같은거 사용가능 멀티바이트)

- __PHP에서 멀티바이트 문자열 처리를 위해 고려해야할 사항__
mbstring PHP 확장 모듈을 설치하면 `mb_*` 로 시작하는 함수와 관련 클래스 사용가능.
  ```# REPL 실행
  $ php -a

  > echo strlen('가나다');
  # 9
  > echo mb_strlen('가나다');
  # 3
  ```
  utf-8 한글은 3byte 문자, 이모티콘은 4byte 문자기 때문에 항상 mb_* 함수를 쓰는 것이 안전하며 라라벨은 전부 mb_* 함수를 사용한다.


- __외부입력값 사용시 주의사항__
```$mode = filter_input(INPUT_GET, 'mode', FILTER_SANIITIZE_STRING);```
```$mode = $_GET['mode'];```
두가지 방법중 전자 방식으로 처리하는게 좋다.
-- http://php.net/manual/en/function.filter-input.php  (PHP 5 >= 5.2.0, PHP 7)
-- http://php.net/manual/en/filter.filters.sanitize.php 3번째 옵션리스트

- __프론트컨트롤러란__
프론트컨트롤러는 라우터를 포함하는 범주이기는 하나 세션, 서비스프로바이더 등록, 오토로더 등의 처리를 하며, 라라벨에서는 라우터는 따로 분리되어있다.
라라벨에서 index.php파일에 해당하며 주로 MVC + 프론트컨트롤러 형태의 방식이 많이 사용된다. MVC프레임워크는 페이지컨트롤러와는 맞지않음

- __PHP는 멀티프로그래밍패러다임__
(절차지향 + 객체지향) 함수형프로그래밍에선 익명함수사용

- __PHP에서 글로벌 멤버는 사용하지 않는게 좋음__
php에서는 기명이든, 익명이든 함수 내부에서 외부의 자원에 접근불가(PHP의 스코프 단위는 {}레벨 단위 스코프??)
기명이면 파라미터로 전달하여 사용, 익명은 클로저로써 해당자원을 참조하도록 처리하여사용 bindTo(), use 등

- __익명함수__
콜백함수와 클로저를 만드는 방법이 익명함수로 PHP에서는 콜백함수와 클로저를 구분이 없다.


##Question
1. 간단한 흉내와 대규모의 차이는 뭐가 있을까?
장비성능이슈 말고 웹서버나, 스위치같은 장비에서 트레픽분산하고 멤캐시, 레디스같거 사용해서 캐싱처리하고 젠드오피캐시, 제너레이터 사용하면 성능에 문제가 있나?
>메모리 이슈가 있다고는 하는데 좀 더 알아봐야할 듯

2. 오토로더 사용하려면 php7이면 그냥사용되는건가?? composer로 설치해서 사용해야하나? psr-n은 php버전에 탑재되어 있는건가?

3. 로그쌓는거 설정? PSR-3로거인터페이스를 구현한 패키지를 설치해서 쓰면되나?

4. api나 link등의 url관리는 그냥 하드코딩?
> 라우터 작성시 ->name('라우터명'); <br/>
route('라우터명')으로 꺼내 쓸 수 있다

5. api가이드 문서 같은거 쉽게 만드는 javadoc, jsdoc같은거 있나?

6. 라라벨 설치시 composer, installer 두 방식의 차이점? 라라벨 필요로하는 모든 의존 패키지들은 수동으로 해줘야하나?
>잘 모르면 일단 composer를 사용
