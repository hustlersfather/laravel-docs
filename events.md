# Events

- [Introduction](#introduction)
- [Registering Events / Listeners](#registering-events-and-listeners)
- [Defining Events](#defining-events)
- [Defining Listeners](#defining-listeners)
- [Firing Events](#firing-events)
- [Queued Event Listeners](#queued-event-listeners)
- [Event Subscribers](#event-subscribers)

<a name="introduction"></a>
## Introduction

Laravel's events provides a simple observer implementation, allowing you to subscribe and listen for events in your application. Event classes are typically stored in the `app/Events` directory, while their listeners are stored in `app/Listeners`.

<a name="registering-events-and-listeners"></a>
## Registering Events / Listeners

The `EventServiceProvider` included with your Laravel application provides a convenient place to register all event listeners. The `listen` property contains an array of all events (keys) and their listeners (values). Of course, you may add as many events to this array as your application requires. For example, let's add our `PodcastWasPurchased` event:

	/**
	 * The event listener mappings for the application.
	 *
	 * @var array
	 */
	protected $listen = [
		'App\Events\PodcastWasPurchased' => [
			'App\Listeners\EmailPurchaseConfirmation',
		],
	];

### Generating Event / Listener Classes

Of course, manually creating the files for each event and listener is cumbersome. Instead, simply add listeners and events to your `EventServiceProvider` and use the `event:generate` command. This command will generate any events or listeners that are listed in your `EventServiceProvider`. Of course, events and listeners that already exist will be left untouched:

	php artisan event:generate

<a name="defining-events"></a>
## Defining Events

An event class is simply a data container which holds the information related to the event. For example, let's assume our generated `PodcastWasPurchased` event receives a [Eloquent ORM](/docs/{{version}}/eloquent) object:

	<?php namespace App\Events;

	use App\Podcast;
	use App\Events\Event;
	use Illuminate\Queue\SerializesModels;

	class PodcastWasPurchased extends Event
	{
	    use SerializesModels;

	    public $podcast;

	    /**
	     * Create a new event instance.
	     *
	     * @param  Podcast  $podcast
	     * @return void
	     */
	    public function __construct(Podcast $podcast)
	    {
	        $this->podcast = $podcast;
	    }
	}

As you can see, this event class contains no special logic. It is simply a container for the `Podcast` object that was purchased. The `SerializesModels` trait used by the event will gracefully serialize any Eloquent models if the event object is serialized using PHP's `serialize` function.

<a name="defining-event-listeners"></a>
## Defining Event Listeners

Next, let's take a look at the listener for our example event. Event listeners receive the event instance in their `handle` method. The `event:generate` command will automatically import the proper event class and type-hint the event on the `handle` method. Within the `handle` method, you may perform any logic necessary to respond to the event.

	<?php namespace App\Listeners;

	use App\Events\PodcastWasPurchased;
	use Illuminate\Queue\InteractsWithQueue;
	use Illuminate\Contracts\Queue\ShouldQueue;

	class EmailPurchaseConfirmation
	{
	    /**
	     * Create the event listener.
	     *
	     * @return void
	     */
	    public function __construct()
	    {
	        //
	    }

	    /**
	     * Handle the event.
	     *
	     * @param  PodcastWasPurchased  $event
	     * @return void
	     */
	    public function handle(PodcastWasPurchased $event)
	    {
	        // Access the podcast using $event->podcast...
	    }
	}

Your event listeners may also type-hint any dependencies they need on their constructors. All event listeners are resolved via the Laravel [service container](/docs/{{version}}/container), so dependencies will be injected automatically:

	use Illuminate\Contracts\Mail\Mailer;

	public function __construct(Mailer $mailer)
	{
		$this->mailer = $mailer;
	}

#### Stopping The Propagation Of An Event

Sometimes, you may wish to stop the propagation of an event to other listeners. You may do so using by returning `false` from your listener's `handle` method.

<a name="firing-events"></a>
## Firing Events

To fire an event, you may use the `Event` [facade](/docs/{{version}}/facades), passing an instance of the event to the `fire` method. The `fire` method will dispatch the event to all of its registered listeners:

	event(new PodcastWasPurchased($podcast));

	<?php namespace App\Http\Controllers;

	use Event;
	use App\Podcast;
	use App\Events\PodcastWasPurchased;
	use App\Http\Controllers\Controller;

	class UserController extends Controller
	{
		/**
		 * Show the profile for the given user.
		 *
		 * @param  int  $userId
		 * @param  int  $podcastId
		 * @return Response
		 */
		public function purchasePodcast($userId, $podcastId)
		{
			$podcast = Podcast::findOrFail($podcastId);

			// Purchase podcast logic...

			Event::fire(new PodcastWasPurchased($podcast));
		}
	}

Alternatively, you may use the global `event` helper function to fire events:

	event(new PodcastWasPurchased($podcast));

<a name="queued-event-listeners"></a>
## Queued Event Listeners

Need to [queue](/docs/{{version}}/queues) an event listener? It couldn't be any easier. Simply add the `ShouldQueue` interface to the listener class. Listeners generated by the `event:generate` Artisan command already have this interface imported into the current namespace, so you can use it immediately:

	<?php namespace App\Listeners;

	use App\Events\PodcastWasPurchased;
	use Illuminate\Queue\InteractsWithQueue;
	use Illuminate\Contracts\Queue\ShouldQueue;

	class EmailPurchaseConfirmation implements ShouldQueue
	{
		//
	}

That's it! Now, when this listener is called for an event, it will be queued automatically by the event dispatcher using Laravel's [queue system](/docs/{{version}}/queues). If no exceptions are thrown when the listener is executed by the queue, the queued job will automatically be deleted after it has processed.

#### Manually Accessing The Queue

If you need to access the underlying queue job's `delete` and `release` methods manually, you may do so. The `Illuminate\Queue\InteractsWithQueue` trait, which is imported by default on generated listeners, gives you access to these methods:

	<?php namespace App\Listeners;

	use App\Events\PodcastWasPurchased;
	use Illuminate\Queue\InteractsWithQueue;
	use Illuminate\Contracts\Queue\ShouldQueue;

	class EmailPurchaseConfirmation implements ShouldQueue
	{
		use InteractsWithQueue;

		public function handle(PodcastWasPurchased $event)
		{
			if (true) {
				$this->release(30);
			}
		}
	}

<a name="event-subscribers"></a>
## Event Subscribers

Event subscribers are classes that may subscribe to multiple events from within the class itself, allowing you to define several event handlers within a single class. Subscribers should define a `subscribe` method, which will be passed an event dispatcher instance:

	<?php namespace App\Listeners;

	class UserEventListener {

		/**
		 * Handle user login events.
		 */
		public function onUserLogin($event) {}

		/**
		 * Handle user logout events.
		 */
		public function onUserLogout($event) {}

		/**
		 * Register the listeners for the subscriber.
		 *
		 * @param  Illuminate\Events\Dispatcher  $events
		 * @return array
		 */
		public function subscribe($events)
		{
			$events->listen(
				'App\Events\UserLoggedIn',
				'App\Listeners\UserEventListener@onUserLogin'
			);

			$events->listen(
				'App\Events\UserLoggedOut',
				'App\Listeners\UserEventListener@onUserLogout'
			);
		}

	}

#### Registering An Event Subscriber

Once the subscriber has been defined, it may be registered with the event dispatcher. You may register subscribers using the `$subscribe` property on the `EventServiceProvider`. For example, let's add the `UserEventListener`.

    <?php namespace App\Providers;

    use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * The event listener mappings for the application.
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * The subscriber classes to register.
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventListener',
        ];
    }
