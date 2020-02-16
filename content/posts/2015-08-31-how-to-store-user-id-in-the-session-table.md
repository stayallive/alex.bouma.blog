+++
author = "Alex Bouma"
date = 2015-08-31T21:58:00+01:00
description = ""
draft = false
cover = "/img/ghost/laravel.jpg"
slug = "how-to-store-user-id-in-the-session-table"
tags = ["laravel", "php"]
title = "How to store user id in the Laravel session table"
aliases = ["/how-to-store-user-id-in-the-session-table"]
+++

Sometimes you want to store the user id with the session so you can purge all sessions or even a single session creating a more secure system for your users, or maybe you want to see how many times a user is logged in. There are many use cases but it's not possible by default... so let's implement this nifty feature :)

> Note: this is not needed in Laravel **5.2** since this comes out of the box

<em>Note: this guide only applies for the `database` session driver, other drivers do not support this.</em>

Let's modify the migration first:
```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateSessionTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create(config('session.table'), function (Blueprint $table) {
            $table->string('id')->unique();
            
            $table->integer('user_id')->unsigned()->nullable()->default(null);
            // You can even add a foreign key constraint if you want :)
            //$table->foreign('user_id')->references('id')->on('users')->onUpdate('CASCADE')->onDelete('CASCADE');
            $table->text('payload');
            
            $table->integer('last_activity');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists(config('session.table'));
    }
}
```

We add a unsigned integer field representing our user to the default fields, we allow null since users that are not logged in have no user id. Run `php artisan migrate` to run the migration.

Next we need to add our custom driver since we need to override a method on the database session driver provided by Laravel.

```php
<?php

namespace App\Session;

class DatabaseSessionHandler extends \Illuminate\Session\DatabaseSessionHandler
{
    /**
     * {@inheritDoc}
     */
    public function write($sessionId, $data)
    {
        $user_id = (auth()->check()) ? auth()->user()->id : null;

        if ($this->exists) {
            $this->getQuery()->where('id', $sessionId)->update([
                'payload' => base64_encode($data), 'last_activity' => time(), 'user_id' => $user_id,
            ]);
        } else {
            $this->getQuery()->insert([
                'id' => $sessionId, 'payload' => base64_encode($data), 'last_activity' => time(), 'user_id' => $user_id,
            ]);
        }

        $this->exists = true;
    }
}
```

We also need a service provider to register our new driver with Laravel, I used a different driver name (`app.database`) but you could also override the default database driver (called `database`) if you want.

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\Session\DatabaseSessionHandler;

class SessionServiceProvider extends ServiceProvider
{
    /**
     * Register the service provider.
     *
     * @return void
     */
    public function register()
    {
        $this->app->session->extend('app.database', function ($app) {
            $lifetime   = $this->app->config->get('session.lifetime');
            $table      = $this->app->config->get('session.table');
            $connection = $app->app->db->connection($this->app->config->get('session.connection'));

            return new DatabaseSessionHandler($connection, $table, $lifetime, $this->app);
        });
    }
}
```

Add the service provider to the providers array in your `config/app.php` and change the driver name to `app.database` in your `config/session.php` config file.

If you now browse around your app and log in and out you will see the session table add and remove the user id accordingly.

Now you could add a Eloquent model for the sessions table and add the relation to your users model and query away on the sessions :)
