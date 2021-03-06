[![Latest Version on Packagist][ico-version]][link-packagist]
[![Build Status][ico-travis]][link-travis]
[![Coverage Status][ico-scrutinizer]][link-scrutinizer]
[![Quality Score][ico-code-quality]][link-code-quality]
[![Total Downloads][ico-downloads]][link-downloads]

# Laravel and Non-Laravel Library To Connect to Incomings.io Service

![logo](https://incomings.io/images/logo_wide.jpeg)

Sign up for the service https://incomings.io

Then setup and start watching your processes come in one place instead of 5 plus places!

![watching](https://incomings.io/images/all_the_places.png)

Docs below and at https://incomings.io/help

## Install

Tested on Laravel 4.2 and 5.x more platforms to be tested soon.

Composer install

~~~
composer require alfred-nutile-inc/incomings-client:">=1.0"
~~~

Add to app.php

~~~
'AlfredNutileInc\Incomings\IncomingsServiceProvider',
~~~

NOTE: If you are using Lumen, instead of the above you need to enable the provider in bootstrap/app.php like this:
~~~
/*
|--------------------------------------------------------------------------
| Register Service Providers
|--------------------------------------------------------------------------
|
| Here we will register all of the application's service providers which
| are used to bind services into the container. Service providers are
| totally optional, so you are not required to uncomment this line.
|
*/

$app->register('AlfredNutileInc\Incomings\IncomingsServiceProvider');
~~~

Set in your .env

```
INCOMINGS_URL=https://incomings.io

INCOMINGS_TOKEN=token_of_project

```

## Send Data to the Service

### URL

This is the most simple helper. Each project gets one

>@TODO fix broken image need to find the right one
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
NOTE: If you're using Lumen, make sure to enable facades in bootstrap/app.php with `$app->withFacades();`


Also see Laravel Docs for failed Queue [https://laravel.com/docs/5.2/queues](https://laravel.com/docs/5.2/queues)

For example I can register with my `AppServiceProvider`

```

        Queue::failing(function (JobFailed $event) {
            $message = sprintf("Connection %s, Job %s, Exception %s %s %s",
                    $event->connectionName, implode("\n", $event->data), $event->job->getRawBody()
                );
            $data = ['title' => 'Failed Queue From FooBar', 'message' =>

                json_encode($message, JSON_PRETTY_PRINT)
            ];

            Incomings::send($data);
        });
```
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

### Laravel Exceptions

Just edit your `app/Exceptions/Handler.php` so it uses Incomings Exception handler

 Before

```php
     <?php

     namespace App\Exceptions;

     use Exception;
     use Symfony\Component\HttpKernel\Exception\HttpException;
     use IncomingsExceptionHandler as ExceptionHandler;

     class Handler extends ExceptionHandler
     {

```

 After

```php
    <?php

    namespace App\Exceptions;

    use Exception;
    use Symfony\Component\HttpKernel\Exception\HttpException;
    use AlfredNutileInc\Incomings\IncomingsExceptionHandler as ExceptionHandler;

    class Handler extends ExceptionHandler
    {
```
If you are using Lumen, you will need to use the `IncomingsExceptionHandlerForLumen` instead, like so:
```php
    <?php

    namespace App\Exceptions;

    use Exception;
    use Symfony\Component\HttpKernel\Exception\HttpException;
    use AlfredNutileInc\Incomings\IncomingsExceptionHandlerForLumen as ExceptionHandler;

    class Handler extends ExceptionHandler
    {
```


Then as seen in this route it will send a message first to Incomings.io

```php

Route::get('/example_exception', function() {

    throw new \Exception("Yo Incomings!!!");

});
```

Will send a message like

![](https://dl.dropboxusercontent.com/s/qg0x11cxs9qjtr5/incomings_exception.png?dl=0)


### Bugsnag Too

If you are using a service like BugSnag just follow their directions so your `app/Exceptions/Handler.php` would then look like this.

```php
<?php namespace App\Exceptions;

use Exception;
use Bugsnag\BugsnagLaravel\BugsnagExceptionHandler as ExceptionHandler;
use AlfredNutileInc\Incomings\IncomingsFacade as Incomings;

class Handler extends ExceptionHandler
{

    protected $dontReport = [
        HttpException::class,
    ];

    public function report(Exception $e)
    {
        $data = [
            'title' => 'Application Exception Error',
            'message' => sprintf(
                "Error Filename %s \n on line %d \n with message %s \n with Code %s",
                $e->getFile(),
                $e->getLine(),
                $e->getMessage(),
                $e->getCode()
            ),
        ];
        Incomings::send($data);
        
        return parent::report($e);
    }

}
```

### Filter for Laravel 4.2

As above plug in your provider

If you are not using DotEnv as I write about here [https://alfrednutile.info/posts/113](https://alfrednutile.info/posts/113)

Then update your `.env.php` to have your tokens and url

~~~
<?php

return array(
    'INCOMINGS_URL' => 'https://post.incomings.io',
    'INCOMINGS_TOKEN' => 'foo-bar-foo'
);
~~~

Then in your route

~~~
Route::get('/', ['before' => 'incomings', function()
{
	return View::make('hello');
}]);
~~~

Finally in your filter file add the following `app/filters.php`

~~~

Route::filter('incomings', function() {

    try
    {
        $incomings = new \AlfredNutileInc\Incomings\IncomingsFilter();
        $incomings->handle(\Illuminate\Support\Facades\Request::instance());
    }
    catch(\Exception $e)
    {
        Log::error(sprintf("Error with Incomings :( %s", $e->getMessage());
    }

});

~~~

This will catch any issues and not mess up your application.

![incomings](https://dl.dropboxusercontent.com/s/qg0x11cxs9qjtr5/incomings_exception.png?dl=0)

### Curl

Here is an example of using Curl. In this case I want to see some info from my server every hour.

~~~
curl -k -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data @status.json https://post.incomings.io/incomings/f4ac705d-5087-3432-8182-334de6726fc5
~~~

Then every hour I get to see the updates to that file. The CronJob would run this as root

~~~
01 * * * * apt-get upgrade -s | grep -i security > /tmp/status.json
03 * * * * curl -k -H "Content-Type: application/json" -H "Accept: application/json" -X POST --data @/tmp/status.json https://post.incomings.io/incomings/foobar
~~~

You can even make a bach command to run this all and gather more data like "Last Run" etc.




### Drupal 8

Coming Soon...

### Drupal 7

Coming Soon...



[ico-version]: https://img.shields.io/packagist/v/alfred-nutile-inc/incomings-client.svg?style=flat-square
[ico-license]: https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square
[ico-travis]: https://img.shields.io/travis/alfred-nutile-inc/incomings-client/master.svg?style=flat-square
[ico-scrutinizer]: https://img.shields.io/scrutinizer/coverage/g/alfred-nutile-inc/incomings-client.svg?style=flat-square
[ico-code-quality]: https://img.shields.io/scrutinizer/g/alfred-nutile-inc/incomings-client.svg?style=flat-square
[ico-downloads]: https://img.shields.io/packagist/dt/alfred-nutile-inc/incomings-client.svg?style=flat-square

[link-packagist]: https://packagist.org/packages/alfred-nutile-inc/incomings-client
[link-travis]: https://travis-ci.org/alfred-nutile-inc/incomings-client
[link-scrutinizer]: https://scrutinizer-ci.com/g/alfred-nutile-inc/incomings-client/code-structure
[link-code-quality]: https://scrutinizer-ci.com/g/alfred-nutile-inc/incomings-client
[link-downloads]: https://packagist.org/packages/alfred-nutile-inc/incomings-client
[link-author]: https://github.com/alnutile
[link-contributors]: ../../contributors

