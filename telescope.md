# Laravel Telescope

- [Introduction](#introduction)
- [Installation](#installation)
    - [Configuration](#configuration)
    - [Data Pruning](#data-pruning)
- [Dashboard Authorization](#dashboard-authorization)
- [Available Watchers](#available-watchers)
    - [Cache Watcher](#cache-watcher)
    - [Command Watcher](#command-watcher)
    - [Dump Watcher](#dump-watcher)
    - [Event Watcher](#event-watcher)
    - [Exception Watcher](#exception-watcher)
    - [Job Watcher](#job-watcher)
    - [Log Watcher](#log-watcher)
    - [Mail Watcher](#mail-watcher)
    - [Model Watcher](#model-watcher)
    - [Notification Watcher](#notification-watcher)
    - [Query Watcher](#query-watcher)
    - [Redis Watcher](#redis-watcher)
    - [Request Watcher](#request-watcher)
    - [Schedule Watcher](#schedule-watcher)

<a name="introduction"></a>
## Introduction

Laravel Telescope is an elegant debug assistant for the Laravel framework. Telescope provides insight into the requests coming into your application, exceptions, log entries, database queries, queued jobs, mail, notifications, cache operations, scheduled tasks, variable dumps and more. Telescope makes a wonderful companion to your local Laravel development environment.

<p align="center">
<img src="https://res.cloudinary.com/dtfbvvkyp/image/upload/v1539110860/Screen_Shot_2018-10-09_at_1.47.23_PM.png" width="600" height="347">
</p>

<a name="installation"></a>
## Installation

> {note} Telescope requires Laravel 5.7.7+.

You may use Composer to install Telescope into your Laravel project:

    composer require laravel/telescope

After installing Telescope, publish its assets using the `telescope:install` Artisan command. After installing Telescope, you should also run the `migrate` command:

    php artisan telescope:install

    php artisan migrate

#### Updating Telescope

When updating Telescope, you should re-publish Telescope's assets:

    php artisan telescope:publish

### Installing Only In Specific Environments

If you plan to only use Telescope to assist your local development. You may install Telescope using the `--dev` flag:

    composer require laravel/telescope --dev

After running `telescope:install`, you should remove the `TelescopeServiceProvider` service provider registration from your `app` configuration file. Instead, manually register the service provider in the `register` method of your `AppServiceProvider`:

    use Laravel\Telescope\TelescopeServiceProvider::class;

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        if ($this->app->isLocal()) {
            $this->app->register(TelescopeServiceProvider::class);
        }
    }

<a name="configuration"></a>
### Configuration

After publishing Telescope's assets, its primary configuration file will be located at `config/telescope.php`. This configuration file allows you to configure your watcher options and each configuration option includes a description of its purpose, so be sure to thoroughly explore this file.

If desired, you may disable Telescope's data collection entirely using the `enabled` configuration option:

    'enabled' => env('TELESCOPE_ENABLED', true),

<a name="data-pruning"></a>
### Data Pruning

Without pruning, the `telescope_entries` table can accumulate records very quickly. To mitigate this, you should schedule the `telescope:prune` Artisan command to run daily:

    $schedule->command('telescope:prune')->daily();

By default, all entries older than 24 hours will be pruned. You may use the `hours` option when calling the command to determine how long to retain Telescope data. For example, the following command will delete all records created over 48 hours ago:

    $schedule->command('telescope:prune --hours=48')->daily();

<a name="dashboard-authorization"></a>
## Dashboard Authorization

Telescope exposes a dashboard at `/telescope`. By default, you will only be able to access this dashboard in the `local` environment. Within your `app/Providers/TelescopeServiceProvider.php` file, there is a `gate` method. This authorization gate controls access to Telescope in **non-local** environments. You are free to modify this gate as needed to restrict access to your Telescope installation:

    /**
     * Register the Telescope gate.
     *
     * This gate determines who can access Telescope in non-local environments.
     *
     * @return void
     */
    protected function gate()
    {
        Gate::define('viewTelescope', function ($user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

<a name="available-watchers"></a>
## Available Watchers

Telescope watchers gather application profile data when a request or console command is executed. You may customize the list of watchers that you would like to enable in your application within your `config/telescope.php` file:

    'watchers' => [
        Watchers\CacheWatcher::class => true,
        Watchers\CommandWatcher::class => true,
        ...
    ],

Some watchers also allow you to also customize certain options:

    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 100,
        ],
        ...
    ],

<a name="cache-watcher"></a>
### Cache Watcher

The Cache Watcher records data when a cache key is hit, missed, updated and forgotten.

<a name="command-watcher"></a>
### Command Watcher

The Command Watcher records the arguments, options, exit code and output whenever an Artisan command is executed. If you would like to exclude certain commands to be recorded by the Command Watcher, you may specify the command in the `ignore` option in your `config/telescope.php` file:

    'watchers' => [
        Watchers\CommandWatcher::class => [
            'enabled' => env('TELESCOPE_COMMAND_WATCHER', true),
            'ignore' => ['key:generate'],
        ],
        ...
    ],

<a name="dump-watcher"></a>
### Dump Watcher

The Dump Watcher records and displays your dumps in Telescope. The Telescope Dump Watcher screen must be open in a browser for the recording to occur, otherwise all dumps will be ignored by the watcher.

<a name="event-watcher"></a>
### Event Watcher

The Event Watcher records the payload, listeners and broadcast data for any events fired by your application. The Laravel Framework events are ignored by the Event Watcher.

<a name="exception-watcher"></a>
### Exception Watcher

The Exception Watcher records the data with the stack trace for any reportable Exceptions that are thrown and logged in your application.

<a name="job-watcher"></a>
### Job Watcher

The Job Watcher records the data and status of any jobs dispatched in your application.

<a name="log-watcher"></a>
### Log Watcher

The Log Watcher records the log data for any logs fired by your application. All logs regardless of the log level will be recorded by the Log Watcher, even if the logs do not meet the log level threshold set in your configuration.

<a name="mail-watcher"></a>
### Mail Watcher

The Mail Watcher allows you to view a preview of the mails sent along with some associated data. You may also download the mail as a .eml file.

<a name="model-watcher"></a>
### Model Watcher

The Model Watcher records model changes whenever an Eloquent `created`, `updated`, `restored` or `deleted` event is fired. You may filter these events further via the `events` option:

    'watchers' => [
        Watchers\ModelWatcher::class => [
            'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
            'events' => ['eloquent.created*', 'eloquent.updated*'],
        ],
        ...
    ],

<a name="notification-watcher"></a>
### Notification Watcher

The Notification Watcher records the notification data whenever a Laravel Notification is triggered. If the notification triggers a mail and you have the Mail Watcher enabled, the mail will also be available for preview and download in the Telescope Mail Watcher screen.

<a name="query-watcher"></a>
### Query Watcher

The Query Watcher records the raw SQL, bindings and time taken by the queries fired by your application. This watcher also tags any queries slower than 100ms as `slow`. You may customize this time using the `slow` option:


    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 50,
        ],
        ...
    ],

<a name="redis-watcher"></a>
### Redis Watcher

> {note} Redis events must be enabled for the Redis Watcher to work. You may enable Redis events by calling `Redis::enableEvents()` in the `boot` method of your `app/Providers/AppServiceProvider.php` file.

The Redis Watcher records all Redis commands executed by your application. If you are using Redis for caching, cache commands will also be recorded by the Redis Watcher.

<a name="request-watcher"></a>
### Request Watcher

The Request Watcher records the request, headers, session and response data associated with any requests handled by the application. You may purge your response data via the `size_limit` (in KB) option:

    'watchers' => [
        Watchers\RequestWatcher::class => [
            'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
            'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
        ],
        ...
    ],

<a name="schedule-watcher"></a>
### Schedule Watcher

The Schedule Watcher records the command and output data of any scheduled tasks run by your application.