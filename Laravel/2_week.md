#02 :: 라라벨 스터디

짭스타그램 만들어보기
```
./routes
    /web => url
    /api => api
```
Rest API를 위한 메소드
- get : 조회
- post : 추가
- put : 변경
- patch : 일부변경
- delete : 삭제

라우터 파라미터가 없을경우 default 처리시 파라미터에 ?추가하고 컨트롤러의 파라미터에 디폴트값을 꼭 써줘야한다.
```php
Route::get('/user/{id?}', function (id = 'default') {
    return $id;
});
```


url을 가져올때 라우트헬퍼함수에 등록하여
route메소드에서 해당이름으로 가져올 수 있음
php artisan route:list의 Name칼럼에 이름이 등록된걸 확인할 수 있음
```
Route::get('/user/{id?}', function ($id = 'default') {
    return view('welcome');
})->name('userInfo');

Route::get('user/profile', 'UserController@showProfile')->name('userInfo');

// view.blade.php
<?php echo route('userInfo'); ?>
// request => http://localhost:8000/user/3
// route() value => http://localhost:8000/user
```

라우터컨트롤러는 라우터 파라미터의 이름이아닌 순서대로 파라미터(URI)를 받는다
이름은 같아야하지만 이름으로 맵핑되진 않음.. ㅜ
쿼리스트링은 Request객체에서 꺼내옴

```
Route::get('/user/{grade?}/{id?}', function ( $b = 'id', $a = 'grade') {
    $name = Request::input('name');
    return 'GRADE :: '.$a.' // ID :: '.$b;
})->name('userInfo');
```

뷰단에서 public asset 경로 가져오는법
```
1. <?= asset('css/app.css')?>
2. {{URL::asset('css/app.css')}}
```

View단에 데이터 전달방법
```
Route::get('/greeting', function () {
    return view('greeting', ['name'=>'lee']);
});
꺼낼때 블래이드 템플릿 내에서 아래처럼 사용
<?=name?>
```
```
Route::get('/greeting', function () {
    $modelView = [
        'name' => 'jang',
        'sex' => 'male'
    ];
    return view('greeting', $modelView);
});
꺼낼때 블래이드 템플릿 내에서 아래처럼 사용
<?=name?>
<?=sex?>
```
```
RRoute::get('/greeting', function () {
     return view('greeting', ['name'=>'lee']);
 //    return view('greeting')->with('name','JANG');
 });
```


라라벨은 외부패키지 가져다쓸 수 있는데 가져다 쓸려면 어디다 꽂아야함
그걸 AppServiceProvider.php에 등록
등록시 App::environment(); 로 환경 조건체크하여 환경별 분기 등록한다
https://laravel.kr/docs/5.4/providers#registering-providers
/myapp/app/Providers/AppServiceProvider.php에
```public function register``` 메소드 내부에 ```register``` 메소드로 패키지의 서비스 프로바이더를 등록해준다.
보통 패키지들은 서비스프로바이더 클래스가 존재하며 해당클래스를 차장서 등록하면됨
```
public function register()
    {
        if(\App::environment('local')){
            $this->app->register(\Barryvdh\Debugbar\ServiceProvider::class);
        }
    }
```
```
프로바이더 레지스터시 아래처럼 작성하면 스트링으로 리턴함
 \Barryvdh\Debugbar\ServiceProvider::class
=> "Barryvdh\Debugbar\ServiceProvider"
```



### 블레이드
주석은 이렇게 ```{{-- //확장한 부모의 겹치는 섹션명을 사용한다.--}}```

블래이드템플릿 사용시 아래처럼 하면 ```welcome.member.family``` welcome/member/family.blade.php 파일을 사용한다
```
Route::get('/user/{id?}', function ($id = 'default') {
    return view('welcome.member.family');
})->name('userInfo');
```
블레이드에서 ```{{}}``` 이런식 출력하면 ```htmlentities``` 처리되서 출력되어 XSS에 어느정도 안전하다.
```{!! !!}``` ```htmlentities``` 처리없이 실행되도록 한다.

- @extends('bladeName')
- @section ~ @endsection
- @each('photo', $items, 'item') // viwename, collection, 뷰내에서 참조할 변수이름
- @parent // 내 섹션이아아니고 부모의 중복되는 아이템을 사용 해당 섹션내에서 선언
- @foreach($users as $user) ~ @endforeach
- @unless (Auth::check()) 로그인고고 @endunless

기타 블레이드 문법이 많으니 https://laravel.kr/docs/5.4/blade 자세한건 참고
블레이드도 캐싱된다 변경은 해시로 체크되어 히트시 캐싱처리하는데 해시는 비싼비용이 아님
해시는 파일명, 그럼 파일명+해시파일해서 호출하고 중간단계에서 그걸 붙여주는게 있나?

### 라라벨 내장 로그인 기능 써보기
``` $ php artisan make:auth``` 명령어치면 로그인용 블레이드와
```route/web.php``` 에 아래내용이 추가되어 있다.
user마이그레이션 파일은 라라벨 프로젝트에 기본으로 있으니 사용전 마이그레이션을 한 번 해줘야한다.
```
 Auth::routes();
 Route::get('/home', 'HomeController@index');
```
```
Route::get('/login', function(){
    return view('auth.login');
});
```


##TIP
composer dump-autoload
마이그레이션파일은 psr-4 오토로딩을 하지 않기때문에 파일명 변경시 composer dump-autoload로 컴포저에 적용해줘야함

이미 만들어진 DB Schema를 라라벨 마이그레이션으로 추출하는 컴포넌트입니다.
https://github.com/Xethron/migrations-generator

마이그레이션 생성후 아무것도 안하고
artisan migrate
하고 마이그레이션 파일 스키마수정후
artisan migrate하면 변경할게 없다고 나온다.
이미 마이그레이션이 수행되어 테이블이 생성되었으므로
artisan migrate:rollback 하여 롤백처리하고
다시 artisan migrate 하면된다
##Question


##정리 내공이 부족하여 점점 엉망이 되어가네요.. 개인용이니 혹시 보시는분 계시면 양해 부탁드립니다.
