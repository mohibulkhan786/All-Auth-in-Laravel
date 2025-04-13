<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>


### About Laravel

- Laravel is a web application framework with expressive, elegant syntax. We believe development must be an enjoyable and creative experience to be truly fulfilling. Laravel takes the pain out of development by easing common tasks used in many web projects.

### What is API?

- An API (application programming interface) is simply a way of communication between two or more computer programs.

### What is Laravel Passport?

- Laravel Passport is a tool for adding secure authentication to web applications. It helps developers set up authentication using APIs quickly and easily. Passport generates API tokens that users can use to access protected resources.

### Install Laravel

````
composer create-project "laravel/laravel:^11.0" passport-auth
````
- In Laravel 11, by default, we don't have an api.php route file. So, you just need to run the following command to install passport with api.php file.

````
php artisan install:api --passport
````
````
php artisan passport:client --personal
````


- Now we have to configure three places: the model, the service provider, and the auth config file. So, you just need to follow the changes in those files.
- In the model, we added the HasApiTokens class of Passport.
- In the auth.php file, we added API auth configuration.<b>app/Models/User.php</b>

````
<?php

namespace App\Models;

// use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    /** @use HasFactory<\Database\Factories\UserFactory> */
    use HasFactory, Notifiable, HasApiTokens;

    /**
     * The attributes that are mass assignable.
     *
     * @var list<string>
     */
    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    /**
     * The attributes that should be hidden for serialization.
     *
     * @var list<string>
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }
}
````
- config/auth.php

````
<?php

return [
    
    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],
    
]
````

- Next, we need to create a migration for the posts table using the Laravel 11 php artisan command, so first, fire the command below:
````
php artisan make:migration create_products_table
````
- Database\migrations\2025_04_13_162101_create_products_table.php

````
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('detail');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
````
````
php artisan migrate
````
- After creating the "products" table, you should create a Product model for products. So, first create a file in this path app/Models/Product.php and put the following content in it:<b>app/Models/Product.php</b>

````
php artisan make:model Product
````

````
<?php
  
namespace App\Models;
  
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
  
class Product extends Model
{
    use HasFactory;
  
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'detail'
    ];
}
````
- In this step, we will create API routes. Laravel provides the api.php file for writing web service routes. So, let's add a new route to that file.
````

<?php
  
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
  
use App\Http\Controllers\API\RegisterController;
use App\Http\Controllers\API\ProductController;
 
  
Route::post('register', [RegisterController::class, 'register']);
Route::post('login', [RegisterController::class, 'login']);
     
Route::middleware('auth:api')->group( function () {
    Route::resource('products', ProductController::class);
});
````

- Now, we've created a new controller called BaseController, ProductController, and RegisterController. I created a new folder named "API" in the Controllers folder because we'll have separate controllers for APIs. So, let's create both controllers:

````
php artisan make:controller API/BaseController
````
- app/Http/Controllers/API/BaseController.php

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

````
php artisan make:controller API/RegisterController
````
- app/Http/Controllers/API/RegisterController.php

````
<?php
     
namespace App\Http\Controllers\API;
     
use Illuminate\Http\Request;
use App\Http\Controllers\API\BaseController as BaseController;
use App\Models\User;
use Illuminate\Support\Facades\Auth;
use Validator;
use Illuminate\Http\JsonResponse;
     
class RegisterController extends BaseController
{
    /**
     * Register api
     *
     * @return \Illuminate\Http\Response
     */
    public function register(Request $request): JsonResponse
    {
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
        $success['token'] =  $user->createToken('MyApp')->accessToken;
        $success['name'] =  $user->name;
   
        return $this->sendResponse($success, 'User register successfully.');
    }
     
    /**
     * Login api
     *
     * @return \Illuminate\Http\Response
     */
    public function login(Request $request): JsonResponse
    {
        if(Auth::attempt(['email' => $request->email, 'password' => $request->password])){ 
            $user = Auth::user(); 
            $success['token'] =  $user->createToken('MyApp')-> accessToken; 
            $success['name'] =  $user->name;
   
            return $this->sendResponse($success, 'User login successfully.');
        } 
        else{ 
            return $this->sendError('Unauthorised.', ['error'=>'Unauthorised']);
        } 
    }
}
````

