---
title: Laravel 使用 JWT Restful API
date: 2020-05-15
categories:
- Laravel
- JWT
- Restful API
tags:
- Laravel
- JWT
- Restful API
---
# Laravel 使用 JWT Restful API
>製作者：Ericsiang


###### tags: `Laravel` `JWT` `Restful API`

### 安裝laravel專案

``` php
composer -project --prefer-dist laravel/laravel laravel_jwt
```

###### PS:記得修改.env檔內的DB連線設定

### 安裝及設定JWT認證套件

- Install JWT package
``` php
composer require tymon/jwt-auth:dev-develop
```

- 執行package publish：
``` php
php artisan vendor:publish
```

輸入完上面的指令
選擇 Provider: Tymon\JWTAuth\Providers\LaravelServiceProvider
我這是第9個，因此輸入9
**++成功後會在config中產生一個jwt.php檔案++**

![](https://i.imgur.com/6QmK28B.jpg)

- 產生一組JWT的加密的金鑰
``` php
php artisan jwt:secret
```

- Registering JWT Middleware

建立API用的JWT middleware
打開app/Http/Kernel.php，然後在裡面找到routeMiddleware，加入圖片框選的這行程式碼

![](https://i.imgur.com/wW3leta.jpg)

### API Routes設定(routes\api.php)

```<?php

    Route::post('login', 'ApiController@login');
    Route::post('register', 'ApiController@register');

    Route::group(['middleware' => 'auth.jwt'], function () {
        Route::get('logout', 'ApiController@logout');

        Route::get('tasks', 'TaskController@index');
        Route::get('tasks/{id}', 'TaskController@show');
        Route::post('tasks', 'TaskController@store');
        Route::put('tasks/{id}', 'TaskController@update');
        Route::delete('tasks/{id}', 'TaskController@destroy');
    });


```

### 修改User Model

設定好routes後，將app/User.phpModel加入兩個JWT的function getJWTIdentifier、getJWTCustomClaims

```<?php
namespace App;
+  use Tymon\JWTAuth\Contracts\JWTSubject;(新增)
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;
-  class User extends Authenticatable (原本)
+  class User extends Authenticatable implements JWTSubject(修改後)
{
...
...

     /**
     * Get the identifier that will be stored in the subject claim of the JWT.
     *
     * @return mixed
     */
    public function getJWTIdentifier()
    {
        return $this->getKey();
    }

    /**
     * Return a key value array, containing any custom claims to be added to the JWT.
     *
     * @return array
     */
    public function getJWTCustomClaims()
    {
        return [];
    }
}
```

### 設定Config Auth guard

在 config/auth.php 檔案當中，將認證的部分改為jwt的認證方式
```<?php
'defaults' => [
-   'guard' => 'web',(原本)
+   'guard' => 'api',(修改後)
    'passwords' => 'users',
],
...
'guards' => [
    'api' => [
-       'driver' => 'token',(原本)
+       'driver' => 'jwt',(修改後)
        'provider' => 'users',
-       'hash' => false,(刪除)
    ],
],
```

### 設定API所需要的各項功能

- 創建一個註冊功能用的form request class：
```
php artisan make:request RegistrationFormRequest
```


app\Http\Requests\RegistrationFormRequest.php
用來規定新使用者註冊的規則

```<?php
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class RegistrationFormRequest extends FormRequest
{
    /**
     * 确定是否授權用戶發出此請求
     *
     * @return bool
     */
    public function authorize()
    {
        return true;//這裡要設成true，預設是false
    }

    /**
     * 應用於請求時的驗證規則
     *
     * @return array
     */
    public function rules()
    {
        return [
            'name' => 'required|string',
            'email' => 'required|email|unique:users',
            'password' => 'required|string|min:6|max:10'
        ];
    }
    
    /**
     * 應用於請求不符合驗證規則時，規則錯誤的訊息
     *
     * @return array
     */
    public function messages()
    {
        return [
            'email.required' => 'Email is required!',
            'email.unique'  => 'Email is unique!',
            'name.required' => 'Name is required!',
            'password.required' => 'Password is required!'
        ];
    }
    
     /**
     * 應用於請求不符合驗證規則時，回應json格式
     *
     * @return json
     */
    protected function failedValidation(Validator $validator)
    {
        $errors = (new ValidationException($validator))->errors();
        throw new HttpResponseException(response()->json([
            'success' => false,
            'message' => $errors
        ], 401/*JsonResponse::HTTP_UNPROCESSABLE_ENTITY*/ ));
    }
}
```


- 建立APIController
```
php artisan make:controller APIController
```

app\Http\Controllers\APIController.php
```<?php

namespace App\Http\Controllers;

use JWTAuth;
use App\User;
use Illuminate\Http\Request;
use Tymon\JWTAuth\Exceptions\JWTException;
use App\Http\Requests\RegistrationFormRequest;

class APIController extends Controller
{
      /**
     * @var bool
     */
    public $loginAfterSignUp = true;//註冊後，是否直接執行登入

    /**
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    //登入
    public function login(Request $request)
    {
        $input = $request->only('email', 'password');
        $token = null;

        if (!$token = JWTAuth::attempt($input)) {
            return response()->json([
                'success' => false,
                'message' => 'Invalid Email or Password',
            ], 401);
        }

        return response()->json([
            'success' => true,
            'token' => $token,
        ]);
    }

    /**
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     * @throws \Illuminate\Validation\ValidationException
     */
     //登出
    public function logout(Request $request)
    {
        $this->validate($request, [
            'token' => 'required'
        ]);

        try {
            JWTAuth::invalidate($request->token);

            return response()->json([
                'success' => true,
                'message' => 'User logged out successfully'
            ]);
        } catch (JWTException $exception) {
            return response()->json([
                'success' => false,
                'message' => 'Sorry, the user cannot be logged out'
            ], 500);
        }
    }

    /**
     * @param RegistrationFormRequest $request
     * @return \Illuminate\Http\JsonResponse
     */
     //註冊
    public function register(RegistrationFormRequest $request)
    {
        //dd($validated = $request->validated());

        $input=$request->only('name','email','password'); 
        $input['password']=bcrypt($input['password']);
        $user=User::create($input);

        //註冊後，是否執行登入
        if ($this->loginAfterSignUp) {
            return $this->login($request);
        }

        return response()->json([
            'success'   =>  true,
            'data'      =>  $user
        ], 200);
    }
}

```

- 輸入指令，一次創建 Model、Migration、Controller
```
php artisan make:model Task -mc
```

database\migrations\2020_05_15_031920_create_tasks_table.php
```<?php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateTasksTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->unsignedBigInteger('user_id');
            $table->string('title');
            $table->text('description')->nullable();
            $table->foreign('user_id')
                    ->references('id')
                    ->on('users')
                    ->onDelete('cascade');

            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('tasks');
    }
}
```

建立資料表
```
php artisan migrate
```

- 修改User跟Tasks Model

app\User.php
新增一對多關聯
```<?php
public function tasks()
{
    return $this->hasMany(Task::class);
}
```


app\Task.php
```<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Task extends Model
{
    /**
     * @var string
     */
    protected $table = 'tasks';

    /**
     * @var array
     */
    protected $guarded = [];
}
```

- TaskController建立CRUD

> **以下均經過身份認證取得該用戶下的任務資訊**

> index, 取得所有任務列表
show,    根据 ID 取得該任務
store,   建立任務
update,  根据 ID 更新任務
destroy, 根据 ID 删除該任務



```<?php

namespace App\Http\Controllers;

use JWTAuth;
use App\Task;
use Illuminate\Http\Request;

class TaskController extends Controller
{
    /**
     * @var
     */
    protected $user;

    /**
     * TaskController constructor.
     */
    public function __construct()
    {
        
        $this->user = JWTAuth::parseToken()->authenticate();
        //dd($this->user);
    }

    /**
     * @return mixed
     */
    public function index()
    {
        $tasks = $this->user->tasks()->get(['id','title', 'description'])->toArray();

        if (!$tasks) {
            return response()->json([
                'success' => false,
                'message' => 'Sorry, you don\'t have any task be found.'
            ], 400);
        }
        return $tasks;
    }


     /**
     * @param $id
     * @return \Illuminate\Http\JsonResponse
     */
    public function show($id)
    {
        $task = $this->user->tasks()->find($id);

        if (!$task) {
            return response()->json([
                'success' => false,
                'message' => 'Sorry, task with id ' . $id . ' cannot be found.'
            ], 400);
        }

        return $task;
    }

    /**
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     * @throws \Illuminate\Validation\ValidationException
     */
    public function store(Request $request)
    {
        $this->validate($request, [
            'title' => 'required',
            'description' => 'required',
        ]);

        $input=$request->only('title','description'); 
        $task=$this->user->tasks()->create($input);

        /*
        $task = new Task();
        $task->title = $request->title;
        $task->description = $request->description;
        $this->user->tasks()->save($task)
        */
        if ($task)
            return response()->json([
                'success' => true,
                'task' => $task
            ]);
        else
            return response()->json([
                'success' => false,
                'message' => 'Sorry, task could not be added.'
            ], 500);
    }

    /**
     * @param Request $request
     * @param $id
     * @return \Illuminate\Http\JsonResponse
     */
    public function update(Request $request, $id)
    {
        $task = $this->user->tasks()->find($id);

        if (!$task) {
            return response()->json([
                'success' => false,
                'message' => 'Sorry, task with id ' . $id . ' cannot be found.'
            ], 400);
        }

       // $updated = $task->fill($request->all())->save();
       //dd($request->only('title','description'));
       $updated =$task->fill($request->only('title','description'))->save();
        
        if ($updated) {
            return response()->json([
                'success' => true,
                'message' => 'Update task with id '.$id.' success'
            ]);
        } else {
            return response()->json([
                'success' => false,
                'message' => 'Sorry, task could not be updated.'
            ], 500);
        }
    }

    /**
     * @param $id
     * @return \Illuminate\Http\JsonResponse
     */
    public function destroy($id)
    {
        $task = $this->user->tasks()->find($id);

        if (!$task) {
            return response()->json([
                'success' => false,
                'message' => 'Sorry, task with id ' . $id . ' cannot be found.'
            ], 400);
        }

        if ($task->delete()) {
            return response()->json([
                'success' => true,
                'message' => 'Delete task success',
            ]);
        } else {
            return response()->json([
                'success' => false,
                'message' => 'Task could not be deleted.'
            ], 500);
        }
    }




}

```

- REST API測試
開啟測試server( localhost:8000)：
```
php artisan serve
```

### 各項測試皆使用Postman

~之後再補，哈哈


#### 哇勒  終於搞定 花了蠻長時間 在這記錄一下流程 不然之後一定會忘記  忘記時可以讓我來回追一下

</br>

最後附上谷歌大神上找到的參考網址：
- [在Laravel 6 REST API中應用JSON Web Token (JWT)](https://medium.com/@rommelhong/%E5%9C%A8laravel-6-rest-api%E4%B8%AD%E6%87%89%E7%94%A8json-web-token-jwt-e067e7fc7b1f)
- [[教程] Laravel 中使用 JWT 认证的 Restful API
](https://learnku.com/laravel/t/27760)
- [Laravel 6 Rest API using JWT Authentication
](https://www.larashout.com/laravel-6-jwt-authentication)
- [Laravel 6.0 JWT教學
](https://medium.com/@jungr/laravel-6-0-jwt%E6%95%99%E5%AD%B8-da324e5aa753)
- [Request Validation – Creating REST API in Laravel (Part III)
](http://stacklearning.com/api/rest-api-laravel-request-validation/)




