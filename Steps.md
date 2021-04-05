## Pasos para utilizar Laravel 7.* + Php 7.3.21 + Vue 

### Instalación 
Se deben ejecutar los siguientes comandos en el orden correspondiente:
1. composer create-project --prefer-dist laravel/laravel nombre_del_proyecto "7.*"
2. cd nombre_del_proyecto
3. composer require laravel/ui:^2.4
4. php artisan ui vue --auth
5. npm i

### Solución de problemas de la versión
Laravel 7 presenta una serie de errores menores que se deben solucionar de la siguiente forma:
1. Problema con los estilos
   - Ir a la ruta resources/views/layouts/app.blade.php y los estilos de [bootstrap](https://getbootstrap.com/docs/5.0/getting-started/introduction/)

```
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">

```
2. Problema con largos de datos
   - Ir al archivo ubicado en app/Provider/AppServiceProvider.php y dejarlo de la siguiente forma
```
<?php

namespace App\Providers;
use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Schema;

class AppServiceProvider extends ServiceProvider
{
    public function register()
    {
        //
    }
    public function boot()
    {
        Schema::defaultStringLength(191);
    }
}

```

### Configuración de la base de datos
Para realizar la configuración con la base de datos (local) se debe configurar en primera instancia el archivo .env presente en la raiz del proyecto, en este se debe modificar según corresponda los siguientes parámetros:
- DB_CONNECTION -> el tipo de conexión
- DB_HOST -> dirección de la base de datos (usualmente 127.0.0.1)
- DB_PORT -> puerto utilizado para la base de datos (usualmente 3306)
- DB_DATABASE -> el nombre de la base de datos
- DB_USERNAME -> nombre de usuario (por defecto mysql utiliza 'root')
- DB_PASSWORD -> contraseña (usualmente se deja vacia)

### Creación de modelos y migraciones
Ejecutar en consola el siguiente comando:
```
php artisan make:model Nombre -m
```
Donde el nombre debe estar capitalizado y en singular. El parámetro -m sirve para la creación automática de la migración, la cual se utilizará más adelante con la interacción con la base de datos.

Ahora se procede a modificar la migración creada anteriormente, esta se encuentra en la ruta database/migrations/fecha_id_create_nombre_table.php, donde nombre corresponda al plural en minúscula  del nombre del modelo creado en el paso anterior. en este archivo, en la función up(), dentro de Schemma::create() se agregan los nombres y tipos de valores que tendrá la tabla, por ejemplo:
```
Schema::create('notas', function (Blueprint $table) {
    $table->bigIncrements('id');
    $table->string('nombre');
    $table->text('descripcion');
    $table->bigInteger('user_id')->unsigned();
    $table->foreign('user_id')->references('id')->on('users');
    $table->timestamps();
});
```
### Creación de tablas
Una vez creados los modelos y las migraciones solo se debe ejecutar el siguiente comando para su creación:
```
php artisan migrate
```

### Creación de controladores
Los controladores nos ayudaran a la interacción entre lo visual y la base de datos mediante el CRUD.
Para la creación de este se debe ejcutar la siguiente linea de código:
```
php artisan make:controller NombreController --resource
```
Se suguiere reemplazar NombreController con el nombre del modelo creado en los pasos anteriores seguido de la palabra Controller.

Luego de la creación del controlador se debe configurar el archivo de rutas para su correcto funcionamiento. Se debe abrir el archivo web.php ubicado en resources/routes y agregas bajo Auth::routes();
```
Route::resource('/nombre_ruta', 'NombreController')->middleware('auth');
```

### Ejecución del proyecto
Para la ejecución de un proyecto de laravel que utiliza vue se deben ejecutar los siguientes comando en diferentes consolas, los cuales trabajaran paralelamente
- php artisan serve  -> para la ejecución de laravel
- npm run watch  -> para la compilación automática de archivos .vue

Cada vez que realizamos un cambio nos aparece un mensaje de Laravel, este lo puedes omitir agregando el siguiente código al archivo: webpack.mix.js
```
mix.disableNotifications();
```