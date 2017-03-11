#03 :: 라라벨 스터디

##Study
###csrf
form전송시 csrf
app/Http/Middleware/VerifyCsrfToken.php
토큰예외처리 항목 선언가낭
```php
protected $except = [
    //
];
```
이미지 전송시 ```form```태그의 ```enctype``` 속성을```multipart/form-data```로 해줘야 함
```html
<form action="{{ route('posts.store')}}" method="POST" enctype="multipart/form-data">
```
###미들웨어
모든라우터는 app/Providers/RouteServiceProvider.php에 정의되어있고
웹, api라우터들이 적용되어있다
모든 미들웨어는 app/Http/kernel.php에
- protected $middleware = []; // 모든 라우트에 적용되는 미들웨어
- protected $routeMiddleware = []; // 라우터에만 적용하는 미들웨어 리스트

로그인한 사람만 접근하게할때 미들웨어처리 이용
```php
Route::get('/posts/create', function () {
    return view('posts.create');
})
    ->name('posts.create')
    ->middleware('auth'); // 'auth' 라우터에 미들웨어 사용추가(login으로 리다이렉트시킴)

Route::get('/login', function(){
    return view('auth.login');
})
     ->middleware('guest'); // 로그인한 사용자는 로그인뷰로 안가고 홈으로 이동
```

사용자 미들웨어는 artisan make:middleware 미들웨어이름
``` php artisan make:middleware CheckHasImageFile```
app/Http/Middleware/에 미들웨어 클래스파일 생성됨, handel메소드에 로직구현하고
kenel.php에 미들웨어 등록하는데 특정 라우터에서만 작동해야하니 $routeMiddleware에 등록
'hasFile' => \App\Http\Middleware\CheckHasFile::class,
- ```CheckHasFile.php```에 미들웨어 등록 및 처리 로직, 파리미터 사용 방법
```php
// ->middleware('hasFile:img,zip') 이런식으로 미들웨어 파리미터로 ,구분해서 사용
    // 미들웨어의 파라미터 "커널에 등록된 미들웨어명":"파라미터명1, 파리미터명2" 파리미터는 그냥 변수로
    // 미들웨어 내에서 변수로 사용됨 파일필드 검사 미들웨어 경우 폼필드 네이명을 임력하면 될 듯
    // 미들웨어에서는 파라미터에 디폴트 값을 안주면 값이 안들어 올 경우 에러발생, $field = null로 초기화
    public function handle($request, Closure $next, $field1 = null, $field2 = null)
    {
//        dd($field1, $field2);
//        if($_FILES[$field1]['size'] === 0){
        if($_FILES['img']['size'] === 0){
            return back();
        }
        return $next($request);
    }
```
- ```web.php(라우터)```에서 미들웨어 사용
```php
Route::post('/posts', function () {
//    dd(Request::all());
    // API 리턴시 JSON타입으로 떨궈줄 수 있음 아래처럼
    // array(k=>v) 는 옛날 문법 지양
    return [
      'title' => Request::input('title'),
      'content' => Request::input('content'),
    ];

})->name('posts.store')
    // 미들웨어의 파라미터 "커널에 등록된 미들웨어명":"파라미터명1, 파리미터명2" 파리미터는 그냥 변수로
    // 미들웨어 내에서 변수로 사용됨 파일필드 검사 미들웨어 경우 폼필드 네이명을 임력하면 될 듯
    // 미들웨어에서는 파라미터에 디폴트 값을 안주면 값이 안들어 올 경우 에러발생, $field = null로 초기화
->middleware('hasFile:abc,def');
```
###Request
use \Illuminate\Http\Request
컨트롤러에서 파라미터 받아야하고 리퀘스트 값도 꺼내야하면 아래처럼 Request를 첫번째 파라미터로 넣는다.
```Route::post('/posts', function (Request $request, param1, param2) {}```

###컨트롤러
왠만하면 모델별로 컨트롤러 가지게됨
```artisan make:model -m -c -r Comment```
- -m 마이그레이션 함께 생성
- -c 컨트롤러 파일을 함께 생성
- -r 레스트풀하게 리소스(method)컨트롤러를 생성

```artisan route:list``` 현재 앱에서 사용하는 라우트리스트를 볼 수 있다.

Rest 컨트롤러 사용시 각 라우터별 개별 처리해야하는 부분(라우터 미들웨어)은 컨트롤러 클래스의 생성자에서 처리해줘야한다

###유효성검사
- 미들웨어를 통한 유효성검사
- 컨트롤러의 ValidatesRequest Trait를 사용하는방법 //부모클래스인 컨트롤러 클래스에 트레이트 use로 호출하고있음
- 폼리퀘스트를 사용
- 수동 Validator생성 ?

*트레이트는 컨트롤러의 메소드 안에 추가되는 개념, 컨트롤러가 상속받아서 내장하고 있기 때문에 ```this->validate()```로 사용가능

>유효성 검사규칙 : https://laravel.kr/docs/5.4/validation#available-validation-rules



##TIP
php artisan storage:link
public/storage생김 외부에서 접근가능한 디렉토리

파사드는
\DB 만써도되고
\aaaa\ssss\ddd\DB 다써도된

form url에 파라미터 전달
```{{ method_field('delet')}}``` request method set
route('urlKey('user.destory')','[key('id')=>val($user->id)]');
##Question
