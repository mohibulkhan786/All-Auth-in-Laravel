<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>

## About Laravel

- Laravel is a web application framework with expressive, elegant syntax. We believe development must be an enjoyable and creative experience to be truly fulfilling. Laravel takes the pain out of development by easing common tasks used in many web projects.
- Laravel Jetstream was designed by Tailwind CSS, and it provides authentication scaffolding using Livewire and Inertia. Laravel Jetstream includes features such as login, registration, email verification, two-factor authentication, session management, API via Laravel Sanctum, and team management.
- INSTALL LARAVEL

````
composer create-project "laravel/laravel:^11.0" jetstream-auth
````
- Now, in this step, we need to use the composer command to install Jetstream, so let's run the below command and install the below library.

````
composer require laravel/jetstream
````

- Now, we need to create authentication using the below command. You can create basic login, register, and email verification. If you want to create team management, then you have to pass the additional parameter. You can see the below commands:
````
php artisan jetstream:install inertia
````
- In Laravel 11 Jetstream, all features are configurable. You can find configuration files named `fortify.php` and `jetstream.php`, where you can enable and disable options for these features.config/fortify.php
````
'features' => [
        Features::registration(),
        Features::resetPasswords(),
        Features::emailVerification(),
        Features::updateProfileInformation(),
        Features::updatePasswords(),
        Features::twoFactorAuthentication(),
    ],

````

- config/jetstream.php
````  
'features' => [
        Features::profilePhotos(),
        Features::api(),
        Features::teams(),
    ],
````

- Now Run the migration command:
````
php artisan migrate
````
- All the required steps have been done, now you have to type the given below command and hit enter to run the Laravel app:
````
php artisan serve
````

- Now, Go to your web browser, type the given URL and view the app output:
````
http://localhost:8000
````

- ![Home page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/Jetstream-Auth/demo-img/home-page.png)
- ![Register page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/Jetstream-Auth/demo-img/register-page.png)
- ![Login page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/Jetstream-Auth/demo-img/login-page.png)
- ![Dashoard page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/Jetstream-Auth/demo-img/dashboard-page.png)



