<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>

## About Laravel

- Laravel is a web application framework with expressive, elegant syntax. We believe development must be an enjoyable and creative experience to be truly fulfilling. Laravel takes the pain out of development by easing common tasks used in many web projects.
- Laravel provides a UI package for the easy setup of auth scaffolding. Laravel UI offers simple authentication features, including login, registration, password reset, email verification, and password confirmation, using Bootstrap, React, and Vue.

- INSTALL LARAVEL

````
composer create-project "laravel/laravel:^11.0" scaffolding-auth
````
- Let's run the below command to install the Laravel UI package:

````
composer require laravel/ui
````
- Next, you have to install the Laravel UI package command for creating auth scaffolding using Bootstrap 5. So let's run the below command:
````
php artisan ui bootstrap --auth
````
- Now, let's run the below command to install npm:
````
npm install && npm run build
````
- It will generate CSS and JS min files. Next, run the migration command:
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
- ![Home page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/main/demo-img/home-page.png)
- ![Register page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/main/demo-img/register-page.png)
- ![Login page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/main/demo-img/login-page.png)
- ![Dashboard page](https://github.com/mohibulkhan786/All-Auth-in-Laravel/blob/main/demo-img/dasboard-page.png)
