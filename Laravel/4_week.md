#04 :: 라라벨 스터디

##Study
###soft delete
실제 테이블에서 제거하지 않고 지우는척
마이그레이션으로 해당 테이블 스키마에 소프트 딜리트용 테이블 추가해주고
스키마에 해당하는 모델파일에
use Illuminate\Database\Eloquent\SoftDeletes;
use SoftDeletes;
protected $dates = ['deleted_at'];

artisan make:migration add_deleted_at_to_users_table --table=users
$table->softDeletes();
//소프트 딜리트는 아래처럼 칼럼 추가안해주고 위에처럼하면 자동으로 'deleted_at' 칼럼이 추가됨
//$table->timestamp('deleted_at');


소프트 딜리트하더라도 컨ㅌ롤러에서 아래처럼 쿼리빌더로 하면안됨
$users = \DB::table('users')->get(); 이런식으로하면 소프트딜리트 적용이안됨,,
모델에 적용해줬으니 모델을 이용해 가져와야함

그래서 컨트롤러에서는 모델을 통해 db다루는게 좋음

소프트 딜리트를 하려면 해당모델의 protected $dates = 에 소프트딜리트용 칼럼을 등록해 줘야 하나요?
>소프트딜리트는 `deleted_at`이란 날짜 컬럼을 만들어 놓고 값이 있으면 "이건 지운거구나"라고 판단하는 식으로 작동합니다.

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class AddDeletedAtToUsersTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->softDeletes();
//            $table->timestamp('deleted_at');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('deleted_at');
        });
    }
}

```
```php
<?php

namespace App;

use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;
    use SoftDeletes;

    // 문자열만 들어가고
    // 여기등록하면 카본인스턴스로 변경하여 꺼내서 객체타임으로 사용할 수 있게됨
    // 안하면 들어갈때 나올때 둘다 날짜
    // 하면 들어갈때 날짜 나올때 카본인스턴스
    protected $dates = ['deleted_at'];

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'email', 'password',
    ];

    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password', 'remember_token',
    ];
}
```
###권한 처리 (policy)
artisan make:policy UserPolicy
App\Policies\에 ```UserPolicy```클래스생성
https://laravel.kr/docs/5.4/authorization#via-middleware
```php
<?php

namespace App\Policies;

use App\User;
use Illuminate\Auth\Access\HandlesAuthorization;

class UserPolicy
{
    use HandlesAuthorization;

    /**
     * Create a new policy instance.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    public function delete(User $currentUser, User $user)
    {
        return $currentUser->id === $user->id;
    }
}
```
// 인증프로바이더에 생성한 UserPolicy 등록
```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Gate;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        'App\Model' => 'App\Policies\ModelPolicy',
        'App\User' => 'App\Policies\UserPolicy', // 인증프로바이더에 생성한 UserPolicy 등록
    ];

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        //
    }
}
```

컨트롤러에서 인증프로바이더에 등록한 유저폴리시를 사용하는 코드 일부
```php
<?php
public function destroy(\App\User $user)
{
   // $loginUser = \Auth::user(); // app.php 알리아스에 등록된 파스드 호출
   // $loginUser = Auth::user(); //use Illuminate\Support\Facades\Auth; 해당파일 산단에 추가하여 이방식 사용

   // if($loginUser->id !== $user->id) {
   //     return back();
   // }

   // $this->authorize('delete', $user); // 첫번째 파라미터 문자열이 폴리시의 메소드명이다.

   $user->delete();

   return redirect('/');
}
```

view에서는
```php
@can('delete', $user)
<form method="post" action="{{ route('user.destroy', ['user' => $user->id]) }}">
    {{ csrf_field() }}
    {{ method_field('delete') }}
    <input type="submit" class="btn btn-default" value="탈퇴하기">
</form>
@endcan
```

###ORM Relation
모델파일에 관계맺어줄 아이의 메소드를 만들어서 해당 메소드 리턴값으로
관계맺어줄 모델인스턴스를 반환하여 객체처럼 사용가능
제이쿼리의 객체 extends 하듯이
블레이드에서 쓸때는 메소드 호출하듯이 사용가능


로우쿼리
=>배열
엘로퀀트, 쿼리빌드
=>php모델 객체로 나와서 프로퍼티 접근가능

쿼리가 오백만개, 천만개 이럴때는 orm릴레이션 맺어서 orm내부처리 join보다는
쿼리두번날려서 애플리케이션 단에서 조인처리가 훨씬 빠름

orm도 내부적으로 콜렉션 루프돌면서 처리함
```php
<?php
{{ $post->user->name }}
```
###ORM Eager로딩
포스트의 댓글을 표시하는 것이 대표적인 사례, 매 댓글 마다 작성자 정보를 데이터베이스에 요청하는 대신 미리 로드하여 쿼리 수를 줄인다.
이거로딩을 안하면 뷰에서 쿼리가 발생하므로 아래같은 경우 최악이다.
매번 where절이 걸린 쿼리를 날린다. 디버그바로 쿼리수를 확인할 수 있다.
```php
@forelse($post->comments as $comment)
  <strong>작성자 : {{ $comment->user->name }}</strong>
  <p>ㄴ {{ $comment->content }}</p>
@empty
  <p>댓글이 없습니다 </p>
@endforelse
```

무조건 이거로딩을 사용하여 쿼리를 아끼자


### Seeder
```artisan make:seeder UserTableSeeder```
database/seeds/에 생성됨
```php
<?php

use Illuminate\Database\Seeder;

class UserTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run(Faker\Generator $faker)
    {
        \App\User::create([
            'name' => $faker->name,
            'email' => $faker->unique()->safeEmail,
            'password' => bcrypt('secret'),
            'remember_token' => str_random(10),
        ]);
    }
}

```
```artisan db:seed --class=UserTableSeeder```

###모델 팩토리
위의 시더파일에서 생성하는 부분만 가져와서 시더에 적용

##TIP
- __sadf__
artisan cache clear ??

- __클래스 안에서 ```use키워드``` 사용
트레이트를 사용하겠다.
SoftDeletes가 가장 트레이트를 잘 활용한 예



##Question
1. artisan cache clear ??
>sdfasd

2. orm join이 원하는 컬럼값만 꺼내올 수 있나? 꺼내오는게 좋나 그냥 다 가져와도 성능이슈 없나??

다대다 관계
태그같은거

3. 다대다 템플릿에서 출력사용시 템플릿 파싱될때 쿼리를 날리나??
