<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>

## About Laravel

- Laravel is a web application framework with expressive, elegant syntax. We believe development must be an enjoyable and creative experience to be truly fulfilling. Laravel takes the pain out of development by easing common tasks used in many web projects.

- JWT stands for JSON Web Token, it is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. JWT is commonly used for Authorization, Information Exchange, etc.
- An API (application programming interface) is simply a way of communication between two or more computer programs.


- INSTALL LARAVEL
````
composer create-project "laravel/laravel:^11.0" jwt-auth
````
- By default, laravel 11 API route is not enabled in laravel 11. We will enable the API using the following command:
````
php artisan install:api
````
- Now, if user is not authenticate then exception will call and we will return json response. so, let's update app.php file.<b>bootstrap/app.php</b>

````
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        //
    })
   ->withExceptions(function (Exceptions $exceptions) {
        $exceptions->render(function (AuthenticationException $e, Request $request) {
            if ($request->is('api/*')) {
                return response()->json([
                    'message' => $e->getMessage(),
                ], 401);
            }
        });
    })->create();
````
- Now we will install php-open-source-saver/jwt-auth composer package.
````
composer require php-open-source-saver/jwt-auth
````
- Now, publish the package config file:
````
php artisan vendor:publish --provider="PHPOpenSourceSaver\JWTAuth\Providers\LaravelServiceProvider"
````
- it will create file in config\jwt.php

- Next, generate a secret key. This will add JWT config values on .env file:
````
php artisan jwt:secret
````
- now, we will update auth guard config/auth.php file.

````
  'defaults' => [
        'guard' => 'api',
        'passwords' => 'users',
    ],
      'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
         ],
      ],
````
- In the model, we implement first the Tymon\JWTAuth\Contracts\JWTSubject contract on the User Model and implement the getJWTIdentifier() and getJWTCustomClaims() methods.<b>app/Models/User.php</b>

````
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use PHPOpenSourceSaver\JWTAuth\Contracts\JWTSubject; 

class User extends Authenticatable implements JWTSubject
{
    use HasFactory, Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var array
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * Get the attributes that should be cast.
     *
     * @return array
     */
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }

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
````
- In this step, we will create API routes. Laravel provides the api.php file for writing web service routes. So, let's add a new route to that file.<b>routes/api.php</b>

````
<?php

use Illuminate\Support\Facades\Route;

use App\Http\Controllers\API\AuthController;
 
Route::group([
    'middleware' => 'api',
    'prefix' => 'auth'
], function ($router) {
    Route::post('/register', [AuthController::class, 'register']);
    Route::post('/login', [AuthController::class, 'login']);
    Route::post('/logout', [AuthController::class, 'logout'])->middleware('auth:api');
    Route::post('/refresh', [AuthController::class, 'refresh'])->middleware('auth:api');
    Route::post('/profile', [AuthController::class, 'profile'])->middleware('auth:api');
});
````
- In the next step, we've created a new controller called BaseController and AuthController. I created a new folder named "API" in the Controllers folder because we'll have separate controllers for APIs. So, let's create both controllers:<b>app/Http/Controllers/API/BaseController.php</b>

````
 php artisan make:controller API/BaseController
````
````
<?php
 
namespace App\Http\Controllers\API;
 
use Illuminate\Http\Request;
use App\Http\Controllers\Controller as Controller;
 
class BaseController extends Controller
{
    /**
     * success response method.
     *
     * @return \Illuminate\Http\Response
     */
    public function sendResponse($result, $message)
    {
        $response = [
            'success' => true,
            'data'    => $result,
            'message' => $message,
        ];
 
        return response()->json($response, 200);
    }
 
    /**
     * return error response.
     *
     * @return \Illuminate\Http\Response
     */
    public function sendError($error, $errorMessages = [], $code = 404)
    {
        $response = [
            'success' => false,
            'message' => $error,
        ];
 
        if(!empty($errorMessages)){
            $response['data'] = $errorMessages;
        }
 
        return response()->json($response, $code);
    }
}
````

app/Http/Controllers/API/AuthController.php
````
<?php

namespace App\Http\Controllers\API;
  
use App\Http\Controllers\API\BaseController as BaseController;
use App\Models\User;
use Validator;
use Illuminate\Http\Request;
  
class AuthController extends BaseController
{
 
    /**
     * Register a User.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function register(Request $request) {

        $validator = Validator::make($request->all(), [
            'name' => 'required',
            'email' => 'required|email',
            'password' => 'required',
            'c_password' => 'required|same:password',
        ]);
     
        if($validator->fails()){
            return $this->sendError('Validation Error.', $validator->errors());       
        }
     
        $input = $request->all();
        $input['password'] = bcrypt($input['password']);
        $user = User::create($input);
        $success['user'] =  $user;
   
        return $this->sendResponse($success, 'User register successfully.');
    }
  
  
    /**
     * Get a JWT via given credentials.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function login()
    {
        $credentials = request(['email', 'password']);
  
        if (! $token = auth()->attempt($credentials)) {
            return $this->sendError('Unauthorised.', ['error'=>'Unauthorised']);
        }
  
        $success = $this->respondWithToken($token);
   
        return $this->sendResponse($success, 'User login successfully.');
    }
  
    /**
     * Get the authenticated User.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function profile()
    {
        $success = auth()->user();
   
        return $this->sendResponse($success, 'Refresh token return successfully.');
    }
  
    /**
     * Log the user out (Invalidate the token).
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function logout()
    {
        auth()->logout();
        
        return $this->sendResponse([], 'Successfully logged out.');
    }
  
    /**
     * Refresh a token.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function refresh()
    {
        $success = $this->respondWithToken(auth()->refresh());
   
        return $this->sendResponse($success, 'Refresh token return successfully.');
    }
  
    /**
     * Get the token array structure.
     *
     * @param  string $token
     *
     * @return \Illuminate\Http\JsonResponse
     */
    protected function respondWithToken($token)
    {
        return [
            'access_token' => $token,
            'token_type' => 'bearer',
            'expires_in' => auth()->factory()->getTTL() * 60
        ];
    }
}
````
- Now Run migration
````
php artisan migrate
````

- All the required steps have been done, now you have to type the given below command and hit enter to run the Laravel app:
````
php artisan serve
````





