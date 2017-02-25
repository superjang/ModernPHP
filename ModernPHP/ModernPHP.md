# Learning Modern PHP & PHP
#### 학습 키워드
- 컴포저
- 인터페이스
- 추상클래스
- 트레이트
- 클로저
- PSR-1,2
- PSR-3 로거 인터페이스
- PSR-4 오토로더
- 제너레이터
- 젠드오피캐시
- 내장서버

##제너레이터
기존 PHP 이터레이터는 미리 계산된 데이터를 메모리에 올려 순회하는데 이는 상황에 따라 심한 메모리 낭비가 될 수 있습니다. 제너레이터를 사용하면 요청이 있을 때 마다 그때 그때 계산을 수행합니다.
제너레이터 함수를 호출하면 제너레이터 클래스에 속한 객체를 반환하는데 이 객체는 ```foreach```로 순회할 수 있습니다. 순회과정에서 PHP는 제너레이터 인스턴스에 다음 값을 계산하고 제공하도록 매번 요청합니다.
제너레이터는 정방향 순회만 가능하여 역방향이나 건너뛰기는 불가능하지만 메모리를 아낄 수 있습니다. 데이터를 역순회하거나 건너뛰는 행위는 PHP 이터리에터 인터페이스를 직접 구현하거나 이를 구현한 컴포넌트를 찾아보는게 좋습니다.
* 제너레이터로 할 수 있는건 모두 이터레이터로 처리 가능합니다.
예제 및 사례 :  http://blog.ircmaxell.com/2012/07/what-generators-can-do-for-you.html


##젠드오피캐시
PHP가 실행되면 인터프리터가 코드를 분석하고 기계어(젠드오피코드)로 컴파일해서 바이트코드를 실행하는데 웹서비스에서 모든 요청에 이러한 작업이 반복되면 서버의 부담이 크게됩니다. 젠드오피캐시는 기계어로변환된 코드를 캐시하여 사전 컴파일된 바이트코드를 메모리에서 읽어 즉시 실행하므로 급격한 성능 향상을 기대할 수 있습니다.
PHP 코드의 변경 여부는 php.ini의 ```opcache.validate_timestamps``` 항목으로 알 수 있습니다. 개발환경에서는 ```true``` 로 처리하여 즉각 바이트코드를 갱신하고, 프로덕션환경에서는 ```false``` 로 배포시 바이트코드를 제거해줘야 합니다.
```
opcache.validate_timestamps = false;
opcache.revalidate_freq = 0;
```
젠드오피캐시는 [5.5이상에서 지원](http://php.net/manual/en/intro.opcache.php)하는 듯 합니다.
그리고 [설정](http://bit.ly/php-config)을 통하여 젠드오피캐시를 유용하게 쓸 수 있습니다.

---
__학습한 내용을 정리중이라 정확하지 않은 정보고 있을 수 있습니다.
학습키워드 추가나 내용 수정은 풀리퀘스트 보내주시면 반영하도록 하겠습니다.__
(다국어)멀티바이트문자열처리
php는 기본적으로 문자를 8bit단위로 인식한다.
이는 다국어 지원시 이슈를 발생시킬 수 있기때문에 멀티바이트문자열 인 다국어 처리를위해 mbstring(php.net/manual/book.mbstring.php)확장모듈을 설치하여 처리해줘야한다. mbstring 확장모듈으로 자유롭게 인코딩변환을할 수 있다.
멀티바이트문자열 사용시 php.ini파일에 default_charset="utf-8" 로 선언해주어 php에 인코딩타입을 사용하고 있다고 알려줘야한다. php내부 api에서 문자열처리시 이 값을 참고하여 사용한다.

스트림