# Laravel and Non-Laravel Library To Connect to Incomings Service

![logo](https://dl.dropboxusercontent.com/s/fz0zwwsxawlyj8t/logo_wide.jpeg?dl=0)

Sign up for the service https://incomings.io

Then setup and start watching your processes come in.

More docs soon...

## Install

Composer install

~~~
composer require alfred-nutile-inc/incomings-client
~~~

Add to app.php

~~~
'AlfredNutileInc\Incomings\IncomingsServiceProvider',
~~~

Set in your .env

INCOMINGS_URL=http://dev.incomings.io

INCOMINGS_TOKEN=token_of_project


## Send Data to the Service

### URL

This is the most simple helper. Each project gets one

![url](https://dl.dropboxusercontent.com/s/7tw1cgu5wvlgz10/Screenshot%202015-06-25%2019.22.04.png?dl=0)

So you can for example use that on Iron.io as a PUSH queue route since you can have more than one.

Or even on your server setup a cron job to post every minute your server resource status or security status.

Example Iron.io

![iron](https://dl.dropboxusercontent.com/s/h3q4ojcbmg22ts6/iron_example.png?dl=0)

### Laravel Facade

Say you are about to send off to a queue

~~~
Queue::push("foo", $data);
~~~

Now try

~~~
$data = ['title' => 'Foo Bar', 'message' => [1,2,3]]

Incomings::send($data);

Queue::push("foo", $data);
~~~

For the above Facade to work you might have to add

~~~
use AlfredNutileInc\Incomings\IncomingsFacade as Incomings;
~~~

### Logger

This setup will allow you to use Log::info("Some Message") and all the other Log methods as normal.

All you need to do at the top of your Class is to set use as follow

~~~
use AlfredNutileInc\Incomings\Log;
~~~

From there on your log messages go to Incomings then to Logger

Even better you now can/should do this

~~~
    $send = [
        'title' => 'foo',
        'message' => "bar",
    ];
    Log::info($send);
~~~

The IncomingLogger will pass this array to Incomings.io giving your incoming more context and then it will
just pass the message to Log as normal. So you could even do.


~~~
    $send = [
        'title' => 'foo',
        'message' => print_r($some_array_payload, 1),
    ];
~~~

Like sometimes we do in Log::info as we are watching for non string based info in the logs. Or

~~~
    $send = [
        'title' => 'foo',
        'message' => json_encode($some_array_payload, JSON_PRETTY_PRINT),
    ];
~~~

For nicer looking data.


### MiddleWare

Coming Soon...

#### As a short name

~~~
    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'incomings' => \AlfredNutileInc\Incomings\IncomingsMiddleWare::class
    ];
~~~

Then plug it in

~~~
Route::get('foobar', ['middleware' => 'incomings', function() {

    return "Send to incomings!";

}]);
~~~

Then data coming in via POST, GET, etc will be sent to Incomings for a sense of is the data coming into my system correctly etc.

You can pass a title as well

~~~
Route::get('foobar', ['middleware' => 'incomings:My Title', function() {

    return "Send to incomings!";

}]);
~~~

### Filter for Laravel 4.2

Coming Soon...

### Drupal 8

Coming Soon...

### Drupal 7

Coming Soon...
