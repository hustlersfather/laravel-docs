# Upgrade Guide

- [Upgrading To 5.4.0 From 5.3](#upgrade-5.4.0)

<a name="upgrade-5.4.0"></a>
## Upgrading To 5.4.0 From 5.3

#### Estimated Upgrade Time: 1-2 Hours

> {note} We attempt to document every possible breaking change. Since some of these breaking changes are in obscure parts of the framework only a portion of these changes may actually affect your application.

### Updating Dependencies

Update your `laravel/framework` dependency to `5.4.*` in your `composer.json` file.

#### Laravel Cashier

Laravel Cashier is already compatible with Laravel 5.4.

#### Laravel Passport

Laravel Passport `2.0.0` has been released to provide compatibility with Laravel 5.4 and the [Axios](https://github.com/mzabriskie/axios) JavaScript library. If you are upgrading from Laravel 5.3 and using the pre-built Passport Vue components, you should make sure the Axios library is globally available to your application as `axios`.

#### Laravel Scout

Laravel Scout `3.0.0` has been released to provide compatibility with Laravel 5.4.

#### Laravel Socialite

Laravel Socialite `3.0.0` has been released to provide compatibility with Laravel 5.4.

#### Laravel Tinker

In order to continue using the `tinker` Artisan command, you should also install the `laravel/tinker` package:

    composer require laravel/tinker

Once the package has been installed, you should add the `Laravel\Tinker\TinkerServiceProvider::class` to your `config/app.php` configuration file.

#### Guzzle

Laravel 5.4 requires Guzzle 6.0 or greater.

### Authorization

#### The `getPolicyFor` Method

Previous, when calling the `Gate::getPolicyFor($class)` method, an exception was thrown if no policy could be found. Now, the method will return `null` if no policy is found for the given class. If you call this method directly, make sure you refactor your code to check for `null`:

```php
$policy = Gate::getPolicyFor($class);

if ($policy) {
    // code that was previously in the try block
} else {
    // code that was previously in the catch block
}
```

### Bootstrappers

If you are manually overriding the `$bootstrappers` array on your HTTP or Console kernel, you should rename the `DetectEnvironment` entry to `LoadEnvironmentVariables`.

### Broadcasting

#### Channel Model Binding

When defining channel name placeholders in Laravel 5.3, the `*` character is used. In Laravel 5.4, you should define these placeholders using `{foo}` style placeholders, like routes:

    Broadcast::channel('App.User.{userId}', function ($user, $userId) {
        return (int) $user->id === (int) $userId;
    });

### Collections

#### The `every` Method

The behavior of the `every` method has been moved to the `nth` method to match the method name defined by Lodash.

#### The `random` Method

Calling `$collection->random(1)` will now return a new collection instance with one item. Previously, this would return a single object. This method will only return a single object if no arguments are supplied.

### Container

#### Binding Classes With Leading Slashes

Binding classes into the container with leading slashes is no longer supported. This feature required a significant amount of string formatting calls to the be made within the container. Instead, simply register your bindings without a leading slash:

    $container->bind('Class\Name', function () {
        //
    });

    $container->bind(ClassName::class, function () {
        //
    });

#### `make` Method Parameters

The container's `make` method no longer accepts a second array of parameters. This feature typically indicates a code smell. Typically, you can always construct the object in another way that is more intuitive.

#### Resolving Callbacks

The container's `resolving` and `afterResolving` method now must be provided a class name or binding key as the first argument to the method:

    $container->resolving('Class\Name', function ($instance) {
        //
    });

    $container->afterResolving('Class\Name', function ($instance) {
        //
    });

#### `share` Method Removed

The `share` method has been removed from the container. This was a legacy method that has not been documented in several years. If you are using this method, you should begin using the `singleton` method instead:

    $container->singleton('foo', function () {
        return 'foo';
    });

### Database

#### Custom Connections

If you were previously binding a service container binding for a `db.connection.{driver-name}` key in order to resolve a custom database connection instance, you should now use the `DB::resolverFor` method in the `register` method of your `AppServiceProvider`:

    DB::resolverFor('driver-name', function ($connection, $database, $prefix, $config) {
        //
    });

#### Fetch Mode

Laravel no longer includes the ability to customize the PDO "fetch mode" from your configuration files. Instead, `PDO::FETCH_OBJ` is always used. If you will still like to customize the fetch mode for your application you may listen for the new `Illuminate\Database\Events\StatementPrepared` event:

    Event::listen(StatementPrepared::class, function ($event) {
        $event->statement->setFetchMode(...);
    });

### Eloquent

#### Date Casts

The `date` cast now converts the column to a `Carbon` object and calls the `startOfDay` method on the object. If you would like to preserve the time portion of the date, you should use the `datetime` cast.

#### Foreign Key Conventions

If the foreign key is not explicitly specified when defining a relationship, Eloquent will now use the table name and primary key name for the related model to build the foreign key. For the vast majority of applications, this is not a change of behavior. For example:

    public function user()
    {
        return $this->belongsTo(User::class);
    }

Just like previous Laravel releases, this relationship will typically use `user_id` as the foreign key. However, the behavior could be different from previous releases if you are overriding the `getKeyName` method of the `User` model. For example:

    public function getKeyName()
    {
        return 'key';
    }

When this is the case, Laravel will now respect your customization and determine the foreign key column name is `user_key` instead of `user_id`.

#### Has One / Many `createMany`

The `createMany` method of a `hasOne` or `hasMany` relationship now returns a collection object instead of an array.

#### The `hydrate` Method

If you are currently passing a custom connection name to this method, you should now use the `on` method:

    User::on('connection')->hydrate($records);

#### `hydrateRaw` Method

The `Model::hydrateRaw` method has been renamed to `fromQuery`. If you are passing a custom connection name to this method, you should now use the `on` method:

    User::on('connection')->fromQuery('...');

#### The `whereKey` method

The `whereKey($id)` method will now add a "where" clause for the given primary key value. Previously, this would fall into the dynamic "where" clause builder and add a "where" clause for the "key" column. If you used the `whereKey` method to dynamically add a condition for the `key` column you should now use `where('key', ...)` instead.

#### The `factory` Helper

Calling `factory(User::class, 1)->make()` or `factory(User::class, 1)->create()` will now return a collection with one item. Previously, this would return a single model. This method will only return a single model if the amount is not supplied.

### Events

#### Contract Changes

If you are manually implementing the `Illuminate\Contracts\Events\Dispatcher` interface in your application or package, you should rename the `fire` method to `dispatch`.

#### Event Priority

Support for event handler "priorities" has been removed. This undocumented feature typically indicates an abuse of the event feature. Instead, consider using a series of synchronous method calls. Alternatively, you may dispatch a new event from within the handler of another event in order to ensure that a given event's handler fires after an unrelated handler.

#### Wildcard Event Handler Signatures

Wildcard event handlers now receive the event name as their first argument and the array of event data as their second argument. The `Event::firing` method has been removed:

    Event::listen('*', function ($eventName, array $data) {
        //
    });

#### The `kernel.handled` Event

The `kernel.handled` event is now an object based event using the `Illuminate\Foundation\Http\Events\RequestHandled` class.

#### The `locale.changed` Event

The `locale.changed` event is now an object based event using the `Illuminate\Foundation\Events\LocaleUpdated` class.

#### The `illuminate.log` Event

The `illuminate.log` event is now an object based event using the `Illuminate\Log\Events\MessageLogged` class.

### Exceptions

The `Illuminate\Http\Exception\HttpResponseException` has been renamed to `Illuminate\Http\Exceptions\HttpResponseException`. Note that `Exceptions` is now plural. Likewise, the `Illuminate\Http\Exception\PostTooLargeException` has been renamed to `Illuminate\Http\Exceptions\PostTooLargeException`.

### Mail

#### `Class@method` Syntax

Sending mail using `Class@method` syntax is no longer supported. For example:

    Mail::send('view.name', $data, 'Class@send');

If you are sending mail in this way you should convert these calls to [mailables](/docs/{{version}}/mail).

#### New Configuration Options

In order to provide support for Laravel 5.4's new Markdown mail components, you should add the following block of configuration to the bottom of your `mail` configuration file:

    'markdown' => [
        'theme' => 'default',

        'paths' => [
            resource_path('views/vendor/mail'),
        ],
    ],

#### Queueing Mail With Closures

In order to queue mail, you now must use a [mailable](/docs/{{version}}/mail). Queuing mail using the `Mail::queue` and `Mail::later` methods no longer supports using Closures to configure the mail message. This feature required the use of special libraries to serialize Closures since PHP does not natively support this feature.

### Redis

#### Improved Clustering Support

Laravel 5.4 introduces improved Redis cluster support. If you are using Redis clusters, you should place your cluster connections inside of a `clusters` configuration option in the Redis portion of your `config/database.php` configuration file:

    'redis' => [

        'client' => 'predis',

        'options' => [
            'cluster' => 'redis',
        ],

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', '127.0.0.1'),
                    'password' => env('REDIS_PASSWORD', null),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

### Sessions

#### Symfony Compatibility

Laravel's session handlers no longer implements Symfony's `SessionInterface`. Implementing this interface required us to implement extraneous features that were not needed by the framework. Instead, a new `Illuminate\Contracts\Session\Session` interface has been defined and may be used instead. The following code changes should also be applied:

All calls to the `->set()` method should be changed to `->put()`. Typically, Laravel applications would never call the `set` method since it has never been documented within the Laravel documentation. However, it is included here out of caution.

All calls to the `->getToken()` method should be changed to `->token()`.

All calls to the `$request->setSession()` method should be changed to `setLaravelSession()`.

### Testing

Laravel 5.4's testing layer has been re-written to be simpler and lighter out of the box. If you would like to continue using the testing layer present in Laravel 5.3, you may install the `laravel/browser-kit-testing` [package](https://github.com/laravel/browser-kit-testing) into your application. This package provides full compatibility with the Laravel 5.3 testing layer. In fact, you can run the Laravel 5.4 testing layer side-by-side with the Laravel 5.3 testing layer.

If you have tests written using Laravel 5.3 and would like to run them side-by-side with Laravel's new testing layer, install the `laravel/browser-kit-testing` package:

    composer require laravel/browser-kit-testing

Next, create a copy of your `tests/TestCase.php` file and save it to your `tests` directory as `BrowserKitTest.php`. Then, modify the file to extend the `Laravel\BrowserKitTesting\TestCase` class. Once you have done this, you should have two base test classes in your `tests` directory: `TestCase.php` and `BrowserKitTest.php`.

Tests written on Laravel 5.3 will extend the `BrowserKitTest` class while any new tests that use the Laravel 5.4 testing layer will extend the `TestCase` class. Your `BrowserKitTest` class should look like the following:

    <?php

    use Laravel\BrowserKitTesting\TestCase as BaseTestCase;

    abstract class BrowserKitTestCase extends BaseTestCase
    {
        /**
         * The base URL of the application.
         *
         * @var string
         */
        public $baseUrl = 'http://localhost';

        /**
         * Creates the application.
         *
         * @return \Illuminate\Foundation\Application
         */
        public function createApplication()
        {
            $app = require __DIR__.'/../bootstrap/app.php';

            $app->make(Kernel::class)->bootstrap();

            return $app;
        }
    }

Once you have created this class, make sure to update all of your tests to extend your new `BrowserKitTest` class. This will allow of your tests written on Laravel 5.3 to continue running on Laravel 5.4. If you choose, you can slowly begin to port them over to the new [Laravel 5.4 test syntax](/docs/5.4/http-tests) or [Laravel Dusk](/docs/5.4/dusk).

> {note} If you are writing new tests and want them to use the Laravel 5.4 testing layer, make sure to extend the `TestCase` class.

#### Environment

The Laravel 5.4 test class no longer manually forces `putenv('APP_ENV=testing')` for each test. Instead, the framework utilizes the `APP_ENV` variable from the loaded `.env` file.

#### Event Fake

The `Event` fake's `assertFired` method should be updated to `assertDispatched`. The method signature has not been changed.

#### Mail Fake

The `Mail` fake has been greatly simplified for the Laravel 5.4 release. Instead of using the `assertSentTo` method, you should now simply use the `assertSent` method and utilize the `hasTo`, `hasCc`, etc. helper methods within your callback:

    Mail::assertSent(MailableName::class, function ($mailable) {
        return $mailable->hasTo('email@example.com');
    });

### Translation

#### `{Inf}` Placeholder

If you are using the `{Inf}` placeholder for pluralizing your translation strings, you should update your translation strings to use the `*` character instead:

    {0} First Message|{1,*} Second Message

### Validation

#### Date Format Validation

Date format validation is now more strict and supports the placeholders present within the documentation for the PHP [date function](http://php.net/manual/en/function.date.php). In previous releases of Laravel, the timezone placeholder `P` would accept all timezone formats; however, in Laravel 5.4 each timezone format has a unique placeholder as per the PHP documentation.