````
php artisan make:controller API/ProductController
````
- app/Http/Controllers/API/ProductController.php
````
<?php
       
namespace App\Http\Controllers\API;
       
use Illuminate\Http\Request;
use App\Http\Controllers\API\BaseController as BaseController;
use App\Models\Product;
use Validator;
use App\Http\Resources\ProductResource;
use Illuminate\Http\JsonResponse;
       
class ProductController extends BaseController
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index(): JsonResponse
    {
        $products = Product::all();
        
        return $this->sendResponse(ProductResource::collection($products), 'Products retrieved successfully.');
    }

    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request): JsonResponse
    {
        $input = $request->all();
       
        $validator = Validator::make($input, [
            'name' => 'required',
            'detail' => 'required'
        ]);
       
        if($validator->fails()){
            return $this->sendError('Validation Error.', $validator->errors());       
        }
       
        $product = Product::create($input);
       
        return $this->sendResponse(new ProductResource($product), 'Product created successfully.');
    } 
     
    /**
     * Display the specified resource.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function show($id): JsonResponse
    {
        $product = Product::find($id);
      
        if (is_null($product)) {
            return $this->sendError('Product not found.');
        }
       
        return $this->sendResponse(new ProductResource($product), 'Product retrieved successfully.');
    }
      
    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Product $product): JsonResponse
    {
        $input = $request->all();
       
        $validator = Validator::make($input, [
            'name' => 'required',
            'detail' => 'required'
        ]);
       
        if($validator->fails()){
            return $this->sendError('Validation Error.', $validator->errors());       
        }
       
        $product->name = $input['name'];
        $product->detail = $input['detail'];
        $product->save();
       
        return $this->sendResponse(new ProductResource($product), 'Product updated successfully.');
    }
     
    /**
     * Remove the specified resource from storage.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function destroy(Product $product): JsonResponse
    {
        $product->delete();
       
        return $this->sendResponse([], 'Product deleted successfully.');
    }
}
````

- This is a very important step in creating a REST API in Laravel 11.
- You can use Eloquent API resources with the API. It will help you to maintain the same response layout of your model object. 
- We used it in the ProductController file. Now, we have to create it using the following command:
````
php artisan make:resource ProductResource
````
- app/Http/Resources/ProductResource.php

````
<?php
  
namespace App\Http\Resources;
  
use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;
  
class ProductResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @return array
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'detail' => $this->detail,
            'created_at' => $this->created_at->format('d/m/Y'),
            'updated_at' => $this->updated_at->format('d/m/Y'),
        ];
    }
}
````
- All the required steps have been done, now you have to type the given below command and hit enter to run the Laravel app:

````
php artisan serve
````
- Register API: Verb:GET, URL:http://localhost:8000/api/register

- ![Register page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/Passport-Auth-Using-Rest-Api/demo-img/register-page.png)

- Login API: Verb:GET, URL:http://localhost:8000/api/login
- ![Login page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/Passport-Auth-Using-Rest-Api/demo-img/login-page.png)
- Product List API: Verb:GET, URL:http://localhost:8000/api/products
- ![product page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/Passport-Auth-Using-Rest-Api/demo-img/get-product.png)

- Product Create API: Verb:POST, URL:http://localhost:8000/api/products
- ![add product page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/Passport-Auth-Using-Rest-Api/demo-img/add-product-page.png)

- Product Show API: Verb:GET, URL:http://localhost:8000/api/products/{id}
- ![get product page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/Passport-Auth-Using-Rest-Api/demo-img/get-product-by-id.png)


- Product Update API: Verb:PUT, URL:http://localhost:8000/api/products/{id}
- ![Update product page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/Passport-Auth-Using-Rest-Api/demo-img/update-product.png)

- Product Delete API: Verb:DELETE, URL:http://localhost:8000/api/products/{id}
- ![Delete product page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/Passport-Auth-Using-Rest-Api/demo-img/delete-product.png)







