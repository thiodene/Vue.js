# Vue-Components-Library Package:
This section helps making a successful installation of the Vue-Components-Library to an existing Laravel App.
This tutorial considers every steps starting from laravel/ui.

## Source:
https://vuejs.org/v2/guide/installation.html

## Install Laravel/ui

The Vue scaffolding provided by Laravel is located in the laravel/ui Composer package, which can be installed using Composer:

```composer require laravel/ui:^2.4```

Once the laravel/ui package has been installed, you can install the frontend scaffolding using the ui Artisan command:

```php artisan ui vue```

The additional php artisan ui vue --auth will is not needed as  Laravel-Auth takes care of the Login/Registration parts

Once the Vue scaffolding is installed successfully.
Please run npm install to compile your fresh scaffolding:


```npm install && npm run dev```
